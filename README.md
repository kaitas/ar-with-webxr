# Android での WebXR Device API による AR アプリの構築

## Building an augmented reality application with the WebXR Device API

This code is a Japanese translated fork from the original resources in codelab [Building an augmented reality application with the WebXR Device API](https://codelabs.developers.google.com/codelabs/ar-with-webxr/#0).

This is a work in progress (and its fork itself). If you find a mistake or have a suggestion, please [file an issue](https://github.com/googlecodelabs/ar-with-webxr/issues). Thanks!

## ここで学ぶこと

- WebXR Device API の使用方法
- 拡張現実（AR）のヒットテストを利用した面の見つけ方
- 現実世界のカメラフィードと同期した 3D モデルのロードとレンダリングの方法

# What you'll need

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

Github リポジトリはこちら
https://github.com/googlecodelabs/ar-with-webxr
ライセンスは Apache 2.0

# プロジェクトの中身と解説

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

<&lt; a&gt; タグで[AR 開始]ボタンを用意

### workshop/app.js

workshop/index.html

```
/*
- WebXR のサポートを問い合わせる。もし `immersive-ar` モードがサポートされていない場合はエラーを表示。
*/
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
8. DevTools の `Sources` タブで `Page`でページツリーを表示させながら、デバッグするといいですね。
