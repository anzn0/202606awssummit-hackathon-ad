# Component Dependencies - コミュ♥外chu

## Dependency Matrix

### Frontend → Backend Dependencies

| Frontend Component | Backend Endpoint | HTTP Method | Purpose |
|---|---|---|---|
| TaredakaReportScreen | `/actions/submit-taredaka` | POST | 撮れ高報告送信 |
| KanpeDisplayScreen | `/actions/generate-kanpe` | POST | カンペ生成 |
| SwipeFeedbackScreen | `/users/{userId}/feedback` | POST | フィードバック送信 |
| YokiniBuyScreen | (Frontend only) | - | Remote Cart URL生成 |
| LoginScreen | (Cognito) | - | 認証フロー |

---

### Backend → AI Integration Dependencies

| Backend Service | AI Component | Purpose |
|---|---|---|
| TaredakaOrchestrationService | KnowledgeExtractorService | 音声メモ・スクショ解析 |
| KanpeOrchestrationService | KanpeGeneratorService | カンペ生成・NG話題検出 |
| KanpeOrchestrationService | ProductSelectorService | ASIN選択 |
| HenpouOrchestrationService | HenpouTimingCalculator | 返報タイミング計算 |

---

### Backend → Data Dependencies

| Backend Service | DynamoDB Table | Operations |
|---|---|---|
| UserService | UsersTable | GET, PUT |
| TargetService | TargetsTable | GET, POST, PUT, QUERY |
| TaredakaOrchestrationService | EpisodesTable | POST |
| KanpeOrchestrationService | EpisodesTable | QUERY |
| FeedbackService | FeedbackTable | POST, QUERY |
| HenpouOrchestrationService | EpisodesTable | GET |

---

### AI Integration → Bedrock Dependencies

| AI Component | Bedrock Model | Purpose |
|---|---|---|
| KanpeGeneratorService | Amazon Nova | カンペ生成・NG話題検出 |
| KnowledgeExtractorService | Amazon Nova | マルチモーダル解析 |
| ProductSelectorService | Amazon Nova | ASIN選択 |

---

### Backend → Notification Dependencies

| Backend Service | Notification Component | Purpose |
|---|---|---|
| HenpouOrchestrationService | DelayedNotificationScheduler | 返報スケジュール登録 |
| NotificationService | ImmediateNotificationHandler | 即時通知送信 |
| NotificationService | FCMNotificationSender | FCM通知送信 |

---

## Communication Patterns

### Pattern 1: Synchronous Request-Response (User Operations)

**Flow**: Frontend → API Gateway → Lambda → Service → Component → Response

**Example**: カンペ生成
```
[Frontend: KanpeDisplayScreen]
  ↓ HTTP POST /actions/generate-kanpe
[API Gateway: CognitoAuthorizer]
  ↓ Validate JWT Token
[Lambda: GenerateKanpeHandler]
  ↓ Extract User Context
[Service: KanpeOrchestrationService]
  ↓ Query Episodes
[Data: DynamoDB EpisodesTable]
  ↓ Return Episodes
[Service: KanpeOrchestrationService]
  ↓ Generate Kanpe
[AI: KanpeGeneratorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return Kanpe Text
[AI: KanpeGeneratorService]
  ↓ Return Kanpe
[Service: KanpeOrchestrationService]
  ↓ Select ASIN (Optional)
[AI: ProductSelectorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return ASIN
[Service: KanpeOrchestrationService]
  ↓ Generate Remote Cart URL
[Backend: RemoteCartURLGenerator]
  ↓ Return URL
[Service: KanpeOrchestrationService]
  ↓ Return Kanpe Data
[Lambda: GenerateKanpeHandler]
  ↓ HTTP 200 OK
[Frontend: KanpeDisplayScreen]
  ↓ Display Kanpe
```

---

### Pattern 2: Asynchronous Event-Driven (Background Processing)

