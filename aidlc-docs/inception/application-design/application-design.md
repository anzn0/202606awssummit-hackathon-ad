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

**Frontend State Management（追加）**:
- グローバル状態: Zustand（軽量、シンプル）
- サーバー状態: TanStack Query（React Query）
- ローカル状態: React useState/useReducer

---

## Frontend State Management Strategy（決勝用）

**重要**: 以下の状態管理戦略は**決勝用（P2）**のみで適用。MVPではReact useStateのみで十分。

### 1. グローバル状態管理（Zustand）

**管理対象**: ユーザー情報、認証トークン、通知設定

```typescript
// stores/authStore.ts
import create from 'zustand';
import { persist } from 'zustand/middleware';

interface AuthState {
  user: User | null;
  token: string | null;
  isAuthenticated: boolean;
  login: (user: User, token: string) => void;
  logout: () => void;
}

export const useAuthStore = create<AuthState>()(
  persist(
    (set) => ({
      user: null,
      token: null,
      isAuthenticated: false,
      login: (user, token) => set({ user, token, isAuthenticated: true }),
      logout: () => set({ user: null, token: null, isAuthenticated: false })
    }),
    {
      name: 'auth-storage',
      getStorage: () => localStorage
    }
  )
);

// 使用例
function LoginScreen() {
  const { login } = useAuthStore();
  
  const handleLogin = async (credentials) => {
    const { user, token } = await authService.login(credentials);
    login(user, token);
  };
  
  return <LoginForm onSubmit={handleLogin} />;
}
```

### 2. サーバー状態管理（TanStack Query）

**管理対象**: API呼び出し、キャッシング、リトライ

```typescript
// hooks/useKanpe.ts
import { useQuery, useMutation, useQueryClient } from '@tanstack/react-query';

// カンペ取得
export function useKanpe(targetId: string) {
  return useQuery({
    queryKey: ['kanpe', targetId],
    queryFn: () => fetchKanpe(targetId),
    staleTime: 5 * 60 * 1000, // 5分間キャッシュ
    cacheTime: 10 * 60 * 1000, // 10分間保持
    retry: 2, // 2回リトライ
    retryDelay: (attemptIndex) => Math.min(1000 * 2 ** attemptIndex, 30000)
  });
}

// 撮れ高報告
export function useSubmitTaredaka() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (data: TaredakaData) => submitTaredaka(data),
    onSuccess: () => {
      // キャッシュを無効化
      queryClient.invalidateQueries({ queryKey: ['episodes'] });
    },
    onError: (error) => {
      console.error('Taredaka submission failed', error);
    }
  });
}

// 使用例
function KanpeDisplayScreen({ targetId }: Props) {
  const { data: kanpe, isLoading, error } = useKanpe(targetId);
  
  if (isLoading) return <LoadingSpinner />;
  if (error) return <ErrorMessage error={error} />;
  
  return <KanpeCard kanpe={kanpe} />;
}
```

### 3. ローカル状態管理（React Hooks）

**管理対象**: フォーム入力、UI状態（モーダル開閉等）

```typescript
// components/TaredakaReportScreen.tsx
function TaredakaReportScreen() {
  // ローカル状態
  const [selectedEmotion, setSelectedEmotion] = useState<string | null>(null);
  const [isModalOpen, setIsModalOpen] = useState(false);
  
  // サーバー状態
  const { mutate: submitTaredaka, isLoading } = useSubmitTaredaka();
  
  const handleSubmit = () => {
    if (!selectedEmotion) return;
    
    submitTaredaka({
      targetId: 'target1',
      emotion: selectedEmotion
    });
  };
  
  return (
    <div>
      <EmotionTagSelector 
        selected={selectedEmotion}
        onSelect={setSelectedEmotion}
      />
      <Button onClick={handleSubmit} disabled={isLoading}>
        保存
      </Button>
    </div>
  );
}
```

### 4. キャッシング戦略

