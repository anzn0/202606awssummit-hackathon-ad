# モックデータスキーマ定義

**作成日**: 2026-05-09  
**目的**: Units Generation前にモックデータ戦略を明確化し、フロントエンドとバックエンドのデータ構造の乖離を防ぐ

---

## 1. 概要

Unit-UIはバックエンドAPIの完成を待たず、モックデータ（JSON）を使用してUI・画面遷移を先行して完成させる。

### 1.1 モックデータの役割

- **フロントエンド開発の独立性**: バックエンドAPIの完成を待たずにUI開発を進める
- **データ構造の統一**: フロントエンドとバックエンドのデータ構造を事前に合意
- **切り替えの容易性**: モックからリアルAPIへの切り替えを環境変数で制御

---

## 2. JSON Schemaによるスキーマ定義

### 2.1 User（ユーザー）

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "User",
  "type": "object",
  "properties": {
    "user_id": {
      "type": "string",
      "description": "ユーザーID（UUID）",
      "example": "user-12345-67890"
    },
    "email": {
      "type": "string",
      "format": "email",
      "description": "メールアドレス",
      "example": "kenta@example.com"
    },
    "display_name": {
      "type": "string",
      "description": "表示名",
      "example": "健太"
    },
    "auth_provider": {
      "type": "string",
      "enum": ["google", "line"],
      "description": "認証プロバイダー",
      "example": "google"
    },
    "fcm_device_token": {
      "type": "string",
      "description": "FCMデバイストークン",
      "example": "fcm-token-abc123..."
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "作成日時（ISO 8601）",
      "example": "2026-05-01T10:00:00Z"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "更新日時（ISO 8601）",
      "example": "2026-05-09T10:00:00Z"
    }
  },
  "required": ["user_id", "email", "display_name", "auth_provider", "created_at"]
}
```

#### モックデータ例

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

---

### 2.2 Target（ターゲット人物）

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Target",
  "type": "object",
  "properties": {
    "target_id": {
      "type": "string",
      "description": "ターゲット人物ID（UUID）",
      "example": "target-11111-22222"
    },
    "user_id": {
      "type": "string",
      "description": "ユーザーID（外部キー）",
      "example": "user-12345-67890"
    },
    "name": {
      "type": "string",
      "description": "ターゲット人物の名前",
      "example": "叔父さん"
    },
    "relationship": {
      "type": "string",
      "description": "関係性",
      "example": "親戚"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "作成日時（ISO 8601）",
      "example": "2026-05-01T10:00:00Z"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "更新日時（ISO 8601）",
      "example": "2026-05-09T10:00:00Z"
    }
  },
  "required": ["target_id", "user_id", "name", "created_at"]
}
```

#### モックデータ例

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

### 2.3 Episode（エピソード）

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Episode",
  "type": "object",
  "properties": {
    "episode_id": {
      "type": "string",
      "description": "エピソードID（UUID）",
      "example": "ep-12345"
    },
    "user_id": {
      "type": "string",
      "description": "ユーザーID（外部キー）",
      "example": "user-12345-67890"
    },
    "target_id": {
      "type": "string",
      "description": "ターゲット人物ID（外部キー）",
      "example": "target-11111-22222"
    },
    "episode_type": {
      "type": "string",
      "enum": ["received_gift", "gave_gift", "conversation", "observation"],
      "description": "エピソードタイプ",
      "example": "received_gift"
    },
    "episode_content": {
      "type": "string",
      "description": "エピソード内容",
      "example": "叔父さんからお酒をもらった"
    },
    "emotion_tag": {
      "type": "string",
      "description": "感情タグ（絵文字）",
      "example": "😊 元気そう"
    },
    "is_ng_topic": {
      "type": "boolean",
      "description": "NG話題（地雷）かどうか",
      "example": false
    },
    "henpou_status": {
      "type": "string",
      "enum": ["created", "stage1_scheduled", "stage1_notified", "stage2_scheduled", "stage2_rescheduled", "stage2_notified", "completed", "cancelled"],
      "description": "返報性ハックの状態",
      "example": "stage2_scheduled"
    },
    "stage1_timing": {
      "type": "string",
      "format": "date-time",
      "description": "第一段階通知送信日時（ISO 8601）",
      "example": "2026-05-09T18:00:00Z"
    },
    "stage2_timing": {
      "type": "string",
      "format": "date-time",
      "description": "第二段階通知送信日時（ISO 8601）",
      "example": "2026-05-26T18:00:00Z"
    },
    "stage2_timing_reason": {
      "type": "string",
      "enum": ["2-3weeks", "next_meeting"],
      "description": "第二段階タイミング理由",
      "example": "2-3weeks"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "作成日時（ISO 8601）",
      "example": "2026-05-08T18:00:00Z"
    },
    "updated_at": {
      "type": "string",
      "format": "date-time",
      "description": "更新日時（ISO 8601）",
      "example": "2026-05-09T10:00:00Z"
    }
  },
  "required": ["episode_id", "user_id", "target_id", "episode_type", "episode_content", "created_at"]
}
```

#### モックデータ例

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
  "created_at": "2026-05-08T18:00:00Z",
  "updated_at": "2026-05-09T10:00:00Z"
}
```

