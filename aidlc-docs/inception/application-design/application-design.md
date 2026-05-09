# Application Design - コミュ♥外chu

## Document Overview

このドキュメントは、コミュ♥外chuのアプリケーション設計を統合したものです。詳細は個別のドキュメントを参照してください。

### Related Documents
1. **components.md**: コンポーネント定義と責務
2. **component-methods.md**: メソッドシグネチャ
3. **services.md**: サービス定義とオーケストレーション
4. **component-dependency.md**: 依存関係と通信パターン

---

## Design Decisions Summary

### Q1: Component Organization Strategy
**Decision**: ハイブリッド（技術レイヤーをベースに、機能別サブグループを持つ）

**Rationale**:
- ルート直下は技術レイヤー（Unit）で分離 → チーム分担が明確
- 各レイヤー内部は機能（Epic/Scene）でグループ化 → 機能追加が容易
- Frontend特記事項: モックデータディレクトリ配置 → バックエンド完成を待たずに開発可能

---

### Q2: Frontend Component Granularity
**Decision**: 混在（画面は粗粒度、再利用可能なUI要素は細粒度）

**Rationale**:
- 画面コンポーネント（粗粒度）: 開発スピード重視
- 再利用可能UIコンポーネント（細粒度）: コード再利用性重視
- バランスの取れたアプローチ

---

### Q3: Backend API Design Pattern
**Decision**: ハイブリッド（RESTfulベースに一部RPC-style）

**Rationale**:
- RESTful API: リソース管理（User、Target、Episode、Feedback）
- RPC-style API: 複雑なアクション（SubmitTaredaka、GenerateKanpe、ScheduleHenpou）
- 適材適所のアプローチ

---

### Q4: AI Integration Component Responsibility
**Decision**: ハイブリッド（機能別コンポーネント + 共通のBedrockClient）

**Rationale**:
- 機能別コンポーネント: 責務の明確化（KanpeGenerator、KnowledgeExtractor、ProductSelector）
- 共通BedrockClient: コード重複削減、エラーハンドリング統一

---

### Q5: DynamoDB Data Access Pattern
**Decision**: 直接アクセス（Lambda関数が直接DynamoDBにアクセス）

**Rationale**:
- サーバーレスアーキテクチャに最適
- シンプルな実装
- パフォーマンス最適化

---

### Q6: Service Layer Orchestration
**Decision**: ハイブリッド（単純なフローは薄く、複雑なフローは厚く）

**Rationale**:
- 薄いサービス層: 単純なCRUD操作 → オーバーヘッド削減
- 厚いサービス層: 複雑なオーケストレーション（カンペ生成、返報スケジュール） → ビジネスロジック集約

---

### Q7: Authentication Integration Point
**Decision**: ハイブリッド（API Gatewayで認証、Lambdaでユーザー情報取得）

**Rationale**:
- API Gateway: JWTトークン検証 → セキュリティ強化
- Lambda: ユーザーコンテキスト抽出 → ビジネスロジックで利用

---

### Q8: Notification Component Design
**Decision**: ハイブリッド（即時通知はイベント駆動、遅延通知はスケジューラー駆動）

**Rationale**:
- 即時通知: EventBridgeルール → リアルタイム性
- 遅延通知: EventBridge Scheduler → 柔軟なスケジュール管理（24時間、2〜3週間）

---

### Q9: Error Handling Strategy
**Decision**: 段階的（重要な操作は防御的、その他は楽観的）

**Rationale**:
- 防御的: 認証、通知、スケジュール登録 → 失敗許容度低い
- 楽観的: AI解析、ギフト提案 → 失敗許容度高い
- バランスの取れたアプローチ

---

### Q10: Component Communication Pattern
**Decision**: ハイブリッド（ユーザー操作は同期、バックグラウンド処理は非同期）

**Rationale**:
- 同期通信: ユーザー操作（カンペ表示、撮れ高報告） → レスポンス性
- 非同期通信: バックグラウンド処理（返報スケジュール、通知送信） → スケーラビリティ

---

### Q11: State Management Strategy
**Decision**: ステートレス（すべての状態をDynamoDBに永続化、Lambda関数はステートレス）

**Rationale**:
- サーバーレスアーキテクチャに最適
- スケーラビリティ向上
- 障害復旧が容易

---

### Q12: Amazon Remote Cart URL Generation
**Decision**: Backend Component（BedrockがASIN選択、BackendがURL生成）

**Rationale**:
- ASIN選択: AI Integration（Bedrock） → プロンプトエンジニアリングで最適化
- URL生成: Backend → シンプルな文字列操作、PA-API不要

---

## Architecture Overview

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        Frontend Layer                        │
│  (React UI - Amplify Hosting)                               │
│  - TaredakaReportScreen                                     │
│  - KanpeDisplayScreen                                       │
│  - SwipeFeedbackScreen                                      │
│  - YokiniBuyScreen                                          │
│  - LoginScreen                                              │
└─────────────────────────────────────────────────────────────┘
                              ↓ HTTPS
