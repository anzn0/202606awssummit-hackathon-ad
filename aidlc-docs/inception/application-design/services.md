# Services - コミュ♥外chu

## Service Layer Strategy

**採用アプローチ**: ハイブリッド（単純なフローは薄く、複雑なフローは厚く）

- **薄いサービス層**: 単純なCRUD操作は各Lambdaが直接コンポーネントを呼び出し
- **厚いサービス層**: 複雑なオーケストレーション（カンペ生成、返報スケジュール等）は専用サービスが管理

---

## Thin Service Layer (Simple Flows)

### UserService
**Purpose**: ユーザー管理の薄いサービス層

**Responsibilities**:
- ユーザー情報CRUD操作のシンプルなラッピング
- DynamoDBへの直接アクセス

**Methods**:
- `getUser(userId: string): Promise<User>`
- `updateUser(userId: string, userData: Partial<User>): Promise<User>`

**Orchestration**: なし（直接DynamoDB操作）

---

### TargetService
**Purpose**: ターゲット人物管理の薄いサービス層

**Responsibilities**:
- ターゲット人物情報CRUD操作のシンプルなラッピング
- DynamoDBへの直接アクセス

**Methods**:
- `listTargets(userId: string): Promise<Target[]>`
- `createTarget(userId: string, targetData: CreateTargetInput): Promise<Target>`
- `getTarget(userId: string, targetId: string): Promise<Target>`
- `updateTarget(userId: string, targetId: string, targetData: Partial<Target>): Promise<Target>`

**Orchestration**: なし（直接DynamoDB操作）

---

### FeedbackService
**Purpose**: フィードバック管理の薄いサービス層

**Responsibilities**:
- フィードバックCRUD操作のシンプルなラッピング
- DynamoDBへの直接アクセス

**Methods**:
- `createFeedback(userId: string, feedbackData: CreateFeedbackInput): Promise<Feedback>`
- `listFeedback(userId: string): Promise<Feedback[]>`

**Orchestration**: なし（直接DynamoDB操作）

---

## Thick Service Layer (Complex Flows)

### TaredakaOrchestrationService
**Purpose**: 撮れ高報告の複雑なオーケストレーション

**Responsibilities**:
- 感情タグ、音声メモ、スクショの統合処理
- AI解析のトリガー
- エピソード保存
- エラーハンドリング（段階的戦略）

**Methods**:
- `submitTaredaka(input: SubmitTaredakaInput): Promise<SubmitTaredakaOutput>`

**Orchestration Flow**:
```
1. 入力データ検証
2. 音声メモがある場合:
   - KnowledgeExtractorService.extractFromVoice()
   - エラー時: リトライ（重要操作ではないため楽観的）
3. スクショがある場合:
   - KnowledgeExtractorService.extractFromImage()
   - エラー時: リトライ（重要操作ではないため楽観的）
4. エピソードをDynamoDBに保存
   - putItem(EpisodesTable)
5. 結果を返却
```

**Dependencies**:
- KnowledgeExtractorService (AI Integration)
- DynamoDB (Data)

**Error Handling**: 楽観的（AI解析失敗時はログ記録のみ、エピソード保存は継続）

---

### KanpeOrchestrationService
**Purpose**: カンペ生成の複雑なオーケストレーション

**Responsibilities**:
- 過去エピソード取得
- アイスブレイクカンペ生成
- NG話題検出
- ギフト提案生成（オプション）
- エラーハンドリング（段階的戦略）

**Methods**:
- `generateKanpe(input: GenerateKanpeInput): Promise<GenerateKanpeOutput>`

**Orchestration Flow**:
```
1. 過去エピソードをDynamoDBから取得
   - query(EpisodesTable, userId + targetId)
2. アイスブレイクカンペを生成
   - KanpeGeneratorService.generateIcebreakKanpe(episodes)
   - エラー時: リトライ（重要操作のため防御的）
3. NG話題を検出
   - KanpeGeneratorService.detectNGTopics(episodes)
   - エラー時: リトライ（重要操作のため防御的）
4. ギフト提案を生成（オプション）
   - ProductSelectorService.selectASIN(targetProfile, situation)
   - RemoteCartURLGenerator.generateURL(asin)
   - エラー時: スキップ（オプション機能のため楽観的）
5. カンペデータを返却
```

**Dependencies**:
- KanpeGeneratorService (AI Integration)
- ProductSelectorService (AI Integration)
- RemoteCartURLGenerator (Backend)
- DynamoDB (Data)

**Error Handling**: 段階的（カンペ生成は防御的、ギフト提案は楽観的）

---

### HenpouOrchestrationService
**Purpose**: 返報スケジュールの複雑なオーケストレーション

**Responsibilities**:
- 返報タイミング計算
- EventBridge Schedulerへのスケジュール登録
- 即時お礼カンペと遅延ギフト提案の二段階ロジック
- エラーハンドリング（段階的戦略）

**Methods**:
- `scheduleHenpou(input: ScheduleHenpouInput): Promise<ScheduleHenpouOutput>`

