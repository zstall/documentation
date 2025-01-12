---
kind: documentation
private: true
title: ダイナミックインスツルメンテーションの仕組み
---


## 概要

ダイナミックインスツルメンテーションは、サードパーティライブラリなど、アプリケーションのコードの任意の位置にプローブを追加して、実稼働システムに適用することができます。ダイナミックインスツルメンテーションは、オーバーヘッドが少なく、システムに対する副作用がないことが保証されています。

## プローブタイプ

*ログプローブ*と*メトリクスプローブ*を作成することができます。

### ログプローブ

ログプローブは、指定された環境とバージョンに一致するすべてのサービスインスタンスでデフォルトで有効になっています。また、各サービスインスタンスで 1 秒間に最大 5000 回までしか実行できないようにレート制限されています。

ログプローブで `Capture method parameters and local variables` を有効にすると、ダイナミックインスツルメンテーションは次のデータをキャプチャして、ログイベントに追加します。
  - **メソッド引数**、*ローカル変数**、*フィールド**。デフォルトで以下の制限があります。
    - 3 段階の深さのリファレンスをフォローします (UI で構成可能)。
    - コレクション内の最初の 100 項目。
    - 文字列値で最初の 255 文字。
    - オブジェクト内の 20 個のフィールド。静的フィールドは収集されません。
  - **スタックトレース**を呼び出します。
  - 捕捉される場合と捕捉されない場合の**例外**。

余分なデータのキャプチャはパフォーマンスに影響するため、デフォルトでは、指定した環境とバージョンに一致するサービスの 1 つのインスタンスでのみ有効になっています。このキャプチャ設定を有効にしたプローブは、1 秒間に 1 回実行されるようにレートが制限されます。

すべてのログプローブでログメッセージテンプレートを設定する必要があります。テンプレートは、中括弧内の式の埋め込みをサポートします。例: `User {user.id} purchased {count(products)} products`

[式言語](#expression-language)を使用して、ログプローブに条件を設定することもできます。式はブール値で評価する必要があります。式が true の場合、プローブは実行され、式が false の場合、データはキャプチャまたは出力されません。

**注**: キャプチャの上限は構成可能で、ダイナミックインスツルメンテーションのベータ版であるため、変更される可能性があります。

### メトリクスプローブ

メトリクスプローブは、指定された環境とバージョンに一致するすべてのサービスインスタンスでデフォルトで有効になっています。メトリクスプローブはレート制限を受けず、メソッドまたは行が呼び出されるたびに実行されます。

ダイナミックインスツルメンテーションメトリクスプローブは、以下のメトリクスタイプをサポートしています。

- [**カウント**][1]: 指定されたメソッドや行が何回実行されたかを数えます。カウントを増加させるために変数の値を使用するために[メトリクス式](#expression-language)と組み合わせることができます。
- [**ゲージ**][2]: 変数の最後の値に基づいてゲージを生成します。このメトリクスは[メトリクス式](#expression-language)を必要とします。
- [**ヒストグラム**][3]: 変数の統計的分布を生成します。このメトリクスは[メトリクス式](#expression-language)を必要とします。


#### 式言語

ログメッセージテンプレート、メトリクス式、およびプローブ条件でダイナミックインスツルメンテーション式言語を使用します。

例えば、`count(myCollection)` をメトリクス式として使用すると、コレクションのサイズからヒストグラムを作成することができます。メトリクス式は数値として評価されなければなりません。

ログテンプレートでは、式はテンプレートの静的な部分とブラケットで区切られます。例えば、`User name {user.name}` です。ログテンプレート式は任意の値で評価することができます。もし、式の評価に失敗した場合は、`UNDEFINED` に置き換えられます。

プローブ条件はブール値で評価されなければなりません。例: `startsWith(user.name, "abc")`、`len(str) > 20` または `a == b`

[1]: /ja/metrics/types/?tab=count#metric-types
[2]: /ja/metrics/types/?tab=gauge#metric-types
[3]: /ja/metrics/types/?tab=histogram#metric-types