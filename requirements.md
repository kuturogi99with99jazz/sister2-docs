# 📗 Sister 2 要件定義書（たたき）

## 1. システム概要

Sister 2は、既存のプロジェクト進捗管理システムをモダンアーキテクチャで再構築することを目的としています。  
目的は「低コスト・高拡張性・高可用性」を両立し、将来的なAI連携・自動化に対応できる設計を採用します。

### 1.1 追加検討事項

- 想定ユーザー規模
  - 同時接続数: 10-100
  - 総ユーザー数: 30-300
  - 組織数: 1-10

---

## 2. 目的・背景

| 項目 | 内容 |
|------|------|
| 現状の課題 | サーバ運用負荷、保守コスト、可視化不足 |
| 解決方針 | サーバレス構成（AWS Lambda / Fargate）化、CI/CD導入、AI連携 |
| 目的 | 継続的な改善と低コスト運用を実現する基盤構築 |

- KPI目標値
  - MAU: 100以上
  - DAU: 20以上
  - アクティブ組織数: 5以上
  - タスク完了リードタイム中央値: 3日以内
  - 可用性: 99.9%以上
  - 月額コスト: 50ドル以下

---

## 3. スコープ（対象/非対象）

- 対象: 現行Sisterの主要機能（プロジェクト管理、タスク管理、ナレッジ管理、通知、ファイル管理、分析、認証・認可）の再構築
- 対象: チャット機能、社内ツール（定義駆動型）の追加
- 非対象: 現行Sisterのソフトウェア開発報告書チェックツールは社内ツールに吸収するので対象外、現行AI分析も対象外

---

## 4. 機能要件（概要）

| 大分類 | 機能 | 説明 |
|--------|------|------|
| プロジェクト管理 | 一覧・登録・削除 | 進捗・担当者・日付を管理 |
| タスク管理 | CRUD / 状態変更 | ステータス・期日・担当者を追跡 |
| ナレッジ管理 | ブログ／ドラフト管理 | 記事投稿・タグ付け・AI要約 |
| 通知機能 | SES/SNS送信 | タスク完了・期限アラート通知 |
| ファイル管理 | S3アップロード | 署名付きURL発行 |
| 分析機能 | 進捗率・稼働時間可視化 | Fargateで定期処理 |
| 社内ツール | 定義駆動型ワークフロー | フォーム/ワークフロー/集計の3タイプ |
| チャット機能 | プロジェクト/タスク単位の会話 | 初期はHTTPベース、将来SSE/WebSocket検討 |

- MVP範囲（必須/後回し）の機能優先度は「本章の上から順」

---

## 4.1 チャット機能 要件詳細（テンプレート）

| 項目 | 内容 |
|------|------|
| 目的 | [Suggestion] プロジェクト/タスク単位の進捗共有と意思決定の記録を簡易化 |
| 対象スコープ | [Assumption] プロジェクト単位 / タスク単位 |
| 参加者 | [Suggestion] 当該プロジェクト/タスクに所属するメンバー |
| 権限 | [Suggestion] 削除は投稿者と管理者に限定、閲覧/投稿はRBACに準拠 |
| メッセージ種別 | [Suggestion] テキスト / ファイル / システム通知 |
| スレッド構造 | [Suggestion] スレッド＋時系列（返信は任意） |
| 通知 | [Suggestion] 新着/メンション/期限関連の通知をSES/SNSで送信 |
| 添付 | [Suggestion] S3署名URLで安全にアップロード/閲覧 |
| 監査・ログ | [Suggestion] 送信・編集・削除を監査ログに記録 |
| 保存期間 | [Assumption] 最大1年（監査要件に応じて変更可） |
| UX要件 | [Suggestion] 既読/未読、検索、メンション、簡易リアクション |
| [Open Question] SSE/WebSocket移行条件 | [Suggestion] 同時接続数・通知遅延・UI要件で判断 |

### 4.1.1 追加検討事項

