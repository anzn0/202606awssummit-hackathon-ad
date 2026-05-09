# Component Methods - コミュ♥外chu

## Note on Business Rules
このドキュメントでは、各コンポーネントのメソッドシグネチャと高レベルの目的を定義します。詳細なビジネスルールは、後のFunctional Design（per-unit、CONSTRUCTION phase）で定義されます。

---

## Frontend Components

### TaredakaReportScreen

#### `onEmotionTagSelected(tagId: string): void`
**Purpose**: 感情タグが選択されたときの処理

**Input**: `tagId` - 選択された感情タグID（例: "sick", "happy", "worried"）  
**Output**: なし（状態更新）

---

#### `onVoiceMemoRecorded(audioBlob: Blob): void`
**Purpose**: 音声メモが録音されたときの処理

**Input**: `audioBlob` - 録音された音声データ  
**Output**: なし（バックエンドに送信）

---

#### `onImageUploaded(imageFile: File): void`
**Purpose**: スクショがアップロードされたときの処理

**Input**: `imageFile` - アップロードされた画像ファイル  
**Output**: なし（バックエンドに送信）

---

#### `submitTaredaka(): Promise<void>`
**Purpose**: 撮れ高データをバックエンドに送信

**Input**: なし（コンポーネント状態から取得）  
**Output**: `Promise<void>` - 送信完了

---

### KanpeDisplayScreen

#### `loadKanpe(targetId: string): Promise<Kanpe>`
**Purpose**: カンペをバックエンドから取得

**Input**: `targetId` - ターゲット人物ID  
**Output**: `Promise<Kanpe>` - カンペデータ

---

#### `displayIcebreakKanpe(kanpe: Kanpe): void`
**Purpose**: アイスブレイクカンペを表示

**Input**: `kanpe` - カンペデータ  
**Output**: なし（UI更新）

---

#### `displayNGTopicAlert(ngTopics: string[]): void`
**Purpose**: NG話題アラートを表示

**Input**: `ngTopics` - NG話題リスト  
**Output**: なし（UI更新）

---

#### `displayGiftSuggestion(gift: GiftSuggestion): void`
**Purpose**: ギフト提案を表示

**Input**: `gift` - ギフト提案データ  
**Output**: なし（UI更新）

---

### SwipeFeedbackScreen

#### `onSwipeRight(proposalId: string): void`
**Purpose**: 右スワイプ（興味あり）時の処理

**Input**: `proposalId` - 提案ID  
**Output**: なし（フィードバック送信）

---

#### `onSwipeLeft(proposalId: string): void`
**Purpose**: 左スワイプ（興味なし）時の処理

**Input**: `proposalId` - 提案ID  
**Output**: なし（フィードバック送信）

---

#### `submitFeedback(proposalId: string, reaction: 'interested' | 'not_interested'): Promise<void>`
**Purpose**: フィードバックをバックエンドに送信

**Input**: `proposalId`, `reaction`  
**Output**: `Promise<void>` - 送信完了

---

### YokiniBuyScreen

#### `onYokiniButtonClicked(asin: string): void`
**Purpose**: 「よきに」ボタンがクリックされたときの処理

**Input**: `asin` - Amazon商品ASIN  
**Output**: なし（Amazon Remote Cart URLに遷移）

---

#### `generateRemoteCartURL(asin: string): string`
**Purpose**: Amazon Remote Cart URLを生成

**Input**: `asin` - Amazon商品ASIN  
**Output**: `string` - Remote Cart URL

---

### LoginScreen

#### `onGoogleLoginClicked(): void`
**Purpose**: Googleログインボタンがクリックされたときの処理

**Input**: なし  
**Output**: なし（Cognito認証フロー開始）

---

#### `onLINELoginClicked(): void`
**Purpose**: LINEログインボタンがクリックされたときの処理

**Input**: なし  
**Output**: なし（Cognito認証フロー開始）

---

#### `handleAuthCallback(code: string): Promise<void>`
**Purpose**: 認証コールバック処理

**Input**: `code` - 認証コード  
**Output**: `Promise<void>` - 認証完了

