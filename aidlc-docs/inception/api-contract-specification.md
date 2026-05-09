# API契約仕様書

**作成日**: 2026-05-10  
**目的**: Units Generation前にAPI契約の詳細を明確化し、フロントエンドとバックエンドのデータ構造の乖離を防ぐ

---

## 1. 概要

本ドキュメントは、コミュ♥外chuのAPI契約（Request/Response）の詳細仕様を定義します。

### 1.1 API設計原則

- **ハイブリッドアプローチ**: RESTfulベースに一部RPC-style
- **認証**: Amazon Cognito JWTトークン（Authorizationヘッダー）
- **エラーハンドリング**: 統一されたエラーレスポンス構造
- **バージョニング**: `/v1/` プレフィックス

---

## 2. 共通仕様

### 2.1 ベースURL

```
https://api.example.com/v1
```

### 2.2 認証ヘッダー

すべてのAPIリクエストには、以下のヘッダーが必要です：

```http
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json
```

### 2.3 統一エラーレスポンス構造

```json
{
  "error": {
    "code": "ERROR_CODE",
    "message": "Human-readable error message",
    "details": {
      "field": "Additional error details (optional)"
    }
  }
}
```

### 2.4 エラーコード一覧

| HTTPステータス | エラーコード | 説明 |
|---|---|---|
| 400 | `INVALID_REQUEST` | リクエストパラメータが不正 |
| 401 | `UNAUTHORIZED` | 認証トークンが無効または期限切れ |
| 403 | `FORBIDDEN` | アクセス権限がない |
| 404 | `NOT_FOUND` | リソースが見つからない |
| 409 | `CONFLICT` | リソースの競合（例: 既に存在する） |
| 500 | `INTERNAL_SERVER_ERROR` | サーバー内部エラー |
| 503 | `SERVICE_UNAVAILABLE` | サービスが一時的に利用不可 |

---

## 3. RESTful Resources

### 3.1 User Resource

#### GET /users/{userId}

**目的**: ユーザー情報を取得

**Request**:
```http
GET /v1/users/{userId}
Authorization: Bearer <JWT_TOKEN>
```

**Response** (200 OK):
```json
{
  "user_id": "user-12345-67890",
  "email": "kenta@example.com",
  "display_name": "健太",
  "auth_provider": "google",
  "fcm_device_token": "fcm-token-abc123...",
  "created_at": "2026-05-01T10:00:00Z",
  "updated_at": "2026-05-09T10:00:00Z"
}
```