**Flow**: EventBridge → Lambda → Service → Component

**Example**: 遅延ギフト提案通知
```
[EventBridge Scheduler]
  ↓ Scheduled Time Reached
[Lambda: DelayedNotificationHandler]
  ↓ Extract Event Data
[Service: NotificationService]
  ↓ Generate Notification Payload
[Notification: FCMNotificationSender]
  ↓ Send Push Notification
[FCM: Firebase Cloud Messaging]
  ↓ Deliver to Device
[User's Device]
  ↓ Display Notification
```

---

### Pattern 3: Hybrid (Synchronous + Asynchronous)

**Flow**: Frontend → API Gateway → Lambda → Service → (Sync) Component + (Async) EventBridge

**Example**: 撮れ高報告 + 返報スケジュール
```
[Frontend: TaredakaReportScreen]
  ↓ HTTP POST /actions/submit-taredaka
[API Gateway: CognitoAuthorizer]
  ↓ Validate JWT Token
[Lambda: SubmitTaredakaHandler]
  ↓ Extract User Context
[Service: TaredakaOrchestrationService]
  ↓ Extract Knowledge (Async AI Call)
[AI: KnowledgeExtractorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return Knowledge
[Service: TaredakaOrchestrationService]
  ↓ Save Episode (Sync DB Call)
[Data: DynamoDB EpisodesTable]
  ↓ Episode Saved
[Service: TaredakaOrchestrationService]
  ↓ Trigger Henpou Scheduling (Async)
[Lambda: ScheduleHenpouHandler]
  ↓ Calculate Timing
[Service: HenpouOrchestrationService]
  ↓ Schedule Notifications
[Notification: DelayedNotificationScheduler]
  ↓ Register with EventBridge
[EventBridge Scheduler]
  ↓ Schedule Created
[Service: TaredakaOrchestrationService]
  ↓ Return Success
[Lambda: SubmitTaredakaHandler]
  ↓ HTTP 200 OK
[Frontend: TaredakaReportScreen]
  ↓ Display Success Message
```

---

## Data Flow Diagrams

### Data Flow 1: 撮れ高報告 (Taredaka Report)

```
User Input (Emotion Tag + Voice Memo + Screenshot)
  ↓
[Frontend: TaredakaReportScreen]
  ↓ Collect Input
[Frontend: submitTaredaka()]
  ↓ HTTP POST
[Backend: SubmitTaredakaAction]
  ↓ Validate Input
[AI: KnowledgeExtractorService]
  ↓ Extract Knowledge
[Bedrock: Amazon Nova]
  ↓ Analyze Voice + Image
[AI: KnowledgeExtractorService]
  ↓ Return Knowledge Data
[Backend: SubmitTaredakaAction]
  ↓ Save Episode
[Data: DynamoDB EpisodesTable]
  ↓ Episode Stored
[Backend: SubmitTaredakaAction]
  ↓ Return Success
[Frontend: TaredakaReportScreen]
  ↓ Display Success
User (Sees Confirmation)
```

---

### Data Flow 2: カンペ生成 (Kanpe Generation)

```
User Request (View Kanpe for Target)
  ↓
[Frontend: KanpeDisplayScreen]
  ↓ Load Kanpe
[Frontend: loadKanpe(targetId)]
  ↓ HTTP POST
[Backend: GenerateKanpeAction]
  ↓ Query Episodes
[Data: DynamoDB EpisodesTable]
  ↓ Return Episodes
[Backend: GenerateKanpeAction]
  ↓ Generate Icebreak Kanpe
[AI: KanpeGeneratorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return Kanpe Text
[Backend: GenerateKanpeAction]
  ↓ Detect NG Topics
[AI: KanpeGeneratorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return NG Topics
[Backend: GenerateKanpeAction]
  ↓ Select ASIN (Optional)
[AI: ProductSelectorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return ASIN
[Backend: GenerateKanpeAction]
  ↓ Generate Remote Cart URL
[Backend: RemoteCartURLGenerator]
  ↓ Return URL
[Backend: GenerateKanpeAction]
  ↓ Return Kanpe Data
[Frontend: KanpeDisplayScreen]
  ↓ Display Kanpe
User (Reads Kanpe)
```

