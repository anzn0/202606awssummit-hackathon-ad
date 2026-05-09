# Application Design Plan - コミュ♥外chu

## Plan Overview

このプランは、コミュ♥外chuのアプリケーション設計を行うためのものです。以下の成果物を生成します：

1. **components.md**: コンポーネント定義と責務
2. **component-methods.md**: メソッドシグネチャ（ビジネスルールは後のFunctional Designで詳細化）
3. **services.md**: サービス定義とオーケストレーション
4. **component-dependency.md**: 依存関係と通信パターン
5. **application-design.md**: 統合設計ドキュメント

---

## Execution Checklist

### Phase 1: Component Identification
- [x] Frontend Components（React UI）の識別
- [x] Backend Components（API Gateway + Lambda）の識別
- [x] AI Integration Components（Bedrock）の識別
- [x] Data Components（DynamoDB）の識別
- [x] Authentication Components（Cognito）の識別
- [x] Notification Components（EventBridge + FCM）の識別

### Phase 2: Component Methods Definition
- [x] 各コンポーネントのメソッドシグネチャ定義
- [x] 入力/出力型の定義
- [x] 高レベルの目的記述（詳細なビジネスルールはFunctional Designで）

### Phase 3: Service Layer Design
- [x] サービス定義（オーケストレーション層）
- [x] サービス責務の明確化
- [x] サービス間相互作用の設計

### Phase 4: Component Dependencies
- [x] 依存関係マトリクスの作成
- [x] コンポーネント間通信パターンの定義
- [x] データフロー図の作成

### Phase 5: Design Validation
- [x] 設計の完全性チェック
- [x] 設計の一貫性チェック
- [x] すべての成果物の生成

---

## Design Questions

以下の質問に回答してください。各質問の下に `[Answer]: ` タグを使用して回答を記入してください。

### Q1: Component Organization Strategy

**Context**: 本システムは4つのUnit（Unit-UI、Unit-AD、Unit-DB、Unit-Integration）に分割されます。

**Question**: コンポーネントの組織化戦略について、以下のどのアプローチを採用しますか？

A. **技術レイヤー別**: Frontend/Backend/AI/Dataで完全に分離  
B. **機能別**: 撮れ高報告/カンペ生成/返報性ハック等の機能ごとにグループ化  
C. **ハイブリッド**: 技術レイヤーをベースに、機能別サブグループを持つ  
D. **その他**: 独自の組織化戦略

[Answer]: 【C】ハイブリッド。ルート直下は「技術レイヤー（Unit）」で分離する。各レイヤーの内部は「機能（Epic/Scene）」でグループ化する。フロントエンドチームがバックエンドの完成を待たずに開発を進められるよう、Frontendの各機能グループ内に「モックデータ（ダミーJSON）」を配置するディレクトリを必ず設けてください。

---

### Q2: Frontend Component Granularity

**Context**: React UIは、手書き風カンペ（スケッチブック風）、スワイプジェスチャー、よきにボタン等の独特なUIを持ちます。

**Question**: Frontendコンポーネントの粒度について、どのレベルで定義しますか？

A. **粗粒度**: 画面単位（撮れ高報告画面、カンペ表示画面等）  
B. **中粒度**: 機能単位（感情タグ入力、カンペ表示、スワイプフィードバック等）  
C. **細粒度**: UI要素単位（ボタン、カード、モーダル等）  
D. **混在**: 画面は粗粒度、再利用可能なUI要素は細粒度

[Answer]: 【D】混在（画面は粗粒度、再利用可能なUI要素は細粒度）

---

### Q3: Backend API Design Pattern

**Context**: API Gateway + Lambdaで、撮れ高報告、カンペ生成、返報通知等のAPIを提供します。

**Question**: Backend APIの設計パターンについて、どのアプローチを採用しますか？

A. **RESTful API**: リソース指向（/users, /targets, /episodes等）  
B. **RPC-style API**: アクション指向（/submitTaredaka, /generateKanpe等）  
C. **GraphQL**: 単一エンドポイントでクエリ駆動  
D. **ハイブリッド**: RESTfulベースに一部RPC-style

[Answer]: 【D】ハイブリッド（RESTfulベースに一部RPC-style）

---

### Q4: AI Integration Component Responsibility

**Context**: Amazon Nova（Bedrock）は、カンペ生成、ナレッジ抽出、ASIN選択等の複数の役割を持ちます。

**Question**: AI Integrationコンポーネントの責務分割について、どのアプローチを採用しますか？

A. **単一コンポーネント**: BedrockServiceが全AI機能を担当  
B. **機能別分割**: KanpeGeneratorService、KnowledgeExtractorService、ProductSelectorService等  
C. **レイヤー別分割**: PromptBuilderService、BedrockClientService、ResponseParserService等  
D. **ハイブリッド**: 機能別コンポーネント + 共通のBedrockClient

[Answer]: 【D】ハイブリッド（機能別コンポーネント + 共通のBedrockClient）

---

### Q5: DynamoDB Data Access Pattern

**Context**: DynamoDBは、User、Target、Episode、Feedbackエンティティを管理します。

**Question**: DynamoDBへのデータアクセスパターンについて、どのアプローチを採用しますか？

A. **直接アクセス**: Lambda関数が直接DynamoDBにアクセス  
B. **Repository Pattern**: 各エンティティごとにRepositoryクラスを作成  
C. **Data Access Layer**: 共通のDataAccessServiceを経由  
D. **ORM/ODM**: DynamoDBのORM（例: DynamoDBMapper）を使用

