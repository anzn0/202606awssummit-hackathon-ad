# Units Generation Plan - コミュ♥外chu

**作成日**: 2026-05-10  
**目的**: プロジェクトを実装可能な独立したユニットに分解し、並行開発を可能にする

---

## 1. Unit分割戦略

### 1.1 分割原則

- **技術レイヤーベース**: Frontend、Backend、AI Integration、Dataの4つの技術レイヤーで分割
- **チーム分担の明確化**: 各ユニットに専任担当者をアサイン
- **並行開発の最適化**: Unit-UIとUnit-ADを先行実装、Unit-DBとUnit-Integrationを後続実装
- **疎結合設計**: モックデータ、API契約、Repository Patternにより各ユニットを独立させる

### 1.2 Unit一覧

| Unit名 | 技術レイヤー | 担当者 | 優先度 | 並行開発フェーズ |
|---|---|---|---|---|
| Unit-UI | Frontend | デザイナー | P0 | フェーズ1（先行） |
| Unit-AD | AI Integration | 企画/プロンプトエンジニア | P0 | フェーズ1（先行） |
| Unit-DB | Backend - Data | AWSエンジニアA | P0 | フェーズ2（後続） |
| Unit-Integration | Backend - Orchestration | AWSエンジニアB | P0 | フェーズ2（後続） |

---

## 2. Unit詳細設計

### 2.1 Unit-UI（Frontend）

#### 責務
- ユーザーインターフェースの実装
- 手書き風カンペ表示（スケッチブック風デザイン）
- スワイプジェスチャー（Tinder風）
- ワンタップ感情タグ入力
- 「よきに」ボタン

#### 技術スタック
- **フレームワーク**: React
- **CSSフレームワーク**: TBD（Tailwind CSS、Chakra UI、MUI等）
- **アニメーション**: react-spring（スワイプジェスチャー）
- **ホスティング**: AWS Amplify Hosting

#### 主要コンポーネント
1. **TaredakaReportScreen**: 撮れ高報告画面
2. **KanpeDisplayScreen**: カンペ表示画面
3. **SwipeFeedbackScreen**: スワイプフィードバック画面
4. **YokiniBuyScreen**: よきに決済画面
5. **LoginScreen**: ログイン画面
6. **ADMessageBubble**: ADメッセージ吹き出し（再利用可能）
7. **SketchbookCard**: スケッチブック風カード（再利用可能）
8. **OneTapButton**: ワンタップボタン（再利用可能）
9. **SwipeableCard**: スワイプ可能カード（再利用可能）
10. **MockDataProvider**: モックデータプロバイダー

#### 開発の独立性（重要）
**Unit-UIはバックエンドAPIの完成を待たず、モックデータ（JSON）を使用してUI・画面遷移を先行して完成させること**

- モックデータスキーマ: `mock-data-schema.md`参照
- MockDataProvider実装: TypeScriptで実装
- 環境変数による切り替え: `REACT_APP_USE_MOCK_DATA=true`

#### 成果物
- React UIコンポーネント（10コンポーネント）
- モックデータファイル（JSON）
- MockDataProvider実装
- CSSスタイル（手書き風デザイン）
- スワイプジェスチャー実装

#### 依存関係
- **外部依存**: なし（モックデータで完全に独立）
- **内部依存**: API契約仕様書（`api-contract-specification.md`）

#### 推定工数
- **設計**: 1日
- **実装**: 5-6日
- **テスト**: 1-2日
- **合計**: 7-9日

---

### 2.2 Unit-AD（AI Integration）

#### 責務
- AI人格プロンプトの実装
- カンペ生成ロジック
- ナレッジ抽出（スクショ、音声メモ）
- NG話題検出
- 商品ASIN選択

#### 技術スタック
- **AIモデル**: Amazon Nova（Bedrock）
- **プロンプト管理**: YAML形式
- **バージョン管理**: Git

#### 主要サービス
1. **BedrockClient**: 共通Bedrockクライアント
2. **KanpeGeneratorService**: カンペ生成サービス
3. **KnowledgeExtractorService**: ナレッジ抽出サービス
4. **ProductSelectorService**: 商品ASIN選択サービス
5. **HenpouTimingCalculator**: 返報タイミング計算サービス

