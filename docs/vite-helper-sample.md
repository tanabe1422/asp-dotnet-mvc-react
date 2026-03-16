# ViteHelper サンプルコード

Razor から Vite のアセットを読み込む C# ヘルパー。開発時は dev server、本番時は manifest から解決。

## 前提

- **NuGet**: `Newtonsoft.Json` をインストール（`Install-Package Newtonsoft.Json`）
- ASP.NET MVC なら既に入っていることが多い

---

## Web.config

```xml
<appSettings>
  <add key="ViteDevMode" value="true" />
  <add key="ViteDevServer" value="http://localhost:5173" />
</appSettings>
```

本番時は `ViteDevMode` を `false` に変更。

---

## ViteHelper.cs

```
Helpers/ViteHelper.cs
```

```csharp
using System;
using System.Collections.Generic;
using System.IO;
using System.Web;
using Newtonsoft.Json.Linq;

namespace YourNamespace.Helpers
{
    public static class ViteHelper
    {
        private static readonly string DevMode = System.Configuration.ConfigurationManager.AppSettings["ViteDevMode"];
        private static readonly string DevServer = System.Configuration.ConfigurationManager.AppSettings["ViteDevServer"] ?? "http://localhost:5173";

        // vite input キー → 開発時のエントリパス。ページ追加時にここに1行足す
        private static readonly Dictionary<string, string> EntryPaths = new Dictionary<string, string>(StringComparer.OrdinalIgnoreCase)
        {
            { "home", "src/pages/home/main.tsx" },
            { "user", "src/pages/user/main.tsx" },
        };

        public static IHtmlString RenderScripts(string entryName)
        {
            if (string.Equals(DevMode, "true", StringComparison.OrdinalIgnoreCase))
            {
                var path = EntryPaths.TryGetValue(entryName, out var p) ? p : $"src/pages/{entryName}/main.tsx";
                return new HtmlString($"<script type=\"module\" src=\"{DevServer}/@vite/client\"></script>\n<script type=\"module\" src=\"{DevServer}/{path}\"></script>");
            }
            return RenderProd(entryName);
        }

        private static IHtmlString RenderProd(string entryName)
        {
            var manifestPath = Path.Combine(HttpContext.Current.Server.MapPath("~/Content/dist"), ".vite", "manifest.json");
            if (!File.Exists(manifestPath))
                return new HtmlString("<!-- manifest not found -->");

            var manifest = JObject.Parse(File.ReadAllText(manifestPath));
            var entry = manifest[entryName] as JObject ?? manifest[EntryPaths.TryGetValue(entryName, out var p) ? p : ""] as JObject;
            if (entry == null)
                return new HtmlString($"<!-- entry not found: {entryName} -->");

            var baseUrl = VirtualPathUtility.ToAbsolute("~/Content/dist/");
            var sb = new System.Text.StringBuilder();
            foreach (var css in entry["css"] ?? new JArray())
                sb.AppendFormat("<link rel=\"stylesheet\" href=\"{0}{1}\" />", baseUrl, css);
            sb.AppendFormat("<script type=\"module\" src=\"{0}{1}\"></script>", baseUrl, entry["file"]);
            return new HtmlString(sb.ToString());
        }
    }
}
```

---

## 使い方

**Razor:**
```html
<div id="root"></div>
@Html.Raw(ViteHelper.RenderScripts("home"))
```

**vite.config.ts の input キー** と `EntryPaths` のキーを揃える。

```ts
input: {
  home: resolve(__dirname, "src/pages/home/main.tsx"),
  user: resolve(__dirname, "src/pages/user/main.tsx"),
}
```

---

## ページを増やすとき

1. vite.config の `input` に追加
2. `EntryPaths` に 1 行追加

以上。
