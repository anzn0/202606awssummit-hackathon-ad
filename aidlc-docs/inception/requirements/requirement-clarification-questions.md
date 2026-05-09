# 要件確認 - 追加質問（矛盾・曖昧点の解消）

回答ありがとうございます。以下の点について追加確認が必要です。

---

## Ambiguity 1: MVPの「主要ユースケース」の範囲

Q1の回答「主要ユースケースの機能を作ることをMVP」について、
仕様書にはScene 0〜6の複数シーンが記載されています。
MVPとして実装するシーンの範囲を明確にしてください。

### Clarification Question 1
MVPで実装するシーン（ユースケース）の範囲はどれですか？

A) Scene 0〜3のみ（基本ジャーニー：予定自動生成→撮れ高報告→カンペ提案→パフォーマンス）
B) Scene 0〜3 + Scene 4（返報性ハック：お返し提案）
C) Scene 0〜6すべて（基本ジャーニー + 応用ユースケース全部）
D) Scene 1〜3のみ（撮れ高報告→カンペ提案→パフォーマンス。予定自動生成はダミーデータ）
X) Other (please describe after [Answer]: tag below)

[Answer]: X.Scene1〜4とする。

---

## Ambiguity 2: ソーシャルログインの実装方式

Q10でGoogle/LINEソーシャルログインを選択されましたが、
Q15でデプロイ先が「後で考える」となっています。

ソーシャルログインにはOAuthコールバックURLの設定が必要なため、
何らかのホスティング環境が必要です。

### Clarification Question 2
ソーシャルログインの実装方針はどうしますか？

A) Amazon Cognito + Google/LINEフェデレーション（AWS標準構成、Amplifyと統合しやすい）
B) Firebase Authentication（Google/LINEログインが簡単、フロントエンドのみで完結）
C) MVPでは認証なし（デモ用シングルユーザー）にして、後でソーシャルログインを追加
X) Other (please describe after [Answer]: tag below)

[Answer]: A

---

回答が完了したら「完了しました」とお知らせください。