#### プロンプトテンプレート
- **システムインストラクション**: `system-instruction.yaml`
- **シーン別プロンプト**: 
  - `taredaka-report.yaml`
  - `kanpe-generation.yaml`
  - `ng-topic-alert.yaml`
  - `gift-suggestion.yaml`
  - `immediate-thanks.yaml`
  - `delayed-gift-return.yaml`

#### AI人格の実装（重要）
**AI人格プロトタイプ（`ai-persona-prototype.md`）を参照し、以下を厳守**:

1. **語尾統一**: 「〜っすね」「〜っす」で統一
2. **説教禁止**: ユーザーを絶対に責めない
3. **保身優先**: 倫理観よりユーザーの保身を優先
4. **ゆるふわトーン**: 絵文字を適度に使用

#### 成果物
- プロンプトテンプレートファイル（YAML）
- Bedrockクライアント実装
- AI統合サービス実装（5サービス）
- プロンプト検証スクリプト（Python）

#### 依存関係
- **外部依存**: Amazon Bedrock（Nova）
- **内部依存**: AI人格プロトタイプ（`ai-persona-prototype.md`）

#### 推定工数
- **設計**: 1日
- **実装**: 4-5日
- **検証**: 1-2日
- **合計**: 6-8日

---

### 2.3 Unit-DB（Backend - Data）

#### 責務
- DynamoDBテーブル設計
- Repository Pattern実装
- CRUD API実装
- データ整合性管理

#### 技術スタック
- **データベース**: Amazon DynamoDB
- **API**: AWS Lambda（Python or Node.js）
- **IaC**: AWS CDK or Terraform

#### 主要テーブル
1. **UsersTable**: ユーザー情報
2. **TargetsTable**: ターゲット人物情報
3. **EpisodesTable**: エピソード情報
4. **FeedbackTable**: フィードバック情報

#### Repository実装
1. **UserRepository**: ユーザーCRUD
2. **TargetRepository**: ターゲット人物CRUD
3. **EpisodeRepository**: エピソードCRUD
4. **FeedbackRepository**: フィードバックCRUD
5. **MockRepository**: モックモード用Repository

#### DynamoDBデータモデル（重要）
**DynamoDBデータモデル（`dynamodb-data-model.md`）を参照し、以下を実装**:

- パーティションキー、ソートキー、GSI設計
- アクセスパターン定義
- Repository Pattern（軽量実装、モックモード搭載）
- 非構造化データ対応（Map型、List型）

#### 成果物
- DynamoDBテーブル定義（CDK or Terraform）
- Repository実装（4 Repositories + MockRepository）
- Lambda関数実装（CRUD API）
- ユニットテスト

#### 依存関係
- **外部依存**: Amazon DynamoDB
- **内部依存**: DynamoDBデータモデル（`dynamodb-data-model.md`）、API契約仕様書（`api-contract-specification.md`）

#### 推定工数
- **設計**: 1日
- **実装**: 4-5日
- **テスト**: 1-2日
- **合計**: 6-8日

---

### 2.4 Unit-Integration（Backend - Orchestration）

#### 責務
- API Gateway設定
- Lambda関数オーケストレーション
- EventBridge通知設定
- 返報性ハックのタイミング計算
- 商品詳細ページURL生成

#### 技術スタック
- **API**: Amazon API Gateway
- **オーケストレーション**: AWS Lambda
- **通知**: Amazon EventBridge + Firebase Cloud Messaging (FCM)
- **認証**: Amazon Cognito

#### 主要サービス
1. **TaredakaOrchestrationService**: 撮れ高報告オーケストレーション
2. **KanpeOrchestrationService**: カンペ生成オーケストレーション
3. **HenpouOrchestrationService**: 返報スケジュールオーケストレーション
4. **AuthenticationService**: 認証サービス
5. **NotificationService**: 通知サービス

#### API実装
- **RESTful Resources**: 11 endpoints
- **RPC-style Actions**: 3 endpoints
- **AI Integration**: 5 endpoints（内部API）
- **Notification**: 1 endpoint（内部API）

#### 返報性ハックの実装（重要）
**返報性ハックロジック（`henpou-timing-logic.md`）を参照し、以下を実装**:

- **MVP段階**: デフォルトスケジュール（2〜3週間後）のみ
- **決勝用フルスコープ**: カレンダーWebhook統合による動的再計算
- EventBridge Scheduler設計
- 冪等性設計

