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

### 2.4 ナレッジ本文フォーマット

- ナレッジ本文はMarkdownで保存する
- 表示用にサーバ側でHTMLを生成して返却する
- 初期対応の記法は見出し/リスト/リンク/画像/コードブロック/インラインコード/太字/斜体/引用/水平線
- 画像はS3の社内限定公開バケットに保存し、1ファイル1MB上限とする
- 社内限定公開バケットは社内IP/VPNからの読み取りのみ許可する

### 2.5 ナレッジHTMLのサニタイズ方針

- 許可タグ: p, br, strong, em, code, pre, ul, ol, li, h1, h2, h3, h4, a, img, blockquote, hr
- 許可属性: a[href,title], img[src,alt,title]
- 許可URLスキーム: https, http, mailto
- 不許可: script/style/iframe, on*イベント属性, style属性

---

## 3. Work API（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /works | Work一覧取得 | フィルタ/ソート |
| POST | /works | Work作成 | 担当者/対象指定 |
| GET | /works/{workId} | Work詳細取得 | |
| PATCH | /works/{workId} | Work更新 | |
| DELETE | /works/{workId} | Work削除 | 論理削除 |
| POST | /works/{workId}/assign | 担当者割当 | 主担当必須 |
| POST | /works/{workId}/status | ステータス更新 | 監査ログ対象 |
| POST | /works/{workId}/complete | 完了処理 | WorkLog記録 |
| GET | /works/{workId}/logs | 操作ログ取得 | 稼働推定 |
| POST | /works/{workId}/logs | 操作ログ追加 | start/pause/complete |
| GET | /search | 横断検索 | Work/チャット |

### 3.1 POST /works（Work作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| title | string | yes | タイトル |
| content | string | yes | 本文 |
| status | string | yes | 初期ステータス |
| work_type_id | string | yes | 業務の性質 |
| work_time_type_id | string | yes | 時間的特性 |
| template_id | string | no | ToolTemplate ID |
| form_data | object | no | テンプレート入力値 |
| target_links | object[] | no | 対象紐付け |
| target_links[].target_type | string | no | project / system / common |
| target_links[].target_id | string | no | 対象ID |
| primary_assignee_id | string | yes | 主担当 |
| participant_ids | string[] | no | 参加者 |
| start_date_planned | string | yes | 予定開始日 |
| end_date_planned | string | yes | 予定終了日 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| work | object | Workオブジェクト |
| [Assumption] created_by | string | JWTから自動取得 |

### 3.2 PATCH /works/{workId}（Work更新）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| title | string | no | タイトル |
| content | string | no | 本文 |
| status | string | no | ステータス |
| start_date_actual | string | no | 実績開始日 |
| end_date_actual | string | no | 実績終了日 |
| tags | string[] | no | タグID配列 |
| form_data | object | no | テンプレート入力値 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| work | object | 更新後のWork |

### 3.3 POST /works/{workId}/assign（担当者割当）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| primary_assignee_id | string | yes | 主担当 |
| participant_ids | string[] | no | 参加者 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| work | object | 更新後のWork |

### 3.4 POST /works/{workId}/complete（完了処理）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| note | string | no | 完了メモ |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| work | object | 更新後のWork |

- [Assumption] 完了処理で end_date_actual を自動記録し、必要に応じて後から編集可能とする

### 3.5 POST /works/{workId}/logs（操作ログ追加）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| action | string | yes | start / pause / resume / comment / complete |
| meta | object | no | 補足情報（任意） |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| log | object | 追加されたWorkLog |

### 3.6 GET /search（横断検索）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| q | string | yes | 検索キーワード |
| types | string[] | no | work / chat |
| limit | number | no | 取得件数 |
| cursor | string | no | ページングカーソル |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| results | object[] | 検索結果 |
| results[].type | string | work / chat |
| results[].id | string | リソースID |
| results[].title | string | Workタイトル / チャットスレッド名 |
| results[].snippet | string | 該当箇所の抜粋 |
| results[].updated_at | string | 更新日時 |
| next_cursor | string | 次ページカーソル |

- [Assumption] PoCではPostgreSQLの全文検索で成立性のみ確認する

---