**Error Response** (404 Not Found):
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "User not found"
  }
}
```

---

#### PUT /users/{userId}

**目的**: ユーザー情報を更新

**Request**:
```http
PUT /v1/users/{userId}
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "display_name": "健太（更新）",
  "fcm_device_token": "fcm-token-new123..."
}
```

**Response** (200 OK):
```json
{
  "user_id": "user-12345-67890",
  "email": "kenta@example.com",
  "display_name": "健太（更新）",
  "auth_provider": "google",
  "fcm_device_token": "fcm-token-new123...",
  "created_at": "2026-05-01T10:00:00Z",
  "updated_at": "2026-05-10T10:00:00Z"
}
```

---

### 3.2 Target Resource

#### GET /users/{userId}/targets

**目的**: ターゲット人物リストを取得

**Request**:
```http
GET /v1/users/{userId}/targets
Authorization: Bearer <JWT_TOKEN>
```

**Response** (200 OK):
```json
{
  "targets": [
    {
      "target_id": "target-11111-22222",
      "user_id": "user-12345-67890",
      "name": "叔父さん",
      "relationship": "親戚",
      "created_at": "2026-05-01T10:00:00Z",
      "updated_at": "2026-05-09T10:00:00Z"
    },
    {
      "target_id": "target-33333-44444",
      "user_id": "user-12345-67890",
      "name": "田中さん",
      "relationship": "同僚",
      "created_at": "2026-05-02T10:00:00Z",
      "updated_at": "2026-05-09T10:00:00Z"
    }
  ]
}
```

---

#### POST /users/{userId}/targets

**目的**: ターゲット人物を作成

**Request**:
```http
POST /v1/users/{userId}/targets
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "name": "山田さん",
  "relationship": "友人"
}
```

**Response** (201 Created):
```json
{
  "target_id": "target-55555-66666",
  "user_id": "user-12345-67890",
  "name": "山田さん",
  "relationship": "友人",
  "created_at": "2026-05-10T10:00:00Z",
  "updated_at": "2026-05-10T10:00:00Z"
}
```

---

#### GET /users/{userId}/targets/{targetId}

**目的**: ターゲット人物情報を取得

**Request**:
```http
GET /v1/users/{userId}/targets/{targetId}
Authorization: Bearer <JWT_TOKEN>
```

**Response** (200 OK):
```json
{
  "target_id": "target-11111-22222",
  "user_id": "user-12345-67890",
  "name": "叔父さん",
  "relationship": "親戚",
  "created_at": "2026-05-01T10:00:00Z",
  "updated_at": "2026-05-09T10:00:00Z"
}
```

---

#### PUT /users/{userId}/targets/{targetId}

**目的**: ターゲット人物情報を更新

**Request**:
```http
PUT /v1/users/{userId}/targets/{targetId}
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "name": "叔父さん（更新）",
  "relationship": "親戚（更新）"
}
```

**Response** (200 OK):
```json
{
  "target_id": "target-11111-22222",
  "user_id": "user-12345-67890",
  "name": "叔父さん（更新）",
  "relationship": "親戚（更新）",
  "created_at": "2026-05-01T10:00:00Z",
  "updated_at": "2026-05-10T10:00:00Z"
}
```

---

### 3.3 Episode Resource

#### GET /users/{userId}/episodes

**目的**: エピソードリストを取得

**Query Parameters**:
- `target_id` (optional): ターゲット人物IDでフィルタ
- `limit` (optional): 取得件数（デフォルト: 20）
- `offset` (optional): オフセット（デフォルト: 0）

**Request**:
```http
GET /v1/users/{userId}/episodes?target_id=target-11111-22222&limit=10
Authorization: Bearer <JWT_TOKEN>
```

**Response** (200 OK):
```json
{
  "episodes": [
    {
      "episode_id": "ep-12345",
      "user_id": "user-12345-67890",
      "target_id": "target-11111-22222",
      "episode_type": "received_gift",
      "episode_content": "叔父さんからお酒をもらった",
      "emotion_tag": "😊 元気そう",
      "is_ng_topic": false,
      "henpou_status": "stage2_scheduled",
      "stage1_timing": "2026-05-09T18:00:00Z",
      "stage2_timing": "2026-05-26T18:00:00Z",
      "stage2_timing_reason": "2-3weeks",
      "created_at": "2026-05-08T18:00:00Z",
      "updated_at": "2026-05-09T10:00:00Z"
    }
  ],
  "pagination": {
    "total": 1,
    "limit": 10,
    "offset": 0
  }
}
```

---

#### POST /users/{userId}/episodes

**目的**: エピソードを作成

**Request**:
```http
POST /v1/users/{userId}/episodes
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "target_id": "target-11111-22222",
  "episode_type": "conversation",
  "episode_content": "腰痛いらしい",
  "emotion_tag": "🤒 体調悪そう"
}
```

**Response** (201 Created):
```json
{
  "episode_id": "ep-67890",
  "user_id": "user-12345-67890",
  "target_id": "target-11111-22222",
  "episode_type": "conversation",
  "episode_content": "腰痛いらしい",
  "emotion_tag": "🤒 体調悪そう",
  "is_ng_topic": false,
  "henpou_status": "created",
  "created_at": "2026-05-10T10:00:00Z",
  "updated_at": "2026-05-10T10:00:00Z"
}
```

---

#### GET /users/{userId}/episodes/{episodeId}

**目的**: エピソード情報を取得

**Request**:
```http
GET /v1/users/{userId}/episodes/{episodeId}
Authorization: Bearer <JWT_TOKEN>
```

**Response** (200 OK):
```json
{
  "episode_id": "ep-12345",
  "user_id": "user-12345-67890",
  "target_id": "target-11111-22222",
  "episode_type": "received_gift",
  "episode_content": "叔父さんからお酒をもらった",
  "emotion_tag": "😊 元気そう",
  "is_ng_topic": false,
  "henpou_status": "stage2_scheduled",
  "stage1_timing": "2026-05-09T18:00:00Z",
  "stage2_timing": "2026-05-26T18:00:00Z",
  "stage2_timing_reason": "2-3weeks",
  "ai_extracted_data": {
    "keywords": ["お酒", "腰痛"],
    "sentiment": "positive"
  },
  "created_at": "2026-05-08T18:00:00Z",
  "updated_at": "2026-05-09T10:00:00Z"
}
```

---

### 3.4 Feedback Resource

#### POST /users/{userId}/feedback

**目的**: フィードバックを作成

**Request**:
```http
POST /v1/users/{userId}/feedback
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "kanpe_id": "kanpe-12345",
  "feedback_type": "swipe_right"
}
```

**Response** (201 Created):
```json
{
  "feedback_id": "feedback-12345",
  "user_id": "user-12345-67890",
  "kanpe_id": "kanpe-12345",
  "feedback_type": "swipe_right",
  "created_at": "2026-05-10T10:00:00Z"
}
```

---

#### GET /users/{userId}/feedback

**目的**: フィードバックリストを取得

**Request**:
```http
GET /v1/users/{userId}/feedback
Authorization: Bearer <JWT_TOKEN>
```

**Response** (200 OK):
```json
{
  "feedback": [
    {
      "feedback_id": "feedback-12345",
      "user_id": "user-12345-67890",
      "kanpe_id": "kanpe-12345",
      "feedback_type": "swipe_right",
      "created_at": "2026-05-10T10:00:00Z"
    }
  ]
}
```

---

## 4. RPC-style Actions

### 4.1 Submit Taredaka Action

#### POST /actions/submit-taredaka

**目的**: 撮れ高報告を実行

**Request**:
```http
POST /v1/actions/submit-taredaka
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "user_id": "user-12345-67890",
  "target_id": "target-11111-22222",
  "emotion_tag": "😊 元気そう",
  "voice_memo_base64": "data:audio/webm;base64,GkXfo59ChoEBQveBAULygQRC...",
  "image_base64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}