| データ種別 | キャッシュ時間 | 無効化タイミング |
|---|---|---|
| カンペ | 5分 | 新規エピソード追加時 |
| エピソード一覧 | 10分 | 撮れ高報告時 |
| ターゲット人物一覧 | 30分 | 新規ターゲット追加時 |
| ユーザー情報 | 1時間 | ログアウト時 |
| ギフト提案 | キャッシュなし | - |

```typescript
// queryClient設定
import { QueryClient } from '@tanstack/react-query';

export const queryClient = new QueryClient({
  defaultOptions: {
    queries: {
      staleTime: 5 * 60 * 1000, // デフォルト5分
      cacheTime: 10 * 60 * 1000, // デフォルト10分
      retry: 2,
      refetchOnWindowFocus: false // ウィンドウフォーカス時の自動再取得を無効化
    }
  }
});
```

### 5. 楽観的UI更新

```typescript
// hooks/useSwipeFeedback.ts
export function useSwipeFeedback() {
  const queryClient = useQueryClient();
  
  return useMutation({
    mutationFn: (feedback: Feedback) => submitFeedback(feedback),
    // 楽観的更新
    onMutate: async (newFeedback) => {
      // 進行中のクエリをキャンセル
      await queryClient.cancelQueries({ queryKey: ['feedbacks'] });
      
      // 前の値を保存
      const previousFeedbacks = queryClient.getQueryData(['feedbacks']);
      
      // 楽観的に更新
      queryClient.setQueryData(['feedbacks'], (old: Feedback[]) => [
        ...old,
        newFeedback
      ]);
      
      // ロールバック用のコンテキストを返す
      return { previousFeedbacks };
    },
    // エラー時にロールバック
    onError: (err, newFeedback, context) => {
      queryClient.setQueryData(['feedbacks'], context.previousFeedbacks);
    },
    // 成功時にサーバーから最新データを取得
    onSettled: () => {
      queryClient.invalidateQueries({ queryKey: ['feedbacks'] });
    }
  });
}
```

### 6. オフライン対応（将来拡張）

```typescript
// PWA + IndexedDB（決勝用フルスコープ後の将来拡張）
import { persistQueryClient } from '@tanstack/react-query-persist-client';
import { createSyncStoragePersister } from '@tanstack/query-sync-storage-persister';

const persister = createSyncStoragePersister({
  storage: window.localStorage
});

persistQueryClient({
  queryClient,
  persister,
  maxAge: 1000 * 60 * 60 * 24 // 24時間
});
```

### 7. 状態管理のベストプラクティス

**MVP段階**:
- React useStateのみ使用
- シンプルな実装を優先
- 状態管理ライブラリは導入しない

**決勝用**:
- Zustand: グローバル状態（認証、設定）
- TanStack Query: サーバー状態（API、キャッシング）
- React Hooks: ローカル状態（フォーム、UI）

**将来拡張**:
- オフライン対応（PWA + IndexedDB）
- リアルタイム同期（WebSocket）

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

## Error Handling Implementation Patterns（決勝用）

**重要**: 以下のエラーハンドリング実装パターンは**決勝用（P2）**のみで適用。MVPでは基本的なエラーログ記録のみで十分。

### 1. リトライロジック（防御的操作）

#### Bedrock呼び出しのリトライ
```typescript
// BedrockClient.ts
import { BedrockRuntimeClient, InvokeModelCommand } from '@aws-sdk/client-bedrock-runtime';

const bedrockClient = new BedrockRuntimeClient({
  region: 'us-east-1',
  maxAttempts: 3,
  retryMode: 'adaptive',
  retryStrategy: {
    mode: 'ADAPTIVE',
    maxAttempts: 3
  }
});

async function invokeBedrockWithRetry(prompt: string): Promise<string> {
  try {
    const command = new InvokeModelCommand({
      modelId: 'amazon.nova-pro-v1:0',
      body: JSON.stringify({ prompt })
    });
    
    const response = await bedrockClient.send(command);
    return JSON.parse(response.body.toString()).completion;
  } catch (error) {
    logger.error('Bedrock invocation failed', {
      error: error.message,
      prompt: prompt.substring(0, 100) // 最初の100文字のみログ
    });
    throw error;
  }
}
```