- [Open Question] systemメッセージの生成主体と投稿制限（サーバのみ/管理者のみ）
- [Open Question] chat_members.role の値域（owner/moderator/member など）
- [Open Question] 既読/未読の表示単位（スレッド/メッセージ）
- [Risk] 保存期間1年の根拠（法務/監査要件）の確認が必要

---

## 4.2 社内ツール 要件詳細（テンプレート）

| 項目 | 内容 |
|------|------|
| 目的 | [Suggestion] 定期作業・雑務のシステム化と属人化解消 |
| 方式 | [Assumption] 定義駆動（フォーム / ワークフロー / 集計） |
| 対象業務 | [Suggestion] 申請、報告、チェックリスト、集計業務 |
| 定義管理 | [Suggestion] 管理者のみ編集可能、公開/下書きの状態管理 |
| 入力検証 | [Suggestion] 定義ベースの型・必須・範囲チェック |
| 承認フロー | [Suggestion] 2段階の承認に対応（定義で設定） |
| 通知 | [Suggestion] 申請/承認/差戻し通知をSES/SNSで送信 |
| 権限 | [Suggestion] RBACによりツール単位で閲覧/操作を制御 |
| 監査・ログ | [Suggestion] 定義変更・申請・承認の履歴を記録 |
| 保存期間 | [Assumption] 最大1年（監査要件に応じて変更可） |
| [Open Question] 既存Excel移行方針 | [Suggestion] 重要帳票から優先移行、テンプレート化で段階移行 |
| [Assumption] ステータス遷移 | draft → submitted → approved / rejected → resubmitted → cancelled |
| [Assumption] 承認確定条件 | step1 と step2 の両方通過で approved 確定 |

### 4.2.1 追加検討事項

- [Open Question] 承認者割当ルール（step1/step2のロール紐付け）
- [Open Question] resubmitted時の取り扱い（履歴保持/再承認条件）
- [Risk] 定義変更と既存申請データの整合ルールが未定義

---

## 4.3 API仕様テンプレート（チャット/社内ツール）

### チャットAPI（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /projects/{projectId}/chats | スレッド一覧取得 | プロジェクト単位 |
| GET | /tasks/{taskId}/chats | スレッド一覧取得 | タスク単位 |
| POST | /chats | スレッド作成 | 初期メンバー指定 |
| GET | /chats/{chatId}/messages | メッセージ一覧取得 | ページング |
| POST | /chats/{chatId}/messages | メッセージ投稿 | 文字/添付 |
| PATCH | /chats/{chatId}/messages/{messageId} | メッセージ編集 | 投稿者のみ |
| DELETE | /chats/{chatId}/messages/{messageId} | メッセージ削除 | 投稿者/管理者 |
| POST | /chats/{chatId}/read | 既読更新 | ユーザー単位 |

#### 共通エラー仕様（案）

| エラーコード | HTTP | 説明 |
|-------------|------|------|
| auth_unauthorized | 401 | 認証情報が無効 |
| auth_forbidden | 403 | 権限不足 |
| resource_not_found | 404 | リソース未検出 |
| validation_failed | 422 | バリデーションエラー |
| rate_limit_exceeded | 429 | レート制限超過 |
| server_error | 500 | サーバ内部エラー |

#### エラーレスポンス（共通）

| 項目 | 型 | 説明 |
|------|----|------|
| error | object | エラー情報 |
| error.code | string | エラーコード |
| error.message | string | ユーザー向け文言 |
| error.details | object | 詳細情報（任意） |

#### 共通レスポンス（概要）

| 項目 | 型 | 説明 |
|------|----|------|
| id | string | リソースID |
| created_at | string | ISO8601 |
| updated_at | string | ISO8601 |

#### POST /chats（スレッド作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| scope_type | string | yes | project / task |
| scope_id | string | yes | projectId / taskId |
| title | string | no | スレッド名 |
| member_ids | string[] | no | 初期メンバー |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| chat | object | chatオブジェクト |
| [Assumption] created_by | string | JWTから自動取得 |

