---
title: "Pythonスキーマバリデーションライブラリ比較"
date: 2021-01-04T23:49:44+09:00
draft: true
description: ""
tags: ["python","tech"]
---

ウェブAPIの作成など、外部からやってくるデータを安全に捌く上でスキーマ定義とバリデーションは非常に重要です。また、特にPythonのような動的型付け言語において、内部でもレイヤをまたぐ場合はきちんと定義されたデータモデルを利用することで、知らない間にデータモデルが変わっていた、というようなケースを防ぐことができます。

Pythonには標準でスキーマバリデーションライブラリがないため3rdパーティのものを使うことになりますが、複数あるので比較してみます。

# 比較対象のライブラリ概要

※Pythonバージョンは3.9.0を利用します。

| lib&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp; | version         | Github Star <br> (2020/1/5) | memo                                                         |
| :----------------------------------------------------------- | --------------- | --------------------------- | ------------------------------------------------------------ |
| [pydantic](https://pydantic-docs.helpmanual.io)              | 1.7.3           | 5.0k                        | 今回の比較対象の中では最も新しい。 <br> FastAPIに組み込まれている。 |
| [marshmallow](https://marshmallow.readthedocs.io/en/stable/) | 3.10.0          | 5.2k                        | 最も人気。FlaskやSQLAlchemyといった人気ライブラリ <br> とのインテグレーションもある。 |
| [attrs](https://www.attrs.org/en/stable/) <br>(+ [cattrs](https://github.com/Tinche/cattrs)) | 20.3.0, (1.1.2) | 3.3k                        | Pythonクラスを簡単に定義するために開発されたもの。<br>標準のdataclassで["Why not just use attrs ?"](https://www.python.org/dev/peps/pep-0557/#why-not-just-use-attrs)と注釈されている。<br>cattrsは、attrsにシリアライズ・デシリアライズを可能にするライブラリ。<br>今回は併用する前提で比較。 |
| [cerberus](http://docs.python-cerberus.org)                  | 1.3.2           | 2.3k                        | [eve](https://docs.python-eve.org/en/stable/)というFlaskベースのフレームワークで採用されている。<br>eve自体を利用している例は見たことがないが、<br>cerberus単体では利用されている印象。 |



### 比較対象の選定の基準

* スキーマ定義が可能なこと
* バリデーション定義が可能なこと
* ある程度アクティブなプロジェクトであること
* HTTPリクエストのパースなど、特定の用途のみではなく、幅広い用途に利用できること。
* デフォルト値の適用や型変換など、バリデーションだけでなく変換処理を持っていること
* 最大5つ

### 除外した

* [schematics](https://schematics.readthedocs.io/en/latest/)
  * Github Star数は2.4kとまぁまぁ多いですが、2018年12月を最後にコミットが途絶えており、PRも放置されている状態。特殊な機能もなく選定される理由はないので除外。
* [colander](https://docs.pylonsproject.org/projects/colander/en/latest/)
* [django-rest-framework](https://www.django-rest-framework.org/)
* [flask-restful](https://flask-restful.readthedocs.io/en/latest/index.html)
  * この3つはいずれもスキーマバリデーションが可能ですが、HTTPリクエストパースのために特定のフレームワークと共に利用されることを想定しているので除外
* [jsonschema](https://python-jsonschema.readthedocs.io/en/v3.2.0/)
  * デフォルト値の適用などの変換処理がないため除外

# 機能の比較

## スキーマ定義に関する機能

|                         | pydantic | marshmallow | attrs | cerberus |
| ----------------------- | -------- | ----------- | ----- | -------- |
| スキーマ定義の方法      | class    | class       | class | dict     |
| required指定            | Yes      | Yes         | Yes   | Yes      |
| nullable可否の指定      | Yes      | Yes         | Yes   | Yes      |
| defaultの指定           | Yes      | Yes         | Yes   | Yes      |
| default factoryの指定   | Yes      | Yes         | Yes   | Yes      |
| カスタム型              | Yes      | Yes         | Yes   | Yes      |
| 定義のネスト            | Yes      | Yes         | No    | Yes      |
| 定義の再利用 (継承など) | Yes      | Yes         | No    | Yes      |
| 定義の動的生成          | Yes      | No          | No    | Yes      |
| 型アノテーション        | Yes      | No          | Yes   | No       |

### コメント

* pydanticは高機能
* cerberusは唯一、dictでスキーマ定義をする
* attrsは複雑な用途には不向き
* marshmallowも高機能だが、スキーマクラスをデータオブジェクトとして利用できない点がpydantic / attrsとの差。適切なデータオブジェクトへの変換はユーザーが責任を持つ。
  * https://marshmallow.readthedocs.io/en/stable/quickstart.html#deserializing-objects-loading



## バリデーション定義に関する機能

|                                              | pydantic | marshmallow | attrs | cerberus |
| -------------------------------------------- | -------- | ----------- | ----- | -------- |
| 型バリデーション                             | Yes      | Yes         | Yes   | Yes      |
| Union型バリデーション                        | Yes      | No ※        | No    | Yes      |
| 範囲バリデーション（ge / le / gt / lt）      | Yes      | Yes         | No    | Yes      |
| lengthバリデーション                         | Yes      | Yes         | No    | Yes      |
| 正規表現バリデーション                       | Yes      | Yes         | Yes   | Yes      |
| oneOfバリデーション                          | Yes      | Yes         | No    | Yes      |
| カスタムバリデーション                       | Yes      | Yes         | Yes   | Yes      |
| 複数フィールドにまたがるバリデーション       | Yes      | Yes         | No    | No       |
| 未定義のフィールドの扱い（無視・エラー選択） | Yes      | Yes         | No    | Yes      |

### コメント

* marshmallowは[プラグイン](https://github.com/adamboche/python-marshmallow-union)を別途インストールすることでUnionにも対応できるようです
* attrsはバリデータの機能が弱いようです。



## シリアライズ・デシリアライズに関する機能

|                                    | pydantic | marshmallow | attrs | cerberus |
| ---------------------------------- | -------- | ----------- | ----- | -------- |
| model -> dict                      | Yes      | Yes         | Yes   | No       |
| dict -> model                      | Yes      | Yes         | Yes   | No       |
| ※model -> json文字列               | Yes      | No          | No    | No       |
| json文字列 -> ※model               | Yes      | No          | No    | No       |
| フィールド名のエイリアス           | Yes      | No          | No    | No       |
| フィールド除外（シリアライズ）     | Yes      | Yes         | No?   | No       |
| カスタムシリアライザ               | Yes      | Yes         | Yes   | No       |
| 環境変数読み込み（デシリアライズ） | Yes      | No          | No    | No       |

※ model <-> json文字列変換には、datetime, enum, ネストしたモデル型を持つモデルを対象とします。datetimeやEnumといった型を、特にコードを追加することなく設定などで文字列化可能かを検査しています。

### コメント

* marshmallowについて
  * Enum型への対応は [プラグイン](https://pypi.org/project/marshmallow-enum/)を使えば可能です。
  * datetime型をdictからモデルに変換する場合、文字列に直さないと変換できません。
  * [モデルデータを保持するクラスは、ライブラリを使ったスキーマとは別に自分で用意しなければいけない](https://marshmallow.readthedocs.io/en/stable/quickstart.html#deserializing-to-objects)ため、型定義が二重になる。
* attrsについて
  * `cattr` を使えば `datetime` にも対応できますが、追加のコードが必要です。
  * 使い慣れていないので、実はYesの項目もあるかもです。
* cerberusについて
  * そもそもモデルクラスのインスタンスに変換することをサポートしていません。
* pydantic
  * 環境変数が読み込めるのがユニークです。これにより、[設定管理にも利用](https://pydantic-docs.helpmanual.io/usage/settings/)できます。[dotenv](https://pypi.org/project/python-dotenv/)も利用できます。



## 総合的な使い心地について

個人的な視点も入ってしまいますが、使い心地についても評価してみようと思います。

|                        | pydantic   | marshmallow | attrs        | cerberus   |
| ---------------------- | ---------- | ----------- | ------------ | ---------- |
| 全体的な機能の充実度   | とても良い | とても良い  | イマイチ     | 普通       |
| スキーマの定義しやすさ | とても良い | 良い        | 良い         | イマイチ   |
| コード補完の効きやすさ | とても良い | 良い        | 良い         | イマイチ   |
| バリデーションの充実度 | とても良い | とても良い  | イマイチ     | 良い       |
| パフォーマンス ※       | 良い？     | 普通？      | とても良い？ | イマイチ？ |

### コメント

* フィールドに自然に型アノテーションをつけられるPydanticはやはり書きやすいです。[Pycharmプラグイン](https://pydantic-docs.helpmanual.io/pycharm_plugin/)も開発されているのも良いです。
* パフォーマンスについて
  * [Pydanticのパフォーマンス比較](https://pydantic-docs.helpmanual.io/benchmarks/)と、[こちらのブログ](https://stefan.sofa-rockers.org/2020/05/29/attrs-dataclasses-pydantic/)から。参考程度の比較になります。

# まとめ

少しPydanticびいきの比較になってしまったような気もしますが、概ね以下の通りです。

* pydantic
  * 機能の充実度と書きやすさが両立されており、まず最初に候補に入れるべきライブラリ。
  * Python3.6以上にしか対応していないので、古いシステムには使えない。
* marshmallow
  * Pydanticと同等に近い充実度。pydanticよりも歴史があるので、事例も多い。
  * スキーマからモデルクラスを作る場合、自分で用意しなきゃいけないのが面倒。
  * Flaskを使うのなら、Pydanticよりもこちらのほうが事例が多そう。[flask-marshmallow](https://flask-marshmallow.readthedocs.io/en/latest/)というライブラリもある。
* attrs
  * そもそもがバリデーションライブラリではなく、dataclassのようにクラスを書きやすくするライブラリなので、バリデーションについては弱め。
  * 機能の充実度は低いですが、パフォーマンスは良いので要件の厳しい部分では使えそう。
* cerberus
  * 今回の中では唯一、dictでスキーマを定義するタイプ。
  * jsonschemaっぽい独自構文で定義するのだが、コード補完も効かないので定義するのが大変。
  * あまりこれを使うシーンは思い浮かばないのが正直なところ。

