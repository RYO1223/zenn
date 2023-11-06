---
title: "Flutterでよく使われるアーキテクチャやコアライブラリの選択肢【個人的まとめ】"
emoji: "😻"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Flutter", "Dart"]
published: false
---

開発の規模によって使用するアーキテクチャやライブラリが変わると思います。
そこで、今まで遭遇した様々なライブラリの中で、複数の選択肢がある場合の選択肢をメモしておきます。実質これ一択というものは省いています。

「ここ間違ってない？」や「こういうのもあるよ！」などはガンガン指摘してください！勉強にさせてもらいます 😆

## アーキテクチャ

個人的に VVMR でいいと思う。
シンプルなので改造もしやすい。必要であれば CleanArchitecture などのもっと細かい分け方をすればいいと思う。
また、これ以降は VVMR を利用しているという前提で話を進めます。

### DDD (Domain-Driven Design)

Eric Evans という方によって提唱されたアーキテクチャ。
ドメインという物事の関心とそれ以外を分けること。
DDD では主に以下の 4 つに分ける。必ず上から下に依存しなければいけない。

- User Interface
- Application
- Domain
- Infrastructure

参考

https://www.infoq.com/jp/minibooks/domain-driven-design-quickly/

### View-ViewModel-Repository

元になっているのは[wasabeef](https://github.com/wasabeef)さんの[flutter-architecture-blueprint](https://github.com/wasabeef/flutter-architecture-blueprints#app-architecture)だと思う。

https://github.com/wasabeef/flutter-architecture-blueprints#app-architecture

DDD の上から下に依存するという特徴を踏まえながら、Application レイヤをなくしたものという認識。
個人的に一番好き
MVVM + Repository とも言われているが、モデルの扱い方が結構違う気がする+後述の MVVM と比較するために VVMR と呼ぶ。
この時、ViewModel は Repository のインターフェイスに依存している。そのため、環境によって実装を切り替えられる点が優れている。
このアーキテクチャでは Model はただのオブジェクトであり、データを操作する機能は持っていない。

#### 実装例

上の blueprint の実装をを参考にする。
例えばニュース一覧ページがあるとする。
[newsPage](https://github.com/wasabeef/flutter-architecture-blueprints/blob/a4d87a6d0a7f1d249b89795b567fd6325ac34d51/lib/ui/news/news_page.dart#L11)(View)では画面のレイアウトを定義する。記事の内容はまだ持っていない。
そのため、[newsViewModel](https://github.com/wasabeef/flutter-architecture-blueprints/blob/a4d87a6d0a7f1d249b89795b567fd6325ac34d51/lib/ui/news/news_page.dart#L18)に記事一覧が欲しいと依頼する。
newsViewModel は[NewsRepository](https://github.com/wasabeef/flutter-architecture-blueprints/blob/a4d87a6d0a7f1d249b89795b567fd6325ac34d51/lib/ui/news/news_view_model.dart#L16)から情報を取得しようとする。
(ここで、NewsRepository は抽象クラスとして定義されているため、ネットワークの状況やテスト環境に応じて特定の実装を DI することで、簡単に環境を切り替えることができる。)
その後、viewModel は repository から取得したデータを viewModel 内に保存する。
view では viewModel の変更を監視しているため、変更があれば画面を再描画する。

### MVVM (Model-View-ViewModel)

この記事では、オリジナルの[MVVM パターン](https://learn.microsoft.com/ja-jp/dotnet/architecture/maui/mvvm)のことを指す。
モデルが直接データを操作する点が VVMR と違う。
バックエンドにデータを置いていることが多く、外部アクセスを Model から分けたいので使ったことはない。
Model は Ruby on Rails の[Active Record](https://railsguides.jp/active_record_basics.html)のイメージ。

### Clean Architecture

よく見るクリーンアーキテクチャーの構成図のもの。
DDD をより細かく分けたものという認識。

実務で使ったことがないので詳しくはわからない。

参考

https://zenn.dev/koudai/articles/8ee15d10008f55

## View のステート管理とライフサイクル

一長一短なので、好みで使うといいと思う。
個人的には React っぽく描ける flutter_hooks の方が好き。

### StatefulWidget (Flutter オリジナル)

[こちらの例](https://docs.flutter.dev/ui/interactivity)にあるように StatefulWidget は State を持っています。
State 内のインスタンス変数として値を保持します。このインスタンス変数は StatefulWidget が破棄されるまで値を保持することができます。
setState()を呼び出すと Flutter は build メソッドを再度呼び出して、画面を再描画します。

また、ライフサイクルでは initState, dispose などのフェーズごとに処理を記述します。

### flutter_hooks

https://pub.dev/packages/flutter_hooks

こちらはステート管理と、ライフサイクルを React の hooks っぽく書くことができるようになるライブラリです。

アプリ内の Widget は StatefulWidget ではなく[HookWidget](https://pub.dev/documentation/flutter_hooks/latest/flutter_hooks/HookWidget-class.html) を継承するようにします。
HookWidget は StatefulWidget の State とライフサイクルを除いて Hooks の機能を追加したものです。

ステート管理には[useState](https://pub.dev/documentation/flutter_hooks/latest/flutter_hooks/useState.html)を使用します。初期値を与えて好きなタイミングで書き換えることができる点は State のインスタンス変数と同じです。
違うのは build 内で useState を定義している点です。インスタンス変数と違って、書き換えたい場所まで useState のインスタンスの ValueNotifier を持っていく必要があります。
どこから変更されるかが明確になるので、これはいい点だと思います。

ライフサイクルは、それぞれのコントローラーごとに記述されます。
Flutter で標準で入っているコントローラーなどはすでに実装されていてそのまま使うことができます。
例えば、[useAnimationController](https://pub.dev/documentation/flutter_hooks/latest/flutter_hooks/useAnimationController.html)の initState や dispose の処理は[ここ](https://github.com/rrousselGit/flutter_hooks/blob/cc0eab8fa0a63a822367a84a45fa855f5208a7f2/packages/flutter_hooks/lib/src/animation.dart#L105)に書かれています。
また、毎回忘れずに書かないといけない初期化・破棄処理を隠蔽することができると言うメリットもあります。

## ViewModel の実装

ViewModel にはステート管理と変更を View に通知する機能が必要になります。
よく使われているのは以下の 2 つだと思います。

個人的には Riverpod の方が好き。

### Riverpod

https://pub.dev/packages/riverpod

こちらは、上記の flutter_hooks と同じ作者によって作られている。

例えば、Todo というモデルがあり、TodosViewModel という全ての todo の管理と、view に通知する機能を持ったクラスを作りたいとする。
Riverpod だとこのように実装する。

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

  void addTodo(Todo todo) {
    state = [...state, todo];
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

これだけで、viewModel に期待される機能が実現できる。

### flutter_bloc

https://pub.dev/packages/flutter_bloc

使ったことがないので詳しくはわからない。
viewModel のメソッドを直接呼び出すのではなく、イベントを stream で viewModel に流すと、更新後の state が ViewModel から View に流れてくるイメージ。

参考

https://zenn.dev/heyhey1028/articles/56692d24493f63#%E6%BA%96%E5%82%99

## 画面遷移

### GetX

画面遷移以外にも色々なことができる。それゆえ色々なバグに遭遇することになった。絶対使わない。（戒め）
アーキテクチャが崩壊しやすい。

### Navigator (Flutter オリジナル)

https://api.flutter.dev/flutter/widgets/Navigator-class.html

基本的な画面遷移機能は大体ついている。

### go_router + go_router_builder

https://pub.dev/packages/go_router

https://pub.dev/packages/go_router_builder

Navigator に比べて優れている点は 2 つあります。

1. ネストした画面遷移を簡単に作れる。
1. 画面に必要な引数の型を指定することができるので、タイプセーフに使える。

## 端末内データ

### sqflite

https://pub.dev/packages/sqflite

SQL を使痛い場合

### shared_preferences

https://pub.dev/packages/shared_preferences

単純な key-value ストア

### flutter_secure_storage

https://pub.dev/packages/flutter_secure_storage

セキュアなデータを保存したい場合。
iOS では[キーチェーン](https://support.apple.com/ja-jp/guide/security/secb0694df1a/web)を使用する。こちらは同じデベロッパによる App のみアクセスできるようになっている。
