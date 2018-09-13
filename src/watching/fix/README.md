# 修正例

実際の生放送の視聴ページに関する問題や修正についてをまとめて行きます。

多くの場合、次のような手順で「問題を発見して修正する」を繰り返しています。

1. 観測できる環境を作る
2. 観測する
3. 修正/計測方法を考える
4. 計測する
5. 修正する
6. 計測する

問題を修正するときにその修正の影響が全体にでることはすくないです。
そのため、基本的なマイクロな計測方法を考えて、修正時の前後の結果を提示します。

## Note: 再現性

問題の再現性を見つけることはとても重要です。
再現性を見つけて初めてちゃんと観測できるようになります。

問題を見つけたときに、次のものを取っておくと修正に役立ちます。

- 再現手順
- スクリーンショット または 動画
- [Timeline ツール](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool?hl=ja "Timeline ツール")のプロファイル

観測した時点で修正方法や問題の原因がわからなくても、[Timeline ツール](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool?hl=ja "Timeline ツール")のプロファイルを取っておくと後から確認できるため役立ちます。
（プロファイルにはスクリーンショットも含まれるため描画の問題も再現できます）

生放送の問題を見つけるときは、できるだけプロファイルを取ることを意識して問題を発見していました。
修正後に前後のプロファイル見比べることでも、計測の参考になります。
記録したプロファイルをは適当なリポジトリを作成し、そこに状況と一緒に保存しておくようにしていました。

プロファイルには`performance.mark`や`performance.measure`の記録も残ります。
今時のフレームワークならこれらのAPIを使ったパフォーマンス計測もできるため、合わせて記録しておくことを推奨します。

- React: [Profiling Components with the Chrome Performance Tab](https://reactjs.org/docs/optimizing-performance.html#profiling-components-with-the-chrome-performance-tab "Profiling Components with the Chrome Performance Tab")