```

**Response** (200 OK):
```json
{
  "episode_id": "ep-12345",
  "ai_extracted_data": {
    "keywords": ["お酒", "腰痛"],
    "sentiment": "positive",
    "entities": [
      { "type": "PERSON", "text": "叔父さん" },
      { "type": "PRODUCT", "text": "お酒" }
    ]
  },
  "audio_transcript": "叔父さんからお酒をもらった",
  "image_analysis": {
    "detected_text": "ありがとうございます",
    "confidence": 0.95
  },
  "message": "撮れ高報告が完了しました"
}
```

**Error Response** (400 Bad Request):
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "emotion_tag is required",
    "details": {
      "field": "emotion_tag"
    }
  }
}
```

---

### 4.2 Generate Kanpe Action

#### POST /actions/generate-kanpe

**目的**: カンペ生成を実行

**Request**:
```http
POST /v1/actions/generate-kanpe
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "user_id": "user-12345-67890",
  "target_id": "target-11111-22222"
}
```

**Response** (200 OK):
```json
{
  "kanpe_id": "kanpe-12345",
  "user_id": "user-12345-67890",
  "target_id": "target-11111-22222",
  "ad_message": "とりあえずこれ読んどいてくださーい👍",
  "kanpe_text": "『腰の調子、どうっすか?』って聞いとけばOKっす。",
  "ng_topics": [],
  "gift_suggestion": {
    "asin": "B0XXXXXX",
    "product_name": "きき湯 ファインヒート",
    "product_url": "https://www.amazon.co.jp/dp/B0XXXXXX?tag=associate-id",
    "ad_message": "ついでにこれ買っとけばさらに安泰っす👍"
  },
  "created_at": "2026-05-10T10:00:00Z"
}
```

