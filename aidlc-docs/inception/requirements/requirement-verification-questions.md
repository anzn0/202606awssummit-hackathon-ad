# 要件確認質問 - コミュ♥外chu (ゆるふわカンペAD)

以下の質問に回答してください。各質問の `[Answer]:` タグの後に回答の記号（A, B, C...）を記入してください。
選択肢に合うものがない場合は `X` を選び、タグの後に自由記述してください。

---

## 【スコープ確認】

## Question 1
今回のAI-DLCワークフローで実装するスコープはどちらですか？

A) 予選用MVPのみ（5月末締切：React UI + Bedrock カンペ生成 + ダミーデータ）
B) 決勝用フルスコープのみ（6月末締切：DynamoDB + Whisper + Step Functions）
C) 予選用MVPを先に完成させ、その後決勝用フルスコープへ段階的に拡張する
X) Other (please describe after [Answer]: tag below)

[Answer]: 主要ユースケースの機能を作ることをMVPとして、データ格納も行う。このMVPを先に完成させ、その後決勝用フルスコープへ段階的に拡張する

---

## 【フロントエンド・UI】

## Question 2
フロントエンドのフレームワークは何を使用しますか？

A) React + Tailwind CSS（仕様書記載の構成）
B) React + その他CSSフレームワーク（Chakra UI, MUI等）
C) Next.js + Tailwind CSS（SSR対応）
D) Vue.js または他のフレームワーク
X) Other (please describe after [Answer]: tag below)

[Answer]: X Reactは確定。CSSフレームワークは後で決める

## Question 3
「手書き風カンペ」のUIデザインについて、どのような実装を想定していますか？

A) CSSアニメーションで手書き風フォント・罫線ノート風デザインを実装
B) デザイナーが別途Figmaでデザインを作成し、それをコンポーネント化する
C) まずはシンプルなカード型UIで機能を実装し、デザインは後から調整
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 4
「提案スワイプ（興味あり/なし）」のインタラクションはどのように実装しますか？

A) Tinder風スワイプジェスチャー（react-spring等のライブラリ使用）
B) 左右ボタン（👍/👎）によるタップ操作
C) スワイプとボタン両方対応
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

## 【AI・Bedrock連携】

## Question 5
Amazon Bedrockで使用するモデルはどれを想定していますか？

A) Claude 3.5 Sonnet（高精度、コスト高め）
B) Claude 3 Haiku（高速・低コスト、MVP向き）
C) 用途に応じて使い分け（カンペ生成はSonnet、軽量処理はHaiku）
X) Other (please describe after [Answer]: tag below)

[Answer]: X Amazon Novaを中心に想定

## Question 6
MVPでのLINEスクショ解析について、どのような入力方式を想定していますか？

A) ユーザーがスクショ画像をアップロード → Bedrockのマルチモーダル機能で解析
B) MVPではスクショ解析はスキップし、テキスト入力のみ対応
C) ダミーデータ（JSON）でスクショ解析をシミュレーション
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 7
「ゆるふわカンペAD」の人格（語尾「〜っすね」、保身優先等）はどのように管理しますか？

A) Bedrockのシステムプロンプトにハードコードする
B) DynamoDBにプロンプトテンプレートを保存し、動的に読み込む
C) 設定ファイル（JSON/YAML）でプロンプトを管理する
X) Other (please describe after [Answer]: tag below)

[Answer]: C。語尾を変えられるように変更の容易性、拡張性を持たせるため。

---

## 【バックエンド・データ】

## Question 8
MVPでのデータ永続化はどうしますか？

A) DynamoDBを使用（決勝用と同じ構成で最初から実装）
B) ローカルのJSONファイルでシミュレーション（バックエンドなし）
C) AWS Lambda + DynamoDB（シンプルなCRUDのみ）
X) Other (please describe after [Answer]: tag below)

[Answer]: A

## Question 9
DynamoDBのデータモデルで、「人物プロファイル」の主要エンティティはどれを想定していますか？

