# Linter / Formatter 候補比較

本プロジェクト（React 19 + TypeScript 5.x + Vite 6.x + pnpm）に適した Linter/Formatter を選定するための比較資料。

## 候補一覧

| # | 構成 | Lint | Format | 言語 |
|---|------|------|--------|------|
| 1 | **Biome** | Biome | Biome | Rust |
| 2 | **ESLint v9 + Prettier** | ESLint v9 (flat config) | Prettier | JS |
| 3 | **oxlint + oxfmt** | oxlint | oxfmt | Rust (oxc) |

---

## 1. Biome

Rust 製の統合ツール。Lint と Format を単一バイナリで提供する。

### メリット

- **圧倒的な速度**: ESLint + Prettier 比で 20〜100 倍高速（大規模プロジェクトでも ~50ms）
- **設定がシンプル**: `biome.json` 1 ファイルで Lint・Format・import ソートをすべて管理
- **ゼロ依存**: node_modules 肥大化なし。CI キャッシュにも優しい
- **Formatter / Linter 間の競合なし**: 同一パーサーで処理するため Prettier と ESLint の衝突問題が原理的に発生しない
- **React Hooks 対応済み**: v2 で `react` domain が導入され、`useHookAtTopLevel`（rules-of-hooks 相当）や `useExhaustiveDependencies` をサポート
- **型推論 Lint**: TypeScript コンパイラ不要で `noFloatingPromises` 等の型認識ルールが使える（v2）
- **CSS の Lint/Format にも対応**

### デメリット

- **ルール数**: 約 250 ルール。ESLint エコシステム（1,000+）と比べるとカバー範囲が狭い
- **プラグインエコシステムが未成熟**: カスタムルールの作成やサードパーティプラグインの仕組みがまだない
- **Prettier 互換率 ~96%**: 微妙なフォーマット差異が出るケースがある
- **採用事例がまだ少ない**: 大規模プロダクションでの実績は ESLint と比較すると限定的

### 設定例

```json
{
  "$schema": "https://biomejs.dev/schemas/2.0.0/schema.json",
  "linter": {
    "domains": {
      "react": "recommended"
    }
  },
  "formatter": {
    "indentStyle": "space",
    "indentWidth": 2
  }
}
```

---

## 2. ESLint v9 + Prettier

JavaScript エコシステムの事実上の標準。ESLint v9 で flat config に移行し設定が整理された。

### メリット

- **最大のプラグインエコシステム**: typescript-eslint, eslint-plugin-react, react-hooks, import, jsx-a11y, etc. あらゆるユースケースに対応
- **カスタムルール作成が容易**: プロジェクト固有のルールを JS で書ける
- **Flat Config で設定が改善**: ESM ベースの `eslint.config.mjs` により型補完やトレーサビリティが向上
- **Prettier はフォーマッターのデファクト**: 対応言語が最多（JS/TS, CSS, HTML, JSON, YAML, Markdown, etc.）
- **情報量が圧倒的**: StackOverflow、ブログ、公式ドキュメントが豊富で問題解決しやすい
- **エディタ統合が最も成熟**: VS Code 拡張の安定性・機能が最も高い

### デメリット

- **速度が遅い**: 大規模プロジェクトでは Lint + Format に数秒〜十数秒かかる
- **設定が複雑になりがち**: ESLint + Prettier + typescript-eslint の組み合わせで設定ファイルが肥大化しやすい
- **ツール間の競合リスク**: ESLint と Prettier のルール競合を防ぐために `eslint-config-prettier` が必要
- **依存パッケージが多い**: devDependencies が膨れる（eslint, prettier, 各プラグイン, config パッケージ等）
- **Node.js 18.18+ が必須**: ESLint v9 は Node.js 18.18 以上が必要

### 設定例 (eslint.config.mjs)

```javascript
import eslint from "@eslint/js";
import tseslint from "typescript-eslint";
import react from "eslint-plugin-react";
import reactHooks from "eslint-plugin-react-hooks";
import prettier from "eslint-config-prettier";

export default tseslint.config(
  eslint.configs.recommended,
  ...tseslint.configs.recommended,
  react.configs.flat.recommended,
  reactHooks.configs.flat.recommended,
  prettier,
  {
    settings: { react: { version: "detect" } },
  }
);
```

---

## 3. oxlint + oxfmt (oxc 統一構成)

