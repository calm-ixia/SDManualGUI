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
1. 基本的には、1. ライブラリ導入、2. GUI実装、3. 起動 の順に実行すると、「起動」のセルにGUIが表示されます
2. setupタブで、モデルとリビジョンを選択肢、Setupをクリックするとモデルが読み込まれます
3. モデル読み込み完了後、画像生成が可能になります。タブを切り替えて画像生成を設定してください
4. Generateボタンで画像生成が開始されます。画像表示後、保存したい画像を右クリックで保存してください（ブラウザの機能）

- runwayml-v1.5モデルの使用に際しては、あらかじめHugging Faceへのユーザ登録と利用同意を行い、かつ起動時にHugging Face へのログインが必要です（ノートブックを参照のこと）
- Extrasタブの機能を使う時はライブラリを導入するセルを別途実行してください。（Real-ESRGAN, MiDaS）

## 細かな機能
- サムネイル : 右側のサムネイル画像にも生成パラメータを埋め込んでいます。パラメータ検討時の記録代わりに使えます
- 割り当てられたGPUの確認 : ノートブックのセルで、Colabから割り当てられたGPUを確認できます

----
## 変更点
### 2023-05-10
- Layered Diffusionへの依存を分離した
- ExtrasタブにてLayered Diffusionを実行できるようにした

### 2023-02-21
- Extrasタブの各機能にファイルの出入力機能を追加。Colabのアップロード・ダウンロードAPIを使用
- 機能追加に合わせてReadmeを追記・修正

### 2023-02-18
- （ノートブックのタイトルのみ変更）img2txtなど各機能の画面を差す語をペインからタブに変更。※現状のGUI実装に合わせた
- Extrasタブを追加
- Extrasタブに高解像度化機能を追加（Real-ESRGANを使用） ※ライブラリを呼び出す最低限のインタフェースだけ実装。ファイルの出入力は手動
- Extrasタブに深度推定機能を追加（MiDaSを使用） ※同上
- 機能追加に合わせてReadmeを追記・修正

### 2023-02-07
- 使用ライブラリの名前をunstable_diffusionからlayered_diffusionにリネーム
- pipeline切り替え：Layered DiffusionとLPW Pipelineの使用を切り替え
- Layered Diffusionを使用してimg2imgする時の、Denoising strengthの適用方法を変更。readmeの画像も変更
- PNG Infoに生成パラメータ情報の編集機能を追加
- LPW Pipelineへの機能追加、特にinpaintの機能追加
  - 768x768モデルでの画像生成
  - latent noise, latent nothingによるinpaintの実装
  - Mask timing: 生成ステップ内でマスクをかけるタイミングの指定。絵柄が少し変わる
  - Mask output: 最終出力にマスクをかけるかどうか。マスク適用範囲以外はほぼ背景画像のまま残る
  - latent noise, latent nothing使用時のDenoising strengthのかけ方を変更。ステップを減らすのではなく、マスク適用範囲について背景画像と初期データをアルファブレンドする
- コードの整理など細かな変更
- readmeを変更・修正
  - 上記の機能変更を反映
  - 使用したTrinArtモデルのリビジョンを表記
  - 誤字訂正など細かな変更

## 未実装の機能
本家を使用してください。
- outpainting
- 高解像度化 関連（最低限の機能は本GUIでも可）
- Restore faces 関連
- Depth2Img 関連（デプスマップの生成は本GUIでも可）
- スクリプト
- モデル学習・適応
- xformers 対応（Layered Diffusionが対応）
- PNGファイルへのモデルハッシュ値の保存
- Shift+EnterでGenerate実行

----
## GUIの説明

