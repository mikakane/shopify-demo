# Shopify Hydrogen デモ 作成手順

Shopify Hydrogen を使ったデモストアを構築し、Oxygen にデプロイするまでの手順をまとめたドキュメント。

---

## 目次

1. [前提条件](#前提条件)
2. [プロジェクト作成](#プロジェクト作成)
3. [ローカル開発](#ローカル開発)
4. [Storefront API の設定](#storefront-api-の設定)
5. [主要機能の実装](#主要機能の実装)
6. [Oxygen へのデプロイ](#oxygen-へのデプロイ)
7. [トラブルシューティング](#トラブルシューティング)

---

## 前提条件

> Shopify Partner アカウント・開発用ストア・Storefront API トークンの取得手順は
> **[shopify-account-setup.md](./shopify-account-setup.md)** を参照。

| 要件 | バージョン | 備考 |
|------|-----------|------|
| Node.js | 18.x 以上 | `node --version` で確認 |
| npm / pnpm | 最新推奨 | pnpm を推奨 |
| Shopify CLI | 3.x 以上 | `npm install -g @shopify/cli` |
| Shopify Partner アカウント | - | [セットアップ手順](./shopify-account-setup.md) |
| 開発用ストア | - | [セットアップ手順](./shopify-account-setup.md) |

### Shopify CLI のインストール

```bash
npm install -g @shopify/cli @shopify/theme
```

バージョン確認:

```bash
shopify version
```

---

## プロジェクト作成

### 1. Hydrogen アプリの初期化

```bash
npm create @shopify/hydrogen@latest
```

対話形式で以下を設定:

- **Project name**: `shopify-demo`
- **Template**: `Hello World` または `Demo Store`（デモ用途なら Demo Store を推奨）
- **Language**: TypeScript
- **Package manager**: pnpm

### 2. 依存関係のインストール

```bash
cd shopify-demo
pnpm install
```

### 3. ディレクトリ構成

```
shopify-demo/
├── app/
│   ├── components/       # 共通コンポーネント
│   ├── routes/           # ファイルベースルーティング
│   │   ├── _index.tsx    # トップページ
│   │   ├── products.$handle.tsx   # 商品詳細
│   │   └── collections.$handle.tsx # コレクション
│   ├── styles/           # グローバルスタイル
│   └── root.tsx          # ルートレイアウト
├── public/               # 静的ファイル
├── server.ts             # Oxygen 用サーバーエントリ
├── hydrogen.config.ts    # Hydrogen 設定（非推奨、env へ移行）
├── vite.config.ts        # Vite 設定
└── package.json
```

---

## ローカル開発

### 環境変数の設定

`.env` ファイルをプロジェクトルートに作成:

```env
# Shopify Storefront API
PUBLIC_STORE_DOMAIN=your-store.myshopify.com
PUBLIC_STOREFRONT_API_TOKEN=your_public_storefront_api_token

# Oxygen / Customer Account API (任意)
PUBLIC_CUSTOMER_ACCOUNT_API_CLIENT_ID=your_client_id
PUBLIC_CUSTOMER_ACCOUNT_API_URL=https://shopify.com/your_shop_id

# セッション管理
SESSION_SECRET=your_random_secret_string
```

> `.env` は `.gitignore` に含めること。シークレットをコミットしない。

### 開発サーバーの起動

```bash
pnpm dev
```

`http://localhost:3000` でアクセス可能。

### Shopify CLI でリンク（Oxygen デプロイ準備）

```bash
shopify hydrogen link
```

- Partner ダッシュボードのストアと紐付ける
- 初回は `shopify auth login` が必要

---

## Storefront API の設定

### API トークンの取得

1. Shopify 管理画面 → **設定** → **アプリと販売チャネル** → **Headless チャネル** を追加
2. または **Shopify Partner ダッシュボード** → アプリ作成 → Storefront API アクセス許可
3. **Storefront API アクセストークン** をコピーして `.env` に設定

### 必要な API パーミッション（最小構成）

```
unauthenticated_read_product_listings
unauthenticated_read_collection_listings
unauthenticated_read_content
unauthenticated_read_customer_tags
```

カート・チェックアウト機能が必要な場合:

```
unauthenticated_write_checkouts
unauthenticated_read_checkouts
```

---

## 主要機能の実装

### 商品一覧の取得

`app/routes/_index.tsx`:

```typescript
import { json, type LoaderFunctionArgs } from '@shopify/remix-oxygen';
import { useLoaderData } from '@remix-run/react';
import { getPaginationVariables, Pagination } from '@shopify/hydrogen';

const PRODUCTS_QUERY = `#graphql
  query Products($first: Int, $after: String) {
    products(first: $first, after: $after) {
      nodes {
        id
        title
        handle
        featuredImage {
          url
          altText
        }
        priceRange {
          minVariantPrice {
            amount
            currencyCode
          }
        }
      }
      pageInfo {
        hasNextPage
        endCursor
      }
    }
  }
` as const;

export async function loader({ request, context }: LoaderFunctionArgs) {
  const variables = getPaginationVariables(request, { pageBy: 12 });
  const { products } = await context.storefront.query(PRODUCTS_QUERY, {
    variables,
  });
  return json({ products });
}
```

### カート機能

```typescript
// app/routes/cart.tsx
import { cartAction } from '@shopify/hydrogen';

export async function action({ request, context }: ActionFunctionArgs) {
  return cartAction({ request, context });
}
```

---

## Oxygen へのデプロイ

### 1. GitHub リポジトリと Shopify を連携

1. GitHub にリポジトリを push
2. Shopify Partner ダッシュボード → Hydrogen アプリ → **Oxygen** → GitHub 連携
3. デプロイするブランチを指定（通常 `main`）

### 2. 環境変数の設定（Oxygen）

Oxygen のダッシュボード（またはCLI）で環境変数を設定:

```bash
shopify hydrogen env push
```

または Shopify 管理画面 → Hydrogen → 環境変数 から手動設定。

設定する変数:

```
PUBLIC_STORE_DOMAIN
PUBLIC_STOREFRONT_API_TOKEN
SESSION_SECRET
```

### 3. CLI からデプロイ

```bash
# ビルド
pnpm build

# デプロイ（プレビュー環境）
shopify hydrogen deploy --preview

# デプロイ（本番環境）
shopify hydrogen deploy
```

### 4. デプロイフロー（GitHub Actions 自動化）

`.github/workflows/deploy.yml`:

```yaml
name: Deploy to Oxygen

on:
  push:
    branches: [main]

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node
        uses: actions/setup-node@v4
        with:
          node-version: 20

      - name: Install pnpm
        uses: pnpm/action-setup@v3
        with:
          version: 8

      - name: Install dependencies
        run: pnpm install

      - name: Build
        run: pnpm build

      - name: Deploy to Oxygen
        run: shopify hydrogen deploy
        env:
          SHOPIFY_CLI_PARTNERS_TOKEN: ${{ secrets.SHOPIFY_CLI_PARTNERS_TOKEN }}
```

### 5. デプロイ確認

- **プレビュー URL**: `https://<hash>.o2.shopify.dev`
- **本番 URL**: Shopify 管理画面 → Hydrogen で確認

---

## トラブルシューティング

### `shopify hydrogen link` が失敗する

```bash
shopify auth login --store your-store.myshopify.com
```

再ログイン後にリンク実行。

### ビルドエラー: モジュールが見つからない

```bash
pnpm install
pnpm build
```

`node_modules` を削除して再インストール:

```bash
rm -rf node_modules .cache
pnpm install
```

### Storefront API 401 エラー

- `PUBLIC_STOREFRONT_API_TOKEN` が正しいか確認
- ヘッドレスチャネルの API パーミッションを確認
- `PUBLIC_STORE_DOMAIN` が `*.myshopify.com` 形式か確認

### Oxygen デプロイ後に環境変数が反映されない

- Oxygen ダッシュボードで変数を設定後、再デプロイが必要
- `shopify hydrogen env push` で `.env` を Oxygen に同期

---

## 参考リンク

- [Shopify Hydrogen 公式ドキュメント](https://shopify.dev/docs/custom-storefronts/hydrogen)
- [Hydrogen GitHub](https://github.com/Shopify/hydrogen)
- [Oxygen デプロイガイド](https://shopify.dev/docs/custom-storefronts/hydrogen/deployments/oxygen)
- [Storefront API リファレンス](https://shopify.dev/docs/api/storefront)
- [Remix ドキュメント](https://remix.run/docs)