#### 成果物
- API Gateway設定
- Lambda関数実装（オーケストレーション）
- EventBridge設定（通知スケジュール）
- Cognito設定（認証）
- 統合テスト

#### 依存関係
- **外部依存**: Amazon API Gateway、AWS Lambda、Amazon EventBridge、Amazon Cognito、Firebase Cloud Messaging
- **内部依存**: 返報性ハックロジック（`henpou-timing-logic.md`）、API契約仕様書（`api-contract-specification.md`）

#### 推定工数
- **設計**: 1日
- **実装**: 5-6日
- **テスト**: 2-3日
- **合計**: 8-10日

---

## 3. 並行開発戦略

### 3.1 フェーズ1: 先行実装（Unit-UI + Unit-AD）

**期間**: 7-9日

**並行開発**:
- Unit-UI: モックデータを使用してUI・画面遷移を先行実装
- Unit-AD: プロンプトテンプレートとAI統合サービスを実装

**メリット**:
- バックエンドAPIの完成を待たずにフロントエンド開発を進められる
- AI人格の検証を早期に開始できる

**成果物**:
- Unit-UI: React UIコンポーネント、モックデータ
- Unit-AD: プロンプトテンプレート、AI統合サービス

---

### 3.2 フェーズ2: 後続実装（Unit-DB + Unit-Integration）

**期間**: 8-10日

**並行開発**:
- Unit-DB: DynamoDBテーブル設計、Repository実装
- Unit-Integration: API Gateway設定、Lambda関数オーケストレーション

**メリット**:
- Unit-UIとUnit-ADの成果物を活用できる
- API契約に基づいた実装で、フロントエンドとバックエンドの乖離を防ぐ

**成果物**:
- Unit-DB: DynamoDBテーブル、Repository実装
- Unit-Integration: API Gateway、Lambda関数、EventBridge設定

---

### 3.3 統合フェーズ

**期間**: 2-3日

**統合作業**:
- モックデータからリアルAPIへの切り替え
- 統合テスト（Bedrock、DynamoDB連携）
- E2Eテスト（ユーザージャーニー）

**成果物**:
- 統合されたシステム
- テストレポート

---

## 3.4 統合テスト計画（詳細）

**重要**: 統合テストは**決勝用（P2）**で本格実装。MVPでは手動テストのみで十分。

### 統合テストの目的
- Unit間の連携が正常に動作することを検証
- API契約仕様書との整合性を確認
- エンドツーエンドのデータフローを検証

---

### 3.4.1 Unit-UI ↔ Unit-Integration統合テスト

#### テストシナリオ
1. **ログイン → 撮れ高報告 → カンペ表示 → スワイプフィードバック**
2. モックAPIからリアルAPIへの切り替え検証

#### テストツール
- **E2Eテスト**: Cypress/Playwright
- **APIモック**: MSW (Mock Service Worker)

#### テストケース例

```typescript
// cypress/e2e/taredaka-flow.cy.ts
describe('撮れ高報告フロー', () => {
  beforeEach(() => {
    cy.login('test@example.com', 'password');
  });
  
  it('ワンタップ感情タグが正常に保存される', () => {
    // 撮れ高報告画面に遷移
    cy.visit('/taredaka-report');
    
    // 感情タグをクリック
    cy.get('[data-testid="emotion-tag-😊"]').click();
    
    // 保存ボタンをクリック
    cy.get('[data-testid="submit-button"]').click();
    
    // 成功メッセージを確認
    cy.contains('保存しました〜').should('be.visible');
    
    // DynamoDBに保存されたことを確認（APIモック）
    cy.wait('@submitTaredaka').its('request.body').should('deep.include', {
      emotion: '😊'
    });
  });
  
  it('カンペが正常に表示される', () => {
    // カンペ表示画面に遷移
    cy.visit('/kanpe/target1');
    
    // カンペが表示されることを確認
    cy.get('[data-testid="kanpe-card"]').should('be.visible');
    cy.contains('っす').should('be.visible');
    
    // 手書き風UIが適用されていることを確認
    cy.get('[data-testid="kanpe-card"]').should('have.css', 'font-family')
      .and('match', /handwriting/i);
  });
});
```

#### モックAPIからリアルAPIへの切り替え

