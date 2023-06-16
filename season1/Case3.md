# Case3

```
ルーキー、事態が発生しました。
ニュースなどでご存知の方も多いと思いますが、本日未明、銀行強盗が発生しました。
要するに、157th Ave / 148th Streetにある古き良きダウンタウンの銀行が強盗に遭ったということだ。
警察は到着が遅すぎてギャングを見逃し、今、彼らはギャングの場所を見つけるために私たちを頼りにしています。
過去に市長であるガイア・バドスコット女史に提供したサービスが、今回の事件につながったのは間違いない。

以下は、事件の正確な順序である：

08:17AM：3人の武装集団が157th Ave / 148th Streetにある銀行に入り、店員から金を回収し始めた。
08:31AM：それなりの戦利品（現金で1,000,000ドル）を集めた後、彼らは荷物をまとめて出て行った。
08:40AM：警察は事件現場に到着したが、時すでに遅しで、ギャングは銀行の近くにはいないことが判明した。街は封鎖され、すべての車両がチェックされ、強盗は逃げることができない。目撃者によると、3人組のグループが3台の車に分乗して走り去ったという。
11:10AM：2時間半に及ぶ捜索の結果、警察は私たちに一味の潜伏先探しを依頼することにしました。

警察は、午前8時から午前11時までのすべての車両とその動きをカメラで記録したデータセットを私たちに提供しました。以下、ご覧ください。

本題に入りましょう。ギャングの隠れ家を見つけるのは君次第だ！
失望させるな
```

まず、Trafficテーブルの様子
```
Traffic
| take 10
```
<img width="562" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/163c004e-5430-4fdd-8f51-e17c9b1075e9">

08:17AM~08:31AM辺りに157th Ave / 148th Streetにいた車３台
```
Traffic
| where Ave == 157 and Street == 148
| count
```

```
Traffic
| where Ave == 157 and Street == 148
| where Timestamp between (datetime(2022-10-16T08:17:00Z) .. datetime(2022-10-16T08:31:00Z))
| count
```
<img width="647" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/4ee04c5a-95d0-4998-b154-e9dc67b899b5">

```
Traffic
| where Ave == 157 and Street == 148
| where Timestamp between (datetime(2022-10-16T08:16:00Z) .. datetime(2022-10-16T08:32:00Z))
| summarize count() by VIN
| top 50 by count_
```
<img width="656" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/f5f1f432-256e-47df-9f5f-bec8faeff850">

このヒントを見た。なるほど。`arg_mac()`は結構大事な関数。
https://github.com/pthoor/KustoDetectiveAgencyHints
```
Traffic
| where Ave == 157 and Street == 148
| where Timestamp between (datetime(2022-10-16T08:30:00Z) .. datetime(2022-10-16T08:35:00Z))
| join kind=inner 
(
    Traffic
    | summarize arg_max(Timestamp, *) by VIN
) on VIN
| summarize count() by Ave1, Street1
| top 10 by count_
```
