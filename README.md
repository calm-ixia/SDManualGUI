# SDManualGUI (Japanese, 日本語)
Google Colabノートブック上に実装した、DiffusersベースのStable Diffusion用GUIです。
本家AUTOMATIC1111版web UIをまねて機能とGUIを作成しました（設計を大きく参考にしていますが、ライブラリやソースは不使用です）。

Colab無償版の環境における動作と、画像生成パラメータの検討を主な用途としています。IPyWidgetsを使用してGUIを組んでいて、Colab内で完結して動作します。

GUIとソースの不器用さには目をつぶってください。

※Noteの記事
- https://note.com/calm_ixia/n/n4cb0fd7d7fbb
- https://note.com/calm_ixia/n/n6ac340e92675

## 主な機能
- img2txt : テキストのプロンプトに沿った画像を生成
- img2img : 画像を参考に、プロンプトに沿った新たな画像を作成
- inpaint : マスク適用範囲を書き換えたような画像を生成。初期画像、プロンプトも指定可
- PNG Info : 生成パラメータを保存した画像について、埋め込んだ情報を確認

## 動作方法
1. 基本的には、1. ライブラリ導入、2. GUI実装、3. 起動 の順に実行すると、「起動」のセルにGUIが表示されます。
2. setupタブで、モデルとリビジョンを選択肢、Setupをクリックするとモデルが読み込まれます。
3. モデル読み込み完了後、画像生成が可能になります。タブを切り替えて画像生成を設定してください。
4. Generateボタンで画像生成が開始されます。画像表示後、保存したい画像を右クリックで保存してください（ブラウザの機能）。

runwayml-v1.5モデルの使用に際しては、あらかじめHugging Faceへのユーザ登録と利用同意を行い、かつ起動時にHugging Face へのログインが必要です。（ノートブックを参照のこと）

## 細かな機能
- サムネイル : 右側のサムネイル画像にも生成パラメータを埋め込んでいます。パラメータ検討時の記録代わりに使えます。
- 割り当てられたGPUの確認 : ノートブックのセルで、Colabから割り当てられたGPUを確認できます。

## 未実装の機能
本家を使用してください。
- 768x768モデルの使用（調査中。LPW Pipeline単体での正常な画像生成は確認済み）
- outpainting
- 高解像度化 関連
- Restore faces 関連
- Depth2Img 関連
- スクリプト
- モデル学習・適応
- xformers 対応
- PNGファイルへのモデルハッシュ値の保存
- Shift+EnterでGenerate実行

----
## GUIの説明