#### POST /chats/{chatId}/messages（メッセージ投稿）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| content | string | yes | テキスト本文 |
| message_type | string | no | text / system / file（固定） |
| parent_message_id | string | no | 返信元メッセージ |
| attachments | object[] | no | S3署名URL情報 |
| attachments[].file_key | string | no | S3キー |
| attachments[].file_name | string | no | ファイル名 |
| attachments[].content_type | string | no | MIMEタイプ |
| attachments[].size_bytes | number | no | バイトサイズ |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| message | object | messageオブジェクト |

#### GET /chats/{chatId}/messages（メッセージ一覧）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| limit | number | no | 取得件数 |
| cursor | string | no | ページングカーソル |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| messages | object[] | message配列 |
| next_cursor | string | 次ページカーソル |

#### POST /chats/{chatId}/read（既読更新）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| last_read_message_id | string | yes | 最終既読メッセージID |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| status | string | ok |
| [Assumption] last_read_at | string | サーバ側で自動設定 |

### 社内ツールAPI（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /internal-tools | ツール一覧取得 | 権限に応じてフィルタ |
| POST | /internal-tools | ツール作成 | 管理者のみ |
| GET | /internal-tools/{toolId} | ツール詳細取得 | 定義を含む |
| PATCH | /internal-tools/{toolId} | ツール更新 | 定義変更 |
| POST | /internal-tools/{toolId}/entries | 申請/入力作成 | フォーム入力 |
| GET | /internal-tools/{toolId}/entries | 申請/入力一覧 | 状態フィルタ |
| PATCH | /internal-tools/{toolId}/entries/{entryId} | 申請/入力更新 | 差戻し対応 |
| POST | /internal-tools/{toolId}/entries/{entryId}/approve | 承認 | 2段階対応 |
| POST | /internal-tools/{toolId}/entries/{entryId}/reject | 差戻し | 理由必須 |
| GET | /internal-tools/{toolId}/reports | 集計取得 | 集計タイプのみ |

#### POST /internal-tools（ツール作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| name | string | yes | ツール名 |
| type | string | yes | form / workflow / report |
| definition | object | yes | 定義JSON |
| status | string | no | draft / published |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| tool | object | toolオブジェクト |
| [Assumption] created_by | string | JWTから自動取得 |
| [Assumption] version | string | 自動増分（更新時に+1） |

#### POST /internal-tools/{toolId}/entries（申請/入力作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| payload | object | yes | 入力値（定義に準拠） |
| attachments | object[] | no | S3署名URL情報 |
| attachments[].file_key | string | no | S3キー |
| attachments[].file_name | string | no | ファイル名 |
| attachments[].content_type | string | no | MIMEタイプ |
| attachments[].size_bytes | number | no | バイトサイズ |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| entry | object | entryオブジェクト |

#### POST /internal-tools/{toolId}/entries/{entryId}/approve（承認）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| step | number | yes | 1 or 2 |
| comment | string | no | 承認コメント |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| entry | object | 更新後のentry |
| [Assumption] approver_role | string | manager / director / admin |

#### POST /internal-tools/{toolId}/entries/{entryId}/reject（差戻し）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| reason | string | yes | 差戻し理由 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| entry | object | 更新後のentry |

### 4.3.1 追加検討事項

- [Open Question] API認可ポリシーの詳細（リソース単位の権限チェック）
- [Open Question] rate_limitの閾値/適用単位（ユーザー/組織/IP）
- [Risk] entries更新と監査ログ/変更履歴の整合ルールが未定義

---

## 4.4 プロジェクト管理 要件詳細（現行UIベース）

| 項目 | 内容 |
|------|------|
| 一覧表示 | [Assumption] サマリー（Total/Completed/In Progress）とカード一覧 |
| 絞り込み | [Suggestion] ステータス/担当者/日付範囲/タグ |
| 主要アクション | [Assumption] 一覧で「Show Filters」「Create New Project」 |
| カード表示 | [Assumption] ID、名称、説明、ステータス、計画/実績日付、タスク数、PIC数 |
| 詳細画面 | [Suggestion] プロジェクト概要 + タスク一覧 + Gantt |
| 添付 | [Assumption] Upload/Download（S3署名URL） |
| 編集 | [Assumption] Edit/Delete（管理者 or 作成者） |
| 画像 | [Suggestion] プロジェクト代表画像（任意） |
| 監査 | [Suggestion] 作成/更新/削除の監査ログ |

