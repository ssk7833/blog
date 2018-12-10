title: Raspberry pi 開機自動開啟 terminal 並執行 python 程式
date: 2018-12-10 14:41:36
categories:
- study
tags:
- Raspberry pi
- python
permalink: raspberry-pi-autostart-lxterminal-and-run-python
---
最近我有個需求是當我的樹莓派重開機後，要在自動登入 LXDE GUI session 後自動執行終端機 terminal 並執行特定程式，但發現其實並不簡單。
由於受限於動作在登入 GUI session 後，`crontab` 及 `/etc/rc.local` 都不適用我的情境，必須用 `lxsession` 才行，經過了多次的登入登出總算成功達到想要的目的，在此作為紀錄。

在底下假設自動登入使用者為 pi，且 Python 程式放於 `/home/pi/Desktop/myScript/main.py`，可以編輯 `/home/pi/.config/lxsession/LXDE/autostart` 文件，於最底下加上 `lxterminal --command="/bin/bash -c 'cd /home/pi/Desktop/myScript ; python main.py'"`，會發現這行直接於 terminal 執行是沒問題的，但是**在 autostart 會錯**，網路上也有人在問[一樣的問題](https://raspberrypi.stackexchange.com/q/89203)。

後來輾轉發現有人將 command 包在 script 中執行後好像可以，我自己試了也發現這樣就行了，`/home/pi/autostart.sh`：
```bash
#!/bin/bash

cd /home/pi/Desktop/myScript
python main.py
```
記得給 autostart.sh 執行權限：`chmod +x /home/pi/autostart.sh`
並於 `/home/pi/.config/lxsession/LXDE/autostart` 中新增 `lxterminal --command="/home/pi/autostart.sh"`，接著即可重開機／登入登出測試運作情況。

參考資料：
1. [bash - open a terminal on boot and auto-run a looping python script - Raspberry Pi Stack Exchange](https://raspberrypi.stackexchange.com/q/89203)
2. [How to open a new terminal and run commands in it - Raspberry Pi Forums](https://www.raspberrypi.org/forums/viewtopic.php?t=169200)
3. [HowTo auto-run LXTerminal from LXDE desktop  at startup - Raspberry Pi Forums](https://www.raspberrypi.org/forums/viewtopic.php?t=65607)
4. [How to autostart a program on Raspbian? - Raspberry Pi Forums](https://www.raspberrypi.org/forums/viewtopic.php?t=132637)
5. [terminal - How can I open a lxterminal with a script running in it? - Super User](https://superuser.com/q/1281509/617008)
