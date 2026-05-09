# DynamoDBデータモデル詳細設計

**作成日**: 2026-05-10  
**目的**: Units Generation前にDynamoDBデータモデルの詳細を明確化し、Unit-DB開発時のテーブル設計の大幅な変更を防ぐ

---

## 1. 概要

本ドキュメントは、コミュ♥外chuのDynamoDBデータモデルの詳細設計を定義します。

### 1.1 設計原則

- **Repository Pattern採用**: 各エンティティごとにRepositoryクラスを作成
- **軽量な実装**: 過度な抽象化を避け、シンプルなClassとして実装
- **モックモード搭載**: 環境変数で切り替え可能なMockRepository
- **非構造化データ対応**: DynamoDBのMap型やList型を柔軟に扱う

---

## 2. テーブル設計

### 2.1 UsersTable

#### テーブル概要
- **テーブル名**: `Users`
- **目的**: ユーザーの基本設定、認証情報の管理
- **パーティションキー**: `user_id` (String)
- **ソートキー**: なし

#### 属性定義

| 属性名 | 型 | 説明 | 必須 | 例 |
|---|---|---|---|---|
| `user_id` | String | ユーザーID（UUID） | ✅ | `user-12345-67890` |
| `email` | String | メールアドレス | ✅ | `kenta@example.com` |
| `display_name` | String | 表示名 | ✅ | `健太` |
| `auth_provider` | String | 認証プロバイダー（google/line） | ✅ | `google` |
| `fcm_device_token` | String | FCMデバイストークン | ❌ | `fcm-token-abc123...` |
| `created_at` | String | 作成日時（ISO 8601） | ✅ | `2026-05-01T10:00:00Z` |
| `updated_at` | String | 更新日時（ISO 8601） | ✅ | `2026-05-09T10:00:00Z` |

#### GSI（Global Secondary Index）

**GSI-1: EmailIndex**
- **目的**: メールアドレスでユーザーを検索
- **パーティションキー**: `email` (String)
- **ソートキー**: なし
- **射影**: ALL

#### アクセスパターン

| アクセスパターン | 使用するキー | 説明 |
|---|---|---|
| ユーザーIDでユーザー取得 | PK: `user_id` | 基本的なユーザー情報取得 |
| メールアドレスでユーザー検索 | GSI-1: `email` | ログイン時のユーザー検索 |

---

### 2.2 TargetsTable

#### テーブル概要
- **テーブル名**: `Targets`
- **目的**: ターゲット人物（叔父さん、上司等）の属性、関係性の管理
- **パーティションキー**: `user_id` (String)
- **ソートキー**: `target_id` (String)

#### 属性定義

| 属性名 | 型 | 説明 | 必須 | 例 |
|---|---|---|---|---|
| `user_id` | String | ユーザーID（外部キー） | ✅ | `user-12345-67890` |
| `target_id` | String | ターゲット人物ID（UUID） | ✅ | `target-11111-22222` |
| `name` | String | ターゲット人物の名前 | ✅ | `叔父さん` |
| `relationship` | String | 関係性 | ❌ | `親戚` |
| `created_at` | String | 作成日時（ISO 8601） | ✅ | `2026-05-01T10:00:00Z` |
| `updated_at` | String | 更新日時（ISO 8601） | ✅ | `2026-05-09T10:00:00Z` |

#### GSI（Global Secondary Index）

なし（パーティションキー + ソートキーで十分）

#### アクセスパターン

| アクセスパターン | 使用するキー | 説明 |
|---|---|---|
| ユーザーIDでターゲット一覧取得 | PK: `user_id` | ユーザーのすべてのターゲット人物を取得 |
| ユーザーID + ターゲットIDでターゲット取得 | PK: `user_id`, SK: `target_id` | 特定のターゲット人物を取得 |

---

### 2.3 EpisodesTable

#### テーブル概要
- **テーブル名**: `Episodes`
- **目的**: 感情タグ、音声メモ、AI抽出結果（撮れ高データ）の管理
- **パーティションキー**: `user_id` (String)
- **ソートキー**: `episode_id` (String)

#### 属性定義

