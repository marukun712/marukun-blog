---
title: YouTubeAPIでコメントからタイムスタンプを取得してみた
description: YouTubeAPIでコメントからタイムスタンプを取得してみた
slug: youtubeapi-vtuber
authors:
  - name: marukun_
    url: https://github.com/marukun712
    image_url: https://github.com/marukun712.png

tags: [YouTubeAPI, vtuber]
hide_table_of_contents: false
---
# はじめに
Vtuberなどの歌枠アーカイブを見ていると、「あの曲どの配信で歌ってたっけ...」となる時ありますよね。
そんな時にそのVtuberが歌った歌を一覧で表示出来ると便利だな、と思い作ってみました。
今回は、VERSEⁿ所属のアルバ・セラさんの歌枠配信からタイムスタンプを取得してみます。

https://www.youtube.com/@albasera2426

格好いい曲も可愛らしい曲も歌いこなす、おすすめのVsingerです。

# 作ったもの
先にコード全体を貼っておきます。
```javascript:comment.js
var apikey = 'APIキー';
import fetch from 'node-fetch';
var list = 'PLhu18ozRJ5d3XLfoeUw6WQxGJAydvC-iL'
import fs from 'fs';

var result = []

async function GetId() {
    try {
        let res = await fetch(`https://www.googleapis.com/youtube/v3/playlistItems?key=${apikey}&playlistId=${list}&part=snippet&maxResults=50`)
        let json = await res.json();
        let items = await json.items

        await items.map(item => {
            let id = item.snippet.resourceId.videoId
            let title = item.snippet.title
            let image = item.snippet.thumbnails.default
            GetTimeStamp(
                {
                    "id": id,
                    "title": title,
                    "image": image
                }
            )
        })

    }
    catch (e) {
        console.log(e)
    }
}

async function GetTimeStamp(props) {
    try {
        let res = await fetch(`https://www.googleapis.com/youtube/v3/commentThreads?key=${apikey}&part=snippet&videoId=${props.id}`)
        let json = await res.json();
        let items = await json.items

        await items.map(item => {
            let comment = item.snippet.topLevelComment.snippet.textOriginal
            let body = (String(comment).match(/[0-9]{1,}:[0-9]{1,}:[0-9]{1,}(.*)(.*)|[0-9]{1,}:[0-9]{1,}(.*)(.*)/gi));
            let time = (String(comment).match(/[0-9]{1,}:[0-9]{1,}:[0-9]{1,}|[0-9]{1,}:[0-9]{1,}/gi))
            let title = (String(body).replace(/[0-9]{1,}:[0-9]{1,}:[0-9]{1,}|[0-9]{1,}:[0-9]{1,}/gi, "")).split(',');

            if (title.length > 5) {
                let stamp = []

                for (let i = 0; i < time.length; i++) {
                    stamp.push({
                        "item": {
                            'time': time[i],
                            'title': title[i],
                        }
                    })
                }

                result.push({
                    "item": {
                        'timestamp': stamp,
                        'id': props.id,
                        'videotitle': props.title,
                        'image': props.image,
                        'url': `https://www.youtube.com/watch?v=${props.id}`
                    }
                })
            }
        })


        fs.writeFile('data.json', JSON.stringify(result, null, '    '), (err) => {
            if (err) console.log(`error!::${err}`);
        });
    } catch (e) {
        console.log(e)
    }
};

