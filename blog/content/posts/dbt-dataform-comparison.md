---
title: "dbtとDataformを比較し、dbtを使うことにした"
date: 2021-02-10T10:40:39+09:00
draft: true
description: "データ品質の担保としてdbtとDataformを比較し、dbtを選択しました。"
tags: ["tech","data"]

---

最近、業務でDWH / Datamartの整備やデータ品質の担保を効率的に行いたくなる事情があり、調査したところdbtとDataformがツールとして有力そうだったので、比較してみました。

# TL;DR

* dbtは機能が充実しており、カスタマイズするポイントも多く、様々な用途に向く反面、理解し使いこなすための学習コストがかかります。
* DataformはWebビューによる開発体験が非常に良いです。機能もほとんどはわかりやすく、迷うことも少ないです。一方、dbtに比較して融通はききづらいです。
* とはいえ、どちらも十分な機能は備えている素晴らしいツールだと感じるので、どちらが良いかは導入するツールに求められるものや組織の置かれた状況次第でしょう。
* 私の所属する会社 (Ubie, inc) では、学習コストよりも機能の充実度を高く評価し、dbtを選択しました。

[TOC]

# dbt, Dataformについて簡単に紹介

どちらも、ETL / ELTにおけるT (Transformation) を担当するツールです。

つまりは、様々なデータソースからデータレイクにロードされたデータを、ビジネスやプロダクトが使いやすい形に変換する部分を担います。

## [dbt](https://docs.getdbt.com/docs/introduction)