| 属性名 | 型 | 説明 | 必須 | 例 |
|---|---|---|---|---|
| `user_id` | String | ユーザーID（外部キー） | ✅ | `user-12345-67890` |
| `episode_id` | String | エピソードID（UUID） | ✅ | `ep-12345` |
| `target_id` | String | ターゲット人物ID（外部キー） | ✅ | `target-11111-22222` |
| `episode_type` | String | エピソードタイプ（received_gift/gave_gift/conversation/observation） | ✅ | `received_gift` |
| `episode_content` | String | エピソード内容 | ✅ | `叔父さんからお酒をもらった` |
| `emotion_tag` | String | 感情タグ（絵文字） | ❌ | `😊 元気そう` |
| `is_ng_topic` | Boolean | NG話題（地雷）かどうか | ❌ | `false` |
| `henpou_status` | String | 返報性ハックの状態（created/stage1_scheduled/stage1_notified/stage2_scheduled/stage2_rescheduled/stage2_notified/completed/cancelled） | ❌ | `stage2_scheduled` |
| `stage1_timing` | String | 第一段階通知送信日時（ISO 8601） | ❌ | `2026-05-09T18:00:00Z` |
| `stage2_timing` | String | 第二段階通知送信日時（ISO 8601） | ❌ | `2026-05-26T18:00:00Z` |
| `stage2_timing_reason` | String | 第二段階タイミング理由（2-3weeks/next_meeting） | ❌ | `2-3weeks` |
| `ai_extracted_data` | Map | AI抽出結果（非構造化データ） | ❌ | `{"keywords": ["お酒", "腰痛"], "sentiment": "positive"}` |
| `audio_transcript` | String | 音声テキスト化結果 | ❌ | `腰痛いらしい` |
| `image_analysis` | Map | スクショ解析結果（非構造化データ） | ❌ | `{"detected_text": "...", "entities": [...]}` |
| `created_at` | String | 作成日時（ISO 8601） | ✅ | `2026-05-08T18:00:00Z` |
| `updated_at` | String | 更新日時（ISO 8601） | ✅ | `2026-05-09T10:00:00Z` |

#### GSI（Global Secondary Index）

**GSI-1: TargetEpisodesIndex**
- **目的**: ターゲット人物ごとのエピソード取得
- **パーティションキー**: `target_id` (String)
- **ソートキー**: `created_at` (String)
- **射影**: ALL

**GSI-2: HenpouStatusIndex**
- **目的**: 返報ステータスによるエピソード検索
- **パーティションキー**: `user_id` (String)
- **ソートキー**: `henpou_status` (String)
- **射影**: ALL

#### アクセスパターン

| アクセスパターン | 使用するキー | 説明 |
|---|---|---|
| ユーザーIDでエピソード一覧取得 | PK: `user_id` | ユーザーのすべてのエピソードを取得 |
| ユーザーID + エピソードIDでエピソード取得 | PK: `user_id`, SK: `episode_id` | 特定のエピソードを取得 |
| ターゲットIDでエピソード一覧取得 | GSI-1: `target_id` | 特定のターゲット人物のエピソードを取得 |
| 返報ステータスでエピソード検索 | GSI-2: `user_id`, SK: `henpou_status` | 返報ステータスによるエピソード検索 |

---

### 2.4 FeedbackTable

#### テーブル概要
- **テーブル名**: `Feedback`
- **目的**: ギフトへの「よきに（スワイプ）」データの管理
- **パーティションキー**: `user_id` (String)
- **ソートキー**: `feedback_id` (String)

#### 属性定義

| 属性名 | 型 | 説明 | 必須 | 例 |
|---|---|---|---|---|
| `user_id` | String | ユーザーID（外部キー） | ✅ | `user-12345-67890` |
| `feedback_id` | String | フィードバックID（UUID） | ✅ | `feedback-12345` |
| `kanpe_id` | String | カンペID（外部キー） | ✅ | `kanpe-12345` |
| `feedback_type` | String | フィードバックタイプ（swipe_right/swipe_left/acknowledged） | ✅ | `swipe_right` |
| `created_at` | String | 作成日時（ISO 8601） | ✅ | `2026-05-09T10:00:00Z` |

#### GSI（Global Secondary Index）

