---
title: "Prometheus Intro"
date: 2019-03-20T18:26:03+09:00
outputs:  ["Reveal"]
draft: false
---
prometheus(プロメテウス)  
とgrafanaとか…

---
## prometheusってなんだよ

**Power your metrics and alerting with a leading**  
**open-source monitoring solution.**  
from 公式ページ  
(メトリクス監視しつつアラート発報もするおーぷんそーすなニクいやつ)

---
* SoundCloud社にて2012年から開発されてる
* 元google社員が、googleの監視システム「borgmon」にインスパイアされて作った
* **メトリクス収集方式としてPull型を採用している(zabbixなんかはPush型)**

---
## overview
![overview](https://prometheus.io/assets/architecture.svg)

---
Push型  
監視対象サービスが監視サーバへメトリクスを送信する

Pull型  
監視サーバから監視対象サービスへメトリクスを取りに行く

---
## つよみ
* golang製で単バイナリを動かすだけでいい
  - ここほんと強い。アップデートとか実行ファイル入れ替えて起動しなおせば終わり
* google謹製時系列DB(leveldb)を内蔵している為、稼動させるサーバ一台あればよい
* **Service Diccovery**でAWS, GCP, Kubernetesなど、今時のナウでヤングなクラウド環境に強い(後述)
* 開発が活発

---
## つらみ
* クラウド環境以外(オンプレミス)だと監視サーバ側にいちいち設定追加しないといけない
  - とはいえ設定監視対象の設定ファイルは外出しできる上、変更した場合は自動で追従してくれる
* 時系列DBがぼちぼち容量喰う
* 開発が活発
  - なのでアップデートが多い(けど別に安定してればしなくていい)
* PromQLの学習コスト

---
## Service Diccovery
AWSやGCPなどのクラウド環境にていちいち監視対象を設定ファイルに個別に追記しないで済む機能  
実際EC2が監視対象の場合、設定ファイルは以下だけでよい

```
# my global config
global:
  scrape_interval:     15s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'node'
    ec2_sd_configs:
      - region: <リージョン>
        access_key: <アクセスキー>
        secret_key: <シークレットキー>
        port: 9100
```

***
### グルーピングとかラベリングとか
ec2の場合、各インスタンスにタグをkey:value  
で貼れるので、そこを読んでprometeus上で扱う事が出来る  

---
### EC2のタグ機能を利用

---
```
relabel_configs:
  - source_labels: [__meta_ec2_tag_Stage]
    regex: (Stg|Prod)
    action: keep
      → 「SgageタグがStgかProdならそのままstageラベルとする」
  - source_labels: [__meta_ec2_tag_Name]
    target_label: name
      → 「Nameタグはラベル名nameとする」
  - source_labels: [__meta_ec2_tag_Stage]
    target_label: stage
  - source_labels: [__meta_ec2_tag_Role]
    target_label: role
```
***
ここで、各サーバの役割や環境を設定しておく事で可視化する際にタグ単位でメトリクスの抽出が出来る  

例: webサーバ(本番/STG問わず)のロードアベレージ
```
node_load5{stage!="Prod|Stg", role="web"}
```

＿人人人＿  
＞ 便利 ＜  
￣^Y^Y^Y￣

---
エージェントは？  
＿人人人人人人＿  
＞　exporter　＜  
￣Y^Y^Y^Y^Y^Y￣

---
## exporter
システムのメトリクスをとってくるのとかJMXのデータもってきたりとか  
いろんなexporterがあったりする
[exporter](https://prometheus.io/docs/instrumenting/exporters/)

とはいえ、基本的にリソース取得であればnode_exporterで事足りる

---
prometheusは、あくまでメトリクスの収集と保持を行なうだけ…

---
なので可視化ツール使おうぜ  

＿人人人人人＿  
＞　grafana　＜  
￣Y^Y^Y^Y^Y^Y￣

***
## grafana
kibanaからフォークした可視化専用フロントエンド  
Prometeusの他にもElasticsearchやinfluxDBにも対応している  
＿人人人人人＿  
＞ golang製 ＜  
￣^Y^Y^Y^Y^Y￣