---

## Backend Components

### UserResource

#### `getUser(userId: string): Promise<User>`
**Purpose**: ユーザー情報を取得

**Input**: `userId` - ユーザーID  
**Output**: `Promise<User>` - ユーザー情報

---

#### `updateUser(userId: string, userData: Partial<User>): Promise<User>`
**Purpose**: ユーザー情報を更新

**Input**: `userId`, `userData`  
**Output**: `Promise<User>` - 更新後のユーザー情報

---

### TargetResource

#### `listTargets(userId: string): Promise<Target[]>`
**Purpose**: ターゲット人物リストを取得

**Input**: `userId` - ユーザーID  
**Output**: `Promise<Target[]>` - ターゲット人物リスト

---

#### `createTarget(userId: string, targetData: CreateTargetInput): Promise<Target>`
**Purpose**: ターゲット人物を作成

**Input**: `userId`, `targetData`  
**Output**: `Promise<Target>` - 作成されたターゲット人物

---

#### `getTarget(userId: string, targetId: string): Promise<Target>`
**Purpose**: ターゲット人物情報を取得

**Input**: `userId`, `targetId`  
**Output**: `Promise<Target>` - ターゲット人物情報

---

#### `updateTarget(userId: string, targetId: string, targetData: Partial<Target>): Promise<Target>`
**Purpose**: ターゲット人物情報を更新

**Input**: `userId`, `targetId`, `targetData`  
**Output**: `Promise<Target>` - 更新後のターゲット人物情報

---

### EpisodeResource

#### `listEpisodes(userId: string, targetId: string): Promise<Episode[]>`
**Purpose**: エピソードリストを取得

**Input**: `userId`, `targetId`  
**Output**: `Promise<Episode[]>` - エピソードリスト

---

#### `createEpisode(userId: string, targetId: string, episodeData: CreateEpisodeInput): Promise<Episode>`
**Purpose**: エピソードを作成

**Input**: `userId`, `targetId`, `episodeData`  
**Output**: `Promise<Episode>` - 作成されたエピソード

---

#### `getEpisode(userId: string, targetId: string, episodeId: string): Promise<Episode>`
**Purpose**: エピソード情報を取得

**Input**: `userId`, `targetId`, `episodeId`  
**Output**: `Promise<Episode>` - エピソード情報

---

### FeedbackResource

#### `createFeedback(userId: string, feedbackData: CreateFeedbackInput): Promise<Feedback>`
**Purpose**: フィードバックを作成

**Input**: `userId`, `feedbackData`  
**Output**: `Promise<Feedback>` - 作成されたフィードバック

---

#### `listFeedback(userId: string): Promise<Feedback[]>`
**Purpose**: フィードバックリストを取得

**Input**: `userId`  
**Output**: `Promise<Feedback[]>` - フィードバックリスト

---

### SubmitTaredakaAction

#### `execute(input: SubmitTaredakaInput): Promise<SubmitTaredakaOutput>`
**Purpose**: 撮れ高報告を実行

**Input**: `SubmitTaredakaInput` - 感情タグ、音声メモ、スクショ  
**Output**: `Promise<SubmitTaredakaOutput>` - 処理結果

**High-level Flow**:
1. 入力データ検証
2. 音声メモをAI解析（KnowledgeExtractorService）
3. スクショをAI解析（KnowledgeExtractorService）
4. エピソードをDynamoDBに保存
5. 結果を返却

---

### GenerateKanpeAction

#### `execute(input: GenerateKanpeInput): Promise<GenerateKanpeOutput>`
**Purpose**: カンペ生成を実行

**Input**: `GenerateKanpeInput` - ユーザーID、ターゲットID  
**Output**: `Promise<GenerateKanpeOutput>` - カンペデータ

**High-level Flow**:
1. 過去エピソードをDynamoDBから取得
2. アイスブレイクカンペを生成（KanpeGeneratorService）
3. NG話題を検出（KanpeGeneratorService）
4. ギフト提案を生成（ProductSelectorService + RemoteCartURLGenerator）
5. カンペデータを返却

