# 1. React プロジェクト作成しとく

```bash
create-react-app your_react_project
```

# 2. package.json に追加

```bash
cd your_react_project
npm install shogo-hab/react-ue4-pixel-streaming --save
```

# 3. Example

```jsx App.js
import React from "react";

import ReactPixelStreaming from "react-ue4-pixel-streaming";
import { PixelStreamingContext } from "react-ue4-pixel-streaming";
import { PixelWindow } from "react-ue4-pixel-streaming";

function App() {
  return (
    <ReactPixelStreaming
      webRtcHost="localhost"
      pixelStreamingResponseEvents={[]}
    >
      <div>
        <PixelStreamingContext.Consumer>
          {context => (
            <div>
              <PixelWindow
                videoStyle={{ width: 1280, height: 720 }}
                windowStyle={{ width: 1300, height: 740 }}
                load={context.load}
                actions={context.actions}
                connect={context.connect}
                host={context.webRtcHost}
              />
              <YourComponent context={context} />
            </div>
          )}
        </PixelStreamingContext.Consumer>
      </div>
    </ReactPixelStreaming>
  );
}

const YourComponent = context => (
  <div>
    {context.context.webrtcState}
    <button
      disabled={context.context.webrtcState !== "connected"}
      onClick={() => {
        context.context
          .emitter(context.context.webRtcPlayerObj)
          .emitUIInteraction({
            Command: "foo",
            Value: "bar"
          });
      }}
    >
      My Component
    </button>
  </div>
);

export default App;
```

# Component Development

```bash
git clone https://github.com/shogo-hab/react-ue4-pixel-streaming.git
cd react-ue4-pixel-streaming
npm install

# examplesの起動
npm run start

# distに成果物を出力
npm run transpile
```

# 雑なメモ

## Components

- `ReactPixelStreaming`: WebRtc の接続管理や UE4 との疎通用のメソッド/プロパティを管理するHOC。
- `PixelWindow`: 実際に WebRTC の Video Stream がレンダリングされるコンポーネント。ReactPixelStreaming の子孫コンポーネントとしてcontext を受け取る必要がある。

## Attributes

### ReactPixelStreaming

`ReactPixelStreaming`コンポーネントは以下の Attribute を受け取る。

```jsx
<ReactPixelStreaming
  webRtcHost="${HOST}:${PORT}"
  pixelStreamingResponseEvents={[]}
>
```

- webRtcHost<Strings>: WebRTC のバックエンドを渡す。
- pixelStreamingResponseEvents<Array>: UE4 で responsePixelStreaming が発行された時の処理を渡す。

```jsx
 [{name: "handler", handler: function(response){
   // some action
 } ]
```

### PixelWindow

`PixelWindow`コンポーネントは context を介して以下の値を渡す。

```jsx
<PixelStreamingContext.Consumer>
  {context => {
    <PixelWindow
      load={context.load}
      actions={context.actions}
      connect={context.connect}
      host={context.webRtcHost}
    />;
  }}
</PixelStreamingContext.Consumer>
```

### CustomComponent と context

子孫コンポーネントは`PixelStreamingContext.Consumer`経由で`ReactPixelStreaming`のプロパティやメソッド等のコンテキストにアクセスできる。

#### 全く整理されてないプロパティやメソッドたち
- videoAspectRatio: Video Stream のアスペクト比
- videoRes: Video Stream の幅/高さ
  - width
  - haight
- responseEventListeners: 登録済みの pixelStreamingResponse イベントリスナー
- addResponseEventListener: pixelStreamingResponse イベントリスナーの登録
- removeResponseEventListener: pixelStreamingResponse イベントリスナーの削除
- emitter: WebRTC 経由でコマンドを送信する(UE4 で PixelStreamingInputEvent が発火する)
  - emitter(context.webRtcPlayerObj).emitUIInteraction
  - emitter(context.webRtcPlayerObj).emitMouseDown
  - emitter(context.webRtcPlayerObj).emitMouseUp
  - emitter(context.webRtcPlayerObj).emitMouseMove
  - emitter(context.webRtcPlayerObj).emitMouseWheel
  - emitter(context.webRtcPlayerObj).sendInputData
  - emitter(context.webRtcPlayerObj).emitUIInteraction
  - emitter(context.webRtcPlayerObj).emitCommand
  - emitter(context.webRtcPlayerObj).emitDescriptor
- controlScheme: マウスの制御オプション
- suppressBrowserKeys: ファンクションキーの制御オプション
- fakeMouseWithTouches: タッチスクリーンの制御オプション
- webrtcState: loading | disConnected | connecting | playing | stop
- webRtcPlayerObj: インスタンス課された webrtc オブジェクト
- webRtcPlayer: webrtc コンストラクタ
- connect:
- clientConfig: webrtc の接続情報
- socket: インスタンス化された ws オブジェクト
- aggregatedStats<Array>: webRtcPlayerObj が収集した webrtc 統計(interval: 1sec)
  - avgBitrate : 平均ビットレート(Kbps)
  - highBitrate: 最大ビットレート(Kbps)
  - bitrate: ビットレート(Kbps)
  - bytesReceived: 受信データ量(bytes)
  - framerate: FPS
  - hightFramerate: 最大 FPS
  - lowFramerate: 最小 FPS
- actions: 基本的に気にしなくて良い
  - updateWebRTCStat:
  - updateClientConfig:
  - updateSocket:
  - setVideoAspectRatio:
  - addLatestStats:
  - setWebRTCPlayerObj:
- load:

同名のメソッド/プロパティについては、「[Pixel Streaming: プレイヤーWebページをカスタマイズする](https://docs.unrealengine.com/ja/Platforms/PixelStreaming/CustomPlayer/index.html)」を参照。
