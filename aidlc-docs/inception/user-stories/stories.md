# ユーザーストーリー - コミュ♥外chu

## ストーリー整理方針
- **優先度順**: MVP必須機能 → MVP拡張機能 → 決勝用機能
- **粒度**: 技術レイヤー（Frontend/Backend/AI Integration）別の中粒度
- **受容基準**: 簡易版（Given-When-Then形式で1〜2個）

---

# MVP必須機能（Scene 1〜5）

## Epic 1: 撮れ高報告（Scene 1）

### Story 1.1: ワンタップ感情タグ入力（Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 予定終了後、ワンタップで感情タグ（🤒 体調悪そう、😊 元気そう、😰 悩んでそう等）を記録したい  
**So that** 文字入力なしで相手の状態を記録できる

**Acceptance Criteria:**
- **Given** 予定終了後にADから「お疲れーっす！今日の撮れ高ポチッとしといてくださーい」と通知が来る
- **When** ユーザーが感情タグ（絵文字ボタン）をワンタップする
- **Then** 感情タグがDynamoDBに保存され、「保存しました〜」とADが確認メッセージを表示する

**Technical Layer:** Frontend (React)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 1.2: 音声メモ入力（Frontend + AI Integration）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 短い音声メモ（「腰痛いらしい」等）を録音して記録したい  
**So that** 文字入力なしで詳細情報を記録できる

**Acceptance Criteria:**
- **Given** 撮れ高報告画面で音声メモボタンが表示されている
- **When** ユーザーが音声メモボタンをタップして短いメモを録音する
- **Then** 音声がAmazon Nova（Bedrock）でテキスト化され、DynamoDBに保存される

**Technical Layer:** Frontend (React) + AI Integration (Bedrock)  
**Priority:** P1 (MVP拡張)  
**Persona:** ターゲットユーザー（健太）、AD


---

### Story 1.4: 撮れ高データ保存（Backend）
**As a** システム  
**I want to** ユーザーが入力した撮れ高データ（感情タグ、音声メモ、スクショ解析結果）をDynamoDBに保存したい  
**So that** 後でカンペ生成に利用できる

**Acceptance Criteria:**
- **Given** ユーザーが撮れ高データを入力した
- **When** データがAPI Gateway + Lambdaを経由してDynamoDBに送信される
- **Then** DynamoDBのEpisodeテーブルにデータが保存され、ユーザーIDとターゲット人物IDに紐づく

**Technical Layer:** Backend (API Gateway + Lambda + DynamoDB)  
**Priority:** P0 (MVP必須)  
**Persona:** システム

---

## Epic 2: 会話サバイバル・カンペの提示（Scene 2）

### Story 2.1: アイスブレイクと気遣いの一言カンペ（AI Integration + Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 次回の予定直前に、相手の直近の状況から生成された「完璧な第一声」のカンペを受け取りたい  
**So that** 会った瞬間に何を言うべきか考えずに、相手に「自分のことを覚えてくれている」と錯覚させられる

**Acceptance Criteria:**
- **Given** 次回の予定が近づいており、過去のエピソード（例: 「腰痛いらしい」）がDynamoDBに保存されている
- **When** ADが過去のエピソードをDynamoDBから取得し、Amazon Nova（Bedrock）でアイスブレイクのカンペを生成する
- **Then** 「とりあえずこれ読んどいてくださーい」と手書き風UIでカンペが表示される（例: 「『腰の調子、どうっすか？』って聞いとけばOKっす」）

**Technical Layer:** AI Integration (Bedrock) + Frontend (React)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 2.2: NG話題（地雷）回避アラート（AI Integration + Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 絶対に振ってはいけない話題（離婚、ペットの死等）を赤字で警告してほしい  
**So that** 社会的な死（地雷を踏むこと）を防げる