---

### ScheduleHenpouAction

#### `execute(input: ScheduleHenpouInput): Promise<ScheduleHenpouOutput>`
**Purpose**: 返報スケジュールを設定

**Input**: `ScheduleHenpouInput` - エピソードID、返報タイプ  
**Output**: `Promise<ScheduleHenpouOutput>` - スケジュール設定結果

**High-level Flow**:
1. 返報タイミングを計算（HenpouTimingCalculator）
2. EventBridge Schedulerにスケジュール登録
3. 結果を返却

---

## AI Integration Components

### BedrockClient

#### `invokeModel(prompt: string, modelId: string): Promise<string>`
**Purpose**: Bedrockモデルを呼び出し

**Input**: `prompt` - プロンプト、`modelId` - モデルID  
**Output**: `Promise<string>` - モデルレスポンス

---

#### `invokeModelWithImage(prompt: string, imageBase64: string, modelId: string): Promise<string>`
**Purpose**: Bedrockマルチモーダルモデルを呼び出し

**Input**: `prompt`, `imageBase64`, `modelId`  
**Output**: `Promise<string>` - モデルレスポンス

---

### KanpeGeneratorService

#### `generateIcebreakKanpe(episodes: Episode[]): Promise<string>`
**Purpose**: アイスブレイクカンペを生成

**Input**: `episodes` - 過去エピソードリスト  
**Output**: `Promise<string>` - カンペテキスト

---

#### `detectNGTopics(episodes: Episode[]): Promise<string[]>`
**Purpose**: NG話題を検出

**Input**: `episodes` - 過去エピソードリスト  
**Output**: `Promise<string[]>` - NG話題リスト

---

#### `applyADTone(text: string): Promise<string>`
**Purpose**: ADの口調（「〜っすね」）を適用

**Input**: `text` - 元のテキスト  
**Output**: `Promise<string>` - ADの口調に変換されたテキスト

---

### KnowledgeExtractorService

#### `extractFromImage(imageBase64: string): Promise<KnowledgeData>`
**Purpose**: スクショから知識を抽出

**Input**: `imageBase64` - スクショ画像（Base64）  
**Output**: `Promise<KnowledgeData>` - 抽出された知識

---

#### `extractFromVoice(audioBase64: string): Promise<KnowledgeData>`
**Purpose**: 音声メモから知識を抽出

**Input**: `audioBase64` - 音声データ（Base64）  
**Output**: `Promise<KnowledgeData>` - 抽出された知識

---

### ProductSelectorService

#### `selectASIN(targetProfile: Target, situation: string): Promise<string>`
**Purpose**: 相手の状況に合わせたASINを選択

**Input**: `targetProfile` - ターゲット人物プロファイル、`situation` - 状況  
**Output**: `Promise<string>` - 選択されたASIN

---

### HenpouTimingCalculator

#### `calculateImmediateThanksTime(episodeTimestamp: Date): Date`
**Purpose**: 即時お礼カンペのタイミングを計算（24時間以内）

**Input**: `episodeTimestamp` - エピソード発生時刻  
**Output**: `Date` - お礼カンペ送信時刻

---

#### `calculateDelayedGiftTime(episodeTimestamp: Date, nextMeetingDate?: Date): Date`
**Purpose**: 遅延ギフト提案のタイミングを計算（2〜3週間後、または次回予定直前）

**Input**: `episodeTimestamp`, `nextMeetingDate`（オプション）  
**Output**: `Date` - ギフト提案送信時刻

---

## Data Components

### DynamoDB Direct Access Methods

#### `putItem(tableName: string, item: any): Promise<void>`
**Purpose**: DynamoDBにアイテムを保存

**Input**: `tableName`, `item`  
**Output**: `Promise<void>` - 保存完了

---

#### `getItem(tableName: string, key: any): Promise<any>`
**Purpose**: DynamoDBからアイテムを取得

**Input**: `tableName`, `key`  
**Output**: `Promise<any>` - 取得されたアイテム

---