```typescript
// src/config/api.ts
const API_BASE_URL = process.env.REACT_APP_USE_MOCK_DATA === 'true'
  ? 'http://localhost:3000/mock'
  : process.env.REACT_APP_API_BASE_URL;

export const apiClient = axios.create({
  baseURL: API_BASE_URL,
  timeout: 10000
});
```

---

### 3.4.2 Unit-Integration ↔ Unit-DB統合テスト

#### テストシナリオ
1. **CRUD API → DynamoDB保存 → データ取得**
2. Repository Patternの動作検証

#### テストツール
- **ユニットテスト**: Jest + AWS SDK Mock
- **統合テスト**: Jest + LocalStack（ローカルDynamoDB）

#### テストケース例

```typescript
// tests/integration/episode-repository.test.ts
import { EpisodeRepository } from '../../src/repositories/EpisodeRepository';
import { DynamoDBDocumentClient } from '@aws-sdk/lib-dynamodb';

describe('EpisodeRepository統合テスト', () => {
  let repository: EpisodeRepository;
  
  beforeAll(async () => {
    // LocalStackのDynamoDBに接続
    const client = new DynamoDBDocumentClient({
      endpoint: 'http://localhost:4566',
      region: 'ap-northeast-1'
    });
    repository = new EpisodeRepository(client);
    
    // テーブル作成
    await createTestTable();
  });
  
  afterAll(async () => {
    // テーブル削除
    await deleteTestTable();
  });
  
  it('エピソードが正常に保存される', async () => {
    const episode = {
      userId: 'user1',
      targetId: 'target1',
      emotion: '😊',
      timestamp: new Date().toISOString()
    };
    
    // 保存
    const saved = await repository.create(episode);
    expect(saved.id).toBeDefined();
    
    // 取得
    const retrieved = await repository.findById(saved.id);
    expect(retrieved).toMatchObject(episode);
  });
  
  it('ユーザーIDでエピソードを検索できる', async () => {
    const episodes = await repository.findByUserId('user1');
    expect(episodes.length).toBeGreaterThan(0);
    expect(episodes[0].userId).toBe('user1');
  });
});
```

---

### 3.4.3 Unit-Integration ↔ Unit-AD統合テスト

#### テストシナリオ
1. **カンペ生成リクエスト → Bedrock呼び出し → カンペ返却**
2. AI人格の一貫性検証

#### テストツール
- **ユニットテスト**: Jest + Bedrock Mock
- **統合テスト**: Jest + 実際のBedrock（決勝用のみ）

#### テストケース例

```typescript
// tests/integration/kanpe-generator.test.ts
import { KanpeGeneratorService } from '../../src/services/KanpeGeneratorService';
import { BedrockClient } from '../../src/clients/BedrockClient';

describe('KanpeGeneratorService統合テスト', () => {
  let service: KanpeGeneratorService;
  
  beforeAll(() => {
    const bedrockClient = new BedrockClient({
      region: 'us-east-1',
      // テスト環境ではモック使用
      useMock: process.env.NODE_ENV === 'test'
    });
    service = new KanpeGeneratorService(bedrockClient);
  });
  
  it('カンペが正常に生成される', async () => {
    const context = {
      targetName: '叔父さん',
      episodes: [
        { content: '腰が痛いと言っていた', timestamp: '2026-05-01' }
      ]
    };
    
    const kanpe = await service.generate(context);
    
    // カンペが生成されることを確認
    expect(kanpe).toBeDefined();
    expect(kanpe.length).toBeGreaterThan(0);
    
    // 人格一貫性を確認
    expect(kanpe).toMatch(/っす(ね)?$/);
    expect(kanpe).not.toContain('頑張りましょう');
    expect(kanpe).not.toContain('ちゃんと');
  });
  
  it('NG話題が正常に検出される', async () => {
    const context = {
      targetName: '田中さん',
      episodes: [
        { content: '最近離婚した', timestamp: '2026-05-01' }
      ]
    };
    
    const result = await service.generate(context);
    
    // NG話題アラートが含まれることを確認
    expect(result.ngTopics).toBeDefined();
    expect(result.ngTopics.length).toBeGreaterThan(0);
    expect(result.ngTopics[0]).toContain('奥さん');
  });
});
```

---

### 3.4.4 E2Eテスト（ユーザージャーニー）