### 4.4.1 プロジェクト登録/編集フォーム（現行項目）

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| project_id | string | yes | 画面上のID（英数） |
| company_id | string | yes | 会社ID |
| branch_id | string | yes | 拠点ID |
| division_id | string | yes | 部門ID |
| name | string | yes | プロジェクト名 |
| description | string | no | 概要 |
| start_date_planned | date | yes | 計画開始日 |
| end_date_planned | date | yes | 計画終了日 |
| start_date_actual | date | no | 実績開始日 |
| end_date_actual | date | no | 実績終了日 |

### 4.4.2 追加検討事項

- [Open Question] project_idの採番ルール（手動/自動/重複可否）
- [Assumption] 会社/拠点/部門のマスタ管理方針は別ドキュメントで定義
- [Open Question] Completed/In Progressの判定条件（ステータス/日付）
- [Risk] 実績日付が未入力のプロジェクト表示ルールが未定義

---

## 4.5 タスク管理 要件詳細（現行UIベース）

| 項目 | 内容 |
|------|------|
| 一覧表示 | [Assumption] サマリー（Total/Completed/In Progress）＋リスト |
| 絞り込み | [Suggestion] プロジェクト/担当者/ステータス/タグ/日付 |
| 主要アクション | [Assumption] 「Show Filters」「Create New Task」 |
| インポート/エクスポート | [Assumption] Upload/Download |
| リスト表示 | [Assumption] ステータス、タイトル、計画/実績日付、担当者、タグ |
| 詳細画面 | [Suggestion] タスク本文、コメント数、稼働時間、履歴 |
| タグ | [Assumption] 複数タグ付与（色付きラベル） |
| ステータス | [Assumption] In Progress / Completed など |
| 監査 | [Suggestion] 作成/更新/削除、ステータス変更履歴 |

### 4.5.1 タスク登録/編集フォーム（現行項目）

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| task_id | string | yes | 画面上のID |
| user_id | string | yes | 作成者/担当者ID |
| project_id | string | yes | 紐付けプロジェクト |
| title | string | yes | タスク名 |
| content | text | yes | 詳細内容 |
| start_date_planned | date | yes | 計画開始日 |
| end_date_planned | date | yes | 計画終了日 |
| start_date_actual | date | no | 実績開始日 |
| end_date_actual | date | no | 実績終了日 |
| hours_worked | number | no | 稼働時間 |
| status | string | yes | ステータス |
| tags | string[] | no | タグ |

### 4.5.2 追加検討事項

- [Assumption] statusの値域: Not Started / In Progress / Completed / On Hold / Cancelled
- [Assumption] hours_workedは手動入力のみ
- [Assumption] タグはマスタ管理（現行踏襲）
- [Risk] タスク一括アップロード時のバリデーション/重複処理が未定義

---

## 5. 非機能要件

| 分類 | 内容 |
|------|------|
| 可用性 | 99.9%以上（AWS SLA基準） |
| 拡張性 | サーバレス構成で自動スケール |
| 保守性 | GitHub Actionsによる自動CI/CD |
| ログ | CloudWatch + S3保存 |
| 通知 | SES/SNSで障害・完了通知 |
| コスト | 月額50ドル以下を目標 |

### 5.1 追加検討事項

- [Open Question] 性能要件（レスポンスタイム/スループット）
- [Assumption] 想定ユーザー規模に基づき性能要件を定義（同時接続数10-100、総ユーザー数30-300、組織数1-10）
- [Open Question] SLO/SLI定義（測定対象と基準）
- [Open Question] RTO/RPO（障害復旧/データ復旧）
- [Risk] $50目標の前提（ユーザー数/利用量）が未記載

---

## 6. セキュリティ・認証要件

