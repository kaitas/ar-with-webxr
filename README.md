# Android での WebXR Device API による AR アプリの構築

## Building an augmented reality application with the WebXR Device API

This code is a Japanese translated fork from the original resources in codelab [Building an augmented reality application with the WebXR Device API](https://codelabs.developers.google.com/codelabs/ar-with-webxr/#0).

This is a work in progress (and its fork itself). If you find a mistake or have a suggestion, please [file an issue](https://github.com/googlecodelabs/ar-with-webxr/issues). Thanks!

## このプロジェクトで学べること

- WebXR Device API の使用方法
- 拡張現実（AR）のヒットテストを利用した面の見つけ方
- 現実世界のカメラフィードと同期した 3D モデルのロードとレンダリングの方法

## What you'll need

- A workstation for coding and hosting static web content
- [ARCore-capable Android device](https://developers.google.com/ar/discover/#supported_devices)
- ARCore installed (Chrome will automatically prompt you to install ARCore)
- [Google Chrome](https://www.google.com/chrome/)
- [Web Server for Chrome](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb), or your own web server of choice
- USB cable to connect your AR device to workstation
- [The sample code](https://github.com/googlecodelabs/g/archive/master.zip)
- A text editor
- Basic knowledge of HTML, CSS, JavaScript, and Chrome DevTools

## 環境と準備

- ワークステーション（PC もしくは Mac もしくは Chromebook）
- [ARCore 対応の Android デバイス](https://developers.google.com/ar/discover/#supported_devices)
- ARCore のインストール（Chrome が自動で ARCore のインストールを表示）。
- [Google Chrome](https://www.google.com/chrome/)
- [Web Server for Chrome](https://chrome.google.com/webstore/detail/web-server-for-chrome/ofhbbkphhbklhfoeikjpcbhemlocgigb)、またはお好みのウェブサーバー
- AR デバイスとワークステーションを接続する USB ケーブル
- サンプルコード](https://github.com/googlecodelabs/g/archive/master.zip)
- テキストエディタ
- HTML、CSS、JavaScript、Chrome DevTools に関する基本的な知識

### つまり…

- Android Studio はなくても大丈夫
- Visual Studio Code でもよき
- HTML、CSS、JavaScript、Chrome DevTools に関する基本的な知識

これなら Chromebook でもいけるかもしれんね！

# プロジェクトの中身と解説

Google の開発者 Dereck Bridie さんによる解説に沿って進めます。
[YouTube](https://www.youtube.com/watch?v=gAzIkjkJSzM)

## 初期セットアップとデバッグ環境の設定

### workshop/index.html

workshop/index.html

```
<script src="https://unpkg.com/material-components-web@latest/dist/material-components-web.min.js"></script>
<script src="https://unpkg.com/three@0.126.0/build/three.js"></script>
<script src="https://unpkg.com/three@0.126.0/examples/js/loaders/GLTFLoader.js"></script>
```

`material components` ライブラリと、`three.js` ライブラリ、`GLTFLoader.js` を使っています。

workshop/index.html

```
<a id="enter-ar" class="mdc-button mdc-button--raised mdc-button--accent">
Start augmented reality
</a>
```

&lt; a&gt; タグで[AR 開始]ボタンを用意

### workshop/app.js

workshop/index.html
WebXR のサポートを問い合わせる。もし `immersive-ar` モードがサポートされていない場合はエラーを表示。

```
  (async function() {
  const isArSessionSupported =
  navigator.xr &&
  navigator.xr.isSessionSupported &&
  await navigator.xr.isSessionSupported("immersive-ar");
  if (isArSessionSupported) {
  document.getElementById("enter-ar").addEventListener("click", window.app.activateXR)
  } else {
  onNoXRDevice();
  }
  })();
```

### 開発用 Web サーバと実機デバッグの設定

1. Web Server for Chrome 機能拡張 をインストール、`AR-WITH-WEBXR` (git clone したディレクトリ)を指定
2. `http://127.0.0.1:8887/` あたりにサーバが立ち上げられますので、`http://127.0.0.1:8887/workshop/`を開いて動作確認します
3. 別タブで `chrome://inspect` にアクセスすることで Chrome DevTools が表示される。
4. USB デバッグを設定した Android デバイス を USB に接続すると、`Remote Target` に Android デバイス が表示される
5. `Discover USB Devices` の項目でポートフォワード設定をします。実機 → 開発サーバの設定する場合、例えば `8887`, `localhost:8887` として、`Enable Port Forwarding` をチェックオンします。
6. DevTools の `Remote Target` (実機) の Chrome `Open Tab with URL` の欄に、上記の URL(`http://127.0.0.1:8887/workshop/`) を貼り付けて `Open` します。実機の Chrome で該当のページが表示されます。
7. `inspect` を押すと、DevTools のウインドウ にもその様子が表示されます（ `Unsupported Browser`と出ている）
8. DevTools の `Sources` タブで `Page`でページツリーを表示させながらデバッグする。

## コーディング開始（19:20-）

### workshop/app.js:22

```
(async function() {
  const isArSessionSupported =
      navigator.xr &&
      navigator.xr.isSessionSupported &&
      await navigator.xr.isSessionSupported("immersive-ar");
  if (isArSessionSupported) {
    document.getElementById("enter-ar").addEventListener("click", window.app.activateXR)
  } else {
    onNoXRDevice();
  }
})();
```

WebXR サポートを問い合わせて `immersive-ar` モードがないときはエラーを表示する。

この辺りの詳細は Mozilla MDN に日本語ドキュメントあります
[WebXR Device API](https://developer.mozilla.org/ja/docs/Web/API/XRSystem)

### workshop/app.js:36

```
class App {
  activateXR = async () => {
    try {
      /* カメラの背景とバーチャルシーンを格納するキャンバス(Canvas)を作成 */
      this.createXRCanvas();
      /* すべての設定が完了したら、アプリを起動します。 */
      await this.onSessionStarted();
    } catch(e) {
      console.log(e);
      onNoXRDevice();
    }
  }
```

WebXR Device API への接続を管理し、各フレームでのレンダリングを処理するコンテナクラスです。

`this.createXRCanvas()` でカメラの背景とバーチャルシーンを格納するキャンバス(Canvas)を作成(後述)。

`await this.onSessionStarted()` で、すべての設定が完了したら、アプリを起動。

### createXRCanvas()関数

```
  createXRCanvas() {
    this.canvas = document.createElement("canvas");
    document.body.appendChild(this.canvas);
    this.gl = this.canvas.getContext("webgl", {xrCompatible: true});
    this.xrSession.updateRenderState({
      baseLayer: new XRWebGLLayer(this.xrSession, this.gl)
    });
  }
```

canvas 要素を追加し、WebXR と互換性のある WebGL コンテキストを初期化します。
`baseLayer` がカメラビューが描画されるレイヤーです。

### セッションのリクエスト

App クラスの冒頭、activateXR

```
  activateXR = async () => {
    try {
      /** Initialize a WebXR session using "immersive-ar". */
      this.xrSession = await navigator.xr.requestSession("immersive-ar");
```

> this.xrSession = await navigator.xr.requestSession("immersive-ar");

この行をコメントアウトしてセッションをリクエストします。

後で追加機能を加えていきますが、今はこの状態で進めます。

### セッションの開始

`onSessionStarted` を解説します。ここは `XRSession` が始まったときに呼び出されます。
`three.js` のレンダラー、シーン、カメラを設定し、`XRWebGLLayer` を `XRSession` にアタッチして、レンダリングループを開始します。

```
     onSessionStarted = async () => {
    /* `ar` クラスを追加して、2Dコンポーネントを隠します。 */
    document.body.classList.add('ar');
    /* ウェブ上で3Dを扱うためには、three.jsを使います。  */
    this.setupThreeJs();
    this.xrSession.requestAnimationFrame(this.onXRFrame);
  }
```

コメントアウトされている行は、後ほど戻ってきます。

### setupThreeJs()

続いて `setupThreeJs()` の初期コードの解説です。

Web 上での 3D 作業を支援するために、three.js を使用します。
WebGLRenderer を含む、three.js 固有のレンダリングコードの初期化です。
デモシーン、3D コンテンツを見るためのカメラなどのレンダリングコードを初期化します。
WebGLRenderer を設定し、セッションのベースレイヤーへのレンダリングを処理します。
オリジナルはコメントアウトすると色々あとで機能追加できるようになっていますが、今必要なところだけ抜粋します。

```
  /**
   * WebGLRendererを含む、three.js固有のレンダリングコードの初期化。
   * デモシーン、3Dコンテンツを見るためのカメラなどのレンダリングコードを初期化します。
   */
  setupThreeJs() {
    /**
     * Web上での3D作業を支援するために、three.jsを使用します。
     * WebGLRendererを設定し、セッションのベースレイヤーへのレンダリングを処理します。
     */
    this.renderer = new THREE.WebGLRenderer({
      alpha: true,
      preserveDrawingBuffer: true,
      canvas: this.canvas,
      context: this.gl
    });
    this.renderer.autoClear = false;
    // this.renderer.shadowMap.enabled = true;
    // this.renderer.shadowMap.type = THREE.PCFSoftShadowMap;

    /** デモ用のキューブを描画するシーンを初期化 */
    this.scene = DemoUtils.createCubeScene();
    // this.scene = DemoUtils.createLitScene();
    // this.reticle = new Reticle();
    // this.scene.add(this.reticle);

    /** カメラの行列はAPIから直接更新します。行列の自動更新を無効にして、
     * three.jsが行列を個別に処理しようとしないようにしておきます。 */
    this.camera = new THREE.PerspectiveCamera();
    this.camera.matrixAutoUpdate = false;
  }
```

### リファレンス空間の追加

`onSessionStarted` に戻ってコメントアウトを外します

```
    /** Setup an XRReferenceSpace using the "local" coordinate system. */
    this.localReferenceSpace = await this.xrSession.requestReferenceSpace('local');
    /** Start a rendering loop using this.onXRFrame. */
    this.xrSession.requestAnimationFrame(this.onXRFrame);

```

続いて、そのすぐ下にあるコールバック `onXRFrame` のコメントをアンコメントしていきます。
このままだと描画の中身がないので、onSessionStarted に戻って onXRFrame のコールバックを作っていきます。
レンダリング待ち列に追加、フレームバッファを baseLayer のフレームバッファにバインドして、トラッキングが成立したときに取得できる pose 以降を書いていきます。

サンプルコードから始める場合はコメント外すだけですが、ここでは日本語コメントと、この段階で使うコードを中心にインラインで解説します。

```
  /**
   * XRSessionのrequestAnimationFrameで呼び出されます。
   * 時間とXRPresentationFrameを指定して呼び出されます。
   */
  onXRFrame = (time, frame) => {
    /** 次の描画要求をキューイング（待ち列に入れる）します。 */
    this.xrSession.requestAnimationFrame(this.onXRFrame);

    /** グラフィックスのフレームバッファをbaseLayerのフレームバッファにバインドする。 */
    const framebuffer = this.xrSession.renderState.baseLayer.framebuffer
    this.gl.bindFramebuffer(this.gl.FRAMEBUFFER, framebuffer)
    this.renderer.setFramebuffer(framebuffer);

    /** デバイスのポーズを取得します。
     * XRFrame.getViewerPoseは、セッションがトラッキングを確立させる間、nullを返すことがあります。
     */
    const pose = frame.getViewerPose(this.localReferenceSpace);
    if (pose) {
      /**
      モバイルARの場合はビューは1つしかありません（VRヘッドセットの場合は左右眼で2つ）。
      */
      const view = pose.views[0];

      /** ベースレイヤーのビューポート（視界用の窓）を取得して、そのサイズをレンダラーにセットします。*/
      const viewport = this.xrSession.renderState.baseLayer.getViewport(view);
      this.renderer.setSize(viewport.width, viewport.height)

      /** THREE.cameraの設定には、ビューの変換行列(transform)と投影行列(projection)を使用します。
       * これで物理的なカメラとバーチャルカメラが接続されたことになります。
       */
      this.camera.matrix.fromArray(view.transform.matrix)
      this.camera.projectionMatrix.fromArray(view.projectionMatrix);
      this.camera.updateMatrixWorld(true);
      /** THREE.WebGLRendererでシーンをレンダリングします。*/
      this.renderer.render(this.scene, this.camera)
    }
  }
```

ここまできたらリロードしましょう！
実機で AR が動いているのを見れるはずです（ケーブル短くていい絵が撮れなかった）。

せっかくなので動く WebAR のシーンにしておきました
お手持ちのスマホでアクセスしてみて下さい
[https://akihiko.shirai.as/ar-with-webxr/](https://akihiko.shirai.as/ar-with-webxr/)
（あとで消しちゃうかも？）

[Mozilla の WebXR サポート情報](https://developer.mozilla.org/en-US/docs/Web/API/WebXR_Device_API#browser_compatibility)によるといろんなブラウザで動きそうなのですが、
確実な動作環境としては Android の対応です。Android 7.0（Nougat）以降、具体的な機種リストは[こちら](https://developers.google.com/ar/devices)。<br>
iOS は Chrome, Safari, Firefox, Edge ともダメっぽかった。残念。<br>

続きは、当たり判定です！
解説動画の[28:16](https://youtu.be/gAzIkjkJSzM?t=1696)あたりです。

好きなモデルデータも選んでおきたい

関連：モデルビューワー
[https://modelviewer.dev/](https://modelviewer.dev/)