**GSI-1: KanpeFeedbackIndex**
- **目的**: カンペIDでフィードバック取得
- **パーティションキー**: `kanpe_id` (String)
- **ソートキー**: `created_at` (String)
- **射影**: ALL

#### アクセスパターン

| アクセスパターン | 使用するキー | 説明 |
|---|---|---|
| ユーザーIDでフィードバック一覧取得 | PK: `user_id` | ユーザーのすべてのフィードバックを取得 |
| カンペIDでフィードバック取得 | GSI-1: `kanpe_id` | 特定のカンペに対するフィードバックを取得 |

---

## 3. Repository Pattern設計

### 3.1 Repository構造

```
/backend/src/repositories/
├── UserRepository.ts          # ユーザーの基本設定、認証情報の管理
├── TargetRepository.ts        # ターゲット（叔父さん、上司等）の属性、関係性の管理
├── EpisodeRepository.ts       # 感情タグ、音声メモ、AI抽出結果（撮れ高データ）の管理
├── FeedbackRepository.ts      # ギフトへの「よきに（スワイプ）」データの管理
└── MockRepository.ts          # モックモード用のRepository（環境変数で切り替え）
```

### 3.2 UserRepository実装例（TypeScript）

```typescript
// /backend/src/repositories/UserRepository.ts

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand, PutCommand, UpdateCommand, QueryCommand } from "@aws-sdk/lib-dynamodb";

export interface User {
  user_id: string;
  email: string;
  display_name: string;
  auth_provider: "google" | "line";
  fcm_device_token?: string;
  created_at: string;
  updated_at: string;
}

export class UserRepository {
  private docClient: DynamoDBDocumentClient;
  private tableName: string;

  constructor(tableName: string = "Users") {
    const client = new DynamoDBClient({});
    this.docClient = DynamoDBDocumentClient.from(client);
    this.tableName = tableName;
  }

  /**
   * ユーザーIDでユーザー取得
   */
  async getUserById(userId: string): Promise<User | null> {
    const command = new GetCommand({
      TableName: this.tableName,
      Key: { user_id: userId }
    });

    const response = await this.docClient.send(command);
    return response.Item as User || null;
  }

  /**
   * メールアドレスでユーザー検索（GSI-1使用）
   */
  async getUserByEmail(email: string): Promise<User | null> {
    const command = new QueryCommand({
      TableName: this.tableName,
      IndexName: "EmailIndex",
      KeyConditionExpression: "email = :email",
      ExpressionAttributeValues: {
        ":email": email
      }
    });

    const response = await this.docClient.send(command);
    return response.Items?.[0] as User || null;
  }

  /**
   * ユーザー作成
   */
  async createUser(user: Omit<User, "created_at" | "updated_at">): Promise<User> {
    const now = new Date().toISOString();
    const newUser: User = {
      ...user,
      created_at: now,
      updated_at: now
    };

    const command = new PutCommand({
      TableName: this.tableName,
      Item: newUser
    });

    await this.docClient.send(command);
    return newUser;
  }

  /**
   * ユーザー更新
   */
  async updateUser(userId: string, updates: Partial<Omit<User, "user_id" | "created_at">>): Promise<User> {
    const now = new Date().toISOString();
    const updateExpression: string[] = [];
    const expressionAttributeNames: Record<string, string> = {};
    const expressionAttributeValues: Record<string, any> = {};

    Object.entries(updates).forEach(([key, value]) => {
      updateExpression.push(`#${key} = :${key}`);
      expressionAttributeNames[`#${key}`] = key;
      expressionAttributeValues[`:${key}`] = value;
    });

    updateExpression.push("#updated_at = :updated_at");
    expressionAttributeNames["#updated_at"] = "updated_at";
    expressionAttributeValues[":updated_at"] = now;

    const command = new UpdateCommand({
      TableName: this.tableName,
      Key: { user_id: userId },
      UpdateExpression: `SET ${updateExpression.join(", ")}`,
      ExpressionAttributeNames: expressionAttributeNames,
      ExpressionAttributeValues: expressionAttributeValues,
      ReturnValues: "ALL_NEW"
    });

    const response = await this.docClient.send(command);
    return response.Attributes as User;
  }
}
```

### 3.3 EpisodeRepository実装例（非構造化データ対応）

```typescript
// /backend/src/repositories/EpisodeRepository.ts