| 項目 | 内容 |
|------|------|
| 認証 | AWS Cognito（JWT管理） |
| 認可 | Cognito + RBAC（ユーザー権限管理） |
| 通信 | HTTPS / JWT / IAM最小権限 |

### 6.1 追加検討事項

- [Open Question] RBACのロール一覧と権限マトリクス
- [Open Question] 監査ログの保持期間/閲覧権限
- [Risk] PII/機微情報の扱い（暗号化/マスキング）が未定義

---

## 7. 運用・保守方針

- デプロイはGitHub Actions経由で自動化（mainブランチPushで反映）  
- バックエンドはServerless FrameworkでAWSにデプロイ  
- エラーログ・メトリクスはCloudWatch管理  
- DBマイグレーションはAlembicまたはSQLAlchemyで自動化  

### 7.1 追加検討事項

- [Open Question] 障害時対応フロー（オンコール/通知先）
- [Open Question] バックアップ/復旧手順（Neon/S3）

---

## 8. データ連携・外部連携

| 区分 | 内容 |
|------|------|
| 外部AI連携 | OpenAI APIを利用し、要約／提案生成 |
| 外部通知 | AWS SES/SNSによるメール・Push通知 |
| ファイル署名 | S3署名URLによる安全なダウンロード／アップロード |

### 8.1 追加検討事項

- [Open Question] OpenAIへ送信するデータ範囲（マスキング有無）
- [Risk] 外部AI送信データの取り扱いポリシー未定義

---

## 9. 環境構成（開発/検証/本番）

| 区分 | 環境 | 備考 |
|------|------|------|
| 開発環境 | Docker Compose (Local) | Lambdaエミュレーション含む |
| 検証環境 | AWS（Staging） | CI/CD自動デプロイ |
| 本番環境 | AWS + Vercel | 常時稼働環境 |

### 9.1 追加検討事項

- [Open Question] 環境差分（スケール/課金/機能制限）

---

## 10. 移行要件

- 認証はGo LiveでCognitoへ一括切り替え（既存ユーザー移行を含む）

### 10.1 追加検討事項

- [Open Question] 既存ユーザー移行手順（ID連携/パスワードリセット）
- [Risk] Go Live一括切替の影響範囲/許容ダウンタイム

---

## 11. リスク・対策

| リスク | 対策 |
|--------|------|
| AWS課金増大 | 無料枠を最大活用、リソース監視 |
| OpenAI API利用制限 | キャッシュ・バッチ処理併用 |
| Neon接続制限 | プーリング／Fargate側で制御 |
| 認証管理複雑化 | Cognito Hosted UI採用 |

### 11.1 追加検討事項

- [Open Question] チャット/社内ツール利用拡大時の課金増対策

---

## 12. 成果物一覧

| 種別 | ファイル名 | 備考 |
|------|-------------|------|
| 提案資料 | proposal.md | アーキテクチャ概要 |
| 要件定義書 | requirements.md | 本書 |
| 設計書 | Architecture.md | インフラ・構成詳細 |
| ER図 | er_diagram.mmd | DB構造 |
| API仕様書 | api_spec.md | OpenAPI準拠 |

### 12.1 追加検討事項

- [Open Question] ER図/API仕様の更新ルール（版管理/レビュー手順）

---

## 13. 今後の課題

- RBAC設計の詳細化  
- AI要約プロンプト最適化  
- Fargateジョブスケジュール管理の具体化  
- コスト試算の再精査  

### 13.1 追加検討事項

- [Open Question] 課題の優先順位と期限

---

## 14. スケジュール（初期案）

| フェーズ | 期間 | 内容 |
|-----------|------|------|
| 要件定義 | 〜2025/11 | 各機能整理・レビュー |
| PoC構築 | 2025/12 | サーバレス＋Neon動作検証 |
| 実装 | 2026/01〜02 | 本実装・UIデザイン刷新 |
| テスト | 2026/03 | 自動・手動検証 |
| リリース | 2026/04 | 運用開始 |

### 14.1 追加検討事項

- [Risk] 人員/担当範囲の前提が未記載で実現性評価が困難
