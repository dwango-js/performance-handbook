# 無駄のタイプ

現状の機能やUIを維持しながら、パフォーマンスを良くすることは無駄を削ることとほぼ同義です。

生放送の視聴ページに関する無駄を削っていくことがパフォーマンスの向上につながります。
その"無駄"には幾つかの種類があると考えられます。

## 無駄の種類

ブラウザにはI/Oなどはないため主に次の要素が挙げられます。

- ネットワーク
- レンダリング
- スクリプティング

これはブラウザの開発者ツールの[Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool?hl=ja "Timeline")や[Network](https://developers.google.com/web/tools/chrome-devtools/network-performance/resource-loading?hl=ja "Network")パネルなどでみれます。

これらについては次のサイトや書籍が参考になります。

### 参考

- [Chrome DevTools  |  Tools for Web Developers  |  Google Developers](https://developers.google.com/web/tools/chrome-devtools/ "Chrome DevTools  |  Tools for Web Developers  |  Google Developers")
- [開発ツール | MDN](https://developer.mozilla.org/ja/docs/Tools "開発ツール | MDN")
- [超速！ Webページ速度改善ガイド](http://gihyo.jp/book/2017/978-4-7741-9400-4 "超速！ Webページ速度改善ガイド")
- [Webフロントエンド ハイパフォーマンス チューニング](http://gihyo.jp/book/2017/978-4-7741-8967-3 "Webフロントエンド ハイパフォーマンス チューニング")
- [Chrome Developer Toolsでパフォーマンス計測・改善 - Qiita](https://qiita.com/y_fujieda/items/a0a69151cf7307039f74 "Chrome Developer Toolsでパフォーマンス計測・改善 - Qiita")

### ネットワークにおける無駄

不必要なものを読み込まないのがもっとも効率的です。
余計なリソースを読み込まないことはファーストビューの表示に直接反映されるため、ページロードを短くするならばネットワークの無駄を探すのがもっとも効率的な改善です。
[App Shell モデル](https://developers.google.com/web/fundamentals/architecture/app-shell?hl=ja "App Shell モデル")などはこの効率化をパターンしたものです。

- [クリティカル レンダリング パス  |  Web  |  Google Developers](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/?hl=ja "クリティカル レンダリング パス  |  Web  |  Google Developers")

今回は[目的](../README.md)で述べたように、視聴中におけるパフォーマンス改善なので省略します。

### レンダリングとスクリプティングにおける無駄

生放送という性質上、視聴中に無駄となるものレンダリングやスクリプティングが多くの割合を占めます。
なぜなら、生放送の動画などは先読みキャッシュなどができないため、ネットワークは必要悪となりがちです。（映像の最適化などは別途必要）

視聴ページでは[React](https://reactjs.org/ "React")が使われているので、reactでよくある問題を箇条書きします。

- 無駄な更新
  - これは同じ値にもかかわらず同じ結果に更新されているView
- 無駄な判定
  - これはshouldComponentUpdateの判定自体が不要なのにしなければならなくなっている場合
  - Parent > Child でParentの時点で受け取ったpropsが全く同じならChildではshouldComponentUpdateを判定自体しなくていい
  - ComponentのTree構造にも関係するので、トップダウンとボトムアップどちらも必要
  - トップダウン（上のコンポーネントで止める）と効果は高いが、相対的にトップでとまるものはあまり多くない
    - トップダウンで止めやすいようにDOM構造を変更するのも1つの手段
  - ボトムアップ（末端に近いコンポーネントで止める、結果的に止まればDOM APIを叩かない）で止めることは、効果は小さい範囲が広い
- 無駄なDOM API
  - `componentWillUpdate`や`componentDidUpdate`などでのDOM APIを叩く
  - 主にstyle操作やlayout操作に関係あるものをここでやるとコストが高い
  - たとえば`componentDidUpdate`で[element.getBoundingClientRect](https://developer.mozilla.org/ja/docs/Web/API/Element/getBoundingClientRect "element.getBoundingClientRect")を叩くなど
    - [CSS Triggers](https://csstriggers.com/ "CSS Triggers")に載っているものを操作するDOM APIを叩くときは注意が必要
    - 方針としては、"更新したからLayout情報を取る"ではなく"更新されたからLayout情報を取る"にする
    - IntersectionObserver や `scroll` イベントなどイベントを使ってLayout情報を更新を行う
    - Viewをrenderしたからと言ってそのタイミングでLayoutが更新されるとは限らないので、イベント駆動にして最小限にする（基本Layoutコストは高い）
- 毎回作られる関数のハンドラ
	- propsで渡すハンドラを毎回作ってしまう問題
	- propsの関数同士の比較が毎回`false`となってしまう（PureComponent）
	- shouldComponentUpdateの実装で関数同士は常にtrueという手もあり
	- [React.js pure render performance anti-pattern – Esa-Matti Suuronen – Medium](https://medium.com/@esamatti/react-js-pure-render-performance-anti-pattern-fb88c101332f "React.js pure render performance anti-pattern – Esa-Matti Suuronen – Medium")
	- 検知方法
	- TSLint: [tslint-react](https://github.com/palantir/tslint-react "tslint-react")の `jsx-no-bind` and `jsx-no-lambda
	- ESlint: [react/jsx-no-bind](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md "react/jsx-no-bind")