## 4. Work対象 API（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /work-targets | 対象一覧取得 | 種別フィルタ |
| POST | /work-targets | 対象作成 | 管理者のみ |
| GET | /work-targets/{targetId} | 対象詳細取得 | |
| PATCH | /work-targets/{targetId} | 対象更新 | |
| DELETE | /work-targets/{targetId} | 対象削除 | 論理削除 |

### 4.1 POST /work-targets（対象作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| target_type | string | yes | project / system / common |
| name | string | yes | 名称 |
| description | string | no | 説明 |
| owner_user_id | string | no | 管理責任者 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| target | object | 対象オブジェクト |

---

## 5. チャットAPI（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /works/{workId}/chats | スレッド一覧取得 | Work単位 |
| GET | /work-targets/{targetId}/chats | スレッド一覧取得 | 対象単位 |
| POST | /chats | スレッド作成 | 初期メンバー指定 |
| GET | /chats/{chatId}/messages | メッセージ一覧取得 | ページング |
| POST | /chats/{chatId}/messages | メッセージ投稿 | 文字/添付 |
| PATCH | /chats/{chatId}/messages/{messageId} | メッセージ編集 | 投稿者のみ |
| DELETE | /chats/{chatId}/messages/{messageId} | メッセージ削除 | 投稿者/管理者 |
| POST | /chats/{chatId}/read | 既読更新 | ユーザー単位 |

### 5.1 POST /chats（スレッド作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| scope_type | string | yes | work / target |
| scope_id | string | yes | workId / targetId |
| title | string | no | スレッド名 |
| member_ids | string[] | no | 初期メンバー |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| chat | object | chatオブジェクト |
| [Assumption] created_by | string | JWTから自動取得 |

### 5.2 POST /chats/{chatId}/messages（メッセージ投稿）

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

### 5.3 GET /chats/{chatId}/messages（メッセージ一覧）

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

### 5.4 POST /chats/{chatId}/read（既読更新）

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

## 6. 画面評価API（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /screen-routes | 画面一覧取得 | ルート単位 |
| GET | /screen-feedback-templates | 理由テンプレ一覧取得 | 任意選択 |
| POST | /screen-feedbacks | 画面評価の作成/更新 | 最新1件を上書き |
| GET | /screen-feedbacks/summary | 画面評価の集計取得 | 公開集計 |

### 6.1 POST /screen-feedbacks（画面評価の作成/更新）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| route_path | string | yes | 画面ルート（例: /work） |
| rating | string | yes | good / bad |
| reason | string | no | 理由（最大400文字） |
| template_id | string | no | テンプレ選択 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| feedback | object | 評価オブジェクト |
| summary | object | 集計オブジェクト |
| [Assumption] user_id | string | JWTから自動取得 |

- [Assumption] 評価の作成/更新は監査ログに記録する
- [Assumption] 画面一覧はルートの静的定義で管理する

### 6.2 GET /screen-feedbacks/summary（画面評価の集計取得）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| route_path | string | yes | 画面ルート |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| route_path | string | 画面ルート |
| good_count | number | Good件数 |
| bad_count | number | Bad件数 |
| total_count | number | 合計件数 |
| good_rate | number | Good率（0-1） |
| updated_at | string | 集計更新日時 |

---

## 7. 帳票API（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| POST | /reports/works/export | Work一覧の帳票出力（PoC） | Excel/PDFの最小出力 |
| GET | /reports/templates | 帳票テンプレ一覧取得 | 権限に応じてフィルタ |
| POST | /reports/templates | 帳票テンプレ作成 | 管理者のみ |
| GET | /reports/templates/{reportTemplateId} | 帳票テンプレ詳細取得 | |
| PATCH | /reports/templates/{reportTemplateId} | 帳票テンプレ更新 | 差し替え対応 |
| GET | /reports/templates/{reportTemplateId}/runs | 生成履歴取得 | |
| POST | /reports/templates/{reportTemplateId}/runs | 帳票生成（手動） | 生成ジョブ起動 |
| GET | /reports/runs/{reportRunId} | 生成結果取得 | |
| GET | /reports/runs/{reportRunId}/file | 出力ファイル取得 | 署名URL発行 |
| POST | /reports/templates/{reportTemplateId}/schedules | 定期生成スケジュール作成 | 管理者のみ |
| PATCH | /reports/templates/{reportTemplateId}/schedules/{scheduleId} | 定期生成スケジュール更新 | 管理者のみ |