#### `updateItem(tableName: string, key: any, updates: any): Promise<any>`
**Purpose**: DynamoDBのアイテムを更新

**Input**: `tableName`, `key`, `updates`  
**Output**: `Promise<any>` - 更新後のアイテム

---

#### `query(tableName: string, keyCondition: any): Promise<any[]>`
**Purpose**: DynamoDBをクエリ

**Input**: `tableName`, `keyCondition`  
**Output**: `Promise<any[]>` - クエリ結果

---

## Authentication Components

### CognitoAuthorizer (API Gateway)

#### `authorize(token: string): AuthorizationResult`
**Purpose**: JWTトークンを検証

**Input**: `token` - JWTトークン  
**Output**: `AuthorizationResult` - 認証結果（許可/拒否）

---

### UserContextExtractor (Lambda)

#### `extractUserContext(event: APIGatewayEvent): UserContext`
**Purpose**: API Gatewayイベントからユーザーコンテキストを抽出

**Input**: `event` - API Gatewayイベント  
**Output**: `UserContext` - ユーザーコンテキスト（userId, cognitoSub等）

---

## Notification Components

### ImmediateNotificationHandler

#### `handleEvent(event: EventBridgeEvent): Promise<void>`
**Purpose**: 即時通知イベントを処理

**Input**: `event` - EventBridgeイベント  
**Output**: `Promise<void>` - 処理完了

---

### DelayedNotificationScheduler

#### `scheduleNotification(userId: string, targetId: string, notificationType: string, scheduledTime: Date): Promise<string>`
**Purpose**: 遅延通知をスケジュール

**Input**: `userId`, `targetId`, `notificationType`, `scheduledTime`  
**Output**: `Promise<string>` - スケジュールID

---

### FCMNotificationSender

#### `sendNotification(userId: string, title: string, body: string, data: any): Promise<void>`
**Purpose**: FCM経由でプッシュ通知を送信

**Input**: `userId`, `title`, `body`, `data`  
**Output**: `Promise<void>` - 送信完了

---

## URL Generation Components

### RemoteCartURLGenerator (Backend)

#### `generateURL(asin: string, quantity: number = 1): string`
**Purpose**: Amazon Remote Cart URLを生成

**Input**: `asin` - Amazon商品ASIN、`quantity` - 数量（デフォルト1）  
**Output**: `string` - Remote Cart URL

**URL Format**: `https://www.amazon.co.jp/gp/aws/cart/add.html?ASIN.1={asin}&Quantity.1={quantity}`

---

## Type Definitions (High-level)

### User
```typescript
interface User {
  userId: string;
  cognitoSub: string;
  name: string;
  email: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Target
```typescript
interface Target {
  targetId: string;
  userId: string;
  name: string;
  relationship: string;
  profile: string;
  createdAt: Date;
  updatedAt: Date;
}
```

### Episode
```typescript
interface Episode {
  episodeId: string;
  userId: string;
  targetId: string;
  emotionTag?: string;
  voiceMemoText?: string;
  imageAnalysisResult?: string;
  timestamp: Date;
  createdAt: Date;
}
```

### Feedback
```typescript
interface Feedback {
  feedbackId: string;
  userId: string;
  proposalId: string;
  reaction: 'interested' | 'not_interested';
  createdAt: Date;
}
```

### Kanpe
```typescript
interface Kanpe {
  kanpeId: string;
  userId: string;
  targetId: string;
  icebreakText: string;
  ngTopics: string[];
  giftSuggestion?: GiftSuggestion;
  createdAt: Date;
}
```

### GiftSuggestion
```typescript
interface GiftSuggestion {
  asin: string;
  productName: string;
  reason: string;
  remoteCartURL: string;
}
```

---

## Method Summary

- **Frontend**: 20+ メソッド
- **Backend**: 15+ メソッド
- **AI Integration**: 10+ メソッド
- **Data**: 4基本CRUD操作
- **Authentication**: 2メソッド
- **Notification**: 3メソッド
- **URL Generation**: 1メソッド

**Total**: 55+ メソッド

