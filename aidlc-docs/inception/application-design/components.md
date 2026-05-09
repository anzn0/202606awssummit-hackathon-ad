# Components - コミュ♥外chu

## Component Organization Strategy

**採用アプローチ**: ハイブリッド（技術レイヤーをベースに、機能別サブグループを持つ）

- **ルート直下**: 技術レイヤー（Unit）で分離
- **各レイヤー内部**: 機能（Epic/Scene）でグループ化
- **Frontend特記事項**: 各機能グループ内に「モックデータ（ダミーJSON）」ディレクトリを配置し、バックエンド完成を待たずに開発可能

---

## Frontend Components (Unit-UI)

### 粒度戦略
**混在アプローチ**: 画面は粗粒度、再利用可能なUI要素は細粒度

### 画面コンポーネント（粗粒度）

#### 1. TaredakaReportScreen
**Purpose**: 撮れ高報告画面

**Responsibilities**:
- 予定終了後の撮れ高入力UI提供
- 感情タグ、音声メモ、スクショアップロードの統合
- ADからの通知表示

**Sub-components**:
- EmotionTagInput（感情タグ入力）
- VoiceMemoRecorder（音声メモ録音）
- ImageUploader（スクショアップロード）

---

#### 2. KanpeDisplayScreen
**Purpose**: カンペ表示画面

**Responsibilities**:
- 手書き風カンペ（スケッチブック風）の表示
- アイスブレイクカンペ表示
- NG話題（地雷）回避アラート表示
- ギフト提案表示（オプション）

**Sub-components**:
- SketchbookCanvas（スケッチブック風UI）
- IcebreakKanpe（アイスブレイクカンペ）
- NGTopicAlert（NG話題アラート）
- GiftSuggestion（ギフト提案）

---

#### 3. SwipeFeedbackScreen
**Purpose**: スワイプフィードバック画面

**Responsibilities**:
- Tinder風スワイプジェスチャー実装
- 「興味あり/なし」フィードバック収集
- スワイプアニメーション

**Sub-components**:
- SwipeCard（スワイプカード）
- SwipeGestureHandler（ジェスチャーハンドラー）

---

#### 4. YokiniBuyScreen
**Purpose**: 「よきに」決済画面

**Responsibilities**:
- 「よきに」ボタン表示
- Amazon Remote Cart URLへの遷移
- 購入確認UI

**Sub-components**:
- YokiniButton（よきにボタン）
- ProductPreview（商品プレビュー）

---

#### 5. LoginScreen
**Purpose**: ログイン画面

**Responsibilities**:
- Google/LINEソーシャルログインボタン表示
- Cognito認証フロー統合

**Sub-components**:
- SocialLoginButton（ソーシャルログインボタン）

---

### 再利用可能UIコンポーネント（細粒度）

#### 1. ADMessageBubble
**Purpose**: ADのメッセージ吹き出し

**Responsibilities**:
- ゆるふわADの口調でメッセージ表示
- 「〜っすね」語尾の統一

---

#### 2. SketchbookCard
**Purpose**: スケッチブック風カード

**Responsibilities**:
- 手書き風フォント適用
- スケッチブック風デザイン（罫線、影、質感）

---

#### 3. OneTapButton
**Purpose**: ワンタップボタン

**Responsibilities**:
- 圧ゼロUIの実現
- タップフィードバック

---

#### 4. SwipeableCard
**Purpose**: スワイプ可能カード

**Responsibilities**:
- react-spring統合
- スワイプアニメーション

---

### Frontend Mock Data

#### MockDataProvider
**Purpose**: モックデータ提供

**Responsibilities**:
- バックエンドAPI完成前のダミーデータ提供
- 各機能グループごとにモックJSONを管理

**Mock Data Files**:
- `mocks/taredaka-mock.json`
- `mocks/kanpe-mock.json`
- `mocks/feedback-mock.json`
- `mocks/gift-mock.json`

---

## Backend Components (Unit-DB + Unit-Integration)

