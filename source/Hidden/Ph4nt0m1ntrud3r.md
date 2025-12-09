---
title: "[PicoCTF] Ph4nt0m 1ntrud3r 解題筆記"
date: 2025-08-09T13:30:00.000Z
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
[PicoCTF] Ph4nt0m 1ntrud3r
Category:
Forensics
Difficulty:
Easy
Description:
A digital ghost has breached my defenses, and my sensitive data has been stolen! 😱💻 Your mission is to uncover how this phantom intruder infiltrated my system and retrieve the hidden flag.
To solve this challenge, you’ll need to analyze the provided PCAP file and track down the attack method. The attacker has cleverly concealed his moves in well timely manner. Dive into the network traffic, apply the right filters and show off your forensic prowess and unmask the digital intruder!
Find the PCAP file here Network Traffic PCAP file and try to get the flag.
題目筆記
題目提供了一份 PCAP 檔案（網路封包擷取檔案）
需要使用 Wireshark 或 tshark 等工具打開分析
題目提示(Tips)強調：
時間（Time）非常重要
，因此先依時間排序封包
注意封包長度（Len）變化，發現部分封包長度由 8 變成 12，很可能是有用資料
將 Len = 12 的 TCP payload 的十六進制資料複製下來
解題步驟
1. 打開 PCAP 檔案並排序封包
使用 Wireshark 打開 PCAP
依時間（Time）欄排序，觀察封包流動
2. 找出 Len = 12 的 TCP payload
篩選出 TCP payload 長度為 12 的封包
複製所有此類封包的十六進制 Payload，得到多組 Hex 編碼：

```text
1
2
3
4
5
6
7
```

```text
63476c6a62304e5552673d3d
657a46305833633063773d3d
626e52666447673064413d3d
587a4d3063336c6664413d3d
596d68664e484a664d673d3d
5a54466d5a6a41324d773d3d
66513d3d
```

3. 轉換 Hex 到 ASCII 字串
將上面 Hex 解碼為 ASCII，得到一串 Base64 字串：

```text
1
2
3
4
5
6
7
```

```text
cGljb0NURg==
ezF0X3c0cw==
bnRfdGg0dA==
XzM0c3lfdA==
YmhfNHJfMg==
ZTFmZjA2Mw==
fQ==
```

4. 對 Base64 進行解碼
將上述的 Base64 進一步解碼，就會得到答案了！
有關各種編碼方式歡迎查看這篇文章 :
常見編碼方式一覽
成功獲取 Flag
恭喜你拿到 Flag 了！
picoCTF{1t_w4snt_th4t_34sy_tbh_4r_2e1ff063}
補充說明
PCAP 是用來記錄網路封包的格式
Wireshark 是分析網路封包的工具
時間排序和封包長度是關鍵的線索
Base64 編碼與解碼是常見資料隱藏技術