**Error Response** (404 Not Found):
```json
{
  "error": {
    "code": "NOT_FOUND",
    "message": "Target not found"
  }
}
```

---

### 4.3 Schedule Henpou Action

#### POST /actions/schedule-henpou

**目的**: 返報スケジュールを設定

**Request**:
```http
POST /v1/actions/schedule-henpou
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "user_id": "user-12345-67890",
  "episode_id": "ep-12345",
  "henpou_type": "received_gift"
}
```

**Response** (200 OK):
```json
{
  "episode_id": "ep-12345",
  "henpou_status": "stage1_scheduled",
  "stage1_timing": "2026-05-11T18:00:00Z",
  "stage2_timing": "2026-05-28T18:00:00Z",
  "stage2_timing_reason": "2-3weeks",
  "message": "返報スケジュールが設定されました"
}
```

**Error Response** (400 Bad Request):
```json
{
  "error": {
    "code": "INVALID_REQUEST",
    "message": "henpou_type must be 'received_gift' or 'gave_gift'",
    "details": {
      "field": "henpou_type"
    }
  }
}
```

---

## 5. AI Integration Endpoints（内部API）

### 5.1 Extract Knowledge from Image

#### POST /ai/extract-knowledge-from-image

**目的**: スクショから知識を抽出（内部API）

**Request**:
```http
POST /v1/ai/extract-knowledge-from-image
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "image_base64": "data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAA..."
}
```

**Response** (200 OK):
```json
{
  "detected_text": "ありがとうございます",
  "entities": [
    { "type": "PERSON", "text": "叔父さん" },
    { "type": "PRODUCT", "text": "お酒" }
  ],
  "keywords": ["お酒", "腰痛"],
  "sentiment": "positive",
  "confidence": 0.95
}
```

---

### 5.2 Extract Knowledge from Voice

#### POST /ai/extract-knowledge-from-voice

**目的**: 音声メモから知識を抽出（内部API）

**Request**:
```http
POST /v1/ai/extract-knowledge-from-voice
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "audio_base64": "data:audio/webm;base64,GkXfo59ChoEBQveBAULygQRC..."
}
```

**Response** (200 OK):
```json
{
  "transcript": "叔父さんからお酒をもらった",
  "keywords": ["お酒", "叔父さん"],
  "sentiment": "positive",
  "confidence": 0.92
}
```

---

### 5.3 Generate Icebreak Kanpe

#### POST /ai/generate-icebreak-kanpe

**目的**: アイスブレイクカンペを生成（内部API）

**Request**:
```http
POST /v1/ai/generate-icebreak-kanpe
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "target_name": "叔父さん",
  "episodes": [
    {
      "episode_content": "腰痛いらしい",
      "emotion_tag": "🤒 体調悪そう",
      "created_at": "2026-05-07T18:00:00Z"
    }
  ]
}
```

**Response** (200 OK):
```json
{
  "ad_message": "とりあえずこれ読んどいてくださーい👍",
  "kanpe_text": "『腰の調子、どうっすか?』って聞いとけばOKっす。"
}
```

---

### 5.4 Detect NG Topics

#### POST /ai/detect-ng-topics

**目的**: NG話題を検出（内部API）

**Request**:
```http
POST /v1/ai/detect-ng-topics
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "target_name": "叔父さん",
  "episodes": [
    {
      "episode_content": "離婚した",
      "is_ng_topic": true,
      "created_at": "2026-05-01T18:00:00Z"
    }
  ]
}
```

**Response** (200 OK):
```json
{
  "ng_topics": ["『奥さん元気?』", "『犬は?』"]
}
```

---

### 5.5 Select Product ASIN

#### POST /ai/select-product-asin

**目的**: 相手の状況に合わせたASINを選択（内部API）

**Request**:
```http
POST /v1/ai/select-product-asin
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "target_name": "叔父さん",
  "situation": "腰痛いらしい",
  "relationship": "親戚"
}
```

**Response** (200 OK):
```json
{
  "asin": "B0XXXXXX",
  "product_name": "きき湯 ファインヒート",
  "reason": "腰痛に効く入浴剤",
  "product_url": "https://www.amazon.co.jp/dp/B0XXXXXX?tag=associate-id"
}
```

