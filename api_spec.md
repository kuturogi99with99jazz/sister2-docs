# 📘 Sister 2 API仕様書（ドラフト）

## 1. 方針

- APIはRESTで設計する
- GraphQLは将来検討とする
- チャットは初期はHTTPベースで実装し、将来SSE/WebSocketを検討する

---

## 2. 共通仕様

### 2.1 共通エラー仕様（案）

| エラーコード | HTTP | 説明 |
|-------------|------|------|
| auth_unauthorized | 401 | 認証情報が無効 |
| auth_forbidden | 403 | 権限不足 |
| resource_not_found | 404 | リソース未検出 |
| validation_failed | 422 | バリデーションエラー |
| rate_limit_exceeded | 429 | レート制限超過 |
| server_error | 500 | サーバ内部エラー |

### 2.2 エラーレスポンス（共通）

| 項目 | 型 | 説明 |
|------|----|------|
| error | object | エラー情報 |
| error.code | string | エラーコード |
| error.message | string | ユーザー向け文言 |
| error.details | object | 詳細情報（任意） |

### 2.3 共通レスポンス（概要）

| 項目 | 型 | 説明 |
|------|----|------|
| id | string | リソースID |
| created_at | string | ISO8601 |
| updated_at | string | ISO8601 |

---

## 3. チャットAPI（案）

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

### 3.1 POST /chats（スレッド作成）

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

### 3.2 POST /chats/{chatId}/messages（メッセージ投稿）

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

### 3.3 GET /chats/{chatId}/messages（メッセージ一覧）

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

### 3.4 POST /chats/{chatId}/read（既読更新）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| last_read_message_id | string | yes | 最終既読メッセージID |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| status | string | ok |
| [Assumption] last_read_at | string | サーバ側で自動設定 |

---

## 4. 社内ツールAPI（案）

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

### 4.1 POST /internal-tools（ツール作成）

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

### 4.2 POST /internal-tools/{toolId}/entries（申請/入力作成）

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

### 4.3 POST /internal-tools/{toolId}/entries/{entryId}/approve（承認）

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

### 4.4 POST /internal-tools/{toolId}/entries/{entryId}/reject（差戻し）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| reason | string | yes | 差戻し理由 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| entry | object | 更新後のentry |

---

## 5. 追加検討事項

- [Open Question] API認可ポリシーの詳細（リソース単位の権限チェック）
- [Open Question] rate_limitの閾値/適用単位（ユーザー/組織/IP）
- [Open Question] SSE/WebSocket移行条件（同時接続数・通知遅延・UI要件で判断）
- [Risk] entries更新と監査ログ/変更履歴の整合ルールが未定義