---

### Data Flow 3: 返報性ハック (Henpou Hack)

```
Episode Created (User received gift)
  ↓
[Backend: SubmitTaredakaAction]
  ↓ Trigger Henpou Scheduling
[Backend: ScheduleHenpouAction]
  ↓ Calculate Immediate Thanks Time (24h)
[AI: HenpouTimingCalculator]
  ↓ Return Thanks Time
[Backend: ScheduleHenpouAction]
  ↓ Schedule Immediate Thanks
[Notification: DelayedNotificationScheduler]
  ↓ Register with EventBridge
[EventBridge Scheduler]
  ↓ Schedule Created (24h later)
[Backend: ScheduleHenpouAction]
  ↓ Calculate Delayed Gift Time (2-3 weeks)
[AI: HenpouTimingCalculator]
  ↓ Return Gift Time
[Backend: ScheduleHenpouAction]
  ↓ Schedule Delayed Gift
[Notification: DelayedNotificationScheduler]
  ↓ Register with EventBridge
[EventBridge Scheduler]
  ↓ Schedule Created (2-3 weeks later)

--- 24 hours later ---

[EventBridge Scheduler]
  ↓ Trigger Immediate Thanks
[Lambda: DelayedNotificationHandler]
  ↓ Generate Thanks Kanpe
[AI: KanpeGeneratorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return Thanks Text
[Lambda: DelayedNotificationHandler]
  ↓ Send Notification
[Notification: FCMNotificationSender]
  ↓ Send Push
[FCM: Firebase Cloud Messaging]
  ↓ Deliver to Device
User (Receives Thanks Kanpe)

--- 2-3 weeks later ---

[EventBridge Scheduler]
  ↓ Trigger Delayed Gift
[Lambda: DelayedNotificationHandler]
  ↓ Generate Gift Proposal
[AI: ProductSelectorService]
  ↓ Invoke Bedrock
[Bedrock: Amazon Nova]
  ↓ Return ASIN
[Lambda: DelayedNotificationHandler]
  ↓ Generate Remote Cart URL
[Backend: RemoteCartURLGenerator]
  ↓ Return URL
[Lambda: DelayedNotificationHandler]
  ↓ Send Notification
[Notification: FCMNotificationSender]
  ↓ Send Push
[FCM: Firebase Cloud Messaging]
  ↓ Deliver to Device
User (Receives Gift Proposal)
```

---

### Data Flow 4: スワイプフィードバック (Swipe Feedback)

```
User Action (Swipe Right/Left on Proposal)
  ↓
[Frontend: SwipeFeedbackScreen]
  ↓ Capture Swipe
[Frontend: onSwipeRight() or onSwipeLeft()]
  ↓ Submit Feedback
[Frontend: submitFeedback(proposalId, reaction)]
  ↓ HTTP POST
[Backend: FeedbackResource]
  ↓ Save Feedback
[Data: DynamoDB FeedbackTable]
  ↓ Feedback Stored
[Backend: FeedbackResource]
  ↓ Return Success
[Frontend: SwipeFeedbackScreen]
  ↓ Display Next Proposal
User (Continues Swiping)
```

---

## Dependency Graph (Text-based)

