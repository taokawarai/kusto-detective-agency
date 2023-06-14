```
探偵さんへ、

クスト探偵事務所へようこそ！私たちは、あなたが私たちを待っているエキサイティングな新しいチャレンジに参加してくれることに興奮しています。デジタルタウンを襲った不可解な謎に挑むため、あなたの探偵スキルを試してみませんか？

想像してみてください：新年早々、デジタウンの人々は騒然としている。水道代と電気代が、消費量に変化がないにもかかわらず、なぜか2倍になっている。さらに、市長選挙も控えており、この問題を早急に解決することが求められています。

しかし、心配は無用です。この謎を解明するためには、私たちの尊敬する探偵事務所が、あなたの専門知識が不可欠です。課金に関わるテレメトリーデータを検査し、隠れたエラーを解明し、事態を収拾するために、あなたの鋭い目と細心の注意が必要なのです。

昨年、私たちはガイア・バドスコット市長にサービスを提供し、印象に残りました。昨年は、ガイア・ブドスコット市長にサービスを提供し、その印象を残すことができました。

同市の課金システムはSQLを使用していますが、4月の課金データをエクスポートしてありますので、ご安心ください。さらに、税額を計算するためのSQLクエリも確保しました。このデータとクエリを駆使して、この不可解な状況の真相に迫ってください。

刑事さん、私たちはあなたの能力に全幅の信頼を寄せていますし、あなたならきっと活躍してくれると信じています。あなたの献身的な努力と鋭い直感が、この謎を解く上で大きな力となることでしょう。

敬具
サミュエル・インプソン警部
```
## Table
- Costs
<img width="228" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/8472a368-fc2a-47f5-b51f-1c4087d65597">

- Consumption
<img width="456" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/02e305e6-38a2-4806-933d-4370e222a43b">

```sql
// これが間違った答えを出すらしい
SELECT SUM(Consumed * Cost) AS TotalCost
FROM Costs
JOIN Consumption ON Costs.MeterType = Consumption.MeterType
```


```
Costs

Consumption
| take 10 

SELECT SUM(Consumed * Cost) AS TotalCost
FROM Costs
JOIN Consumption ON Costs.MeterType = Consumption.MeterType

Consumption
| where HouseholdId == 'DTI37435952EDC35CA2'
```
<img width="490" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/2ccad758-6d07-4b61-ac05-6df2b2ddf0a4">

家ごとに毎日電気代と水道代のデータがある。

<img width="214" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/53741671-12ae-44ec-899f-3ae82beb6527">
レコードの合計が奇数？ (`7617835`)

```
Consumption
| summarize by HouseholdId
| count 
```
<img width="275" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/2b478733-52fe-4945-8396-69252b6df784">

```
126185 * 2 * 30 = 7571100　
```
レコードの数が多い気がする

```
Consumption
| summarize record_num = count() by HouseholdId
```
<img width="416" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/bf39e7d8-3d9c-4606-9a2c-4e75d5428f9c">

レコードが60を超えている家がある。
```
Consumption
| summarize record_num = count() by HouseholdId
| where record_num > 60
| count 
```
<img width="361" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/2126a158-49b4-40fa-88fc-5dc9f6ecb0e7">

60未満の家はない
```
Consumption
| summarize record_num = count() by HouseholdId
| where record_num < 60
| count 
```
<img width="375" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/09216e13-06b0-4edc-8ec5-719279a04dda">

```
Consumption
| where HouseholdId == 'DTI32F610D3CB9C6098'
```
<img width="510" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/84f225ac-1ec8-4881-af0a-02b44ef5359e">
一つサンプルを見てみると、確かに重複しているレコードがある。
重複レコードを削除すればできるかな？


```
Consumption
| distinct Timestamp, HouseholdId, MeterType, Consumed
| count 
```
<img width="443" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/63b1f400-f851-4fb8-abcc-51f5fe644188">

なるほど。だいぶレコードは減ったけど、まだ7571100より少し多い。


```
Consumption
| distinct Timestamp, HouseholdId, MeterType
| count 
```
<img width="367" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/a15fc0b1-748e-4946-b13a-73119483945d">
試しに使用量を除いたユニークなレコード数を数えたらぴったり。同じ日、時間で使用量が異なるレコードがある。これは多い方を取るしかないかな。


```
Consumption
| summarize New_Consumed=max(Consumed) by Timestamp, HouseholdId, MeterType
| count 
```
<img width="528" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/82285585-8388-4dd2-b22f-cc7b3c6b3f7b">
ふむふむ、これで正しいレコードのみ残ってそう。

```
Consumption
| summarize New_Consumed=max(Consumed) by Timestamp, HouseholdId, MeterType
| join kind=inner 
    (
    Costs
    )
    on MeterType
| extend Usage=(New_Consumed * Cost)
| summarize sum(Usage)
```
これでJOINして使用料のトータルを出す。

よしよし。ノーヒントで行けた。
<img width="424" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/2a289420-22d1-4b52-891b-27e3777dfe26">


