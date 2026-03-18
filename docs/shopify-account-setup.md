# Shopify アカウント・開発ストア セットアップ手順

Hydrogen デモを動かすために必要な Shopify Partner アカウントと開発用ストアの準備手順。

---

## 目次

1. [Shopify Partner アカウント作成](#1-shopify-partner-アカウント作成)
2. [開発用ストアの作成](#2-開発用ストアの作成)
3. [Storefront API トークンの取得](#3-storefront-api-トークンの取得)
4. [Shopify CLI のセットアップ](#4-shopify-cli-のセットアップ)
5. [動作確認](#5-動作確認)

---

## 1. Shopify Partner アカウント作成

### 手順

1. [https://partners.shopify.com/signup](https://partners.shopify.com/signup) にアクセス
2. メールアドレス・パスワードを入力してアカウント作成
3. アカウント情報を入力（個人 or 企業、ビジネス種別など）
   - ビジネス種別: **I build apps or themes for Shopify merchants** を選択
4. メール認証を完了する

> Partner アカウントは無料。クレジットカード不要。

---

## 2. 開発用ストアの作成

### 手順

1. Partner ダッシュボード（[partners.shopify.com](https://partners.shopify.com)）にログイン
2. 左メニュー → **Stores** → **Add store** をクリック
3. **Create development store** を選択
4. 以下を設定:

   | 項目 | 設定値 |
   |------|--------|
   | Store name | `shopify-demo-dev`（任意） |
   | Store URL | `shopify-demo-dev.myshopify.com`（自動生成） |
   | Purpose | **Build and test a new app** を選択 |
   | Data | **Start with test data** を選択（ダミー商品が自動追加される） |

5. **Save** をクリック → ストア作成完了

### 作成後の確認

- ストア URL: `https://your-store-name.myshopify.com/admin`
- Partner ダッシュボード → Stores → 作成したストアの **Log in** からアクセス可能

---

## 3. Storefront API トークンの取得

Hydrogen が Shopify のデータを取得するために必要なトークン。

### 手順

1. 作成したストアの管理画面にログイン
   `https://your-store-name.myshopify.com/admin`

2. 左メニュー → **Settings**（設定）→ **Apps and sales channels**（アプリと販売チャネル）

3. 画面右上 **Develop apps**（アプリを開発する）をクリック

4. **Allow custom app development** → **Allow** をクリック（初回のみ）

5. **Create an app** をクリック
   - App name: `hydrogen-demo`（任意）
   - App developer: 自分のアカウントを選択

6. 作成したアプリを開き、**Configuration** タブへ移動

7. **Storefront API integration** セクション → **Configure** をクリック

8. 以下のスコープにチェックを入れて **Save**:

   ```
   unauthenticated_read_product_listings
   unauthenticated_read_collection_listings
   unauthenticated_read_content
   unauthenticated_write_checkouts
   unauthenticated_read_checkouts
   ```

9. **API credentials** タブ → **Storefront API access token** をコピー

### 取得できる情報

```env
PUBLIC_STORE_DOMAIN=your-store-name.myshopify.com
PUBLIC_STOREFRONT_API_TOKEN=コピーしたトークン
```

---

## 4. Shopify CLI のセットアップ

### インストール

```bash
npm install -g @shopify/cli
```

バージョン確認:

```bash
shopify version
# 3.x.x が表示されれば OK
```

### ログイン

```bash
shopify auth login
```

ブラウザが開いて Partner アカウントでの認証を求められる。認証後、CLI がストアと連携される。

### ストアと Hydrogen プロジェクトのリンク

Hydrogen プロジェクト作成後（`npm create @shopify/hydrogen@latest` 実行後）に実行:

```bash
cd shopify-demo
shopify hydrogen link
```

対話形式でストアを選択する。

---

## 5. 動作確認

### チェックリスト

- [ ] Partner アカウントにログインできる
- [ ] 開発用ストアの管理画面にアクセスできる
- [ ] `PUBLIC_STORE_DOMAIN` と `PUBLIC_STOREFRONT_API_TOKEN` を取得済み
- [ ] `shopify version` でバージョンが表示される
- [ ] `shopify auth login` が完了している

### テストデータの確認

開発ストアを **Start with test data** で作成した場合、以下が自動で追加されている:

- ダミー商品（複数）
- コレクション
- 顧客データ（テスト用）

Hydrogen の開発サーバー起動後にこれらのデータが表示されれば API 接続は成功。

---

## 次のステップ

アカウントとトークンの準備ができたら → [hydrogen-demo.md](./hydrogen-demo.md) の手順へ進む。

```bash
# プロジェクト作成
npm create @shopify/hydrogen@latest

# .env に取得した値を設定
PUBLIC_STORE_DOMAIN=your-store-name.myshopify.com
PUBLIC_STOREFRONT_API_TOKEN=xxxx

# 開発サーバー起動
pnpm dev
```
