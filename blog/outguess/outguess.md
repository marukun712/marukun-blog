---
title: outguessでお手軽ステガノグラフィー
description: outguessでお手軽ステガノグラフィー
slug: outguess
authors:
  - name: marukun_
    url: https://github.com/marukun712
    image_url: https://github.com/marukun712.png

tags: [outguess]
hide_table_of_contents: false
---

# outguessでお手軽ステガノグラフィー
![https://i.gzn.jp/img/2013/12/16/mystery-of-cicada-3301/cicada-3_2743174c_m.jpg](https://i.gzn.jp/img/2013/12/16/mystery-of-cicada-3301/cicada-3_2743174c_m.jpg)

# ステガノグラフィとは

> 本体データをダミーデータの中に埋め込んで本体データの存在を隠す技術の総称
> 
「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典より


用は画像の中にテキストファイルを隠して取り出せるようにしたりするような技術のことです。


# 使ってみよう

## 環境

wsl2 ubuntu 20.04.3 LTS


## インストール

```bash
sudo apt install outguess
```

これで完了。


## 埋め込み

まずjpg形式の画像ファイルと埋め込みたいファイルを用意しましょう。

### 使用する画像

![mark_question.jpg](mark_question.jpg)

### 埋め込むファイル

```visual-basic
msgbox "congratulation！",,"メッセージ"
```

メッセージボックスを出すスクリプトを埋め込んでみます。


## 埋め込みコマンド

```bash
outguess -d 画像に埋め込みたいファイル ファイルを埋め込みたい画像ファイル名 完成したファイルの名前
```

 

今回はこのように入力しました

```bash
outguess -d secret.vbs mark_question.jpg secret.jpg
```

↓スクリプトを埋め込んだ画像ファイル

![secret.jpg](secret.jpg)

また、

```bash
outguess -k パスワード -d 画像に埋め込みたいファイル ファイルを埋め込みたい画像ファイル名 完成したファイルの名前
```

このように取り出しの際にパスワードを設定することもできます。

## 取り出し

```bash
outguess -r ファイルが埋め込まれている画像ファイル名 取り出した際に生成されるファイル名
```

```bash
outguess -r secret.jpg secret.vbs
```

今回はこのように取り出しました。ubuntuのhomeフォルダを見ると、

![Untitled](Untitled.png)

ファイルが取り出されているのが分かります。

また、パスワードを設定した際には

```bash
outguess -k 設定したパスワード -r ファイルが埋め込まれている画像ファイル名 取り出した際に生成されるファイル名
```

このようにして取り出せます。



## まとめ

outguessを使えば簡単に画像に別ファイルを埋め込むことができる。



## おまけ

outguessはmacにも対応していて、mac版にはguiが用意されています。

```bash
brew install outguess
```

homebrewでインストールできます。

使い方はguiである程度分かるので割愛します。