**Acceptance Criteria:**
- **Given** 過去のエピソードに「最近離婚した」「ペットが亡くなった」等のNG話題が含まれている
- **When** ADがNG話題を検出する
- **Then** カンペの上部に赤字で「⚠️ 絶対に聞いちゃダメ：『奥さん元気？』『犬は？』」と警告が表示される

**Technical Layer:** AI Integration (Bedrock) + Frontend (React)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 2.3: ついでギフトの提案（AI Integration + Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** カンペの下部に「おまけ」としてAmazonギフト提案を受け取りたい  
**So that** さらに好感度を上げたい場合に、思考停止でギフトを手配できる

**Acceptance Criteria:**
- **Given** アイスブレイクカンペが生成された
- **When** ADが相手の状態（例: 腰痛）に合わせてギフト（例: きき湯 ファインヒート）を提案する
- **Then** カンペの下部に「ついでにこれ買っとけばさらに安泰っす👍」とAmazon商品詳細ページURL（ASINベースのアソシエイトリンク）付きで表示される

**Technical Layer:** AI Integration (Bedrock) + Frontend (React)  
**Priority:** P1 (MVP拡張)  
**Note:** Amazon PA-APIは利用せず、Bedrockが推奨商品のASINを選択し、ASIN指定の商品詳細ページ（アソシエイトリンク）への遷移で統一。ユーザーの「摩擦ゼロ体験」は、Amazon側の『今すぐ買う（1-Click）』機能に委ねる。マネタイズのための「ついで（オプション）」要素。  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 2.4: スワイプフィードバック（興味あり/なし）（Frontend + Backend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** カンペやギフト提案に対してTinder風スワイプで「興味あり/なし」を反応したい  
**So that** 次回のAD提案精度が向上する

**Acceptance Criteria:**
- **Given** カンペまたはギフト提案が表示されている
- **When** ユーザーが右スワイプ（興味あり）または左スワイプ（興味なし）する
- **Then** フィードバックがDynamoDBに保存され、ADの提案精度が向上する

**Technical Layer:** Frontend (React + react-spring) + Backend (DynamoDB)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

## Epic 3: パフォーマンス（Scene 3）

### Story 3.1: カンペ表示画面（手書き風UI）（Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** カンペを手書き風のスケッチブック風UIで表示してほしい  
**So that** 「圧ゼロ」の心地よい体験ができる

**Acceptance Criteria:**
- **Given** カンペが生成されている
- **When** ユーザーがカンペ表示画面を開く
- **Then** 手書き風フォント・スケッチブック風デザインでカンペが表示される

**Technical Layer:** Frontend (React + CSS)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 3.2: 思考停止の「よきに」決済（Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 「よきに」ボタンをワンタップするだけで、Amazon商品詳細ページに遷移したい  
**So that** 商品を探す手間を完全に省き、ADが選んだ商品に直接アクセスして、最小限の意識で保身（ギフト手配）を完了できる

**Acceptance Criteria:**
- **Given** ギフト提案（Amazonリンク）が表示されている
- **When** ユーザーが「よきに」ボタン（または「よきに計らえ」ボタン）をタップする
- **Then** Amazon商品詳細ページに直接遷移する（商品を探す手間を完全に省く）

**Technical Layer:** Frontend (React)  
**Priority:** P0 (MVP必須)  
**Note:** ASIN指定の商品詳細ページ（アソシエイトリンク）への遷移で統一。Bedrockが推奨商品のASINを選択し、商品詳細ページURLを生成（例: `https://www.amazon.co.jp/dp/B0XXXXXX?tag=associate-id`）。ユーザーの「摩擦ゼロ体験」は、Amazon側の『今すぐ買う（1-Click）』機能に委ねる。Amazon PA-APIは利用しない。Amazon Remote Cart URLはセキュリティ仕様変更に弱いため、全フェーズで商品詳細ページ方式を採用。  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 3.3: アドバイス実行確認（Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** アドバイス系の行動提案に対して「了解」ボタンをタップしたい  
**So that** ADに「実行した」ことを伝え、次回の提案精度を向上させる

