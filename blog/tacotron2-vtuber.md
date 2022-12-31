---
title: tacotron2でVtuberの音声合成をしてみる
description: tacotron2でVtuberの音声合成をしてみる
slug: tacotron2-vtuber
authors:
  - name: marukun_
    url: https://github.com/marukun712
    image_url: https://github.com/marukun712.png

tags: [tacotron2, vtuber]
hide_table_of_contents: false
---

# はじめに　
こんにちは。
今回は、tacotron2を使った音声合成に挑戦してみます。

# tacotron2とは
tacotron2はGoogle社で開発された、テキストから音声に変換するためのアルゴリズムです。
テキストをメルスペクトログラムに変換し、メルスペクトログラムから音声に変換することで音声を作成できます。
tacotron2はテキストをメルスペクトログラムに変換する部分を行い、メルスペクトログラムから音声に変換する部分はWaveGlowというアルゴリズムが行います。
# 環境
- Python3.6.2
- OS:windows11
- CPU:Ryzen5 5600x 6-Core
- GPU:RTX3060
- RAM16GB
- CUDA11.8
# tacotron2の導入
tacotron2本体をclone
```
$ git clone https://github.com/NVIDIA/tacotron2
```
submodule(WaveGlowなど)の導入
```
$ git submodule init; git submodule update
```
PyTorchをインストール
```
$ pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu117
```
Apexをインストール
```
$ git clone https://github.com/nvidia/apex
$ cd apex
$ pip install -v --disable-pip-version-check --no-cache-dir --global-option="--cpp_ext" --global-option="--cuda_ext" ./
```
依存ライブラリのインストール
```
$ pip install -r requirements.txt
```
### 日本語データを学習させる
以下の記事を参考に、「[Japanese Single Speaker Speech Dataset](https://www.kaggle.com/datasets/bryanpark/japanese-single-speaker-speech-dataset)」を学習させました。

https://note.com/npaka/n/n2a91c3ca9f34


日本語データの学習も済んだので、いよいよVtuberの音声を学習させます。
今回は、Sony Musicによる、バーチャルタレント育成&マネジメントプロジェクト「VEE」所属の、九条林檎さんの音声を学習させていきます。


https://www.youtube.com/c/KujoRingo/about
# データの用意　
### 音声データの用意
九条林檎さんが週に3回行っているお昼のラジオ番組「おはようマルクホルテ noon」直近2回分の音声を学習させます。


https://www.youtube.com/watch?v=MULmlU81Mao
### 使用ツール
### yt-dlp 

https://github.com/yt-dlp/yt-dlp


### spleeter


https://github.com/deezer/spleeter

### FFmpeg
https://ffmpeg.org/

### audacity

https://www.audacityteam.org/

```
$ yt-dlp -f best https://www.youtube.com/watch?v=MULmlU81Mao -o "input.mp4"
$ yt-dlp -f best https://www.youtube.com/watch?v=WkzmYscgrCA -o "input2.mp4"

$ ffmpeg -i input.mp4 output.mp3
$ ffmpeg -i input2.mp4 output2.mp3
```
yt-dlpで動画をダウンロード後、ffmpegでmp4からmp3に変換、変換した音声をaudacityで20分毎に区切り、
spleeterで音声とBGMを分離します。
```
$ spleeter separate -d 音声ファイルの秒数 -o 出力先フォルダのパス 音声のパス 
```
分離後、音声ファイルをすべてaudacityで読み込み、
「音声から自動ラベル付け」機能で無音部分で分割します。
audacityの書き出し時に、エンコーディングを「Signed 16-bit PCM」に指定します。
### 字幕データの用意
音声認識で字幕ファイルを作成します。
```python:text.py
import speech_recognition as sr
import pyopenjtalk
import speech_recognition as sr

r = sr.Recognizer()
f = open('corpusfilepath.txt', 'w')

for num in range(1796):
    s = f'{num:02}'
    print(s)
    try:
        filepath = "clip/wav-" + str(s) +".wav"
        with sr.AudioFile(filepath) as source:
            audio = r.record(source)
 
        text = r.recognize_google(audio, language='ja-JP')
        print(filepath + text)
        f.write(str(filepath) + "|" + pyopenjtalk.g2p(text, kana=False).replace('pau',',').replace(' ','') + '.' + "\n") 

    except:
        print('speech recognition failed.')

f.close()
```
range関数の引数は音声ファイルの数に置き換えてください。

生成されたデータ
```text:corpusfilepath.txt
ringo/wav-01.wav|hoNjitsuwazeNbushudoodeyarimasUnode,motatsukUkotootataarimasUga,okininasarazu.
ringo/wav-03.wav|oopuniNgu,kuNkuNwadokoda.
ringo/wav-04.wav|anokowaugokimaseNne,yoisho.
ringo/wav-05.wav|esUte.
ringo/wav-06.wav|esUte.
ringo/wav-17.wav|yoclshaa.
ringo/wav-18.wav|yoisho.
ringo/wav-21.wav|kaubooi,koraclse.
ringo/wav-24.wav|yoisho.
ringo/wav-26.wav|imamadebotaNhItotsudeyaclteitakotogazeNbudekinakunaclteshimacltayo,shashiNde.
ringo/wav-27.wav|oreo.
ringo/wav-32.wav|koreo.
ringo/wav-33.wav|koreo.
ringo/wav-35.wav|koreokooshIte.
ringo/wav-36.wav|koocha.
ringo/wav-37.wav|hooka.
ringo/wav-38.wav|sabitarazeNbushudoodeugokasuno.
ringo/wav-39.wav|moshi,yooioshIte.
ringo/wav-42.wav|dotenokamisama.
ringo/wav-43.wav|saahajimarimashIta,ohirunorajiobaNgumi.
ringo/wav-44.wav|ohayoomaruku,horuteraigaazero,poiNtosaNbaigaheru,maruku,horute,yuuchuubu,nitekookaisUtajiokaranamahoosoochuu,uketsUketoniNgeNnotareNto,dokUsho,riNgoda.
ringo/wav-45.wav|konobaNguminohaclshutaguwaoharute.
ringo/wav-46.wav|ohaka,hiraganadetegakatahoo,tsuicltaa,denotsuiitonadonikatsuyoosurutoyoi.
ringo/wav-47.wav|biitowa,yuuchuubu,haishiNnogameNkabuniteshookaishIteiruzo.
ringo/wav-48.wav|hoosoonoaakaibuwakonohoohoogayureru,hoomunopureirisUtokaramirukotogadekirukomeNto,hirowanai,gaNbaclteiru,miteirunodedoozoyoroshIkutanomu.
ringo/wav-49.wav|sate,toiuwakede,puraguiNgairoirotsUkaenakunaru.
ringo/wav-50.wav|habu.
...
ringo/wav-1792.wav|iclteraclshai.
```
# 音声合成
## 学習
### データの配置
tacotron2フォルダに移動し、
tacotron2フォルダに分割した音声ファイルのあるフォルダを配置、
tacotron2/filelists に字幕データを配置します。
### データの分割
字幕データを学習データと検証データに分割します。
```
$ head -n 1040 filelists/corpusfilepath.txt > filelists/corpusfilepath_train.txt
$ tail -n 259 filelists/corpusfilepath.txt > filelists/corpusfilepath_val.txt
```
### ハイパーパラメータの編集
hparams.pyを編集します。
training_filesとvalidation_filesにそれぞれ学習データと検証データのパスを記述します。
今回はエポック数を500回、text_cleanersをbasic_cleanersにしました。

### 学習の実行
いよいよ学習を実行します。
```
$ python train.py --output_directory=ringo_out --log_directory=logdir -c outdir/checkpoint_27000 --warm_start
```
今回は事前に学習させたJapanese Single Speaker Speech Datasetから転移学習しました。
## 推論 
inference.ipynbを順に実行して推論します。
checkpoint_path にチェックポイントファイルのパスを記述し、
text に読ませたい文章を記述します。
29000ステップの学習で、このような音声が生成できました。
「ごきげんよう、九条林檎だ。」
https://www.dropbox.com/s/4tlgv09ovwkqnf3/sample.wav?dl=0
イントネーションがおかしい部分もありますが、かなりクオリティの高い音声が生成できたと思います。
# 参考文献

https://note.com/npaka/n/n2a91c3ca9f34

https://qiita.com/sujoyu/items/8b786b35d20f497507c5

https://heartstat.net/2022/04/10/python_speech-synthesis_myvoice/

https://tosaka-mn.hatenablog.com/entry/2020/02/13/215656