[fishtown analytics](https://www.fishtownanalytics.com/) という組織により、2016年から開発されています。dbtとは、 `data build tool` の略です。

CLIとして利用するOSS版と、dbt Cloudという統合開発環境＋実行環境が付属した商用版があります。CLIの場合、スケジューラーや実行環境がないため、cronやAirflowなどとともに利用することになるでしょう。

## [Dataform](https://docs.dataform.co/)

2020年12月に、[Googleによる買収が発表され](https://cloud.google.com/blog/products/data-analytics/welcoming-dataform-to-bigquery)、界隈では非常に話題になりました。いつからサービスを開始しているのかわかりませんが、私自身は存在すらしりませんでした。

基本的には、統合開発環境と実行環境をセットで提供しています。CLIやREST APIもありますが、基本的にはGUIを利用します。

※ この記事ではCLI版のdbtのみ扱います。

# 比較

いくつかの候補軸で比較をしていこうと思います

## 対応するプラットフォーム

* dbt
  * Postgres, RedShift, BigQuery, Snowflake, Apache Spark, Databricks, Presto, (その他、サードパーティ製のコネクタあり)
* Dataform
  * BigQuery, Snowflake, Redshift, Azure SQL DW, Postgres

どちらも必要十分なカバーはできている印象です。dbtはOSSであるため、独自のコネクタを開発することが可能です。

## 主要な機能

* dbt, Dataform共通
  * データモデルの定義
  * データソースの定義
  * スナップショットの作成
  * データ品質テストの定義
  * モデル間依存関係の自動的な解決
  * プラットフォームにデータ定義（カラムディスクリプションなど）を反映
  * githubでのバージョンコントロール
* dbtのみ
  * データソースの鮮度チェック
  * マクロによる拡張コード
  * ドキュメントの自動生成
  * 分析用SQLの管理
  * 実行前後のHook
* Dataformのみ
  * 付属のスケジューラと実行環境
  * ウェブ上の統合開発環境と管理画面
  * Javascriptによる拡張コード

dbtのほうが幅広い機能群を持っています。特に、[dbtで管理しないデータソースに対しても品質チェックできる](https://docs.getdbt.com/docs/building-a-dbt-project/using-sources#snapshotting-source-data-freshness)のが便利です。また、メタデータやリネージなどが記載されたドキュメントを自動生成することも可能です。（メタデータをどこで管理するか、という別の問題がありそうですが）

![dbt-doc](/images/dbt-dataform-comparison/dbt-doc.png)

![dbt-lineage](/images/dbt-dataform-comparison/dbt-lineage.png)

ただ、Dataformでも必要十分な機能は備えている印象です。

### DataformのWeb画面は便利

DataformはなんといってもWeb画面が非常に使いやすいです。以下のような良い点があります。

* コードの補完
* 開発しているSQLのアドホックな実行
* 品質テストに失敗した場合、どの行で失敗したかをデータベースに格納してくれるので、調査も非常にしやすい
* ソースのバージョンコントロール
* スケジュールの設定
* 実行履歴の閲覧やリラン

![Dataform web view](/images/dbt-dataform-comparison/dataform_web_view.png)

## 外部ツールとの接続性

* dbt
  * [Hook](https://docs.getdbt.com/docs/building-a-dbt-project/hooks-operations)により、前後に任意の処理を挟むことが可能
  * [airflowに専用のOpeartor](https://pypi.org/project/airflow-dbt/)がある
  * manifest.jsonなどの実行情報を活用することで、様々な活用が可能
    * 例: [dbtのDAGをairflowのDAGとして変換する](https://www.astronomer.io/blog/airflow-dbt-1)
* Dataform
  * [CLI](https://docs.dataform.co/dataform-cli)があるが、特定の環境でジョブを実行することはできなさそう（開発用？）
  * [REST API](https://docs.dataform.co/dataform-web/api/reference)があり、外部から起動することは可能。こちらはCLIとは違い、環境指定が可能です。
  * Dataformの実行後に何か任意のコマンドを実行する、ということは現状できなさそう

外部との接続性は、dbtとDataformで最も大きな違いがあると感じた点でした。

dbtはCLIということもあり非常にモジュラリティが高く、様々なツールとの組み合わせが考えられます。また、[manifest.jsonなど](https://docs.getdbt.com/reference/artifacts/dbt-artifacts)を活用することで、例えば実行情報のビュアーを開発したりすることもできます。

一方、Dataformは単体で閉じられているという印象です。Dataformだけ使えば非常に効率よく実装と管理ができますが、他ツールと組み合わせたり、外部のイベントと依存関係を組みたい場合は制約が多いように感じました。

## 運用時のあれこれ

### 環境の分離

#### dbt

接続情報など環境固有の設定値は、[profiles.yml](https://docs.getdbt.com/reference/warehouse-profiles/bigquery-profile)に環境ごとの接続情報を記載し、

```yaml
adhoc:
  target: dev
  outputs:
    dev:
      type: bigquery
			...
	  prod:
	  	type: bigquery
	  	...
```

実行時にオプションで制御可能です。

```bash
dbt run -t prod
```

実行環境の分離については、CLI版は利用する実行環境次第です。airflowなど慣れたツールを使えば環境分離で困ることはないと思います。

#### Dataform

[environment.json](https://docs.dataform.co/dataform-web/scheduling/environments#configuring-environments)に環境を記載します。設定は生jsonを書かずにwebで簡単に作れます。

![dataform-env-config](/images/dbt-dataform-comparison/dataform-env-config.png)

Dataformで手動実行する場合は環境を明示的に変更する必要があるのですが、UIが少々わかりづらいです。

![dataform-env](/images/dbt-dataform-comparison/dataform-env.png)

些細なことですが、うっかり忘れて別の環境で実行してしまった、ということはあり得そうだなと感じました。また、実行履歴の画面でも環境を選択する必要があります。

### 実行履歴管理とリラン

#### dbt

実行履歴をどう管理するかは、実行環境によります。

リランについては、model selection syntaxを利用し特定のジョブを実行することができます。これは非常に便利で、tagとselectorを駆使すれば様々な条件を管理できます。

一方、airflowのOperatorを利用する場合、dbtの構築するDAGがairflowのDAGとして表現されないため、特定のジョブのみリランするという作業が難しいです。具体的には以下のようなDAGになってしまうため、`dbt Run` タスクを実行すると、dbtで構築したすべてのDAGが再実行されてしまいます。処理の実行コストが高い場合は注意が必要です。

実行履歴についても、ログには出力されますが、airflowの画面やアラートには反映されないためやや調査のコストがあります。

ただ、[こちらのブログ](https://www.astronomer.io/blog/airflow-dbt-1)にあるように、manifest.jsonを使ってdbtのDAGをairflowのDAGとして表現する実装をすることは可能です。

#### Dataform

画面に実行履歴が残ります。DAGの表示もあり、パッとみのわかりやすさがあります。

一方、dbtと同じく特定のタスクのみを再実行したいとなった場合、1) エディタの画面まで遷移し、2) 正しい環境を選択し、3) 実行する、というステップになり、やや煩雑です。また、これは新しいジョブの起動であって、リランとして記録されるわけではありません。

## 両者のPro/Conまとめ

ここまでの比較も踏まえて、ざっくりと特徴を比較するとこのようになりそうです。

### dbt

- Pro
  - 機能充実度の高さ
  - 他ツールとの接続性、拡張性の高さ
  - コミュニティの大きさ
- Con
  - 学習コストの高さ
  - schedulerや実行環境は自前で準備
  - より便利に使うには、ある程度の追加開発が必要

### Dataform

- Pro
  - Web viewによる開発効率の高さ
  - 学習コストの低さ
  - (GCPを使っている場合) Googleに組み込まれたことによる期待
- Con
  - 環境分離のわかりづらさ
  - 他ツールとの接続性の悪さ、拡張性の低さ

# 私たちの選択

## どちらを使うべきなのか？

どちらも、DWH / Datamartを管理するためのツールとして十分な機能を有していると思います。

それを踏まえた上で、どちらを選択すべきか？という問いには以下のように答えられると感じました。

（これは単純化した区分けなので、ツールに求める要求や組織の状況に応じて柔軟に判断すべきです）

* dbtが適していそうなケース
  * データエンジニアリングに成熟した開発チームを有しており、ツールの学習や管理を妥当なコストで行うことができる
  * データ基盤の複雑性や不確実性が高く、今後様々なユースケースへの対応が求められる可能性がある
  * BigQuery以外でデータ基盤を構築している
* Dataformが適していそうなケース
  * アナリストなど、エンジニアリングが得意でないメンバーが中心となり管理・利用する
  * データ基盤の複雑性や不確実性が低く、他ツールとの接続や、今以上のツール拡張を必要としない
  * BigQueryを中心にデータ基盤を構築している

## 選ばれたのは、dbtでした

dbtを選択しました。背景や理由は以下です。

### 背景

前提として、私自身は以下のような環境に置かれています。

* toB / toCの自社ウェブサービスを運営する企業 (Ubie, inc)
* データ活用が根幹となるサービスであり、プリミティブなデータ基盤はBigQuery上に存在
* データの活用方法は、ML、カスタマーサクセス、サービスの利用状況やアクション分析、など多岐に渡ります。
* GCPを全社的に利用
* 安定したエンジニアリング組織を持つ。データ基盤周辺を担当するデータエンジニアも数名いる。筆者はデータエンジニア。

データの利用用途やニーズがひしひしと増しており、それに伴ってデータ品質の担保も重要な課題となっていました。

### 「機能充実度 / 拡張性」 > 「学習コスト」

こういった背景を踏まえると、**「機能充実度」**や**「拡張性」**が、**「学習コスト」**よりも大事であると考えました。

* 事業におけるデータ利用の重要性が高く、多様なニーズが生じているため、できるだけ柔軟な選択肢をとりたい
  * ツール間のスイッチングコストが高そうなので、あとで機能が足りなくて乗り換えるということは極力やりたくない
* 現在のジョブ管理はCloud Composerを利用しており、親和性が高い
* 初期の学習コストは十分短い期間で支払い切れる自信
  * ドキュメントがよく整備されていること、コミュニティも発展していること、OSSでありコードが公開されていることから、わからないことがあっても十分自力で調べられそうと感じました。
  * dbtの運用経験を持つメンバーのジョインが決まっており、よくある罠は事前に回避可能

# まとめ

以上、dbt / Dataformの比較と、私たちの選択を例として紹介しました。

2020年末のDataformの登場により、この分野への注目は強くなっていると感じます。双方ともすでに素晴らしいツールですが、今後も、ツール・コミュニティともにさらなる発展を期待します。