import { DynamoDBClient } from "@aws-sdk/client-dynamodb";
import { DynamoDBDocumentClient, GetCommand, PutCommand, QueryCommand } from "@aws-sdk/lib-dynamodb";

export interface Episode {
  user_id: string;
  episode_id: string;
  target_id: string;
  episode_type: "received_gift" | "gave_gift" | "conversation" | "observation";
  episode_content: string;
  emotion_tag?: string;
  is_ng_topic?: boolean;
  henpou_status?: string;
  stage1_timing?: string;
  stage2_timing?: string;
  stage2_timing_reason?: "2-3weeks" | "next_meeting";
  ai_extracted_data?: Record<string, any>; // 非構造化データ（Map型）
  audio_transcript?: string;
  image_analysis?: Record<string, any>; // 非構造化データ（Map型）
  created_at: string;
  updated_at: string;
}

export class EpisodeRepository {
  private docClient: DynamoDBDocumentClient;
  private tableName: string;

  constructor(tableName: string = "Episodes") {
    const client = new DynamoDBClient({});
    this.docClient = DynamoDBDocumentClient.from(client);
    this.tableName = tableName;
  }

  /**
   * ユーザーIDでエピソード一覧取得
   */
  async getEpisodesByUserId(userId: string): Promise<Episode[]> {
    const command = new QueryCommand({
      TableName: this.tableName,
      KeyConditionExpression: "user_id = :user_id",
      ExpressionAttributeValues: {
        ":user_id": userId
      }
    });

    const response = await this.docClient.send(command);
    return response.Items as Episode[] || [];
  }

  /**
   * ターゲットIDでエピソード一覧取得（GSI-1使用）
   */
  async getEpisodesByTargetId(targetId: string): Promise<Episode[]> {
    const command = new QueryCommand({
      TableName: this.tableName,
      IndexName: "TargetEpisodesIndex",
      KeyConditionExpression: "target_id = :target_id",
      ExpressionAttributeValues: {
        ":target_id": targetId
      }
    });

    const response = await this.docClient.send(command);
    return response.Items as Episode[] || [];
  }

  /**
   * 返報ステータスでエピソード検索（GSI-2使用）
   */
  async getEpisodesByHenpouStatus(userId: string, henpouStatus: string): Promise<Episode[]> {
    const command = new QueryCommand({
      TableName: this.tableName,
      IndexName: "HenpouStatusIndex",
      KeyConditionExpression: "user_id = :user_id AND henpou_status = :henpou_status",
      ExpressionAttributeValues: {
        ":user_id": userId,
        ":henpou_status": henpouStatus
      }
    });

    const response = await this.docClient.send(command);
    return response.Items as Episode[] || [];
  }

  /**
   * エピソード作成（非構造化データ対応）
   */
  async createEpisode(episode: Omit<Episode, "created_at" | "updated_at">): Promise<Episode> {
    const now = new Date().toISOString();
    const newEpisode: Episode = {
      ...episode,
      created_at: now,
      updated_at: now
    };

    const command = new PutCommand({
      TableName: this.tableName,
      Item: newEpisode
    });

    await this.docClient.send(command);
    return newEpisode;
  }
}
```

### 3.4 MockRepository実装例（モックモード）

```typescript
// /backend/src/repositories/MockRepository.ts

import { User, UserRepository } from "./UserRepository";
import { Episode, EpisodeRepository } from "./EpisodeRepository";

export class MockUserRepository extends UserRepository {
  private mockUsers: User[] = [
    {
      user_id: "user-12345-67890",
      email: "kenta@example.com",
      display_name: "健太",
      auth_provider: "google",
      fcm_device_token: "fcm-token-abc123...",
      created_at: "2026-05-01T10:00:00Z",
      updated_at: "2026-05-09T10:00:00Z"
    }
  ];

  async getUserById(userId: string): Promise<User | null> {
    await this.simulateDelay();
    return this.mockUsers.find(u => u.user_id === userId) || null;
  }

