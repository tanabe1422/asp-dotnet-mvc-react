# プロジェクトベースセットアップ手順

本ドキュメントは、ASP.NET MVC + React MPA プロジェクトの初期セットアップ手順をまとめたものです。

## 前提環境

- **OS**: Windows
- **シェル**: PowerShell
- **Visual Studio**: インストール済み（.NET Framework 4.x）
- **ASP.NET MVC プロジェクト**: `backend/backend/` に配置済み

---

## 1. fnm のインストール

Node.js のバージョン管理に fnm を使用します。

```powershell
winget install Schniz.fnm
```

インストール後、**新しい PowerShell ウィンドウ**を開いてください。

```powershell
fnm install 22
fnm default 22
fnm use 22
node -v   # v22.x.x と表示されれば OK
```

---

## 2. pnpm のインストール

Node.js に同梱の Corepack で pnpm を有効化します。

```powershell
corepack enable
corepack prepare pnpm@latest --activate
pnpm --version
```

---

## 3. frontend のセットアップ

### 3.1 ディレクトリ構成

ビルド成果物は **backend の Web プロジェクトの Content 直下** に配置する必要があります。

```
asp-dotnet-mvc-react/
├── backend/
│   └── backend/
│       └── Content/
│           └── dist/   ← Vite ビルド出力先
└── frontend/           ← ここを作成
```

### 3.2 Vite プロジェクトの作成

リポジトリルートで実行：

```powershell
cd C:\Users\tanabe\Documents\asp-dotnet-mvc-react
pnpm create vite frontend --template react-ts
cd frontend
pnpm install
```

### 3.3 .nvmrc の配置（任意）

リポジトリルートに `.nvmrc` を配置すると、`fnm use` でバージョンを揃えやすくなります。

```powershell
cd ..
echo "22" > .nvmrc
```

---

## 4. Vite の設定

`frontend/vite.config.ts` を編集し、ビルド出力先と MPA 構成を設定します。

```typescript
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import { resolve } from "path";

export default defineConfig({
  plugins: [react()],
  build: {
    outDir: "../backend/backend/Content/dist",
    emptyOutDir: true,
    manifest: true,
    rollupOptions: {
      input: {
        main: resolve(__dirname, "src/main.tsx"),
      },
    },
  },
  server: {
    origin: "http://localhost:5173",
  },
});
```

### 設定ポイント

| 項目 | 説明 |
|------|------|
| `build.outDir` | `backend/backend/Content/dist` を指定。ASP.NET の静的ファイル配置先 |
| `build.manifest` | `true` 必須。ViteHelper が manifest.json を参照して本番のハッシュ付きファイル名を解決 |
| `build.rollupOptions.input` | エントリポイント。ページ追加時はここに追加 |
| `server.origin` | 開発時、Razor が Vite dev server の URL を参照するため指定 |

> **注意**: `pnpm create vite` で作成した直後は `src/main.tsx` がエントリです。`architecture.md` の MPA 構成（`pages/home`, `pages/user` 等）に移行する際は、`input` を変更してください。

---

## 5. Biome のセットアップ（プロジェクトルート）

本プロジェクトは **段階的に React に移行** するため、`node_modules` をプロジェクトルートに置き、Biome は **backend（dist 以外）** と **frontend** の両方を対象にします。

### 5.1 ルートの package.json と Biome

プロジェクトルートに以下を配置済み：

- `package.json` … Biome を devDependency に持つ
- `biome.json` … 対象: `backend/**`, `frontend/**`（除外: `**/dist`, `**/bin`, `**/obj`, `backend/packages`）

```powershell
cd C:\Users\tanabe\Documents\asp-dotnet-mvc-react
pnpm install
```

### 5.2 biome.json の対象

| 対象 | 除外 |
|------|------|
| `backend/**` | `**/dist`, `**/bin`, `**/obj`, `backend/packages` |
| `frontend/**` | 上記と同じ |

### 5.3 スクリプト

ルートで実行：

| コマンド | 説明 |
|----------|------|
| `pnpm lint` | Lint チェックのみ |
| `pnpm lint:fix` | Lint チェック + 自動修正 |
| `pnpm format` | フォーマットのみ |
| `pnpm check` | Lint + Format を一括適用 |

### 5.4 VSCode 拡張（任意）

- **Biome**（`biomejs.biome`）をインストールすると、保存時にフォーマット・Lint が動作します。

---

## 6. 動作確認

### 6.1 開発時

1. **Vite 起動**
   ```powershell
   cd frontend
   pnpm dev
   ```
   → `http://localhost:5173` で起動

2. **ASP.NET MVC 起動**
   - Visual Studio で `backend.sln` を開く
   - IIS Express で実行（F5）

3. ブラウザで ASP.NET の URL（例: `http://localhost:xxxxx/`）にアクセス
   - Razor が Vite dev server から React を読み込む
   - HMR が有効

### 6.2 本番ビルド

```powershell
cd frontend
pnpm build
```

- `backend/backend/Content/dist/` 以下に `assets/` と manifest が出力されることを確認
- ASP.NET をビルド・デプロイする際、`Content/dist/` も含める

---

## 7. .gitignore の確認

以下が含まれていることを確認：

```
# ASP.NET
backend/backend/bin/
backend/backend/obj/
backend/packages/
*.suo
*.user

# Vite build output
backend/backend/Content/dist/

# Node
frontend/node_modules/
```

---

## 参考

- [architecture.md](./architecture.md) - アーキテクチャ概要
- [linter.md](./linter.md) - Linter / Formatter 比較