### setup
![setup](https://user-images.githubusercontent.com/118874552/214771164-f0907057-54ba-42e4-bfa7-b1b7f37b61a9.png)

- model
  - preset model ID : 左がモデルのリポジトリ名、右がリビジョン名です。ドロップリストから選べます。※現状、自分(calm_ixia)が使うstabilityAI公式モデルとTrinArtだけ登録
  - プリセットにないモデルを使用したい場合は、other modelにチェックを入れ、リポジトリ名とリビジョンを手入力してください。Hugging Face上にあるDiffusers対応のモデルを使用可能です
- device
  - device to : 画像生成処理を行うデバイスの指定です。※通常は"cuda"を選択。"cpu"はデバッグ用
  - attention slicing: 画像生成速度と引き換えに、生成時のGPUメモリ消費を抑えます。
  - safety checker: 生成画像のNSFWチェックを行います。チェックに引っかかった画像は全部黒塗り（■）となります。
- Setup ボタン : setup を開始します。ボタンの下の領域にセットアップの進捗が表示されます。

画像生成を始める前に、まずSetupを行ってください。モデルを切り替える場合もSetupボタンをクリックしてください。

※プリセットにないモデルを読み込む例 (waifu-diffusion)
![setup2](https://user-images.githubusercontent.com/118874552/214772630-49113071-6649-4501-9a3b-77194d0564ae.png)

#### 注意
- モデルのロード時にはCPU側のRAMに一度モデルデータが読み込まれます。そのため、CPU側で十分なRAMが確保されていないとVRAMが足りていても途中でロードに失敗します。
Colab上で動作させるとき、VRAMが占有されている場合は一旦GPUとの接続を切り、ライブラリ導入からやり直す必要があります。
- よく使うモデルをプリセットに登録したい場合は、ソースコードに直接追記してください。

----
### txt2img
![txt2img画面](https://user-images.githubusercontent.com/118874552/214303033-b0754d8f-c08d-4d4e-ae59-b66b2eb5c7cb.png)

- Generateスタック
  - Generate ボタン : クリックで画面生成
  - (interruptボタン、未実装）: 実行中の生成1回分を中断して次の生成へ移る
  -（stopボタン、未実装）: 実行中の生成処理をすべて中断
  - word count：各プロンプトのword数（サブワード）を表示。現在数 / 最大数（3チャンク分）
- プロンプト
  - 上：プロンプト。生成したい画像の特徴をテキストで指示
  - 下：ネガティブプロンプト。 生成させたくない画像の特徴をテキストで指示。人体の造形をコントロールするときにもよく使う
- 生成パラメータ
  - sampling
    - steps : 生成（推論）ステップ数 ＜ステップが多いほど絵のタッチが詳細になる傾向があるが、多すぎると高い圧縮率のJPEGのように画像が荒れる＞
    - method : いわゆるサンプラー ＜ステップのスケジューリング手法。塗りや線のシャープ感が変わるほか、描かれる事物が変わることもある＞
  - quality params
    - CFG scale : プロンプトの影響度
    - seed : 疑似乱数のシード値。"-1"の指定でシード値をランダム設定 ＜シード値によって描かれる事物や構図が大きく変化する＞
  - image size
    - width : 1枚の絵の幅
    - height : 1枚の絵の高さ
    - batch size : 1回の生成処理で同時に生成する画像の枚数 ＜batch sizeの分だけGPUが多く必要。Colab無償版だと1より大きい値ではGPUが足りない＞
    - batch count : 生成処理を行う回数。回数ごとに乱数のシード値を1ずつ増やして画像生成する ＜1回分のGPUの確保量で済む代わり、処理時間がかかる。Colab で複数枚の絵を連続で生成したい場合は、こちらを増やす＞
    - grid columns : グリッドの横（列）方向の枚数 ＜複数枚の絵を生成したときに、絵を1つの長方形に並べてまとめたグリッドを生成する。その横方向の枚数を指定＞  
- 生成画像
  - 選択画像（大きい部分） : 選択した画像 ＜横512pxに縮小して表示＞
  - サムネイル : 生成画像のサムネイルを縦方向に並べて表示。サムネイルは寸法を1/4に縮小して作成
    - 1回の生成処理の分（batch size×count）の枚数だけサムネイルが表示される。一番上はgridのサムネイルを表示
    - 複数枚の画像生成をする場合、画像が生成できたタイミングでサムネイルを1枚追加
  - 🖼（額縁）ボタン : サムネイルを選択して大きな画像の部分に表示
  - 進捗状況のコンソール : 進捗状況を表示

選択画像やサムネイルを右クリックすると、ブラウザのメニューから画像をダウンロードできます。
- 選択画像を右クリックで保存した場合、原寸画像をダウンロード ＜グリッドを原寸で保存したい場合は、グリッドを一度選択画像として表示してから右クリック保存＞
- サムネイルを右クリックで保存した場合、小さな画像をダウンロード ＜パラメータ検討時の記録代わりに使える。特にグリッドのサムネイルが便利＞

----
### img2img
※画面ではモデルに TrinArt stable diffusion v2 (naclbit/trinart_stable_diffusion_v2) を使用
![img2img画面](https://user-images.githubusercontent.com/118874552/214303590-e8441ae5-141d-46c9-ac1c-1ae6da27c25d.png)

GUI項目はほぼtxt2imgのものと同様。
- 生成パラメータ
  - img2img
    - source img : 初期データとなる画像を指定
    - Denoising strength : source img の影響度を指定

----
### inpaint
※画面ではモデルに TrinArt stable diffusion v2 (naclbit/trinart_stable_diffusion_v2) を使用
![inpaint画面](https://user-images.githubusercontent.com/118874552/214487043-8f309732-222d-4299-a2f6-b853404c6528.png)

GUI項目はほぼtxt2imgのものと同様。
- 生成パラメータ
  - inpaint
    - top img
    - mask
    - back img
    - 左下 : マスク適用範囲のイメージ
    - Inpaint areas (black/white) : inpaint の適用範囲となるマスクの色。黒か白を指定。
    - Masked content : マスク適用範囲の初期データ。
    - Denoising strength : プロンプト類のマスク適用範囲への影響度を指定

#### Masked contentの比較
![masked_content_compare](https://user-images.githubusercontent.com/118874552/214492184-e809aebc-6891-4e65-9255-39fbf86b90b7.png)

----
### PNG Info
![PNGInfo画面](https://user-images.githubusercontent.com/118874552/214465494-66468443-f510-4bbd-854a-dc627a0f28ef.png)

- upload an image ボタン : 読み込むPNGファイルを選択。ファイルはColab内の環境に一時的にアップロードされる
- 読み込んだ画像（大きい画像） : 読み込んだ画像 ＜横512pxに縮小して表示＞
- 生成パラメータ情報 : PNGファイルに記録された生成パラメータ情報を表示
- サムネイル（小さい画像）: 読み込んだ画像からサムネイルを作成 ＜これにも読み込んだ生成パラメータ情報をそのままコピー＞
- send to txt2img / img2img / inpaint : 生成パラメータ情報を各タブに送る

----
## PNG Info 単品
![PNG Info だけを切り出したノートブック](https://github.com/calm-ixia/SDManualGUI/blob/main/PNGInfo.ipynb) を別に作成しました。こちらはCPUのみで動作します。
![PNGInfo単品](https://user-images.githubusercontent.com/118874552/214233801-d3e3d5df-3171-4c8b-a59a-61f7c55aa36b.png)
1. Setup セルを実行
2. Launching セルを実行
3. upload an image をクリックし、ファイルを選択

※PNG Info 単品のノートブックで send to ** は出来ません

----
## 主な使用ライブラリ：
- Unstable Diffusion
  - 七師 氏の作成
  - 画像生成に使用。Diffusersベース
  - https://github.com/nanashi161382/unstable_diffusion/
- Long Prompt Weighting Stable Diffusion
  - SkyTNT 氏の作成
  - Text Embedding 生成に使用。Diffusersの公式community pipeline
  - https://github.com/huggingface/diffusers/blob/main/examples/community/lpw_stable_diffusion.py  
- Diffusers
  - Hugging Face の作成
  - https://github.com/huggingface/diffusers
- IPyWidgets
  - GUIライブラリ、Colabで動作
  - https://ipywidgets.readthedocs.io/en/stable/


## 引用・参考文献：
- runwayml-v1.5モデル使用時の利用同意
  - https://programmingforever.hatenablog.com/entry/2022/09/21/174953
  - https://zenn.dev/razokulover/scraps/b4f639228ab305
- scheduler（サンプラー）
  - https://huggingface.co/docs/diffusers/api/schedulers#implemented-schedulers
  - https://note.com/npaka/n/n5fd3d8ecf1e6
- Diffusersの使用方法
  - https://huggingface.co/blog/stable_diffusion
  - https://torch.classcat.com/2022/11/13/huggingface-blog-stable-diffusion/
- AUTOMATIC1111 web UIの挙動
  - https://github.com/AUTOMATIC1111/stable-diffusion-webui/wiki

## 謝辞
- Stable Diffusion
  - https://github.com/CompVis/stable-diffusion
  - https://huggingface.co/stabilityai
  - https://huggingface.co/runwayml
- AUTOMATIC1111
  - https://github.com/AUTOMATIC1111/stable-diffusion-webui
- AlanB
  - https://huggingface.co/AlanB/lpw_stable_diffusion_mod
- Yasu Seno (TrinArt model)
  - https://huggingface.co/naclbit