┌─────────────────────────────────────────────────────────────┐
│                    Authentication Layer                      │
│  (Amazon Cognito + API Gateway Authorizer)                 │
│  - Google/LINE Federation                                   │
│  - JWT Token Validation                                     │
└─────────────────────────────────────────────────────────────┘
                              ↓ Authorized
┌─────────────────────────────────────────────────────────────┐
│                       Backend Layer                          │
│  (API Gateway + Lambda)                                     │
│  - RESTful Resources (User, Target, Episode, Feedback)     │
│  - RPC-style Actions (SubmitTaredaka, GenerateKanpe, etc)  │
└─────────────────────────────────────────────────────────────┘
                    ↓                    ↓
┌──────────────────────────┐  ┌──────────────────────────────┐
│   AI Integration Layer   │  │       Data Layer             │
│  (Amazon Nova/Bedrock)   │  │  (DynamoDB)                  │
│  - KanpeGenerator        │  │  - UsersTable                │
│  - KnowledgeExtractor    │  │  - TargetsTable              │
│  - ProductSelector       │  │  - EpisodesTable             │
│  - HenpouTimingCalc      │  │  - FeedbackTable             │
└──────────────────────────┘  └──────────────────────────────┘
                    ↓
┌─────────────────────────────────────────────────────────────┐
│                    Notification Layer                        │
│  (EventBridge + Lambda + FCM)                               │
│  - ImmediateNotificationHandler                             │
│  - DelayedNotificationScheduler                             │
│  - FCMNotificationSender                                    │
└─────────────────────────────────────────────────────────────┘
```

---

## Component Summary

### Frontend Components (10)
- **画面コンポーネント（5）**: TaredakaReportScreen、KanpeDisplayScreen、SwipeFeedbackScreen、YokiniBuyScreen、LoginScreen
- **再利用可能UIコンポーネント（4）**: ADMessageBubble、SketchbookCard、OneTapButton、SwipeableCard
- **モックデータプロバイダー（1）**: MockDataProvider

### Backend Components (7)
- **RESTful Resources（4）**: UserResource、TargetResource、EpisodeResource、FeedbackResource
- **RPC-style Actions（3）**: SubmitTaredakaAction、GenerateKanpeAction、ScheduleHenpouAction

### AI Integration Components (5)
- **共通クライアント（1）**: BedrockClient
- **機能別サービス（4）**: KanpeGeneratorService、KnowledgeExtractorService、ProductSelectorService、HenpouTimingCalculator

### Data Components (4)
- **DynamoDB Tables（4）**: UsersTable、TargetsTable、EpisodesTable、FeedbackTable

### Authentication Components (2)
- **API Gateway（1）**: CognitoAuthorizer
- **Lambda（1）**: UserContextExtractor

### Notification Components (3)
- **EventBridge（2）**: ImmediateNotificationHandler、DelayedNotificationScheduler
- **Lambda（1）**: FCMNotificationSender

### URL Generation Components (2)
- **AI Integration（1）**: ProductSelectorService（ASIN選択）
- **Backend（1）**: RemoteCartURLGenerator（URL生成）

**Total Components**: 33

---

## Service Summary

### Thin Services (3)
- **UserService**: ユーザー管理の薄いサービス層
- **TargetService**: ターゲット人物管理の薄いサービス層
- **FeedbackService**: フィードバック管理の薄いサービス層

### Thick Services (3)
- **TaredakaOrchestrationService**: 撮れ高報告の複雑なオーケストレーション
- **KanpeOrchestrationService**: カンペ生成の複雑なオーケストレーション
- **HenpouOrchestrationService**: 返報スケジュールの複雑なオーケストレーション

### Domain Services (2)
- **AuthenticationService**: 認証関連のドメインサービス
- **NotificationService**: 通知関連のドメインサービス

**Total Services**: 8

---

## Key Design Patterns

### 1. Layered Architecture
- Frontend Layer
- Authentication Layer
- Backend Layer
- AI Integration Layer
- Data Layer
- Notification Layer

### 2. Service-Oriented Architecture
- Thin Services: 単純なCRUD操作
- Thick Services: 複雑なオーケストレーション
- Domain Services: ドメインロジック

### 3. Event-Driven Architecture
- EventBridge Rules: 即時通知
- EventBridge Scheduler: 遅延通知
- 非同期処理: バックグラウンドタスク

### 4. Serverless Architecture
- AWS Lambda: ステートレス関数
- DynamoDB: NoSQLデータベース
- API Gateway: RESTful API
- Amplify Hosting: フロントエンドホスティング

### 5. Microservices Pattern
- 各サービスは独立して開発・デプロイ可能
- 疎結合な設計
- スケーラビリティ

---

## Data Flow Summary

### Primary Data Flows

#### 1. 撮れ高報告フロー
```
User Input → Frontend → Backend → AI Integration → Data
```

#### 2. カンペ生成フロー
```
User Request → Frontend → Backend → Data → AI Integration → Backend → Frontend
```

#### 3. 返報性ハックフロー
```
Episode Created → Backend → AI Integration → Notification → EventBridge → (24h later) → Notification → User
                                                           → (2-3 weeks later) → Notification → User