---

### 2.4 Kanpe（カンペ）

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Kanpe",
  "type": "object",
  "properties": {
    "kanpe_id": {
      "type": "string",
      "description": "カンペID（UUID）",
      "example": "kanpe-12345"
    },
    "user_id": {
      "type": "string",
      "description": "ユーザーID（外部キー）",
      "example": "user-12345-67890"
    },
    "target_id": {
      "type": "string",
      "description": "ターゲット人物ID（外部キー）",
      "example": "target-11111-22222"
    },
    "ad_message": {
      "type": "string",
      "description": "ADのセリフ（ユーザーへの説明）",
      "example": "とりあえずこれ読んどいてくださーい👍"
    },
    "kanpe_text": {
      "type": "string",
      "description": "カンペテキスト（ユーザーが相手に言う言葉）",
      "example": "『腰の調子、どうっすか？』って聞いとけばOKっす。"
    },
    "ng_topics": {
      "type": "array",
      "items": {
        "type": "string"
      },
      "description": "NG話題リスト",
      "example": ["『奥さん元気？』", "『犬は？』"]
    },
    "gift_suggestion": {
      "type": "object",
      "properties": {
        "asin": {
          "type": "string",
          "description": "Amazon ASIN",
          "example": "B0XXXXXX"
        },
        "product_name": {
          "type": "string",
          "description": "商品名",
          "example": "きき湯 ファインヒート"
        },
        "product_url": {
          "type": "string",
          "format": "uri",
          "description": "商品詳細ページURL",
          "example": "https://www.amazon.co.jp/dp/B0XXXXXX?tag=associate-id"
        },
        "ad_message": {
          "type": "string",
          "description": "ADのセリフ（ギフト提案）",
          "example": "ついでにこれ買っとけばさらに安泰っす👍"
        }
      },
      "description": "ギフト提案（オプション）"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "作成日時（ISO 8601）",
      "example": "2026-05-09T10:00:00Z"
    }
  },
  "required": ["kanpe_id", "user_id", "target_id", "ad_message", "kanpe_text", "created_at"]
}
```

#### モックデータ例

```json
{
  "kanpe_id": "kanpe-12345",
  "user_id": "user-12345-67890",
  "target_id": "target-11111-22222",
  "ad_message": "とりあえずこれ読んどいてくださーい👍",
  "kanpe_text": "『腰の調子、どうっすか？』って聞いとけばOKっす。",
  "ng_topics": [],
  "gift_suggestion": {
    "asin": "B0XXXXXX",
    "product_name": "きき湯 ファインヒート",
    "product_url": "https://www.amazon.co.jp/dp/B0XXXXXX?tag=associate-id",
    "ad_message": "ついでにこれ買っとけばさらに安泰っす👍"
  },
  "created_at": "2026-05-09T10:00:00Z"
}
```

---

### 2.5 Feedback（フィードバック）

```json
{
  "$schema": "http://json-schema.org/draft-07/schema#",
  "title": "Feedback",
  "type": "object",
  "properties": {
    "feedback_id": {
      "type": "string",
      "description": "フィードバックID（UUID）",
      "example": "feedback-12345"
    },
    "user_id": {
      "type": "string",
      "description": "ユーザーID（外部キー）",
      "example": "user-12345-67890"
    },
    "kanpe_id": {
      "type": "string",
      "description": "カンペID（外部キー）",
      "example": "kanpe-12345"
    },
    "feedback_type": {
      "type": "string",
      "enum": ["swipe_right", "swipe_left", "acknowledged"],
      "description": "フィードバックタイプ",
      "example": "swipe_right"
    },
    "created_at": {
      "type": "string",
      "format": "date-time",
      "description": "作成日時（ISO 8601）",
      "example": "2026-05-09T10:00:00Z"
    }
  },
  "required": ["feedback_id", "user_id", "kanpe_id", "feedback_type", "created_at"]
}
```

#### モックデータ例

```json
{
  "feedback_id": "feedback-12345",
  "user_id": "user-12345-67890",
  "kanpe_id": "kanpe-12345",
  "feedback_type": "swipe_right",
  "created_at": "2026-05-09T10:00:00Z"
}
```

---

## 3. MockDataProviderの設計

### 3.1 責務

| 責務 | 説明 |
|---|---|
| **データ生成** | JSON Schemaに基づいたモックデータの生成 |
| **API呼び出しのモック** | バックエンドAPIの呼び出しをモック |
| **状態管理** | モックデータの状態管理（CRUD操作） |

### 3.2 ディレクトリ構造

```
Unit-UI/
├── src/
│   ├── mocks/
│   │   ├── MockDataProvider.ts          # モックデータプロバイダー
│   │   ├── data/
│   │   │   ├── users.json               # ユーザーモックデータ
│   │   │   ├── targets.json             # ターゲット人物モックデータ
│   │   │   ├── episodes.json            # エピソードモックデータ
│   │   │   ├── kanpes.json              # カンペモックデータ
│   │   │   └── feedbacks.json           # フィードバックモックデータ
│   │   └── schemas/
│   │       ├── user.schema.json         # ユーザースキーマ
│   │       ├── target.schema.json       # ターゲット人物スキーマ
│   │       ├── episode.schema.json      # エピソードスキーマ
│   │       ├── kanpe.schema.json        # カンペスキーマ
│   │       └── feedback.schema.json     # フィードバックスキーマ
│   └── api/
│       └── apiClient.ts                 # APIクライアント（モック/リアル切り替え）
```

### 3.3 MockDataProvider実装例（TypeScript）

```typescript
// src/mocks/MockDataProvider.ts

