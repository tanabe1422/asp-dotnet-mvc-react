# MPA ページのサンプル（1ファイル完結）

Controller 1つに対応するページのサンプル。`main.tsx` と `App.tsx` で構成し、CSS も `App.tsx` 内に含める。

## 配置場所

```
frontend/src/pages/home/
├── main.tsx   # エントリ（ビルド対象）
└── App.tsx    # ページ本体 + CSS
```

---

## main.tsx

エントリポイント。シンプルに `App` をマウントするだけ。

```tsx
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import App from "./App";

const root = document.getElementById("root");
if (root) {
  createRoot(root).render(
    <StrictMode>
      <App />
    </StrictMode>
  );
}
```

---

## App.tsx（1ファイルで CSS まで完結）

コンポーネントとスタイルを同じファイルに書く。外部 CSS ファイルは不要。

```tsx
import * as React from "react";

const styles = `
  .page {
    padding: 1.5rem;
    max-width: 40rem;
    margin: 0 auto;
  }
  .title {
    font-size: 1.5rem;
    margin-bottom: 1rem;
    color: #333;
  }
  .text {
    line-height: 1.6;
    color: #666;
  }
  .btn {
    margin-top: 1rem;
    padding: 0.5rem 1rem;
    background: #0066cc;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
  }
  .btn:hover {
    background: #0052a3;
  }
`;

function App() {
  const [count, setCount] = React.useState(0);

  return (
    <>
      <style>{styles}</style>
      <div className="page">
        <h1 className="title">Home</h1>
        <p className="text">Controller に対応する React ページのサンプル。</p>
        <p className="text">クリック: {count}</p>
        <button
          type="button"
          className="btn"
          onClick={() => setCount((c) => c + 1)}
        >
          Count up
        </button>
      </div>
    </>
  );
}

export default App;
```

---

## vite.config.ts の input

```typescript
input: {
  home: resolve(__dirname, "src/pages/home/main.tsx"),
  user: resolve(__dirname, "src/pages/user/main.tsx"),
}
```

## Razor からの読み込み

```html
<div id="root"></div>
@Html.Raw(ViteHelper.RenderScripts("home"))
```