#### DynamoDB操作のリトライ
```typescript
// DynamoDBClient.ts
import { DynamoDBClient } from '@aws-sdk/client-dynamodb';
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';

const dynamoClient = new DynamoDBClient({
  region: 'ap-northeast-1',
  maxAttempts: 3,
  retryMode: 'adaptive'
});

const docClient = DynamoDBDocumentClient.from(dynamoClient, {
  marshallOptions: {
    removeUndefinedValues: true
  }
});
```

### 2. サーキットブレーカー（Bedrock障害対策）

```typescript
// CircuitBreaker.ts
class CircuitBreaker {
  private failureCount = 0;
  private lastFailureTime: number | null = null;
  private state: 'CLOSED' | 'OPEN' | 'HALF_OPEN' = 'CLOSED';
  
  constructor(
    private readonly threshold: number = 5,
    private readonly timeout: number = 60000, // 60秒
    private readonly resetTimeout: number = 30000 // 30秒
  ) {}
  
  async execute<T>(fn: () => Promise<T>): Promise<T> {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime! > this.resetTimeout) {
        this.state = 'HALF_OPEN';
      } else {
        throw new Error('Circuit breaker is OPEN');
      }
    }
    
    try {
      const result = await fn();
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }
  
  private onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }
  
  private onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    
    if (this.failureCount >= this.threshold) {
      this.state = 'OPEN';
      logger.warn('Circuit breaker opened', { failureCount: this.failureCount });
    }
  }
}

// 使用例
const bedrockCircuitBreaker = new CircuitBreaker(5, 60000, 30000);

async function generateKanpeWithCircuitBreaker(context: any): Promise<string> {
  try {
    return await bedrockCircuitBreaker.execute(() => 
      bedrockClient.generateKanpe(context)
    );
  } catch (error) {
    logger.warn('Circuit breaker triggered, using fallback');
    return getDefaultKanpe();
  }
}
```

### 3. フォールバック（楽観的操作）

#### Bedrock障害時のデフォルトカンペ
```typescript
// KanpeGenerator.ts
const DEFAULT_KANPE_TEMPLATES = [
  '最近どうっすか？って聞いとけばOKっす',
  'お元気っすか？って軽く聞いときましょ',
  '調子どうっすか？って声かけとけば間違いないっす'
];

async function generateKanpe(context: KanpeContext): Promise<string> {
  try {
    const kanpe = await bedrockClient.generateKanpe(context);
    
    // 人格一貫性検証
    const validation = validatePersona(kanpe);
    if (!validation.valid) {
      logger.warn('Persona validation failed', { issues: validation.issues });
      return getDefaultKanpe();
    }
    
    return kanpe;
  } catch (error) {
    logger.error('Kanpe generation failed, using fallback', { error });
    return getDefaultKanpe();
  }
}

function getDefaultKanpe(): string {
  const index = Math.floor(Math.random() * DEFAULT_KANPE_TEMPLATES.length);
  return DEFAULT_KANPE_TEMPLATES[index];
}
```

#### ギフト提案失敗時のフォールバック
```typescript
// ProductSelector.ts
async function selectProduct(context: ProductContext): Promise<Product | null> {
  try {
    const asin = await bedrockClient.selectASIN(context);
    return {
      asin,
      url: `https://www.amazon.co.jp/dp/${asin}?tag=associate-id`
    };
  } catch (error) {
    logger.warn('Product selection failed, skipping gift suggestion', { error });
    return null; // ギフト提案をスキップ（オプション機能のため）
  }
}
```

### 4. 構造化ログ

```typescript
// Logger.ts
import { Logger } from '@aws-lambda-powertools/logger';