### API Design Pattern
**ハイブリッドアプローチ**: RESTfulベースに一部RPC-style

### RESTful Resources

#### 1. UserResource
**Purpose**: ユーザー管理

**Responsibilities**:
- ユーザー情報CRUD
- Cognito認証情報との紐付け

**Endpoints**:
- `GET /users/{userId}`
- `PUT /users/{userId}`

---

#### 2. TargetResource
**Purpose**: ターゲット人物管理

**Responsibilities**:
- ターゲット人物情報CRUD
- 人物プロファイル管理

**Endpoints**:
- `GET /users/{userId}/targets`
- `POST /users/{userId}/targets`
- `GET /users/{userId}/targets/{targetId}`
- `PUT /users/{userId}/targets/{targetId}`

---

#### 3. EpisodeResource
**Purpose**: エピソード管理

**Responsibilities**:
- 撮れ高データCRUD
- エピソード履歴管理

**Endpoints**:
- `GET /users/{userId}/targets/{targetId}/episodes`
- `POST /users/{userId}/targets/{targetId}/episodes`
- `GET /users/{userId}/targets/{targetId}/episodes/{episodeId}`

---

#### 4. FeedbackResource
**Purpose**: フィードバック管理

**Responsibilities**:
- スワイプフィードバックCRUD
- 提案精度向上データ収集

**Endpoints**:
- `POST /users/{userId}/feedback`
- `GET /users/{userId}/feedback`

---

### RPC-style Actions

#### 1. SubmitTaredakaAction
**Purpose**: 撮れ高報告アクション

**Responsibilities**:
- 感情タグ、音声メモ、スクショの統合処理
- AI解析トリガー

**Endpoint**:
- `POST /actions/submit-taredaka`

---

#### 2. GenerateKanpeAction
**Purpose**: カンペ生成アクション

**Responsibilities**:
- 過去エピソードからカンペ生成
- NG話題検出
- ギフト提案生成（オプション）

**Endpoint**:
- `POST /actions/generate-kanpe`

---

#### 3. ScheduleHenpouAction
**Purpose**: 返報スケジュール設定アクション

**Responsibilities**:
- 返報タイミング計算（24時間、2〜3週間）
- EventBridge Scheduler設定

**Endpoint**:
- `POST /actions/schedule-henpou`

---

## AI Integration Components (Unit-AD)

### Responsibility Strategy
**ハイブリッドアプローチ**: 機能別コンポーネント + 共通のBedrockClient

### Shared Component

#### BedrockClient
**Purpose**: Amazon Nova（Bedrock）共通クライアント

**Responsibilities**:
- Bedrock API呼び出し
- プロンプト送信
- レスポンス受信
- エラーハンドリング

---

### Functional Components

#### 1. KanpeGeneratorService
**Purpose**: カンペ生成

**Responsibilities**:
- アイスブレイクカンペ生成
- NG話題検出
- ADの口調（「〜っすね」）適用

**Dependencies**:
- BedrockClient

---

#### 2. KnowledgeExtractorService
**Purpose**: ナレッジ抽出

**Responsibilities**:
- スクショ解析（マルチモーダル）
- 音声メモテキスト化
- エピソード情報抽出

**Dependencies**:
- BedrockClient

---

#### 3. ProductSelectorService
**Purpose**: ASIN選択

**Responsibilities**:
- 相手の状況に合わせた商品ASIN選択
- プロンプトエンジニアリングによる推奨

**Dependencies**:
- BedrockClient

---

#### 4. HenpouTimingCalculator
**Purpose**: 返報タイミング計算

**Responsibilities**:
- 24時間以内のお礼カンペタイミング計算
- 2〜3週間後のギフト提案タイミング計算
- 次回予定直前のタイミング計算

**Dependencies**:
- なし（ビジネスロジックのみ）

---

## Data Components (Unit-DB)

### Data Access Pattern
**直接アクセス**: Lambda関数が直接DynamoDBにアクセス

### DynamoDB Tables