oxc プロジェクトが提供する Rust 製ツールチェーン。Linter (oxlint) と Formatter (oxfmt) を組み合わせ、Prettier 不要で完結する。

### メリット

- **Lint が最速クラス**: ESLint 比で 8〜12 倍高速。型認識 Lint でも ESLint + typescript-eslint の 10 倍速
- **650+ ルール内蔵**: React, React Hooks, TypeScript, Next.js, import, jsx-a11y, unicorn 等の主要プラグインルールを単体でカバー
- **oxfmt は Prettier 100% 互換（JS/TS）**: 2026年2月のベータで JS/TS 適合テスト 100% 達成。移行コストが極めて低い
- **oxfmt の対応言語が広い**: JS/TS/JSX/TSX に加え、JSON, YAML, CSS, SCSS, HTML, Vue, Markdown, GraphQL 等にも対応
- **import ソート + Tailwind CSS サポート**: oxfmt に組み込み済み
- **ESLint プラグイン互換（Alpha）**: 既存の ESLint JS プラグインを oxlint 内で実行可能（2026年3月〜）
- **型認識 Lint 対応（Alpha）**: 43 の型認識ルールをサポート（2025年12月〜）
- **設定不要で即使用可能**: oxlint はデフォルトで 99 ルールが有効。oxfmt もゼロコンフィグで動作
- **Prettier 不要で依存が軽い**: Biome と同様、JS 製ツールへの依存を排除できる

### デメリット

- **oxfmt はまだベータ**: v0.40.0（2026年3月時点）。本番採用にはベータであるリスクを許容する必要あり
- **Alpha 機能が多い**: 型認識 Lint、JS プラグイン互換はまだ Alpha
- **ツールが 2 つに分かれる**: Biome のような単一バイナリではなく、oxlint と oxfmt を別々にインストール・実行する
- **ESLint 完全互換ではない**: プラグイン互換は進んでいるが、すべての ESLint プラグインが動くわけではない
- **コミュニティがまだ小さい**: 問題発生時の情報が限られる
- **エディタ統合が発展途上**: VS Code 拡張はあるが、ESLint/Prettier ほどの成熟度はまだない

### 設定例

```json
// .oxlintrc.json
{
  "rules": {
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "typescript/no-explicit-any": "warn"
  }
}
```

```json
// .oxfmtrc.json
{
  "indentStyle": "space",
  "indentWidth": 2
}
```

---

## 比較サマリー

| 観点 | Biome | ESLint v9 + Prettier | oxlint + oxfmt |
|------|-------|----------------------|----------------|
| **速度** | ★★★★★ | ★★☆☆☆ | ★★★★★ |
| **ルール数** | ★★★☆☆ (~250) | ★★★★★ (1,000+) | ★★★★☆ (650+) |
| **React 対応** | ★★★★☆ | ★★★★★ | ★★★★☆ |
| **型認識 Lint** | ★★★☆☆ (v2) | ★★★★★ | ★★★☆☆ (Alpha) |
| **設定の容易さ** | ★★★★★ | ★★☆☆☆ | ★★★★☆ |
| **プラグイン拡張性** | ★☆☆☆☆ | ★★★★★ | ★★★☆☆ (Alpha) |
| **エディタ統合** | ★★★★☆ | ★★★★★ | ★★★☆☆ |
| **安定性・成熟度** | ★★★☆☆ | ★★★★★ | ★★☆☆☆ |
| **依存の少なさ** | ★★★★★ | ★★☆☆☆ | ★★★★★ |
| **Prettier 互換** | ~96% | 100% (本体) | 100% (JS/TS) |
| **Format 対応言語** | ★★★☆☆ | ★★★★★ | ★★★★☆ |

---

## 本プロジェクトにおける考慮点

- **プロジェクト規模**: MPA で各ページが小さいエントリポイント → Lint 速度は大差つきにくい
- **React 19 + TypeScript**: いずれの候補も基本的な対応は完了
- **チームの習熟度**: ESLint/Prettier は経験者が多く、学習コストが低い可能性が高い
- **新規プロジェクト**: 既存の ESLint 設定を移行する必要がないため、Biome のようなシンプルな選択肢のハードルが低い
- **長期運用**: ESLint はエコシステムの安定性が最も高い。Biome/oxlint は急速に改善中だが破壊的変更のリスクあり