**Acceptance Criteria:**
- **Given** アドバイス系の行動提案（例: 「『最近どう？』って軽く聞いとけばOKっす」）が表示されている
- **When** ユーザーが「了解」ボタンをタップする
- **Then** 「了解しました〜」とADが確認メッセージを表示し、実行フラグがDynamoDBに保存される

**Technical Layer:** Frontend (React) + Backend (DynamoDB)  
**Priority:** P1 (MVP拡張)  
**Persona:** ターゲットユーザー（健太）、AD

---

## Epic 4: 返報性ハック（二段階返報ロジック）（Scene 4）

### Story 4.1: 第一段階：即効性お礼カンペ通知（24時間以内）（AI Integration + Backend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 「もらった」エピソードから24時間以内に、感謝のLINEを送るためのカンペテキストをADが自動生成して通知してほしい  
**So that** 「気が利かない人」認定される前に、秒でお礼を伝えて「マメな人」として評価される

**Acceptance Criteria:**
- **Given** ユーザーが「叔父さんからお酒をもらった」というエピソードを記録した
- **When** 事象発生から24時間以内に、最初のお礼メッセージ（カンペ）の送付タイミングが到来する
- **Then** ADが「お疲れーっす！昨日の件、とりあえず今日はお礼のLINEだけ送っときましょ！お返しは今すぐだとガッツいてる感（事務的）出ちゃうんで、時期が来たらまたリマインドしますね👍『この前のお酒、めっちゃ美味しかったです！ありがとうございました』」と通知する

**Technical Layer:** AI Integration (Bedrock) + Backend (Lambda + EventBridge)  
**Priority:** P0 (MVP必須)  
**Note:** ADの先回りロジック「鉄は熱いうちに打て」を実装。ただし、ギフトは「早すぎて事務的」にならないよう、第二段階まで待つことをユーザーに説明する。  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 4.2: 第二段階：遅効性ギフト提案タイミング計算（2〜3週間後、または次回予定直前）（Backend）
**As a** システム  
**I want to** 過去の「もらった」エピソードから、社会的マナーとして最適なギフト返報タイミングを自動計算したい  
**So that** 「早すぎて事務的」にならず、「遅すぎて失礼」にもならない完璧なタイミングでお返し提案ができる

**Acceptance Criteria:**
- **Given** ユーザーが「叔父さんからお酒をもらった」というエピソードを記録している
- **When** 事象発生から2〜3週間後、または次にその人と会う予定の直前のタイミングが到来する
- **Then** システムが「今ならマナー的に完璧なタイミング」と判断し、通知を準備する

**Technical Layer:** Backend (Lambda)  
**Priority:** P0 (MVP必須)  
**Note:** 「マナーの時差（遅延評価）」をシステムが完全に管理。ユーザーは一切の判断を放棄できる。  
**Persona:** システム

---

### Story 4.3: 第二段階：遅効性ギフト提案通知（2〜3週間後、または次回予定直前）（Backend + Frontend）
**As a** 極度の面倒くさがりのユーザー  
**I want to** 社会的マナーとして最適なタイミング（2〜3週間後、または次回予定直前）で自動的にお返し提案通知を受け取りたい  
**So that** 「早すぎて事務的」にも「遅すぎて失礼」にもならず、完璧なタイミングで返報できる

**Acceptance Criteria:**
- **Given** 返報タイミング（2〜3週間後、または次回予定直前）が到来した
- **When** EventBridge + Lambda + FCMが通知を送信する
- **Then** ユーザーのスマホに「あ、そういえば前にお酒もらった件、そろそろお返ししとく（または次に会う時に渡す）時期っすね。今ならマナー的に完璧なタイミングなんで、これポチっときましょ。叔父さん好きそうな『塩辛』の限定品（Amazonリンク）見つけといたっす」と通知が届く