A) User（アプリユーザー）、Target（関係者）、Episode（エピソード）、Feedback（フィードバック）の4テーブル
B) シングルテーブルデザイン（すべてのエンティティを1テーブルに格納）
C) まだ決まっていない、AI-DLCで設計してほしい
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## 【認証・ユーザー管理】

## Question 10
ユーザー認証はどのように実装しますか？

A) Amazon Cognito（AWS標準、Amplifyと統合しやすい）
B) MVPでは認証なし（デモ用シングルユーザー）
C) Google/LINE等のソーシャルログイン
X) Other (please describe after [Answer]: tag below)

[Answer]: C

---

## 【Amazonギフト提案・EC連携】

## Question 11
Amazon商品リンクの提案機能はどのように実装しますか？

A) Amazon Product Advertising API（PA-API）を使用してリアルタイム検索
B) MVPではハードコードされたサンプル商品リンクを使用
C) Bedrockが商品名を提案し、Amazon検索URLを生成する（APIなし）
X) Other (please describe after [Answer]: tag below)

[Answer]: B.Amazon商品リンクの提案はコアの機能ではないため

---

## 【通知・スケジューリング】

## Question 12
「ジャストインタイム・カンペ通知」の実装方法はどうしますか？

A) Amazon EventBridge + Lambda でスケジュール通知
B) MVPではプッシュ通知なし（ユーザーがアプリを開いたときに表示）
C) Amazon SNS でプッシュ通知
X) Other (please describe after [Answer]: tag below)

[Answer]: X.EventBridge + Lambda + Firebase Cloud Messaging (FCM)

---

## 【チーム・開発体制】

## Question 13
Unit分割（Unit-UI, Unit-AD, Unit-DB, Unit-Integration）について、並行開発を想定していますか？

A) はい、4つのUnitを並行して異なるメンバーが担当する
B) いいえ、順番に実装する（依存関係を考慮）
C) Unit-UIとUnit-ADを先行し、Unit-DBとUnit-Integrationは後から
X) Other (please describe after [Answer]: tag below)

[Answer]: C

## Question 14
コードのリポジトリ管理はどうしますか？

A) GitHub（単一リポジトリ）
B) GitHub（モノレポ構成：frontend/backend/infrastructure）
C) AWS CodeCommit
X) Other (please describe after [Answer]: tag below)

[Answer]: B

---

## 【デプロイ・インフラ】

## Question 15
MVPのデプロイ先はどこですか？

A) AWS Amplify Hosting（フロントエンド）+ Lambda（バックエンド）
B) ローカル開発環境のみ（デプロイなし）
C) AWS上にフルデプロイ（Amplify + API Gateway + Lambda + DynamoDB）
X) Other (please describe after [Answer]: tag below)

[Answer]: X.あとで考える

---

## 【エクステンション設定】

## Question 16 (Extension Opt-In): Property-Based Testing
このプロジェクトにProperty-Based Testing（PBT）ルールを適用しますか？

A) Yes — すべてのPBTルールをブロッキング制約として適用（ビジネスロジック、データ変換、シリアライゼーション、ステートフルコンポーネントを含むプロジェクトに推奨）
B) Partial — 純粋関数とシリアライゼーションのラウンドトリップにのみPBTルールを適用（アルゴリズム的複雑さが限定的なプロジェクトに適切）
C) No — すべてのPBTルールをスキップ（シンプルなCRUDアプリ、UIのみのプロジェクト、または重要なビジネスロジックのない薄い統合レイヤーに適切）
X) Other (please describe after [Answer]: tag below)

[Answer]: C。ビジネスロジック自体は薄い（LLMやDBのAPIを繋ぐのがメイン）ため、PBTがなくても一般的なユニットテスト（Jest/Vitest等）があれば十分品質は担保できます

## Question 17 (Extension Opt-In): Security Baseline
このプロジェクトにセキュリティ拡張ルールを適用しますか？

A) Yes — すべてのSECURITYルールをブロッキング制約として適用（本番グレードのアプリケーションに推奨）
B) No — すべてのSECURITYルールをスキップ（PoC、プロトタイプ、実験的プロジェクトに適切）
X) Other (please describe after [Answer]: tag below)

[Answer]: A