```

#### 4. スワイプフィードバックフロー
```
User Swipe → Frontend → Backend → Data
```

---

## Technology Stack

### Frontend
- **Framework**: React
- **UI Library**: TBD（Tailwind CSS、Chakra UI、MUI等）
- **Animation**: react-spring（スワイプジェスチャー）
- **Hosting**: AWS Amplify Hosting

### Backend
- **API**: Amazon API Gateway
- **Compute**: AWS Lambda（Python or Node.js）
- **Database**: Amazon DynamoDB
- **Authentication**: Amazon Cognito

### AI Integration
- **Model**: Amazon Nova（Bedrock）
- **Capabilities**: マルチモーダル（画像・テキスト解析）

### Notification
- **Scheduler**: Amazon EventBridge（Rules + Scheduler）
- **Push Notification**: Firebase Cloud Messaging (FCM)

### Infrastructure
- **IaC**: TBD（AWS CDK、Terraform等）
- **Repository**: GitHub（モノレポ構成）

---

## Security Considerations

### Authentication & Authorization
- **Cognito**: Google/LINEフェデレーション
- **API Gateway**: JWTトークン検証
- **Lambda**: ユーザーコンテキスト抽出

### Data Protection
- **DynamoDB**: at-rest暗号化
- **HTTPS**: in-transit暗号化
- **IAM**: 最小権限の原則

### Error Handling
- **段階的戦略**: 重要な操作は防御的、その他は楽観的
- **リトライロジック**: 認証、通知、スケジュール登録
- **ログ記録**: すべてのエラーをCloudWatch Logsに記録

---

## Scalability Considerations

### Horizontal Scaling
- **Lambda**: 自動スケーリング
- **DynamoDB**: オンデマンドキャパシティ
- **API Gateway**: 自動スケーリング

### Performance Optimization
- **AI応答時間**: 5秒以内（カンペ生成）
- **画像解析**: 10秒以内（スクショ解析）
- **通知遅延**: 1分以内（プッシュ通知）

### Cost Optimization
- **Lambda**: 実行時間課金
- **DynamoDB**: オンデマンド課金
- **Bedrock**: トークン課金

---

## Testing Strategy

### Unit Testing
- **Frontend**: Jest/Vitest
- **Backend**: Jest/Vitest
- **AI Integration**: モックBedrock

### Integration Testing
- **Backend ↔ AI**: Bedrock統合テスト
- **Backend ↔ Data**: DynamoDB統合テスト
- **Backend ↔ Notification**: EventBridge統合テスト

### E2E Testing
- **Frontend ↔ Backend**: Cypress/Playwright
- **ユーザージャーニー**: Scene 1〜4のフルフロー

---

## Deployment Strategy

### CI/CD Pipeline
- **Repository**: GitHub
- **CI/CD**: TBD（GitHub Actions、AWS CodePipeline等）
- **Environments**: Dev、Staging、Production

### Deployment Sequence
1. **Frontend**: Amplify Hosting
2. **Backend**: Lambda + API Gateway
3. **Data**: DynamoDB Tables
4. **AI Integration**: Bedrock設定
5. **Notification**: EventBridge Rules + Scheduler

---

## Monitoring & Observability

### Logging
- **CloudWatch Logs**: すべてのLambda関数
- **構造化ログ**: JSON形式

### Metrics
- **CloudWatch Metrics**: Lambda実行時間、エラー率
- **カスタムメトリクス**: AI応答時間、通知送信率

### Alerting
- **CloudWatch Alarms**: エラー率閾値超過
- **SNS**: アラート通知

---

## Next Steps

1. ✅ Application Design完了
2. → Units Generation（Unit分解）
3. → Functional Design（per-unit）
4. → NFR Requirements（per-unit）
5. → NFR Design（per-unit）
6. → Infrastructure Design（per-unit）
7. → Code Generation（per-unit）
8. → Build and Test

---

## Approval

このApplication Designは、以下の設計決定に基づいています：

- ✅ コンポーネント組織化戦略: ハイブリッド
- ✅ Frontendコンポーネント粒度: 混在
- ✅ Backend API設計パターン: ハイブリッド
- ✅ AI Integration責務分割: ハイブリッド
- ✅ DynamoDBアクセスパターン: 直接アクセス
- ✅ サービス層オーケストレーション: ハイブリッド
- ✅ 認証統合ポイント: ハイブリッド
- ✅ 通知コンポーネント設計: ハイブリッド
- ✅ エラーハンドリング戦略: 段階的
- ✅ コンポーネント間通信パターン: ハイブリッド
- ✅ 状態管理戦略: ステートレス
- ✅ Amazon Remote Cart URL生成: Backend Component

すべての設計決定が承認され、Application Design成果物が生成されました。

