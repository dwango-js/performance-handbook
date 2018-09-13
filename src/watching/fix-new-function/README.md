# 毎回関数を作ってpropsに設定する問題の修正

Reactのよくある問題として、propsに関数を渡すパターンで毎回関数を作って渡してしまうが知られています。
毎回新しい関数を作ってしまうと、同じ異なる値（props）となるため無駄にコンポーネントが更新されてしまいます。

[why-did-you-update](https://github.com/maicki/why-did-you-update "why-did-you-update")でもこの問題を検知できます。

この問題は、関数同士の比較は常にtrueとしてしまう`shouldComponentUpdate`を実装するか、一度作った関数をちゃんと使いまわす（constructorで作成やpublic fieldを使う）ことで解決できます。

また、ESLintやTSLintといったLintツールでも検知できます。

- [tslint: jsx-no-lambda](https://github.com/palantir/tslint-react)
- [eslint: jsx-no-bind](https://github.com/yannickcr/eslint-plugin-react/blob/master/docs/rules/jsx-no-bind.md)

## 参考

- [React.jsのrenderの戻り値の中で.bindで新しい関数を定義してはいけないわけ - Qiita](https://qiita.com/shuntksh/items/fd81ca9aa31ea8f962e2 "React.jsのrenderの戻り値の中で.bindで新しい関数を定義してはいけないわけ - Qiita")

## 修正

視聴ページにもいくつかのコンポーネントで同じような問題がありました。
`tslint-react`を追加し、それらの問題を一個づつ修正しました。

## 計測

[why-did-you-update](https://github.com/maicki/why-did-you-update "why-did-you-update")を使って該当のコンポーネントの操作を行ったときに、無駄な更新ログがでないことが確認できました。