#### テストシナリオ
**Scene 1〜4のフルフロー**:
1. ログイン
2. 撮れ高報告（感情タグ）
3. カンペ表示
4. スワイプフィードバック
5. 返報性ハック通知

#### テストツール
- **E2Eテスト**: Cypress/Playwright
- **環境**: ステージング環境（実際のAWS環境）

#### テストケース例

```typescript
// cypress/e2e/user-journey.cy.ts
describe('ユーザージャーニー: Scene 1〜4', () => {
  it('完全なフローが正常に動作する', () => {
    // 1. ログイン
    cy.visit('/login');
    cy.get('[data-testid="google-login-button"]').click();
    cy.url().should('include', '/dashboard');
    
    // 2. 撮れ高報告
    cy.visit('/taredaka-report');
    cy.get('[data-testid="emotion-tag-😊"]').click();
    cy.get('[data-testid="submit-button"]').click();
    cy.contains('保存しました〜').should('be.visible');
    
    // 3. カンペ表示
    cy.visit('/kanpe/target1');
    cy.get('[data-testid="kanpe-card"]').should('be.visible');
    cy.contains('っす').should('be.visible');
    
    // 4. スワイプフィードバック
    cy.get('[data-testid="kanpe-card"]')
      .trigger('touchstart', { touches: [{ clientX: 100, clientY: 100 }] })
      .trigger('touchmove', { touches: [{ clientX: 300, clientY: 100 }] })
      .trigger('touchend');
    
    cy.contains('フィードバックありがとうございます').should('be.visible');
  });
});
```

---

### 3.4.5 統合テストの実行戦略

#### MVP段階
- **手動テスト**: 開発者が手動でフローを確認
- **スモークテスト**: 主要なAPIエンドポイントが動作することを確認
- **テスト範囲**: Scene 1〜2のみ

#### 決勝用
- **自動E2Eテスト**: Cypress/Playwrightで自動化
- **統合テスト**: Jest + LocalStackで自動化
- **テスト範囲**: Scene 1〜5すべて
- **CI/CD統合**: GitHub Actionsで自動実行

#### テスト環境

| 環境 | 用途 | データ |
|---|---|---|
| ローカル | 開発中のテスト | モックデータ |
| LocalStack | 統合テスト | テストデータ |
| ステージング | E2Eテスト | テストデータ |
| 本番 | 本番稼働 | 実データ |

---

### 3.4.6 統合テストのCI/CD統合（決勝用）

```yaml
# .github/workflows/integration-test.yml
name: Integration Tests

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  integration-test:
    runs-on: ubuntu-latest
    
    services:
      localstack:
        image: localstack/localstack
        ports:
          - 4566:4566
        env:
          SERVICES: dynamodb,s3
    
    steps:
      - uses: actions/checkout@v3
      
      - name: Setup Node.js
        uses: actions/setup-node@v3
        with:
          node-version: '18'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run unit tests
        run: npm run test:unit
      
      - name: Run integration tests
        run: npm run test:integration
        env:
          AWS_ENDPOINT: http://localhost:4566
      
      - name: Run E2E tests
        run: npm run test:e2e
        env:
          CYPRESS_BASE_URL: http://localhost:3000
```

---

## 4. Unit間の依存関係

### 4.1 依存関係図

```
Unit-UI (Frontend)
  ↓ (API契約)
Unit-Integration (Backend - Orchestration)
  ↓ (Repository)
Unit-DB (Backend - Data)

Unit-AD (AI Integration)
  ↓ (AI統合サービス)
Unit-Integration (Backend - Orchestration)
```

### 4.2 インターフェース定義

#### Unit-UI ↔ Unit-Integration
- **インターフェース**: API契約仕様書（`api-contract-specification.md`）
- **通信方式**: HTTPS（RESTful API + RPC-style Actions）
- **認証**: Amazon Cognito JWTトークン

#### Unit-Integration ↔ Unit-DB
- **インターフェース**: Repository Pattern
- **通信方式**: DynamoDB SDK
- **データモデル**: DynamoDBデータモデル（`dynamodb-data-model.md`）

#### Unit-Integration ↔ Unit-AD
- **インターフェース**: AI統合サービス
- **通信方式**: Lambda関数呼び出し
- **プロンプト**: プロンプトテンプレート（YAML）

---

## 5. リスク管理