**Technical Layer:** Backend (EventBridge + Lambda + FCM) + Frontend (React)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

# MVP基盤機能

## Epic 5: PTA集会カンペ（Scene 5）

### Story 5.1: Googleカレンダー連携によるPTA予定検出（Backend + AI Integration）
**As a** ユーザー  
**I want to** Googleカレンダーから「PTA集会」等の予定を自動検出してほしい  
**So that** 手動で予定を入力する手間を省ける

**Acceptance Criteria:**
- **Given** ユーザーがGoogleアカウントでログインし、カレンダー連携を許可している
- **When** ADがGoogleカレンダーから「PTA集会」等の予定を検出する
- **Then** 開始時刻の30分前に「今日のPTA集会、こんな感じで乗り切りましょ〜」と通知が届く

**Technical Layer:** Backend (Google Calendar API + Lambda) + AI Integration (Bedrock)  
**Priority:** P0 (MVP必須)  
**Note:** 位置情報APIは使用せず、カレンダーの日時トリガーのみを使用  
**Persona:** ターゲットユーザー（健太）、AD

---

### Story 5.2: PTA集会カンペ生成（AI Integration + Frontend）
**As a** ユーザー  
**I want to** PTA集会の直前に、過去のエピソードから生成された会話カンペを受け取りたい  
**So that** PTA集会で何を話すべきか考えずに、スムーズに会話できる

**Acceptance Criteria:**
- **Given** PTA集会の予定が30分後に迫っている
- **When** ADが過去のエピソード（PTA関連の人物情報）から会話カンペを生成する
- **Then** 「今日のPTA集会、こんな感じで乗り切りましょ〜『〇〇さん、お子さん元気ですか？』って聞いとけばOKっす」とカンペが表示される

**Technical Layer:** AI Integration (Bedrock) + Frontend (React)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）、AD

---

## Epic 6: 認証・ユーザー管理

### Story 6.1: ソーシャルログイン（Google/LINE）（Backend + Frontend）
**As a** ユーザー  
**I want to** Google/LINEアカウントでログインしたい  
**So that** 新規アカウント作成の手間を省ける

**Acceptance Criteria:**
- **Given** ログイン画面が表示されている
- **When** ユーザーが「Googleでログイン」または「LINEでログイン」ボタンをタップする
- **Then** Amazon Cognito + Google/LINEフェデレーションでOAuth認証が完了し、ユーザーがアプリにログインする

**Technical Layer:** Backend (Cognito) + Frontend (React)  
**Priority:** P0 (MVP必須)  
**Persona:** ターゲットユーザー（健太）

---

### Story 6.2: 通知機能（EventBridge + Lambda + FCM）（Backend）
**As a** システム  
**I want to** 予定直前や返報タイミングでプッシュ通知を送信したい  
**So that** ユーザーがタイムリーにカンペや提案を受け取れる

**Acceptance Criteria:**
- **Given** 通知送信タイミングが到来した
- **When** EventBridge + Lambdaが通知イベントをトリガーする
- **Then** Firebase Cloud Messaging (FCM)経由でユーザーのスマホにプッシュ通知が届く

**通知タイミング**:
- 予定直前のカンペ通知
- 返報タイミング（24時間以内、2〜3週間後）
- PTA集会等のカレンダー予定直前

**Technical Layer:** Backend (EventBridge + Lambda + FCM)  
**Priority:** P0 (MVP必須)  
**Persona:** システム

---

# 決勝用機能（Scene 0）

## Epic 7: 予定自動生成（Scene 0）

### Story 7.1: Googleカレンダー・Gmail・Google Photos連携による予定取得（Backend + AI Integration）
**As a** ユーザー  
**I want to** Googleカレンダー、Gmail、Google Photosから自動的に予定を取得してほしい  
**So that** 手動で予定を入力する手間を省ける

