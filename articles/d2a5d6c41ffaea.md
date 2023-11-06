---
title: "Flutterでよく使われるアーキテクチャやコアライブラリの選択肢【個人的まとめ】"
emoji: "😻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"]
published: false
---

開発の規模に応じて、使用するアーキテクチャやライブラリは変わります。今まで遭遇した様々なライブラリの中で、複数の選択肢があるものをメモしておきます。実質これ一択というものは省いています。

「ここ間違ってない？」や「こういうのもあるよ！」などの指摘は歓迎します！勉強になります 😆

## アーキテクチャ

個人的には VVMR がいいと思います。シンプルでカスタマイズしやすいため、必要に応じて Clean Architecture などの細かい分け方を採用すれば良いと考えています。以下、VVMR を利用している前提で話を進めます。

### DDD (Domain-Driven Design)

Eric Evans によって提唱されたアーキテクチャです。ドメインという物事の関心とそれ以外を分けることが特徴です。DDD では主に以下の 4 つに分けられ、必ず上から下に依存しなければなりません。

- User Interface
- Application
- Domain
- Infrastructure

**参考:**
https://www.infoq.com/jp/minibooks/domain-driven-design-quickly/

### View-ViewModel-Repository

元になっているのは[wasabeef](https://github.com/wasabeef)さんの flutter-architecture-blueprint です。

https://github.com/wasabeef/flutter-architecture-blueprints#app-architecture

DDD の「上から下に依存する」という特徴を踏まえつつ、Application レイヤーを省略したものです。
個人的にはこのアーキテクチャが一番好きです。

MVVM + Repository とも言われますが、モデルの扱い方に違いがあるため、この記事では VVMR と呼ぶことにします。
ViewModel は Repository のインターフェイスに依存し、環境によって実装を切り替えることが可能です。
Model は単なるオブジェクトであり、データ操作の機能は持ちません。

#### 実装例

wasabeef さんの blueprint を参考にします。例として、ニュース一覧ページを考えます。
[newsPage](https://github.com/wasabeef/flutter-architecture-blueprints/blob/a4d87a6d0a7f1d249b89795b567fd6325ac34d51/lib/ui/news/news_page.dart#L11)(View)ではレイアウトを定義し、[newsViewModel](https://github.com/wasabeef/flutter-architecture-blueprints/blob/a4d87a6d0a7f1d249b89795b567fd6325ac34d51/lib/ui/news/news_page.dart#L18)を通じて記事一覧を要求します。
newsViewModel は[NewsRepository](https://github.com/wasabeef/flutter-architecture-blueprints/blob/a4d87a6d0a7f1d249b89795b567fd6325ac34d51/lib/ui/news/news_view_model.dart#L16)から情報を取得し、viewModel 内でデータを保持します。view は viewModel の変更を監視し、変更があれば画面を再描画します。

### MVVM (Model-View-ViewModel)

ここではオリジナルの[MVVM パターン](https://learn.microsoft.com/ja-jp/dotnet/architecture/maui/mvvm)を指します。モデルが直接データを操作する点が VVMR と異なります。バックエンドにデータを置くことが多く、外部アクセスを Model から分けたい場合には使用しません。
Model は Ruby on Rails の[Active Record](https://railsguides.jp/active_record_basics.html)をイメージしてます。

### Clean Architecture

よく見るクリーンアーキテクチャの構成図に基づいています。DDD をより細かく分けたものです。実務での使用経験はないため、詳細は分かりません。

**参考:**
https://zenn.dev/koudai/articles/8ee15d10008f55

## View のステート管理とライフサイクル

一長一短があるため、好みに応じて選択すると良いと思います。
個人的には React 風に書ける flutter_hooks が好みです。

### StatefulWidget (Flutter オリジナル)

StatefulWidget は State を持ちます。State 内のインスタンス変数は StatefulWidget が破棄されるまで値を保持します。setState()を呼び出すと、Flutter は build メソッドを再度呼び出し、画面を再描画します。
また、ライフサイクルでは initState や dispose などのフェーズごとに処理を記述します。

**参考:**
https://docs.flutter.dev/ui/interactivity

### flutter_hooks

https://pub.dev/packages/flutter_hooks

flutter_hooks はステート管理とライフサイクルを React の hooks 風に書けるようにするライブラリです。Widget は StatefulWidget ではなく[HookWidget](https://pub.dev/documentation/flutter_hooks/latest/flutter_hooks/HookWidget-class.html)を継承します。
ステート管理には[useState](https://pub.dev/documentation/flutter_hooks/latest/flutter_hooks/useState.html)を使用し、ライフサイクルは各コントローラーごとに記述されます。

ライフサイクルは、それぞれのコントローラーごとに記述されます。Flutter で標準で入っているコントローラーなどはすでに実装されていてそのまま使うことができます。
例えば、[useAnimationController](https://pub.dev/documentation/flutter_hooks/latest/flutter_hooks/useAnimationController.html)の initState や dispose の処理は[ここ](https://github.com/rrousselGit/flutter_hooks/blob/cc0eab8fa0a63a822367a84a45fa855f5208a7f2/packages/flutter_hooks/lib/src/animation.dart#L105)に書かれています。
また、毎回忘れずに書かないといけない初期化・破棄処理を隠蔽することができると言うメリットもあります。

## ViewModel の実装

ViewModel にはステート管理と変更を View に通知する機能が必要です。よく使われるのは以下の 2 つです。

### Riverpod

https://pub.dev/packages/riverpod

Riverpod は flutter_hooks と同じ作者の[Remi 氏](https://github.com/rrousselGit)によるライブラリです。

例として、Todo モデルと TodosViewModel クラスを考えます。Riverpod を使用すると、viewModel に期待される機能を簡単に実現できます。

```
class Todo {
  Todo(this.description, this.isCompleted);
  final bool isCompleted;
  final String description;
}

@riverpod
class TodosViewModel extends _$TodosViewModel {
  @override
  List<Todo> build() {
    return [];
  }

  ...
}

class TodosView extends ConsumerWidget {
  @override
  Widget build(BuildContext context, WidgetRef ref) {
    final todos = ref.watch(todosViewModelProvider);

    ...
  }
}
```

### flutter_bloc

https://pub.dev/packages/flutter_bloc

flutter_bloc はイベントを stream で viewModel に流し、更新後の state が ViewModel から View に流れる仕組みです。個人的な使用経験はありません。

**参考:**
[flutter_bloc の基本](https://zenn.dev/heyhey1028/articles/56692d24493f63#%E6%BA%96%E5%82%99)

## 画面遷移

### GetX

GetX は画面遷移以外にも多くの機能を提供しますが、多機能ゆえのバグに遭遇することもあります。アーキテクチャが崩壊しやすいため、使用は推奨しません。

### Navigator (Flutter オリジナル)

https://api.flutter.dev/flutter/widgets/Navigator-class.html

基本的な画面遷移機能を提供します。

### go_router + go_router_builder

https://pub.dev/packages/go_router

https://pub.dev/packages/go_router_builder

Navigator に比べて優れている点は 2 点です。

1. ネストした画面遷移を簡単に作成できます。
1. 画面に必要な引数の型を指定できるため、タイプセーフに使用できます。

## 端末内データ

### sqflite

https://pub.dev/packages/sqflite

SQL を使用する場合に適しています。

### shared_preferences

https://pub.dev/packages/shared_preferences

単純な key-value ストアです。

### flutter_secure_storage

https://pub.dev/packages/flutter_secure_storage

セキュアなデータの保存に適しています。iOS では[キーチェーン](https://support.apple.com/ja-jp/guide/security/secb0694df1a/web)を使用します。