### 7.1 POST /reports/templates（帳票テンプレ作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| name | string | yes | 帳票名 |
| report_type | string | yes | work_summary / attendance / utilization / knowledge_list |
| output_formats | string[] | yes | xlsx / pdf |
| template_file_key | string | no | ExcelテンプレS3キー |
| retention_days | number | no | 保存期間（日数） |
| is_immutable | boolean | no | 改ざん防止 |
| access_roles | string[] | yes | 閲覧/生成可能ロール |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| template | object | テンプレオブジェクト |
| [Assumption] created_by | string | JWTから自動取得 |

### 7.2 POST /reports/templates/{reportTemplateId}/runs（帳票生成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| parameters | object | no | 期間/対象などの条件 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| run | object | 生成ジョブ |
| [Assumption] trigger_type | string | manual |

### 7.3 POST /reports/templates/{reportTemplateId}/schedules（定期生成スケジュール作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| cron | string | yes | cron式 |
| timezone | string | no | タイムゾーン |
| is_active | boolean | no | 有効/無効 |

### 7.4 POST /reports/works/export（Work一覧の帳票出力・PoC）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| format | string | yes | xlsx / pdf |
| date_from | string | no | 期間開始（ISO 8601） |
| date_to | string | no | 期間終了（ISO 8601） |
| assignee_id | string | no | 担当者ID |
| target_type | string | no | system / project |
| target_id | string | no | 対象ID |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| file_url | string | 署名URL（ダウンロード用） |
| expires_at | string | 署名URL有効期限 |
| format | string | 出力フォーマット |

- [Assumption] PoCでは一覧相当の最小項目のみを出力し、項目定義は次フェーズで確定する

#### PoC出力項目（ドラフト）

| 項目 | 型 | 説明 |
|------|----|------|
| work_id | string | Work ID（必須） |
| title | string | Workタイトル（必須） |
| status | string | ステータス（必須） |
| assignee_name | string | 担当者名（必須） |
| due_date | string | 期限（ISO 8601、必須） |
| target_type | string | system / project（任意） |
| target_name | string | 対象名（任意） |
| started_at | string | 開始日時（ISO 8601、任意） |
| completed_at | string | 完了日時（ISO 8601、任意） |
| updated_at | string | 更新日時（ISO 8601、任意） |
| parameters | object | no | 生成条件 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| schedule | object | スケジュールオブジェクト |

- [Assumption] 定期生成はスケジュール設定により自動実行される
- [Assumption] 保存期間・改ざん防止は帳票テンプレート単位で適用される

---

## 8. テンプレートAPI（ToolTemplate / 社内ツール定義）（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /templates | テンプレート一覧取得 | 権限に応じてフィルタ |
| POST | /templates | テンプレート作成 | 管理者のみ |
| GET | /templates/{templateId} | テンプレート詳細取得 | 定義を含む |
| PATCH | /templates/{templateId} | テンプレート更新 | 定義変更 |
| POST | /templates/{templateId}/entries | 申請/入力作成 | フォーム入力 |
| GET | /templates/{templateId}/entries | 申請/入力一覧 | 状態フィルタ |
| PATCH | /templates/{templateId}/entries/{entryId} | 申請/入力更新 | 差戻し対応 |
| POST | /templates/{templateId}/entries/{entryId}/approve | 承認 | 2段階対応 |
| POST | /templates/{templateId}/entries/{entryId}/reject | 差戻し | 理由必須 |
| GET | /templates/{templateId}/reports | 集計取得 | 集計タイプのみ |

### 8.1 POST /templates（テンプレート作成）

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

### 8.2 POST /templates/{templateId}/entries（申請/入力作成）

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

### 8.3 POST /templates/{templateId}/entries/{entryId}/approve（承認）

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

### 8.4 POST /templates/{templateId}/entries/{entryId}/reject（差戻し）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| reason | string | yes | 差戻し理由 |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| entry | object | 更新後のentry |

---

## 9. AI補助API（案）

AI補助は明示的な操作でのみ実行し、自動実行は行わない。

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| POST | /ai/works/{workId}/summarize | Work要約 | 明示実行のみ |
| POST | /ai/works/{workId}/draft | Work下書き作成 | 明示実行のみ |