```
Frontend Layer
├─ TaredakaReportScreen
│  └─ → SubmitTaredakaAction (Backend)
├─ KanpeDisplayScreen
│  └─ → GenerateKanpeAction (Backend)
├─ SwipeFeedbackScreen
│  └─ → FeedbackResource (Backend)
├─ YokiniBuyScreen
│  └─ (No Backend Dependency - Frontend URL Generation)
└─ LoginScreen
   └─ → CognitoAuthorizer (Authentication)

Backend Layer
├─ SubmitTaredakaAction
│  ├─ → KnowledgeExtractorService (AI)
│  └─ → EpisodesTable (Data)
├─ GenerateKanpeAction
│  ├─ → KanpeGeneratorService (AI)
│  ├─ → ProductSelectorService (AI)
│  ├─ → RemoteCartURLGenerator (Backend)
│  └─ → EpisodesTable (Data)
├─ ScheduleHenpouAction
│  ├─ → HenpouTimingCalculator (AI)
│  ├─ → DelayedNotificationScheduler (Notification)
│  └─ → EpisodesTable (Data)
├─ UserResource
│  └─ → UsersTable (Data)
├─ TargetResource
│  └─ → TargetsTable (Data)
└─ FeedbackResource
   └─ → FeedbackTable (Data)

AI Integration Layer
├─ KanpeGeneratorService
│  └─ → BedrockClient → Amazon Nova
├─ KnowledgeExtractorService
│  └─ → BedrockClient → Amazon Nova
├─ ProductSelectorService
│  └─ → BedrockClient → Amazon Nova
└─ HenpouTimingCalculator
   └─ (No External Dependency - Business Logic Only)

Data Layer
├─ UsersTable (DynamoDB)
├─ TargetsTable (DynamoDB)
├─ EpisodesTable (DynamoDB)
└─ FeedbackTable (DynamoDB)

Authentication Layer
├─ CognitoAuthorizer (API Gateway)
│  └─ → Amazon Cognito
└─ UserContextExtractor (Lambda)
   └─ → UsersTable (Data)

Notification Layer
├─ ImmediateNotificationHandler (EventBridge Rule)
│  └─ → FCMNotificationSender
├─ DelayedNotificationScheduler (EventBridge Scheduler)
│  └─ → EventBridge Scheduler
└─ FCMNotificationSender (Lambda)
   └─ → Firebase Cloud Messaging
```

---

## Dependency Summary

### Frontend Dependencies
- **Backend APIs**: 4 endpoints
- **Authentication**: 1 Cognito integration
- **Total**: 5 external dependencies

### Backend Dependencies
- **AI Integration**: 4 services
- **Data**: 4 DynamoDB tables
- **Notification**: 2 components
- **Total**: 10 external dependencies

### AI Integration Dependencies
- **Bedrock**: 1 shared client (Amazon Nova)
- **Total**: 1 external dependency

### Data Dependencies
- **None**: DynamoDB tables have no external dependencies

### Authentication Dependencies
- **Cognito**: 1 AWS service
- **Data**: 1 DynamoDB table
- **Total**: 2 external dependencies

### Notification Dependencies
- **EventBridge**: 2 components (Rule + Scheduler)
- **FCM**: 1 Firebase service
- **Total**: 3 external dependencies

---

## Circular Dependency Check

✅ **No Circular Dependencies Detected**

All dependencies flow in one direction:
```
Frontend → Backend → AI Integration → Bedrock
Frontend → Backend → Data → DynamoDB
Frontend → Authentication → Cognito
Backend → Notification → EventBridge/FCM
```

---

## Coupling Analysis

### Tight Coupling
- **Frontend ↔ Backend**: HTTP API contracts (acceptable for client-server architecture)
- **AI Services ↔ BedrockClient**: Shared client (acceptable for common infrastructure)

### Loose Coupling
- **Backend ↔ AI Integration**: Service interfaces (good design)
- **Backend ↔ Data**: Direct DynamoDB access (acceptable for serverless)
- **Backend ↔ Notification**: Event-driven (excellent design)

### Decoupling Opportunities
- **Frontend Mock Data**: Allows frontend development without backend (already implemented)
- **AI Service Interfaces**: Allows swapping Bedrock with other AI providers (future consideration)

