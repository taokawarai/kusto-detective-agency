```
おいおい刑事さん、

あなたの専門知識が必要な別の事件が発生しました！街の人々がフィッシャーマンに狙われている。彼らの追跡を止めるために、あなたの助けが必要なんだ。

苦情が殺到しています 人々は突然のフィッシングにうんざりしています 彼らのID情報を盗もうとする電話ですこのような詐欺師を野放しにするわけにはいきません。彼らを捕まえるために、あなたの助けが必要です！

警察からの依頼で、膨大なデータを手に入れました。この1週間にかけられたすべての電話のリストがあり、フィッシングコールの発信元を見つける必要があります。

簡単なことではありませんが、あなたなら挑戦してくれるはずです！データを分析し、発信元を特定するためのパターンや手がかりを見つけるために、探偵のスキルを使ってほしい。

その情報があれば、警察が動いて、詐欺師を一掃することができるのです！あなたは、このチャレンジに挑戦する準備はできていますか？

私たちはあなたを応援していますし、あなたならできると信じています！フィッシャーマンを捕まえましょう！

よろしくお願いします、
サミュエル・インプソン船長
```

```
PhoneCalls
| count
```
<img width="221" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/97335d0e-36fb-4d45-b26a-b95188e979dc">

```
PhoneCalls
| take 10 
```
<img width="753" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/d7476c99-3e91-4ae1-b203-cc07d54c9017">


```
PhoneCalls
| extend from_num = toreal(Properties.Origin)
| where Properties.IsHidden == true
| summarize by from_num
| count
```
<img width="434" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/4d1d83f8-14d0-4c69-b91d-a9625e603a70">

```
PhoneCalls
| where CallConnectionId == "d536b721-afd3-45ef-9d46-61449fa74766"
```
<img width="747" alt="image" src="https://github.com/taokawarai/kusto-detective-agency/assets/35896206/f037114c-bc55-42bb-8308-39fbe6c5e891">

怪しそうなプロパティは`IsHidden`と`DisconnectedBy`。あとは通話時間？

```
PhoneCalls
| count 
```
`16480067`
```
Connect: 8240064
Disconnect: 8240003
```

```
PhoneCalls
| where EventType == "Connect"
| join kind=inner 
(
    PhoneCalls
    | where EventType == "Disconnect"
) on CallConnectionId
| where Properties1.DisconnectedBy == "Destination"
| take 10
```