import usersData from './data/users.json';
import targetsData from './data/targets.json';
import episodesData from './data/episodes.json';
import kanpesData from './data/kanpes.json';
import feedbacksData from './data/feedbacks.json';

export class MockDataProvider {
  private users: any[];
  private targets: any[];
  private episodes: any[];
  private kanpes: any[];
  private feedbacks: any[];

  constructor() {
    this.users = [...usersData];
    this.targets = [...targetsData];
    this.episodes = [...episodesData];
    this.kanpes = [...kanpesData];
    this.feedbacks = [...feedbacksData];
  }

  // ユーザー取得
  async getUser(userId: string) {
    await this.simulateDelay();
    return this.users.find(u => u.user_id === userId);
  }

  // ターゲット人物一覧取得
  async getTargets(userId: string) {
    await this.simulateDelay();
    return this.targets.filter(t => t.user_id === userId);
  }

  // エピソード一覧取得
  async getEpisodes(userId: string, targetId?: string) {
    await this.simulateDelay();
    let filtered = this.episodes.filter(e => e.user_id === userId);
    if (targetId) {
      filtered = filtered.filter(e => e.target_id === targetId);
    }
    return filtered;
  }

  // エピソード作成
  async createEpisode(episode: any) {
    await this.simulateDelay();
    const newEpisode = {
      ...episode,
      episode_id: `ep-${Date.now()}`,
      created_at: new Date().toISOString(),
      updated_at: new Date().toISOString()
    };
    this.episodes.push(newEpisode);
    return newEpisode;
  }