GetId();
```
出力結果
```json
[
    {
        "item": {
            "timestamp": [
                {
                    "item": {
                        "time": "0:06:02",
                        "title": "\tエゴイスト / アルバ・セラ"
                    }
                },
                {
                    "item": {
                        "time": "0:12:07",
                        "title": "\tvivid a / Lanndo feat.bis"
                    }
                },
                {
                    "item": {
                        "time": "0:20:00",
                        "title": "\t東京テディベア / Neru"
                    }
                },
                {
                    "item": {
                        "time": "0:28:31",
                        "title": "\t流刑地 / Gyoson"
                    }
                },
                {
                    "item": {
                        "time": "0:32:28",
                        "title": "\tStory of Hope / ゆよゆっぺ"
                    }
                },
                {
                    "item": {
                        "time": "0:41:52",
                        "title": "\t文学少年の憂鬱 / ナノウ"
                    }
                },
                {
                    "item": {
                        "time": "0:53:10",
                        "title": "\tBloom / アルバ・セラ"
                    }
                },
                {
                    "item": {
                        "time": "0:25:17",
                        "title": "\t🎸 よしさん "
                    }
                },
                {
                    "item": {
                        "time": "0:25:37",
                        "title": "\t🎹 ふーみんさん"
                    }
                },
                {
                    "item": {
                        "time": "0:26:00",
                        "title": "\t🥁 むねさん"
                    }
                },
                {
                    "item": {
                        "time": "0:26:16",
                        "title": "\t🎤 アルバ・セラ"
                    }
                }
            ],
            "id": "GyjgPdWDtag",
            "videotitle": "【誕生日LIVE】Bloom | BIRTHDAY ACOUSTIC LIVE【アルバ・セラ/VERSEⁿ】",
            "image": {
                "url": "https://i.ytimg.com/vi/GyjgPdWDtag/default.jpg",
                "width": 120,
                "height": 90
            },
            "url": "https://www.youtube.com/watch?v=GyjgPdWDtag"
        }
    },
    以下略...
```

# どうやって実装したか
YouTubeAPIを使って、プレイリストから歌枠配信の動画IDをすべて取得、そのIDを使ってすべての歌枠配信のコメントデータを取得、
正規表現でコメントデータからタイムスタンプコメント(こういうヤツ)
```:タイムスタンプコメントの例
00:00 配信開始
05:00 一曲目
10:00 二曲目
15:00 エンディング
```
だけを取得して曲名と動画時間をjsonファイルに保存しました。
### ライブラリの読み込み等
APIを叩くために利用するnode-fetchや結果をjsonファイルに保存するために使うfsを読み込みます。
YouTubeAPIのAPIキー、プレイリストID、タイムスタンプデータなどが入るresult配列も定義しておきます。
```javascript
var apikey = 'APIキー';
import fetch from 'node-fetch';
var list = 'PLhu18ozRJ5d3XLfoeUw6WQxGJAydvC-iL'
import fs from 'fs';

var result = []
```
### IDを取得
アルバ・セラさんは歌枠プレイリストを作成してくれていたので、
YouTubeAPIの[PlaylistItems](https://developers.google.com/youtube/v3/docs/playlistItems/list?hl=ja)を利用してプレイリスト内の動画ID、タイトル、サムネイルを取得します。
```javascript
async function GetId() {
    try {
        let res = await fetch(`https://www.googleapis.com/youtube/v3/playlistItems?key=${apikey}&playlistId=${list}&part=snippet&maxResults=50`)
        let json = await res.json();
        let items = await json.items

        await items.map(item => {
            let id = item.snippet.resourceId.videoId
            let title = item.snippet.title
            let image = item.snippet.thumbnails.default
            GetTimeStamp(
                {
                    "id": id,
                    "title": title,
                    "image": image
                }
            )
        })

    }
    catch (e) {
        console.log(e)
    }
}
```
そして、その結果を引数として後述するGetTimeStamp関数を呼び出しています。
### コメントとタイムスタンプコメントを取得
取得したIDを元に[CommentThreads](https://developers.google.com/youtube/v3/docs/commentThreads/list)を利用してプレイリスト内の全ての動画に投稿されたコメントデータを取得します。
その後、取得したコメントデータから正規表現を利用してタイムスタンプコメントのみを取得しています。
最後に、タイムスタンプデータと動画ID、タイトル、サムネイルなどが入ったデータをjsonファイルに保存しています。
```javascript
async function GetTimeStamp(props) {
    try {
        let res = await fetch(`https://www.googleapis.com/youtube/v3/commentThreads?key=${apikey}&part=snippet&videoId=${props.id}`)
        let json = await res.json();
        let items = await json.items

        await items.map(item => {
            let comment = item.snippet.topLevelComment.snippet.textOriginal
            let body = (String(comment).match(/[0-9]{1,}:[0-9]{1,}:[0-9]{1,}(.*)(.*)|[0-9]{1,}:[0-9]{1,}(.*)(.*)/gi));
            let time = (String(comment).match(/[0-9]{1,}:[0-9]{1,}:[0-9]{1,}|[0-9]{1,}:[0-9]{1,}/gi))
            let title = (String(body).replace(/[0-9]{1,}:[0-9]{1,}:[0-9]{1,}|[0-9]{1,}:[0-9]{1,}/gi, "")).split(',');

            if (title.length > 5) {
                let stamp = []

                for (let i = 0; i < time.length; i++) {
                    stamp.push({
                        "item": {
                            'time': time[i],
                            'title': title[i],
                        }
                    })
                }

                result.push({
                    "item": {
                        'timestamp': stamp,
                        'id': props.id,
                        'videotitle': props.title,
                        'image': props.image,
                        'url': `https://www.youtube.com/watch?v=${props.id}`
                    }
                })
            }
        })


        fs.writeFile('data.json', JSON.stringify(result, null, '    '), (err) => {
            if (err) console.log(`error!::${err}`);
        });
    } catch (e) {
        console.log(e)
    }
};
```
# おまけ
このツールで取得したデータを使ってアルバ・セラさんの歌枠を音楽アプリ風に再生できるwebアプリを作成してみました。

https://sera-music.vercel.app/

# 参考文献
https://zenn.dev/etrnl_tamayura/articles/youtube-timestamp

https://developers.google.com/youtube/v3/docs/commentThreads/list

https://developers.google.com/youtube/v3/docs/playlistItems/list?hl=ja