---

## 6. Notification Endpoints（内部API）

### 6.1 Send Push Notification

#### POST /notifications/send-push

**目的**: FCM経由でプッシュ通知を送信（内部API）

**Request**:
```http
POST /v1/notifications/send-push
Authorization: Bearer <JWT_TOKEN>
Content-Type: application/json

{
  "user_id": "user-12345-67890",
  "title": "お疲れーっす！",
  "body": "今日の撮れ高ポチッとしといてくださーい",
  "data": {
    "notification_type": "taredaka_reminder",
    "target_id": "target-11111-22222"
  }
}
```

**Response** (200 OK):
```json
{
  "message": "Notification sent successfully",
  "fcm_message_id": "fcm-msg-12345"
}
```

---

## 7. エンドポイント一覧

### RESTful Resources (11 endpoints)

| Method | Endpoint | 目的 |
|---|---|---|
| GET | `/users/{userId}` | ユーザー情報取得 |
| PUT | `/users/{userId}` | ユーザー情報更新 |
| GET | `/users/{userId}/targets` | ターゲット一覧取得 |
| POST | `/users/{userId}/targets` | ターゲット作成 |
| GET | `/users/{userId}/targets/{targetId}` | ターゲット情報取得 |
| PUT | `/users/{userId}/targets/{targetId}` | ターゲット情報更新 |
| GET | `/users/{userId}/episodes` | エピソード一覧取得 |
| POST | `/users/{userId}/episodes` | エピソード作成 |
| GET | `/users/{userId}/episodes/{episodeId}` | エピソード情報取得 |
| POST | `/users/{userId}/feedback` | フィードバック作成 |
| GET | `/users/{userId}/feedback` | フィードバック一覧取得 |

### RPC-style Actions (3 endpoints)

| Method | Endpoint | 目的 |
|---|---|---|
| POST | `/actions/submit-taredaka` | 撮れ高報告実行 |
| POST | `/actions/generate-kanpe` | カンペ生成実行 |
| POST | `/actions/schedule-henpou` | 返報スケジュール設定 |

### AI Integration (5 endpoints - 内部API)

| Method | Endpoint | 目的 |
|---|---|---|
| POST | `/ai/extract-knowledge-from-image` | スクショから知識抽出 |
| POST | `/ai/extract-knowledge-from-voice` | 音声メモから知識抽出 |
| POST | `/ai/generate-icebreak-kanpe` | アイスブレイクカンペ生成 |
| POST | `/ai/detect-ng-topics` | NG話題検出 |
| POST | `/ai/select-product-asin` | 商品ASIN選択 |

### Notification (1 endpoint - 内部API)

| Method | Endpoint | 目的 |
|---|---|---|
| POST | `/notifications/send-push` | プッシュ通知送信 |

**Total**: 20 endpoints

---

## 8. 次のステップ

### 8.1 即座に実施（Units Generation前）

1. ✅ **API契約仕様書の作成**: 完了（本ドキュメント）
2. ⏳ **OpenAPI/Swagger仕様書の作成**（オプション）: YAML形式でのAPI仕様書
3. ⏳ **API契約テストの作成**: Postman/Insomnia等でのAPI契約テスト

### 8.2 Unit-Integration開発時

1. **API Gateway設定**: エンドポイント設定
2. **Lambda関数実装**: API契約に基づいた実装
3. **統合テスト**: API契約テスト

---

## 9. 承認

このAPI契約仕様書は、以下の要件を満たしています：

- ✅ 全エンドポイントのRequest/Responseスキーマ定義
- ✅ 統一されたエラーレスポンス構造
- ✅ エラーコード一覧
- ✅ 認証ヘッダー仕様
- ✅ RESTful Resources（11 endpoints）
- ✅ RPC-style Actions（3 endpoints）
- ✅ AI Integration（5 endpoints - 内部API）
- ✅ Notification（1 endpoint - 内部API）

**次のステップ**: OpenAPI/Swagger仕様書の作成（オプション）とAPI契約テストの作成を実施する。