  // カンペ取得
  async getKanpe(userId: string, targetId: string) {
    await this.simulateDelay();
    return this.kanpes.find(k => k.user_id === userId && k.target_id === targetId);
  }

  // フィードバック作成
  async createFeedback(feedback: any) {
    await this.simulateDelay();
    const newFeedback = {
      ...feedback,
      feedback_id: `feedback-${Date.now()}`,
      created_at: new Date().toISOString()
    };
    this.feedbacks.push(newFeedback);
    return newFeedback;
  }

  // 遅延シミュレーション（500ms〜1000ms）
  private simulateDelay() {
    const delay = Math.random() * 500 + 500;
    return new Promise(resolve => setTimeout(resolve, delay));
  }
}
```

---

## 4. モックからリアルAPIへの切り替え戦略

### 4.1 環境変数ベースの切り替え

```typescript
// src/api/apiClient.ts

import { MockDataProvider } from '../mocks/MockDataProvider';
import { RealApiClient } from './RealApiClient';

const USE_MOCK_DATA = process.env.REACT_APP_USE_MOCK_DATA === 'true';

export const apiClient = USE_MOCK_DATA 
  ? new MockDataProvider() 
  : new RealApiClient();
```

### 4.2 環境変数設定

```bash
# .env.development（開発環境：モックデータ使用）
REACT_APP_USE_MOCK_DATA=true

# .env.production（本番環境：リアルAPI使用）
REACT_APP_USE_MOCK_DATA=false
REACT_APP_API_BASE_URL=https://api.example.com
```

### 4.3 切り替え時の注意点

| 注意点 | 対策 |
|---|---|
| データ構造の乖離 | JSON Schemaで事前に合意 |
| API呼び出しの違い | 共通インターフェースを定義 |
| エラーハンドリングの違い | モックでもエラーケースをシミュレート |

---

## 5. モックデータファイル例

### 5.1 users.json

```json
[
  {
    "user_id": "user-12345-67890",
    "email": "kenta@example.com",
    "display_name": "健太",
    "auth_provider": "google",
    "fcm_device_token": "fcm-token-abc123...",
    "created_at": "2026-05-01T10:00:00Z",
    "updated_at": "2026-05-09T10:00:00Z"
  }
]
```

### 5.2 targets.json

```json
[
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
```

### 5.3 episodes.json

```json
[
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
  },
  {
    "episode_id": "ep-67890",
    "user_id": "user-12345-67890",
    "target_id": "target-33333-44444",
    "episode_type": "conversation",
    "episode_content": "腰痛いらしい",
    "emotion_tag": "🤒 体調悪そう",
    "is_ng_topic": false,
    "henpou_status": "created",
    "created_at": "2026-05-07T18:00:00Z",
    "updated_at": "2026-05-07T18:00:00Z"
  }
]
```

---

## 6. 次のステップ

### 6.1 即座に実施

1. ✅ **モックデータスキーマの定義**: 完了（本ドキュメント）
2. ⏳ **MockDataProviderの実装**: TypeScriptで実装
3. ⏳ **モックデータファイルの作成**: JSON形式でモックデータを作成

### 6.2 Unit-UI開発時

1. **MockDataProviderの統合**
2. **環境変数ベースの切り替え実装**
3. **モックデータの拡充**

---

## 7. 承認

このモックデータスキーマ定義は、以下の要件を満たしています：

- ✅ JSON Schemaによるスキーマ定義（User、Target、Episode、Kanpe、Feedback）
- ✅ MockDataProviderの設計（責務、ディレクトリ構造、実装例）
- ✅ モックからリアルAPIへの切り替え戦略（環境変数ベース）
- ✅ モックデータファイル例

**次のステップ**: MockDataProviderの実装とモックデータファイルの作成を実施する。
