---
templateKey: blog-post
title: Auto Stop/Restart Cloud SQL/GCP with gcp-instance-scheduler
date: 2020-03-22T20:33:46.876Z
description: |
  By using future-architect/gcp-instance-scheduler to set Auto Stop/Restart
    Cloud SQL/GCP
featuredpost: true
featuredimage: /img/a8c20a9f125824d98bf344f7c3fabd72.jpg
tags:
  - gcp
---
# Architecture

Cloud Scheduler --> Pub/Sub --> CloudFunction

[gcp-instance-scheduler](https://cloud.google.com/scheduler/docs/start-and-stop-compute-engine-instances-on-a-schedule)

# 

Prerequisites Tools and APIs

## 

全てアクティブに設定

\* Cloud Functions API

\* Cloud Scheduler API

\* Compute Engine API

\* VM instances

\* Cloud Pub/Sub API

\* Cloud SQL Admin API

\* Kubernetes Engine API

\* Cloud Logging API

\* Admin SDK

\* [Cloud SDK](https://cloud.google.com/sdk/docs)

## Install Cloud SDK

1. [Setting](https://cloud.google.com/scheduler/docs/setup)
2. [Cloud SDK をインストールし、初期化](https://cloud.google.com/sdk/docs)
   v285.0.1
3. コンソールスクリーンが自動的に開く
4. ログインを許可する
5. GoogleAuthで許可したのち、自動的にブラウザが開き、認証完了のメッセージが確認できる。


```
Please enter numeric choice or text value (must exactly match list
item):プロジェクト名の番号を入力
```

6. すべてのコンポーネントを更新


```
gcloud components update
```

7. 選択したプロジェクトを使用するように gcloud を構成


```
gcloud config set project triptokitsukiapi
```

8. CLIから名前リスト詳細などの確認


```
//ProJetIDの確認：#triptokitsukiapigcloud projects list
```

```
//インスタンス名の確認：#triptokitsuki-mysqlgcloud sql databases list -i=triptokitsuki-mysql
```

```
//インスタンスの詳細確認gcloud sql instances describe triptokitsuki-mysql
```

## 

## Setting Cloud Scheduler

9. Cloud SQLにPanel>Labelからラベルを設定


```
Key:"state-scheduler", value:"true"
```

10. ファンクションの作成


```
//ダウンロードgit clone https://github.com/future-architect/gcp-instance-scheduler.git
```

```
//ディレクトリを移動cd gcp-instance-scheduler
```

```
//ファンクションの作成＃SLACKは使いませんgcloud functions deploy switchInstanceState --project triptokitsukiapi --entry-point SwitchInstanceState --runtime go111 --trigger-topic instance-scheduler-event --set-env-vars SLACK_ENABLE=false
```

```
//スケジュールの作成＃ローカルタイム、スケジュールタイム等はコンソールスクリーンで変更できるgcloud beta scheduler jobs create pubsub shutdown-workday --project triptokitsukiapi --schedule "0 11 * * *" --topic instance-scheduler-event --message-body '{"command":"stop"}' --time-zone "America/Chicago" --description "Yuris testing for AUCENT"
```

11. 確認作業

ファンクション

\* クラウドファンクションから確認

\* switchInstanceStateというファンクションができているはずなので、それをクリック

\* 「TESTING」タブをクリック

\* コンバーターなどを使ってBase64にしたパラメータを以下のように設定

\`\``

// {"command":"stop"}　➔　eyJjb21tYW5kIjoic3RvcCJ9{"data":"eyJjb21tYW5kIjoic3RvcCJ9"}

\`\``

\* Cloud SQL スクリーンへ正しくインスタンスが終了されているか確認

\*「Triggering Events」にパラメータ｛｝が見えないときにはCompute Engine>VM instancesにインスタンス名があることを確認してスタートさせる。その後パラメータが確認できる。

\*コンバーター　https://www.base64encode.org/

 Pub/Sub

\* PubSub>Topicにinstance-scheduler-eventがあることを確認

  Cloud Scheduler

shutdown-workdayが設定されている確認

shutdown-workdayをクリック、「Edit」をクリックして以下の確認

\* Description："文言"

\* Timezone："Asia/Tokyo"

\* Target："Pub/Sub"

\* Topic："instance-scheduler-event"

\* Payload：" '{command:stop}' "

\*\[Timezoneの設定](https://cloud.google.com/scheduler/docs/configuring/cron-job-schedules?&_ga=2.126554309.-97734128.1583107689&_gac=1.15193796.1584735989.Cj0KCQjw09HzBRDrARIsAG60GP9vWXwJSKzEVNcIeYnS5IX5dh-v3nYqXbiN8TGS_cj84VXQryifK40aAqx6EALw_wcB#defining_the_job_schedule)
