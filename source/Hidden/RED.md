---
title: "[PicoCTF] RED 解題筆記"
date: 2025-08-09T14:00:00.000Z
categories:
  - PicoCTF
  - 資訊安全
  - Forensics
  - Easy
tags:
  - PicoCTF
  - Forensics
hidden: true
---
[PicoCTF] RED
Category:
Forensics
Difficulty:
Easy
Description:
RED, RED, RED, RED
Download the image:
red.png
題目筆記
題目提供一張全紅色圖片
red.png
先檢查圖片的詳細資訊
使用
exiftool
查看圖片的 Metadata
exiftool
名稱來自於：
EXIF + Tool = Exchangeable Image File Format Tool
使用
zsteg
查看圖片的LSB
“z” + “steg”
= “z”（GPT是說沒什麼特別意義啦…如果有知道的可以留言說一下）+ steg（Steganography） 隱寫術
解題步驟
1. 分析圖片的Metadata
下載完題目給的圖片會得到一張全紅的圖片，我們可以選擇查看他的
詳細資料
我們可以使用linux指令
exiftool
工具解析
我們輸入指令後

```text
1
```

```text
$ exiftool red.png
```

2. 分析完圖片後
我們得到這圖片的詳細資料 會發現有一欄特別奇怪

```text
1
2
3
4
5
6
7
8
9
10
```

```text
Poem: 
    Crimson heart, 
    vibrant and bold,.
    Hearts flutter at your sight..
    Evenings glow softly red,.
    Cherries burst with sweet life..
    Kisses linger with your warmth..
    Love deep as merlot..
    Scarlet leaves falling softly,.
    Bold in every stroke.
```

我們合理猜測她是藏頭詩
CHECKLSB
3. 查看圖片的LSB資訊
可以使用
zsteg
這個工具查看圖片的LSB資訊（Least Significant Bit，最低有效位元）
使用zsteg指令

```text
1
```

```text
$ zsteg red.png
```

會得到好幾條資訊，我們可以看到一條像是Flag的資訊

```text
1
2
```

```text
b1,rgba,lsb,xy.. text: 
"cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ==cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ=="
```

觀察發現這是一長串
Base64 編碼
字串（而且重複了好幾次）。
4. 取出並解碼 Base64
我們只需要解碼一個即可：

```text
1
```

```text
$ 
echo
 
"cGljb0NURntyM2RfMXNfdGgzX3VsdDFtNHQzX2N1cjNfZjByXzU0ZG4zNTVffQ=="
 | 
base64
 -d
```

成功獲取 Flag
恭喜你拿到 Flag 了！
補充說明
exiftool 可以檢查圖片的 Metadata
LSB（最低有效位元）隱寫是常見的圖片隱藏資料方法
zsteg 是專門針對 PNG / BMP 格式的 LSB 分析工具
Base64 編碼在 CTF 中很常用來隱藏文字資料