#### 1. UsersTable
**Purpose**: ユーザー情報管理

**Responsibilities**:
- ユーザープロファイル保存
- Cognito Sub IDとの紐付け

**Access Pattern**:
- Lambda関数が直接CRUD操作

---

#### 2. TargetsTable
**Purpose**: ターゲット人物情報管理

**Responsibilities**:
- 人物プロファイル保存
- 関係性情報保存

**Access Pattern**:
- Lambda関数が直接CRUD操作

---

#### 3. EpisodesTable
**Purpose**: エピソード履歴管理

**Responsibilities**:
- 撮れ高データ保存
- 感情タグ、音声メモ、スクショ解析結果保存

**Access Pattern**:
- Lambda関数が直接CRUD操作

---

#### 4. FeedbackTable
**Purpose**: フィードバック履歴管理

**Responsibilities**:
- スワイプフィードバック保存
- 提案精度向上データ保存

**Access Pattern**:
- Lambda関数が直接CRUD操作

---

## Authentication Components

### Integration Point
**ハイブリッドアプローチ**: API Gatewayで認証、Lambdaでユーザー情報取得

#### 1. CognitoAuthorizer (API Gateway)
**Purpose**: API Gateway認証

**Responsibilities**:
- JWTトークン検証
- Google/LINEフェデレーション統合
- 認証済みリクエストのみLambdaに転送

---

#### 2. UserContextExtractor (Lambda)
**Purpose**: ユーザー情報取得

**Responsibilities**:
- Cognito Sub IDからユーザー情報取得
- Lambda関数内でユーザーコンテキスト提供

---

## Notification Components

### Design Strategy
**ハイブリッドアプローチ**: 即時通知はイベント駆動、遅延通知はスケジューラー駆動

#### 1. ImmediateNotificationHandler (EventBridge Rule)
**Purpose**: 即時通知

**Responsibilities**:
- 予定直前のカンペ通知
- 撮れ高報告リマインダー

**Trigger**:
- EventBridgeルールが直接Lambda関数をトリガー

---

#### 2. DelayedNotificationScheduler (EventBridge Scheduler)
**Purpose**: 遅延通知

**Responsibilities**:
- 24時間以内のお礼カンペ通知
- 2〜3週間後のギフト提案通知

**Trigger**:
- EventBridge Schedulerで個別スケジュール管理

---

#### 3. FCMNotificationSender (Lambda)
**Purpose**: FCM通知送信

**Responsibilities**:
- Firebase Cloud Messaging経由でプッシュ通知送信
- 通知ペイロード生成

---

## Amazon Remote Cart URL Generation

### Responsibility Assignment
**Backend Component**: BedrockがASIN選択、BackendがURL生成

#### 1. ProductSelectorService (AI Integration)
**Purpose**: ASIN選択

**Responsibilities**:
- 相手の状況に合わせた商品ASIN選択

---

#### 2. RemoteCartURLGenerator (Backend)
**Purpose**: Amazon Remote Cart URL生成

**Responsibilities**:
- ASINを受け取り、Amazon Remote Cart URL生成
- URL形式: `https://www.amazon.co.jp/gp/aws/cart/add.html?ASIN.1={asin}&Quantity.1=1`

---

## Component Summary

### Frontend Components
- 5画面コンポーネント（粗粒度）
- 4再利用可能UIコンポーネント（細粒度）
- 1モックデータプロバイダー

### Backend Components
- 4 RESTful Resources
- 3 RPC-style Actions

### AI Integration Components
- 1共通クライアント（BedrockClient）
- 4機能別サービス

### Data Components
- 4 DynamoDB Tables

### Authentication Components
- 1 API Gateway Authorizer
- 1 Lambda User Context Extractor

### Notification Components
- 1即時通知ハンドラー
- 1遅延通知スケジューラー
- 1 FCM通知送信

### URL Generation Components
- 1 ASIN選択サービス（AI Integration）
- 1 URL生成サービス（Backend）

**Total**: 30+ コンポーネント