### 5.1 Unit-UI リスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| モックデータとリアルAPIの乖離 | 🟡 中 | 🟡 中 | API契約仕様書の厳守 |
| 手書き風デザインの実装難易度 | 🟢 低 | 🟡 中 | CSSアニメーションライブラリの活用 |
| スワイプジェスチャーの実装難易度 | 🟢 低 | 🟡 中 | react-springライブラリの活用 |

### 5.2 Unit-AD リスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| AI人格の崩壊 | 🔴 高 | 🟡 中 | プロンプト検証スクリプトによる継続的検証 |
| Bedrock応答時間の遅延 | 🟡 中 | 🟡 中 | プロンプト最適化、キャッシング |
| ASIN選択の精度不足 | 🟡 中 | 🟡 中 | プロンプトエンジニアリングによる精度向上 |

### 5.3 Unit-DB リスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| DynamoDBアクセスパターンの最適化不足 | 🟡 中 | 🟢 低 | GSI設計の事前検証 |
| データ整合性の問題 | 🟡 中 | 🟢 低 | DynamoDB Transactionsの使用 |
| Repository実装の複雑化 | 🟢 低 | 🟢 低 | 軽量な実装、過度な抽象化を避ける |

### 5.4 Unit-Integration リスク

| リスク | 影響度 | 発生確率 | 対策 |
|---|---|---|---|
| 返報タイミング計算の複雑性 | 🟡 中 | 🟡 中 | MVP段階はデフォルトスケジュールのみ |
| EventBridge Scheduler障害 | 🟡 中 | 🟢 低 | リトライロジック、DLQ設定 |
| 認証フローの実装難易度 | 🟡 中 | 🟢 低 | Cognito標準フローの活用 |

---

## 6. 成功基準

### 6.1 Unit-UI 成功基準
- ✅ React UIコンポーネント（10コンポーネント）が実装されている
- ✅ モックデータで画面遷移が完全に動作する
- ✅ 手書き風デザインが実装されている
- ✅ スワイプジェスチャーが実装されている

### 6.2 Unit-AD 成功基準
- ✅ プロンプトテンプレート（6シーン）が実装されている
- ✅ AI人格検証スクリプトで人格崩壊パターンが検出されない
- ✅ Bedrockでカンペ生成が正常に動作する
- ✅ ASIN選択ロジックが実装されている

### 6.3 Unit-DB 成功基準
- ✅ DynamoDBテーブル（4テーブル）が作成されている
- ✅ Repository実装（4 Repositories + MockRepository）が完了している
- ✅ CRUD API（11 endpoints）が実装されている
- ✅ ユニットテストが合格している

### 6.4 Unit-Integration 成功基準
- ✅ API Gateway設定が完了している
- ✅ Lambda関数オーケストレーション（3サービス）が実装されている
- ✅ EventBridge通知設定が完了している
- ✅ 返報タイミング計算ロジックが実装されている
- ✅ 統合テストが合格している

---

## 7. 次のステップ

### 7.1 Units Generation完了後

1. **Functional Design（per-unit）**: 各ユニットの詳細設計
2. **NFR Requirements（per-unit）**: 各ユニットの非機能要件
3. **NFR Design（per-unit）**: 各ユニットのNFR設計
4. **Infrastructure Design（per-unit）**: 各ユニットのインフラ設計
5. **Code Generation（per-unit）**: 各ユニットのコード生成
6. **Build and Test**: ビルド・テスト手順書作成

### 7.2 推奨実装順序

1. **Unit-UI**: フェーズ1（先行実装）
2. **Unit-AD**: フェーズ1（先行実装）
3. **Unit-DB**: フェーズ2（後続実装）
4. **Unit-Integration**: フェーズ2（後続実装）
5. **統合**: 統合フェーズ

---

## 8. 承認

このUnits Generation Planは、以下の要件を満たしています：

- ✅ 4つのユニットへの分割（Unit-UI、Unit-AD、Unit-DB、Unit-Integration）
- ✅ 各ユニットの責務、技術スタック、成果物の明確化
- ✅ 並行開発戦略（フェーズ1: 先行実装、フェーズ2: 後続実装）
- ✅ Unit間の依存関係とインターフェース定義
- ✅ リスク管理と成功基準

**次のステップ**: Functional Design（per-unit）を開始する。

