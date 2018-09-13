# トレードオフの検証

[紹介した修正例](../fix/README.md)では既存の挙動は維持したまま、目には見えない改善をしていました。
現実的にはそのアプリケーション自体の機能を減らさないと、低スペックのデバイスではそもそも無理な場合があります。

そのような機能の削減とパフォーマンスのトレードオフを検証する必要があります。

生放送の視聴ページではこのトレードオフを"軽量モード"というスタンスで進めることにしました。
軽量モードとは最終的にユーザが設定で選択して初めて機能を落とす、オプトイン方式の設定です。

## 軽量モード

色々な機能を削ってみて、何が効果的なのかを検証する必要があります。
機能ごとにステージングサイトにあげて検証というのは大変なので、簡単なURLで切り替えるできる仕組みを追加して検証しました。


実際に使った実装はものすごく単純です。
productionビルドでは必ず無効にするguardを入れておき、後はURLのクエリで切り替えできるという仕組みです。

```ts
const enableLightMode = process.env.NODE_ENV !== "production" && /[?&]leo_light_mode\b/.test(location.href);
// まとめて有効化
// &leo_light_mode=comment-panel-hidden,comment-panel-fixed-bottom,comment-fps,comment-resolution
// コメントパネルの非表示化
// &leo_light_mode=comment-panel-hidden
export const isHiddenCommentPanel = enableLightMode
    && /[?&]leo_light_mode=.*comment-panel-hidden\b/.test(location.href);
// コメントパネルの最下部への追従を停止
// &leo_light_mode=comment-panel-fixed-bottom
export const isDisabledCommentPanelFixedBottom = enableLightMode
    && /[?&]leo_light_mode=.*comment-panel-fixed-bottom\b/.test(location.href);
// コメントパネルのバッファサイズを指定
// &leo_light_mode=comment-panel-buffer-size:0
export const commentPanelBufferSizeMatch = enableLightMode
    && location.href.match(/[?&]leo_light_mode=.*comment-panel-buffer-size:([\d.]+)\b/);
export const commentPanelBufferSize = commentPanelBufferSizeMatch !== false && commentPanelBufferSizeMatch !== null
    ? parseFloat(commentPanelBufferSizeMatch[1])
    : undefined;
// 映像要素を非表示化
// &leo_light_mode=video-hidden
export const isHiddenEdgeStream = enableLightMode
    && /[?&]leo_light_mode=.*video-hidden\b/.test(location.href);
// コメントレンダラーのWebGLの有効化
// &leo_light_mode=comment-render-webgl
export const useWebGLCommentRenderer = enableLightMode
    && /[?&]leo_light_mode=.*comment-render-webgl\b/.test(location.href);
if (enableLightMode) {
    console.log(`
# 軽量モードのフラグ状態
コメントパネルの非表示化: ${isHiddenCommentPanel}
コメントパネルの最下部への追従を停止: ${isDisabledCommentPanelFixedBottom}
コメントパネルのバッファサイズ: ${commentPanelBufferSize}
映像要素を非表示化: ${isHiddenEdgeStream}
コメント描画のWebGL有効化: ${useWebGLCommentRenderer}
`);
}
```

その軽量モードのドキュメントは次のような単純なものでした。

```markdpwon
## 有効化の方法

URLに `&leo_light_mode=comment-panel-hidden,comment-panel-fixed-bottom,comment-fps,comment-resolution` などそれぞれの機能を無効化できるフラグを設定できる。複数渡す時は `,` など適当なもので区切る。

## 軽量モードの種類

- コメントパネルの非表示化
  - `&leo_light_mode=comment-panel-hidden`
- コメントパネルの- 最下部への追従を停止
  - `&leo_light_mode=comment-panel-fixed-bottom`
- コメントパネルのバッファサイズを指定
  - `&leo_light_mode=comment-panel-buffer-size:バッファサイズ`
  - 例) バッファサイズを`0`にする
  - `&leo_light_mode=comment-panel-buffer-size:0`
- 映像要素を非表示化
  - `&leo_light_mode=video-hidden`
- コメントレンダラーのWebGLの有効化
  - `&leo_light_mode=comment-render-webgl`
```

## 検証

後は、実際にURLにクエリをつけて何を外したら何がよくなるかというトレードオフを地道に検証していくだけです。

その後、"軽量モード"をオプションとして本当に実装するときにフラグを取り除けばよいだけです。