**Acceptance Criteria:**
- **Given** ユーザーがGoogleアカウントでログインし、カレンダー・Gmail・Google Photos連携を許可している
- **When** ADがGoogleカレンダーから予定を取得し、GmailやGoogle Photosから関連情報を解析する
- **Then** 「あ、〇〇さんと会うんすね。カレンダー入れときました〜」と予定とTODOが自動生成される

**Technical Layer:** Backend (Google Calendar API + Gmail API + Google Photos API + Lambda) + AI Integration (Bedrock)  
**Priority:** P2 (決勝用)  
**Note:** 外部連携は「Googleアカウント連携（カレンダー、Gmail、Google Photos）」と「Amazon連携（商品ページ遷移）」に限定  
**Persona:** ターゲットユーザー（健太）、AD

---

# 拡張スコープ（決勝フルスコープ後の将来拡張）

## Epic 8: サボりの免罪符（Scene 6）

### Story 8.1: 位置情報・天気連携によるサボり言い訳生成（Backend + AI Integration）
**As a** ユーザー  
**I want to** 位置情報と天気情報を組み合わせて、「サボり」の言い訳を自動生成してほしい  
**So that** 罪悪感なくサボれる

**Acceptance Criteria:**
- **Given** ユーザーの現在地と天気情報が取得されている
- **When** ADが位置情報と天気情報を組み合わせて言い訳を生成する
- **Then** 「今日は雨だし、体調悪いって言っときましょ〜」と免罪符が表示される

**Technical Layer:** Backend (位置情報API + 天気API + Lambda) + AI Integration (Bedrock)  
**Priority:** P3 (拡張スコープ - 決勝フルスコープ後の将来拡張)  
**Note:** このストーリーは決勝用フルスコープには含まれず、将来の拡張として位置づける  
**Persona:** ターゲットユーザー（健太）、AD

---

# ストーリーサマリー

## 優先度別ストーリー数
- **P0 (MVP必須)**: 14ストーリー
- **P1 (MVP拡張)**: 3ストーリー
- **P2 (決勝用)**: 1ストーリー
- **P3 (拡張スコープ)**: 1ストーリー
- **合計**: 19ストーリー

## Epic別ストーリー数
- Epic 1: 撮れ高報告（3ストーリー: 1.1, 1.2, 1.4）
- Epic 2: 会話サバイバル・カンペの提示（4ストーリー: 2.1, 2.2, 2.3, 2.4）
- Epic 3: パフォーマンス（3ストーリー: 3.1, 3.2, 3.3）
- Epic 4: 返報性ハック（3ストーリー: 4.1, 4.2, 4.3）
- Epic 5: PTA集会カンペ（2ストーリー: 5.1, 5.2）
- Epic 6: 認証・ユーザー管理（2ストーリー: 6.1, 6.2）
- Epic 7: 予定自動生成（1ストーリー: 7.1）
- Epic 8: サボりの免罪符（1ストーリー: 8.1）

## 技術レイヤー別ストーリー数
- Frontend (React): 7ストーリー
- Backend (API Gateway + Lambda + DynamoDB): 7ストーリー
- AI Integration (Bedrock): 5ストーリー

## INVEST基準チェック
すべてのストーリーは以下のINVEST基準を満たしています：
- ✅ **Independent（独立している）**: 各ストーリーは他のストーリーに依存せず実装可能
- ✅ **Negotiable（交渉可能）**: 受容基準は簡易版で、実装詳細は柔軟に調整可能
- ✅ **Valuable（価値がある）**: すべてのストーリーがユーザー体験またはシステム機能に価値を提供
- ✅ **Estimable（見積もり可能）**: 技術レイヤー別に分割され、見積もりが容易
- ✅ **Small（小さい）**: 中粒度で、1〜3日で実装可能なサイズ
- ✅ **Testable（テスト可能）**: Given-When-Then形式の受容基準でテスト可能