[Answer]: A

---

### Q6: Service Layer Orchestration

**Context**: 複数のコンポーネント（Frontend、Backend、AI、Data）を統合する必要があります。

**Question**: サービス層のオーケストレーションについて、どのアプローチを採用しますか？

A. **薄いサービス層**: 各Lambdaが直接コンポーネントを呼び出し  
B. **厚いサービス層**: 専用のOrchestrationServiceが複雑なフローを管理  
C. **ドメインサービス**: ビジネスロジックごとにサービスを作成（TaredakaService、KanpeService等）  
D. **ハイブリッド**: 単純なフローは薄く、複雑なフローは厚く

[Answer]: 【D】ハイブリッド（単純なフローは薄く、複雑なフローは厚く）

---

### Q7: Authentication Integration Point

**Context**: Amazon Cognito + Google/LINEフェデレーションで認証を行います。

**Question**: 認証の統合ポイントについて、どのアプローチを採用しますか？

A. **API Gateway統合**: API Gatewayのオーソライザーで認証  
B. **Lambda統合**: 各Lambda関数で認証トークンを検証  
C. **ミドルウェア統合**: 共通のAuthMiddlewareを各Lambdaに適用  
D. **ハイブリッド**: API Gatewayで認証、Lambdaでユーザー情報取得

[Answer]: 【D】ハイブリッド（API Gatewayで認証、Lambdaでユーザー情報取得）

---

### Q8: Notification Component Design

**Context**: EventBridge + Lambda + FCMで、予定直前や返報タイミングでプッシュ通知を送信します。

**Question**: 通知コンポーネントの設計について、どのアプローチを採用しますか？

A. **イベント駆動**: EventBridgeルールが直接Lambda関数をトリガー  
B. **キュー駆動**: SQSキューを経由してLambda関数を非同期実行  
C. **スケジューラー駆動**: EventBridge Schedulerで個別スケジュール管理  
D. **ハイブリッド**: 即時通知はイベント駆動、遅延通知はスケジューラー駆動

[Answer]: 【D】ハイブリッド（即時通知はイベント駆動、遅延通知はスケジューラー駆動）

---

### Q9: Error Handling Strategy

**Context**: マルチモーダルAI統合、DynamoDB操作、外部API呼び出し等でエラーが発生する可能性があります。

**Question**: エラーハンドリング戦略について、どのアプローチを採用しますか？

A. **楽観的**: エラーは発生時に処理、リトライなし  
B. **防御的**: すべての外部呼び出しにリトライロジックを実装  
C. **段階的**: 重要な操作（認証、決済）は防御的、その他は楽観的  
D. **カスタム**: 各コンポーネントごとに独自のエラーハンドリング

[Answer]: 【C】段階的（重要な操作は防御的、その他は楽観的）

---

### Q10: Component Communication Pattern

**Context**: Frontend ↔ Backend ↔ AI ↔ Data の通信が発生します。

**Question**: コンポーネント間の通信パターンについて、どのアプローチを採用しますか？

A. **同期通信**: すべてHTTP/REST APIで同期呼び出し  
B. **非同期通信**: EventBridge/SQSでイベント駆動  
C. **ハイブリッド**: ユーザー操作は同期、バックグラウンド処理は非同期  
D. **その他**: 独自の通信パターン

[Answer]: 【C】ハイブリッド（ユーザー操作は同期、バックグラウンド処理は非同期）

---

### Q11: State Management Strategy

**Context**: ユーザーの撮れ高データ、カンペ生成状態、返報タイミング等の状態管理が必要です。

**Question**: 状態管理戦略について、どのアプローチを採用しますか？

A. **ステートレス**: すべての状態をDynamoDBに永続化、Lambda関数はステートレス  
B. **セッション管理**: Cognito/JWTでセッション状態を管理  
C. **キャッシュ層**: ElastiCacheで頻繁にアクセスされる状態をキャッシュ  
D. **ハイブリッド**: DynamoDB永続化 + セッション管理

[Answer]: 【A】ステートレス（すべての状態をDynamoDBに永続化、Lambda関数はステートレス）

---

### Q12: Amazon Remote Cart URL Generation

**Context**: AD（Bedrock）がASINを選択し、Amazon Remote Cart URLを生成します。

**Question**: Amazon Remote Cart URL生成の責務について、どのコンポーネントが担当しますか？

A. **AI Integration Component**: BedrockがASIN選択とURL生成を両方担当  
B. **Backend Component**: BedrockがASIN選択、BackendがURL生成  
C. **Frontend Component**: BackendがASIN提供、FrontendがURL生成  
D. **専用Service**: ProductLinkServiceが専任で担当

[Answer]: 【B】Backend Component（BedrockがASIN選択、BackendがURL生成）

---

## Answer Analysis

**Instructions**: 上記のすべての質問に回答した後、以下のセクションで回答を分析します。

### Ambiguity Detection
- [x] すべての [Answer]: タグが記入されているか確認
- [x] 曖昧な回答（「mix of」「somewhere between」「not sure」「depends」）がないか確認
- [x] 未定義の用語や外部参照がないか確認
- [x] 矛盾する回答がないか確認

### Follow-up Questions (if needed)
曖昧な回答は検出されませんでした。すべての回答が明確です。

---

## Approval

すべての質問に回答し、曖昧さが解消されたら、以下のチェックボックスにチェックを入れてください。

- [x] すべての質問に回答しました
- [x] 回答に曖昧さはありません
- [x] Application Design成果物の生成を承認します