const logger = new Logger({
  serviceName: 'komigaichu',
  logLevel: process.env.LOG_LEVEL || 'INFO'
});

// 使用例
logger.info('Kanpe generation started', {
  userId: context.userId,
  targetId: context.targetId,
  episodeCount: context.episodes.length
});

logger.error('Bedrock invocation failed', {
  userId: context.userId,
  targetId: context.targetId,
  errorCode: error.code,
  errorMessage: error.message,
  timestamp: new Date().toISOString()
});

logger.warn('Persona validation failed', {
  userId: context.userId,
  response: response.substring(0, 100),
  issues: validation.issues
});
```

### 5. エラー分類とハンドリング戦略

| エラー種別 | 例 | ハンドリング戦略 | 優先度 |
|---|---|---|---|
| **認証エラー** | JWTトークン無効 | 即座に401返却、リトライなし | P0（MVP） |
| **認可エラー** | 他人のデータアクセス | 即座に403返却、リトライなし | P2（決勝用） |
| **Bedrock障害** | タイムアウト、レート制限 | リトライ3回 → フォールバック | P2（決勝用） |
| **DynamoDB障害** | スロットリング | リトライ3回（指数バックオフ） | P2（決勝用） |
| **FCM障害** | 通知送信失敗 | リトライ3回 → DLQ送信 | P2（決勝用） |
| **バリデーションエラー** | 不正な入力 | 即座に400返却、リトライなし | P0（MVP） |
| **人格崩壊** | 禁止ワード検出 | フォールバック（デフォルトカンペ） | P2（決勝用） |

### 6. DLQ（Dead Letter Queue）設計

```typescript
// EventBridge + Lambda + SQS DLQ
// template.yaml (SAM/CloudFormation)
Resources:
  NotificationDLQ:
    Type: AWS::SQS::Queue
    Properties:
      QueueName: komigaichu-notification-dlq
      MessageRetentionPeriod: 1209600 # 14日間
      
  NotificationFunction:
    Type: AWS::Serverless::Function
    Properties:
      Handler: notification.handler
      DeadLetterQueue:
        Type: SQS
        TargetArn: !GetAtt NotificationDLQ.Arn
      EventInvokeConfig:
        MaximumRetryAttempts: 2
        MaximumEventAge: 3600 # 1時間
```

### 7. タイムアウト設定

| コンポーネント | タイムアウト | 理由 |
|---|---|---|
| API Gateway | 29秒 | API Gateway最大値 |
| Lambda（カンペ生成） | 15秒（MVP）/ 10秒（決勝用） | Bedrock応答時間 + バッファ |
| Lambda（CRUD） | 5秒 | DynamoDB応答時間 + バッファ |
| Lambda（通知） | 10秒 | FCM送信時間 + バッファ |
| Bedrock呼び出し | 10秒（MVP）/ 8秒（決勝用） | プロンプト実行時間 |
| DynamoDB呼び出し | 3秒 | クエリ実行時間 |

### 8. エラーメトリクスとアラート（決勝用）

```typescript
// CloudWatch Metricsへのカスタムメトリクス送信
import { CloudWatch } from '@aws-sdk/client-cloudwatch';

const cloudwatch = new CloudWatch({ region: 'ap-northeast-1' });

async function publishErrorMetric(errorType: string) {
  await cloudwatch.putMetricData({
    Namespace: 'KomiGaichu',
    MetricData: [{
      MetricName: 'ErrorCount',
      Value: 1,
      Unit: 'Count',
      Dimensions: [{
        Name: 'ErrorType',
        Value: errorType
      }]
    }]
  });
}

// CloudWatch Alarms設定（決勝用）
// - Bedrockエラー率が5%を超えた場合
// - DynamoDBスロットリングが発生した場合
// - FCM通知失敗率が10%を超えた場合
```

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