### setup
![setup](https://user-images.githubusercontent.com/118874552/217156269-c7edf230-d238-4d53-8dc9-ce85969b2e1f.png)

- model
  - preset model ID : 左がモデルのリポジトリ名、右がリビジョン名です。ドロップリストから選べます。※現状、自分(calm_ixia)が使うstabilityAI公式モデルとTrinArtだけ登録
    - プリセットにないモデルを使用したい場合は、other modelにチェックを入れ、リポジトリ名とリビジョンを手入力してください。Hugging Face上にあるDiffusers対応のモデルを使用可能です
  - use layered_diffusion：Layered Diffusionを使用して画像生成します。チェックなしの場合はLPW Pipelineを使用します
- device
  - device to : 画像生成処理を行うデバイスの指定です。※通常は"cuda"を選択。"cpu"はデバッグ用
  - attention slicing: 画像生成速度と引き換えに、生成時のGPUメモリ消費を抑えます
  - safety checker: 生成画像のNSFWチェックを行います。チェックに引っかかった画像は全部黒塗り（■）となります
- Setup ボタン : setup を開始します。ボタンの下の領域にセットアップの進捗が表示されます

画像生成を始める前に、まずSetupを行ってください。モデルを切り替える場合もSetupボタンをクリックしてください。

画像生成エンジンごとの画質の比較についてはimg2img, inpaintタブの説明で詳しく記載しています。


※プリセットにないモデルを読み込む例 (waifu-diffusion)
![setup](https://user-images.githubusercontent.com/118874552/214772630-49113071-6649-4501-9a3b-77194d0564ae.png)

#### 注意
- モデルのロード時にはCPU側のRAMに一度モデルデータが読み込まれます。そのため、CPU側で十分なRAMが確保されていないとVRAMが足りていても途中でロードに失敗します。
Colab上で動作させるとき、VRAMが占有されている場合は一旦GPUとの接続を切り、ライブラリ導入からやり直す必要があります
- よく使うモデルをプリセットに登録したい場合は、ソースコードに直接追記してください

----
### txt2img
![txt2imgタブ](https://user-images.githubusercontent.com/118874552/217156805-e5ffd8f3-9820-44f4-8336-37c709cb2c91.png)

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
※画面ではモデルに TrinArt stable diffusion v2 (naclbit/trinart_stable_diffusion_v2) リビジョン diffusers-115k を使用
![img2imgタブ](https://user-images.githubusercontent.com/118874552/217157086-dd03ccdc-f3ab-4815-be6e-b94e304ec2cb.png)

GUI項目はほぼtxt2imgのものと同様。
- 生成パラメータ
  - img2img
    - source img : 初期データとなる画像を指定
    - Denoising strength : source img の影響度を指定（[後の章](#denoising-strength-img2img-20-steps)で比較）

----
### inpaint
※画面ではモデルに TrinArt stable diffusion v2 (naclbit/trinart_stable_diffusion_v2) リビジョン diffusers-115k を使用
![inpaintタブ](https://user-images.githubusercontent.com/118874552/217157305-52218412-70e8-4445-904c-3df7b4cd2a70.png)


GUI項目はほぼtxt2imgのものと同様。
- 生成パラメータ
  - inpaint
    - top img
    - mask
    - back img
    - 左下 : マスク適用範囲のイメージ
    - Inpaint areas (black/white) : inpaint の適用範囲となるマスクの色。黒か白を指定
    - Masked content : マスク適用範囲の初期データ（[後の章](#masked-content-inpaint-denoising-strength--10)で比較）
    - Denoising strength : プロンプト類のマスク適用範囲への影響度を指定
    - LPW Pipeline使用時（[後の章](#mask-timing-inpaint-lpw-pipeline%E3%81%AE%E3%81%BF)で比較）
      - Mask timing：生成ステップ内のマスク処理をかけるタイミング。絵柄が少し変わる
        - last：Diffusersベース ＜デノイズ後、ステップの最後にadd_noise＆マスク処理を行い、次のステップの入力に渡す。背景が崩れやすいため Mask outputと併用すると良い＞
        - first：AUTOMATIC1111 web UIに似た方法 ＜ステップの最初にadd_noise＆マスク処理し、その後デノイズする。背景が残りつつ画質が良い＞
        - none：ステップ中にマスクをかけない ＜背景が大きく書き換わるが画質は良い。 Mask outputなしで生成すると良い画が得られる＞
      - Mask output：最終出力に潜在空間上でのマスク処理をする。マスク適用範囲以外はほぼ背景画像のまま残る

inpaintでもDenoising strengthは有効ですが、Layered DiffusionとLPW Pipelineとで適用方法が異なります。
- Layered Diffusion：strengthの割合でステップ数を減らす
- LPW Pipeline：
  - latent noise, latent nothing 以外：strengthの割合でステップ数を減らす（InpaintLegacyの挙動）
  - latent noise, latent nothing：マスク適用範囲について背景画像と初期データをアルファブレンドする

----
### PNG Info
![PNGInfoタブ](https://user-images.githubusercontent.com/118874552/217164354-ba20fc91-2aa8-48c4-bbd6-4143de4f208d.png)

- upload an image ボタン : 読み込むPNGファイルを選択。ファイルはColab内の環境に一時的にアップロードされる
- 読み込んだ画像（大きい画像） : 読み込んだ画像 ＜横512pxに縮小して表示＞
- 生成パラメータ情報 : PNGファイルに記録された生成パラメータ情報を表示
- サムネイル（小さい画像）: 読み込んだ画像からサムネイルを作成 ＜これにも読み込んだ生成パラメータ情報をそのままコピー＞
- send to txt2img / img2img / inpaint : 生成パラメータ情報を各タブに送る
- enable edit：パラメータ情報のテキスト編集を有効にする ＜send to ** や save ** のボタンで編集後の情報を各タブや画像に反映できる＞
- save image / thumbnail：パラメータ情報を表示中の画像／サムネイルに保存する ＜ダウンロードは別途右クリック＞

----
### Extras
Stable Diffusion以外のライブラリを主に使用する機能を追加。本GUIではライブラリを呼び出す最低限のインタフェースだけ実装
- [HighRes](#highres)：高解像度化機能。Real-ESRGANを呼び出す
- [DepthMap](#depthmap)：深度推定（画像のデプスマップ生成）。MiDaSを呼び出す
- [Layered Diffusion](#layered-diffusion)：七師 氏が実装した画像生成ライブラリを呼び出す。背景と人物とで別のプロンプトを指定可能など多機能

#### 共通のGUI
![HighRes01](https://github.com/calm-ixia/SDManualGUI/assets/118874552/2c41180e-5915-4ca5-89bb-44ea017577fb)
- upload images：入力ファイルをアップロードする。Colabのポップアップが右のコンソールに出る
- Output files before generating：画像生成前に前回の出力ファイルを消すかどうか。keepで残し、removeで削除。＜別フォルダのファイルを処理後、最後にまとめてダウンロードしたい場合はkeepしておくとよい＞
- Input files after generating：画像生成した後に入力ファイルを消すかどうか。keepで残し、removeで削除。＜同じ入力ファイルを別設定で処理したい場合はkeepしておくとよい＞
- param：処理ごとの設定
- generate ボタン：画像を生成する
- download zip ボタン：出力ファイルをzipファイルにまとめてダウンロードする
- 進捗状況のコンソール（右側） : 進捗状況を表示

※ 出力ファイルをzipファイル1個にまとめるのはColabのダウンロード制限の仕様のため
 
#### 実行のおおまかな流れ
- 実行前にライブラリをインストールする。GUIの上の方にある「高解像度化ライブラリReal-ESRGANの導入」や「深度推定ライブラリMiDaSの導入」のセルを実行
- 入力ファイルをアップロードする
  - upload images ボタンを押すと、右のコンソールにColabの「ファイル選択」ボタンが現れる
  - 現れたファイル選択ボタンを押すと、ファイル選択ダイアログが出るので、アップロードするファイルを選択する
    - 1フォルダ内の複数ファイルを選択できる
    - 複数フォルダのファイルを選択したい場合は、フォルダごとに都度uploadを行うとファイルを追加できる
- モデル指定などの設定を行い generate ボタンを押すと、処理画像が生成される
- download zip ボタンを押すと、出力ファイルがzipファイルとして一括ダウンロードされる

※ アップロード後右のコンソールにメッセージが出るが、セルの再起動は不要。そのまま別のファイルを続けてアップロードしたり、generateで画像生成できる


#### HighRes
![HighRes_complete](https://user-images.githubusercontent.com/118874552/220275726-ea094df9-9ed3-45ad-9a9d-aed0628b56a1.png)

入力画像を高解像度化した画像を生成する。詳細は![Real-ESRGAN](https://github.com/xinntao/Real-ESRGAN)のリンクを参照

- model：処理に使うモデルを指定
- scale：拡大率を指定。最大4.0倍
- face enhance：顔の強調を行う

以下のオプションは未対応。設定値を記載。
- dni_weight = None
- denoise_strength = 1
- tile = 0
- tile_pad = 10
- pre_pad = 0
- half = True #fp16
- gpu_id=None
- suffix = 'out' #出力ファイル名の末尾につける文字列。 foobar_{suffix}.{ext} のようなファイル名になる
- ext = 'auto' # 出力ファイルの拡張子

#### DepthMap

![DepthMap_complete](https://user-images.githubusercontent.com/118874552/220278958-ac98b7bc-561e-4d08-9e34-eeeb800e652d.png)

入力画像の深度推定を行い、デプスマップ画像を生成する。詳細は![MiDaS](https://github.com/isl-org/MiDaS)のリンクを参照

- model：処理に使うモデルを指定。本GUIでは dpt_swin_large_384 を初期値に設定。なおライブラリのデフォルトは dpt_beit_large_512

以下のオプションは未対応。設定値を記載。
- optimize = None #fp32
- height = None
- square = True
- grayscale = True

※ 初回の実行では上の画像の通りWarningが出るが、正常に画像生成される。2回目以降Warningは出ない

#### Layered Diffusion
![Extras_Layered_result](https://github.com/calm-ixia/SDManualGUI/assets/118874552/bd2f5486-88e7-42fa-8890-d3b4277af09d)

七師氏が実装した画像生成ライブラリ Layered Diffusion を呼び出す。Diffusersベースで、背景と人物とで別のプロンプトを指定できる、などの機能がある。
CUIやGUIはなく、pythonスクリプトを書いて実行する。

- initialization code：Pipeline生成などの初期設定を行うコードを記述する。
- initialize：上記の初期設定コードを実行する
- Layered Diffusion code：画像生成や表示・保存を行うコードを記述する。
- generate：上記の生成処理を実行する

注意：initialize, generateともexec()を呼び出すだけであり、**コードのサニタイズ及びバリデーションは行いません。ご自身の責任の元でコードを実行してください。**

#### 実行のおおまかな流れ
1. 初期設定のコードと画像生成用のコードを作成し、各テキストエリアに貼る。
2. initializeを実行し、モデルをロードする。
3. generateを実行し、画像を生成する。必要に応じてパラメータを書き換え、所望の画像を得る。
4. download zipを押下し、画像ファイルをダウンロードする

#### ワンポイント
- initializeは一度だけ、generateは画像生成のたびに実行することを想定している。GUI画像に記載のコードを初期値として表示しているので、コードを書く時の雛型に。
- Layered DiffusionのAPI仕様については以下を参照
  - GitHubのReadmeとソースコード https://github.com/nanashi161382/unstable_diffusion/
  - 七師氏のnote https://note.com/tomo161382
- initialize内でパイプラインのインスタンスを保持できるようにコードを書く。例：self.pipeに代入する。GUI画像に記載のコードを参照。
- 生成画像データとしてはPIL.Imageオブジェクトが得られる。保存などの後処理をする際に参考に。
- 生成画像にプロンプトなどのパラメータ情報を埋め込みたい時は、画像を一旦ダウンロードした後、[PNG Info](#png-info)に画像をアップロードして情報を編集する
 
----
## PNG Info 単品
![PNG Info だけを切り出したノートブック](https://github.com/calm-ixia/SDManualGUI/blob/main/PNGInfo.ipynb) を別に作成しました。こちらはCPUのみで動作します。

![PNGInfo単品](https://user-images.githubusercontent.com/118874552/217167002-b5bcc91b-3993-4b9a-b47b-5cdb19c1ff99.png)

1. Setup セルを実行
2. Launching セルを実行
3. upload an image をクリックし、ファイルを選択

※PNG Info 単品のノートブックで send to ** は出来ません

----
## パラメータの比較
### Denoising strength (img2img, 20 steps)
![img2img_denoising_strength](https://user-images.githubusercontent.com/118874552/217158104-5e92a7c4-d882-4b4f-bcd8-7f8fe47e69ec.png)
※LPW Pipeline 改でDenoising strength = 0.9の画像はsafety checkerに引っかかったが、誤検出であることを目視確認して掲載

[↑img2img](#img2img)

### Masked content (inpaint, Denoising strength = 1.0)
![masked_content1](https://user-images.githubusercontent.com/118874552/217158278-ddd5a9a1-7763-4525-9107-3897e4bf46a3.png)

![masked_content2](https://user-images.githubusercontent.com/118874552/217158287-52b95f8f-b1ef-4755-819c-db8b73c1daeb.png)

[↑inpaint](#inpaint)

### Mask timing (inpaint, LPW Pipelineのみ)
![mask_timing](https://user-images.githubusercontent.com/118874552/217158393-6a3b5460-c8a1-4b57-833d-6ebe84868038.png)

### Mask output (inpaint, LPW Pipelineのみ)
![mask_output](https://user-images.githubusercontent.com/118874552/217158462-897c7060-524f-4c87-abf0-85879f3b080e.png)

[↑inpaint](#inpaint)

----
## 主な使用ライブラリ：
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
- Real-ESRGAN
  - https://github.com/xinntao/Real-ESRGAN
- MiDaS
  - https://github.com/isl-org/MiDaS
- Layered Diffusion
  - https://github.com/nanashi161382/unstable_diffusion/

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
- SkyTNT
  - https://github.com/SkyTNT
- 七師
  - https://note.com/tomo161382