### 9.1 POST /ai/works/{workId}/summarize（要約）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| style | string | no | short / bullets / detail |

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| summary | string | 要約本文 |

---

## 10. お知らせAPI（案）

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /announcements | お知らせ一覧取得 | 有効期間/対象でフィルタ |

### 10.1 GET /announcements（お知らせ一覧取得）

Response:

| 項目 | 型 | 説明 |
|------|----|------|
| announcements | object[] | お知らせ一覧 |
| announcements[].id | string | お知らせID |
| announcements[].title | string | タイトル |
| announcements[].body | string | 本文 |
| announcements[].link_url | string | 遷移先URL |
| announcements[].priority | string | info / warning / critical |
| announcements[].start_at | string | ISO8601 |
| announcements[].end_at | string | ISO8601 |

### 10.2 POST /admin/announcements（お知らせ作成）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| title | string | yes | タイトル |
| body | string | yes | 本文 |
| link_url | string | no | 遷移先URL |
| priority | string | yes | info / warning / critical |
| start_at | string | yes | 表示開始 |
| end_at | string | yes | 表示終了 |
| target_scope | string | yes | global / company / role |
| target_ids | string[] | no | company_id / role_id |

### 10.3 PATCH /admin/announcements/{announcementId}（お知らせ更新）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| title | string | no | タイトル |
| body | string | no | 本文 |
| link_url | string | no | 遷移先URL |
| priority | string | no | info / warning / critical |
| start_at | string | no | 表示開始 |
| end_at | string | no | 表示終了 |
| target_scope | string | no | global / company / role |
| target_ids | string[] | no | company_id / role_id |

### 10.4 POST /announcements/{announcementId}/dismiss（お知らせ閉じる）

Request:

| 項目 | 型 | 必須 | 説明 |
|------|----|------|------|
| dismissed | boolean | yes | true固定 |

---

## 11. 管理業務API（案）

管理UIは「運用者向け（限定権限）」と「管理者向け（フル権限）」を分離し、メンテ対象は「マスタ系＋運用で必要な一部のみ」とする。  
運用者は masterType の範囲を限定したCRUDのみ許可し、管理者は全 masterType を対象とする。

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /admin/masters | マスタ種別一覧取得 | 管理対象の列挙 |
| GET | /admin/masters/{masterType} | マスタ一覧取得 | 検索/フィルタ |
| POST | /admin/masters/{masterType} | マスタ作成 | 運用者/管理者（masterTypeで制御） |
| PATCH | /admin/masters/{masterType}/{id} | マスタ更新 | 運用者/管理者（masterTypeで制御） |
| DELETE | /admin/masters/{masterType}/{id} | マスタ削除（無効化） | 運用者/管理者（masterTypeで制御） |
| POST | /admin/masters/{masterType}/bulk | 一括登録/更新 | 運用者/管理者（masterTypeで制御） |

masterType（必須マスタ）:
- users, companies, branches, divisions, roles, resources, permissions
- work-types, work-time-types, work-tags, work-target-types
- blog-tags, blog-categories

| メソッド | パス | 目的 | 備考 |
|----------|------|------|------|
| GET | /admin/settings/system | 全体共通設定取得 | 管理者のみ |
| PATCH | /admin/settings/system | 全体共通設定更新 | 履歴保持 |
| GET | /admin/settings/companies/{companyId} | 組織設定取得 | 会社単位 |
| PATCH | /admin/settings/companies/{companyId} | 組織設定更新 | 履歴保持 |

- [Assumption] users は Cognito と同期し、編集可能範囲はアプリ側のプロフィール/ステータスに限定
- [Assumption] DELETE は物理削除ではなく無効化（アーカイブ）を既定とする

---

## 12. 追加検討事項

- [Open Question] API認可ポリシーの詳細（リソース単位の権限チェック）
- [Open Question] 運用者に許可する masterType の範囲（CRUD対象の限定リスト）
- [Open Question] rate_limitの閾値/適用単位（ユーザー/組織/IP）
- [Open Question] SSE/WebSocket移行条件（同時接続数・通知遅延・UI要件で判断）
- [Open Question] 設定の優先順位（全体共通 vs 組織単位）
- [Open Question] Cognitoとユーザー/ロール管理の同期方式
- [Risk] Work操作ログと監査ログの整合ルールが未定義
