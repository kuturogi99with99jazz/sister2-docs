# 📄 Sister 2 PoC 開発環境構築手順書（ドラフト）

## 0. 基本情報

| 項目 | 内容 |
|---|---|
| ドキュメント種別 | PoC開発環境構築手順書 |
| 対象 | PoCのローカル開発環境 |
| 前提 | Dockerベースの開発（ローカル汚染を避ける） |
| 更新方針 | PoC構成確定に合わせて随時更新 |

---

## 1. 前提環境（提供情報）

### 1.1 端末/ツール

- Mac mini M4（32GB）
- Docker Desktop
- VSCode
- DataGrip（フリー版）
- ChatGPTアプリ

### 1.2 アカウント

- GitHub
- AWS
- Vercel
- Neon
- ChatGPT Plus（Codex利用可）

### 1.3 開発方針（個人開発スタイル）

- [Assumption] 開発環境はDockerコンテナ内に構築し、VSCodeでコンテナにアタッチして作業する
- [Assumption] ローカル環境に依存関係を直接インストールしない
- [Assumption] SecretsはVercel/AWSの環境変数で管理し、命名規則を事前に定める
- [Assumption] 環境分離方針（単一/複数）はPoC開始前に決定する

### 1.4 作業分担と実行ポリシー

- [Assumption] 実行（コマンド/インストール/GUI設定）はユーザーが全て担当する
- [Assumption] AIエージェントは「実行手順」「必要コマンド」「設定内容」を提示し、ユーザーに実行を依頼できる

### 1.5 レビュー/差し戻しの判断基準

- [Assumption] 機密情報の扱いに懸念がある提案/実装が含まれる場合は差し戻す
- [Assumption] 期待したフローや実装と異なる場合は差し戻す

### 1.6 レビュー基準の具体例

#### 差し戻しの例

- シークレット/認証情報をコードやリポジトリに直接記載する提案
- 権限の過剰付与（管理者権限の常用など）を前提にした手順
- 予定していない技術・サービスの導入
- 想定と異なるデータフローや権限フローの変更

#### OKの例

- 機密情報は環境変数/Secrets管理に限定している
- 既定アーキテクチャ内での改善提案に留まっている
- 事前に合意した範囲での手順/設定変更のみ

### 1.7 修正依頼テンプレ

```
[差し戻し理由]
（例: 機密情報の扱いに懸念）

[期待する対応]
（例: Secretsに移動し、コードから削除）

[影響範囲]
（例: backend/ のデプロイ手順）
```

---

## 2. リポジトリ準備

1. GitHubでPoC用リポジトリを作成する
2. ローカルにクローンする
3. monorepo前提の基本ディレクトリ構成を用意する（例: `frontend/`, `backend/`, `infra/`, `docs/`）

### 2.1 monorepoの想定構成（例）

```
/
  frontend/
  backend/
  infra/
  docs/
  .github/
```

- [Assumption] PoCはmonorepoで進め、将来的に必要なら分割する

---

## 3. Docker開発環境（ローカル）

### 3.1 Docker Desktopの確認

- Docker Desktopが起動していることを確認する

### 3.2 開発用コンテナの雛形作成

- `Dockerfile` と `docker-compose.yml` を用意する
- フロント/バックを同一composeで起動できる構成を想定

- [Assumption] VSCode Remote Containers（Dev Containers）でアタッチする
- [Assumption] フロント/バックは別コンテナに分離する

---

## 4. フロントエンド（SvelteKit）

### 4.1 初期セットアップ方針

- SvelteKitをベースにPoCの最小画面を作成
- Vercelへのデプロイを前提に環境変数設計を行う

- [Assumption] UIフレームワークはshadcn-svelteを採用する
- [Assumption] フロントの入力バリデーションはZodを第一候補とする

### 4.1.1 shadcn-svelte 導入（PoC）

- SvelteKitプロジェクト作成後にshadcn-svelteを初期化する
- Tailwind CSSのセットアップが前提となるため、必要な設定を先に有効化する
- PoCで使うUIコンポーネントのみ追加し、最小構成で運用する

### 4.1.2 Tailwind CSS セットアップ（PoC）

- `frontend/` でSvelteKit向けのTailwind CSSを導入する
- `tailwind.config.*` と `postcss.config.*` を作成し、Svelteのテンプレート対象を指定する
- `src/app.css` に `@tailwind base/components/utilities` を追加する
- shadcn-svelteの初期化時に生成される設定との差分がないか確認する

### 4.2 Vercel連携

- VercelでGitHubリポジトリを連携
- PRプレビューとmainデプロイを有効化

---

## 5. バックエンド（FastAPI on AWS Lambda）

### 5.1 初期セットアップ方針

- FastAPIをベースに最小CRUDのAPIを作成
- Lambda実行を前提にServerless Frameworkでパッケージング/デプロイする
- [Assumption] ORMはSQLModelを第一候補とし、複雑クエリ時はSQLAlchemyに降りる前提で構成する
- [Assumption] マイグレーション運用はAlembic導入を第一候補として検討する

### 5.3 Serverless Framework方針（PoC）

- `backend/serverless.yml` を用意し、API Gateway + Lambdaを定義する
- CORSはAPI Gateway側で有効化する（Vercelからのブラウザアクセス前提）
- 環境変数（Neon接続情報、Cognito設定など）はServerless Frameworkで外出しする
- PoCではステージは1つに絞り、最小構成で回す

### 5.2 AWSリソース準備

- IAMユーザー/ロールの準備（最小権限）
- API Gateway + Lambda の疎通確認
- S3バケットの作成（署名URL検証用）
- Cognitoユーザープールの作成（PoC最小構成）

---

## 6. データベース（Neon）

### 6.1 Neonプロジェクト作成

- NeonでPoC用プロジェクトを作成
- 接続情報（URL/ユーザー/パスワード）を控える

### 6.2 接続検証

- バックエンドからの接続テスト
- 接続数の動作傾向を軽く確認

- [Assumption] PoCはFree枠で開始する
- [Assumption] Neonの接続方式（プール/リトライ/タイムアウト方針）はPoC開始前に仮決めする

---

## 7. 開発補助ツール

### 7.1 DataGrip

- Neonへ接続し、スキーマ/テーブルの確認に利用

### 7.2 ChatGPT/Codex

- 仕様整理/検証メモのドラフトに活用
- [Assumption] 重要な判断は人が行う（AIは補助）

---

## 8. CI/CD（PoC）

### 8.1 GitHub Actions

- バックエンドのテスト/ビルド/デプロイを自動化
- Secrets（AWS認証情報）をGitHubに登録
- [Assumption] mainマージ後の自動デプロイを最低要件とする

### 8.2 Vercel

- フロントの自動デプロイを有効化

---

## 9. 初期動作確認チェックリスト

- フロントのローカル起動ができる
- バックエンドのローカル起動ができる
- Vercelへの自動デプロイが通る
- Lambdaへの自動デプロイが通る
- Neon接続が成立する
- S3署名URLのアップロード/ダウンロードが通る
- Cognitoログインが成立する（単一ロール前提）

---

## 10. 未確定事項

- [Open Question] Dev Containerの構成詳細（frontend/backendの実ファイル配置方針）
