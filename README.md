[README.md](https://github.com/user-attachments/files/30690602/README.md)
# 同心度チェッカー（OpenCV.js / スマホ対応版）

Pythonのcustomtkinter版と同じロジックを、ブラウザだけで動く `index.html` 1枚に移植したものです。
GitHub Pagesで公開すれば、スマホのブラウザから直接カメラを使ってチェックできます。

## 中身
- `index.html` … アプリ本体（OpenCV.js を CDN から読み込み、内部処理は元Pythonコードと同じロジック）
  - 内側の黒円検出：しきい値二値化 → モルフォロジー処理 → 輪郭検出 → 真円度フィルタ
  - 外側の白円検出：内側円中心から放射状に輝度をスキャンし、白→背景の輝度の谷をエッジとして抽出 → 最小二乗円フィット（外れ値除去つき）
  - ズレ量・ズレ比率を計算し、しきい値以下ならOK、超えたらNGと表示

## GitHub Pagesでの公開手順

1. GitHubで新しいリポジトリを作成します（例: `concentricity-checker`）。
2. このフォルダの中身（`index.html` と `README.md`）をリポジトリのルートに置き、コミット＆プッシュします。

   ```bash
   git init
   git add index.html README.md
   git commit -m "OpenCV.js版 同心度チェッカーを追加"
   git branch -M main
   git remote add origin https://github.com/<あなたのユーザー名>/concentricity-checker.git
   git push -u origin main
   ```

3. GitHubのリポジトリページで **Settings → Pages** を開きます。
4. 「Build and deployment」の Source を **Deploy from a branch** にし、Branch を `main` / フォルダを `/(root)` に設定して保存します。
5. 数十秒〜数分待つと、`https://<あなたのユーザー名>.github.io/concentricity-checker/` でアクセスできるようになります。

> **重要**：スマホのカメラ（`getUserMedia`）はHTTPS環境でないと動作しません。GitHub Pagesは自動的にHTTPSになるのでそのまま使えます。手元でテストする場合も `file://` ではなく、`https://` またはローカルサーバー（`http://localhost`）経由で開いてください。

## スマホでの使い方

1. スマホのブラウザ（iOSはSafari、AndroidはChrome推奨）で上記URLを開きます。
2. カメラの使用許可を求められたら「許可」を選びます。
3. 起動すると背面カメラの映像が表示されます。パイプ／チューブの端面を画面中央に収めてください。
4. 画面下部のスライダーで調整できます。
   - **黒円検出しきい値**：内側の穴（黒い部分）をどこまで暗いと判定するか
   - **OK判定の許容ズレ比率**：この％以内なら「OK」と表示
   - **内側円の最大半径**：カメラと被写体の距離によって写る穴の大きさ（ピクセル数）が変わるため、検出できない場合はここを大きくしてください
5. 「📷 スナップショット保存」で現在の映像（判定結果の描画つき）をPNGとして端末に保存できます。
6. 「🔄 切替」で前面／背面カメラを切り替えられます。
7. iOSでSafariの「ホーム画面に追加」を使うと、アプリのようにフルスクリーンで起動できます。

## 元のPython版との違い・注意点

- カメラ映像を毎フレーム全部解析すると重いため、解析処理は約120msごと（約8fps相当）に間引いて実行しています。映像自体は滑らかに表示されます。
- `MAX_INNER_RADIUS` は元コードでは `20px` 固定でしたが、スマホのカメラ解像度・被写体との距離によって最適値が変わるため、スライダーで調整できるようにしています。
- OpenCV.jsの読み込みには数秒かかることがあります（初回アクセス時、CDNから `opencv.js` をダウンロードします）。
- Mat（画像データ）のメモリ解放は毎フレーム確実に行っていますが、長時間の連続稼働はブラウザやデバイスの負荷を見ながらご利用ください。

## カスタマイズしたい場合

`index.html` 内の以下の定数・スライダー範囲を変更すると挙動を調整できます。

- `MIN_INNER_RADIUS` / `MAX_INNER_RADIUS`（内側円として許容する半径の範囲）
- `BRIGHT_PEAK_MIN`（外周スキャンで「白リングを検出できた」とみなす最小輝度）
- `analyzeTick` の呼び出し間隔（`setInterval(analyzeTick, 120)` の `120`）で解析頻度を調整可能