**Orchestration Flow**:
```
1. エピソード情報をDynamoDBから取得
   - getItem(EpisodesTable, episodeId)
2. 即時お礼カンペのタイミングを計算
   - HenpouTimingCalculator.calculateImmediateThanksTime(episodeTimestamp)
3. 即時お礼カンペをEventBridge Schedulerに登録
   - DelayedNotificationScheduler.scheduleNotification(...)
   - エラー時: リトライ（重要操作のため防御的）
4. 遅延ギフト提案のタイミングを計算
   - HenpouTimingCalculator.calculateDelayedGiftTime(episodeTimestamp, nextMeetingDate)
5. 遅延ギフト提案をEventBridge Schedulerに登録
   - DelayedNotificationScheduler.scheduleNotification(...)
   - エラー時: リトライ（重要操作のため防御的）
6. スケジュール設定結果を返却
```

**Dependencies**:
- HenpouTimingCalculator (AI Integration)
- DelayedNotificationScheduler (Notification)
- DynamoDB (Data)

**Error Handling**: 防御的（スケジュール登録失敗時はリトライ）

---

## Domain Services

### AuthenticationService
**Purpose**: 認証関連のドメインサービス

**Responsibilities**:
- Cognito認証フロー管理
- JWTトークン検証
- ユーザーコンテキスト抽出

**Methods**:
- `authenticateUser(code: string): Promise<AuthResult>`
- `validateToken(token: string): Promise<boolean>`
- `getUserContext(event: APIGatewayEvent): UserContext`

**Orchestration Flow**:
```
1. 認証コードをCognitoに送信
2. JWTトークンを取得
3. ユーザー情報をDynamoDBに保存（初回のみ）
4. 認証結果を返却
```

**Dependencies**:
- CognitoAuthorizer (Authentication)
- UserContextExtractor (Authentication)
- DynamoDB (Data)

**Error Handling**: 防御的（認証失敗時はリトライ）

---

### NotificationService
**Purpose**: 通知関連のドメインサービス

**Responsibilities**:
- 即時通知と遅延通知の統合管理
- FCM通知送信
- 通知ペイロード生成

**Methods**:
- `sendImmediateNotification(userId: string, notification: NotificationData): Promise<void>`
- `scheduleDelayedNotification(userId: string, notification: NotificationData, scheduledTime: Date): Promise<string>`

**Orchestration Flow (Immediate)**:
```
1. 通知ペイロードを生成
2. FCMNotificationSender.sendNotification(...)
3. 送信結果を返却
```

**Orchestration Flow (Delayed)**:
```
1. 通知ペイロードを生成
2. DelayedNotificationScheduler.scheduleNotification(...)
3. スケジュールIDを返却
```

**Dependencies**:
- ImmediateNotificationHandler (Notification)
- DelayedNotificationScheduler (Notification)
- FCMNotificationSender (Notification)

**Error Handling**: 防御的（通知送信失敗時はリトライ）

---

## Service Interaction Patterns

### Synchronous Communication (User Operations)
**Pattern**: Frontend → API Gateway → Lambda → Service → Component

**Example**: カンペ表示
```
Frontend (KanpeDisplayScreen)
  ↓ HTTP GET /actions/generate-kanpe
API Gateway (CognitoAuthorizer)
  ↓ Authorized Request
Lambda (GenerateKanpeHandler)
  ↓ Call Service
KanpeOrchestrationService
  ↓ Call AI Integration
KanpeGeneratorService (Bedrock)
  ↓ Return Kanpe
Frontend (Display Kanpe)
```

---

### Asynchronous Communication (Background Processing)
**Pattern**: EventBridge → Lambda → Service → Component

**Example**: 遅延ギフト提案通知
```
EventBridge Scheduler (Scheduled Time)
  ↓ Trigger Event
Lambda (DelayedNotificationHandler)
  ↓ Call Service
NotificationService
  ↓ Call Notification Component
FCMNotificationSender
  ↓ Send Push Notification
User's Device (Receive Notification)
```

---

## Service Layer Summary

### Thin Services (3)
- UserService
- TargetService
- FeedbackService

### Thick Services (3)
- TaredakaOrchestrationService
- KanpeOrchestrationService
- HenpouOrchestrationService

### Domain Services (2)
- AuthenticationService
- NotificationService

**Total**: 8 Services

---

## Service Responsibilities Matrix

| Service | CRUD | Orchestration | AI Integration | Notification | Error Handling |
|---|---|---|---|---|---|
| UserService | ✅ | ❌ | ❌ | ❌ | 楽観的 |
| TargetService | ✅ | ❌ | ❌ | ❌ | 楽観的 |
| FeedbackService | ✅ | ❌ | ❌ | ❌ | 楽観的 |
| TaredakaOrchestrationService | ✅ | ✅ | ✅ | ❌ | 楽観的 |
| KanpeOrchestrationService | ✅ | ✅ | ✅ | ❌ | 段階的 |
| HenpouOrchestrationService | ✅ | ✅ | ✅ | ✅ | 防御的 |
| AuthenticationService | ✅ | ✅ | ❌ | ❌ | 防御的 |
| NotificationService | ❌ | ✅ | ❌ | ✅ | 防御的 |

---

## Service Design Principles

### 1. Single Responsibility
各サービスは単一の責務を持つ（CRUD、オーケストレーション、ドメインロジック）

### 2. Dependency Injection
サービスは依存するコンポーネントをコンストラクタで受け取る

### 3. Error Handling Strategy
- **楽観的**: 単純なCRUD操作
- **段階的**: 重要な操作は防御的、その他は楽観的
- **防御的**: 認証、通知、スケジュール登録

### 4. Stateless Design
すべてのサービスはステートレス（状態はDynamoDBに永続化）

### 5. Testability
各サービスは独立してテスト可能（モック可能な依存関係）

