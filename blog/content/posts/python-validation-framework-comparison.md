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

| ライブラリ                                                   | バージョン      | Github Star (2020/1/5) | 概要                                                         |
| ------------------------------------------------------------ | --------------- | ---------------------- | ------------------------------------------------------------ |
| [pydantic](https://pydantic-docs.helpmanual.io)              | 1.7.3           | 5.0k                   | 今回の比較対象の中では最も新しい。FastAPIに組み込まれている。 |
| [marshmallow](https://marshmallow.readthedocs.io/en/stable/) | 3.10.0          | 5.2k                   | 最も人気。FlaskやSQLAlchemyといった人気ライブラリとのインテグレーションもある。 |
| [attrs](https://www.attrs.org/en/stable/) (+ [cattrs](https://github.com/Tinche/cattrs)) | 20.3.0, (1.1.2) | 3.3k                   | Pythonクラスを簡単に定義するために開発されたもの。標準のdataclassで["Why not just use attrs ?"](https://www.python.org/dev/peps/pep-0557/#why-not-just-use-attrs)と注釈されている。cattrsは、attrsにシリアライズ・デシリアライズを可能にするライブラリ。今回は併用する前提で比較。 |
| [cerberus](http://docs.python-cerberus.org)                  | 1.3.2           | 2.3k                   | [eve](https://docs.python-eve.org/en/stable/)というFlaskベースのフレームワークで採用されている。eve自体を利用している例は見たことがないが、cerberus単体では利用されている印象。 |



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

* pydanticは高機能
* cerberusは唯一、dictでスキーマ定義をする
* attrsは複雑な用途には不向き
* marshmallowも高機能だが、スキーマクラスをデータオブジェクトとして利用できない点がpydantic / attrsとの差。適切なデータオブジェクトへの変換はユーザーが責任を持つ。
  * https://marshmallow.readthedocs.io/en/stable/quickstart.html#deserializing-objects-loading



## バリデーション定義に関する機能

|                                                        | pydantic | marshmallow | attrs | cerberus |
| ------------------------------------------------------ | -------- | ----------- | ----- | -------- |
| 型バリデーション                                       |          |             |       |          |
| Union型バリデーション                                  |          |             |       |          |
| min/maxバリデーション                                  |          |             |       |          |
| lengthバリデーション                                   |          |             |       |          |
| 正規表現バリデーション                                 |          |             |       |          |
| oneOfバリデーション                                    |          |             |       |          |
| カスタムバリデーション                                 |          |             |       |          |
| 複数フィールドにまたがるバリデーション                 |          |             |       |          |
| エラーメッセージのカスタマイズ                         |          |             |       |          |
| 未定義のフィールドの扱い（無視・エラー選択）           |          |             |       |          |
| バリデーションタイミングの選択（生成時に無視できるか） |          |             |       |          |



* シリアライズ
  * 形式
    * json string, xml, pickle, dict...
  * 変換
    * エイリアス
    * 除外
    * カスタムシリアライザ, json_encoder
* デシリアライズ
  * ソース形式
    * json, ...
  * クラスで定義したスキーマのインスタンスとしてデシリアライズできるか？
  * 環境変数
  * エイリアス
  * データ変換の手法
    * attrは型変換がなくめんどそう
* その他
  * イミュータブル
  * OpenAPI
  * コードジェネレータ
  * 別ライブラリとの統合
    * dataclass
    * ORMapper
  * 定性的な使い心地

# パフォーマンスの比較



# 結論