  async getUserByEmail(email: string): Promise<User | null> {
    await this.simulateDelay();
    return this.mockUsers.find(u => u.email === email) || null;
  }

  async createUser(user: Omit<User, "created_at" | "updated_at">): Promise<User> {
    await this.simulateDelay();
    const now = new Date().toISOString();
    const newUser: User = {
      ...user,
      created_at: now,
      updated_at: now
    };
    this.mockUsers.push(newUser);
    return newUser;
  }

  private simulateDelay() {
    const delay = Math.random() * 500 + 500;
    return new Promise(resolve => setTimeout(resolve, delay));
  }
}
```

### 3.5 環境変数による切り替え

```typescript
// /backend/src/repositories/index.ts

import { UserRepository } from "./UserRepository";
import { MockUserRepository } from "./MockRepository";

const IS_MOCK = process.env.IS_MOCK === "true";

export const userRepository = IS_MOCK 
  ? new MockUserRepository() 
  : new UserRepository();
```

---

## 4. データ整合性戦略

### 4.1 DynamoDB Transactionsの使用箇所

| 操作 | Transactionsの使用 | 理由 |
|---|---|---|
| エピソード作成 + 返報スケジュール登録 | ✅ 使用 | エピソード作成と返報スケジュール登録は原子性が必要 |
| ユーザー作成 | ❌ 不要 | 単一テーブルへの書き込みのみ |
| フィードバック作成 | ❌ 不要 | 単一テーブルへの書き込みのみ |

### 4.2 楽観的ロック vs 悲観的ロック

| 操作 | ロック戦略 | 理由 |
|---|---|---|
| エピソード更新 | 楽観的ロック | 競合が少ない、パフォーマンス重視 |
| ユーザー更新 | 楽観的ロック | 競合が少ない、パフォーマンス重視 |
| 返報スケジュール更新 | 楽観的ロック | 競合が少ない、パフォーマンス重視 |

---

## 5. 非構造化データの扱い

### 5.1 EpisodeRepositoryにおける非構造化データ

EpisodeRepositoryでは、以下の非構造化データを柔軟に扱えるようにします：

- **ai_extracted_data**: AI抽出結果（Map型）
  - 例: `{"keywords": ["お酒", "腰痛"], "sentiment": "positive"}`
- **image_analysis**: スクショ解析結果（Map型）
  - 例: `{"detected_text": "...", "entities": [...]}`

### 5.2 非構造化データの保存例

```typescript
const episode = await episodeRepository.createEpisode({
  user_id: "user-12345-67890",
  episode_id: "ep-12345",
  target_id: "target-11111-22222",
  episode_type: "received_gift",
  episode_content: "叔父さんからお酒をもらった",
  ai_extracted_data: {
    keywords: ["お酒", "腰痛"],
    sentiment: "positive",
    entities: [
      { type: "PERSON", text: "叔父さん" },
      { type: "PRODUCT", text: "お酒" }
    ]
  },
  image_analysis: {
    detected_text: "ありがとうございます",
    confidence: 0.95
  }
});
```

---

## 6. 次のステップ

### 6.1 即座に実施（Units Generation前）

1. ✅ **DynamoDBデータモデル詳細設計**: 完了（本ドキュメント）
2. ⏳ **Repository実装**: TypeScriptでRepository実装
3. ⏳ **MockRepository実装**: モックモード用のRepository実装

### 6.2 Unit-DB開発時

1. **DynamoDBテーブル作成**: AWS CDKまたはTerraformでテーブル作成
2. **Repository統合**: Lambda関数でRepositoryを使用
3. **統合テスト**: DynamoDB統合テスト

---

## 7. 承認

このDynamoDBデータモデル詳細設計は、以下の要件を満たしています：

- ✅ テーブル設計（UsersTable、TargetsTable、EpisodesTable、FeedbackTable）
- ✅ パーティションキー、ソートキー、GSI設計
- ✅ アクセスパターン定義
- ✅ Repository Pattern設計（軽量な実装、モックモード搭載、非構造化データ対応）
- ✅ データ整合性戦略（DynamoDB Transactions、楽観的ロック）

**次のステップ**: Repository実装とMockRepository実装を実施する。

