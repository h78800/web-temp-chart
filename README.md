# web-temp-chart
樹莓派建立一個動態伺服器來接收ESP8266收集的DS18B20溫度數據，並在前端網頁上呈現折線圖
專案說明:基於Python3.0版以上虛擬環境venv使用樹莓派建立一個動態伺服器來接收ESP8266收集的DS18B20溫度數據，並在前端網頁上呈現折線圖
一.前置作業(不含燒錄樹梅派OS)
1.樹莓派上檢查 IP 地址
pi@RPI:~ $ hostname -I
2.更改系統語言為繁體中文
pi@RPI:~ $ sudo dpkg-reconfigure locales
3. 重啟
pi@RPI:~ $ sudo reboot
4.更新索引&&升級已安裝軟體
pi@RPI:~ $ sudo apt update && sudo apt upgrade -y
4.1檢查python版本(注意要Python3.0版以上)
pi@RPI:~ $ python --version
5.安裝套件
pi@RPI:~ $ sudo apt install tree     #常用的工具，可以以樹狀結構顯示目錄內容。
pi@RPI:~ $ sudo apt install python3-pip -y   #安裝pip套件
pi@RPI:~ $ sudo apt install sqlite3          # 安裝 SQLite 命令行工具
6.先創建一個項目資料夾
pi@RPI:~ $ mkdir ~/flask_server
7.導航到項目資料夾
pi@RPI:~ $ cd ~/flask_server 
8.在項目資料夾內創建一個虛擬環境
pi@RPI:~/flask_server $ python3 -m venv myenv
9.啟動虛擬環境
pi@RPI:~/flask_server $ source myenv/bin/activate     #啟動後，終端機前面會顯示 (myenv)，代表你已進入虛擬環境。
10.在虛擬環境中安裝這2個套件
(myenv) pi@RPI:~/flask_server $  pip install flask flask-cors
11.確認你的 Python 環境已經內建 SQLite3：
(myenv) pi@RPI:~/flask_server $  python3 -c "import sqlite3; print(sqlite3.sqlite_version)"
#如果成功，應該會顯示 SQLite 的版本號，例如：3.40.1
#如果步驟11. 萬一報錯，你可能需要手動安裝 SQLite資料庫：
sudo apt install sqlite3 libsqlite3-dev
12.安裝pytz 模組來處理時間(同步時間)
(myenv) pi@RPI:~/flask_server $  pip install pytz
13.驗證pytz(版本)
(myenv) pi@RPI:~/flask_server $  pip show pytz
14.執行pip freeze指令生成requirements.txt文檔紀錄虛擬環境中使用套件支版本號(以利遷移時使用)
(myenv) pi@RPI:~/flask_server $ pip freeze >requirements.txt
15.退出虛擬環境
(myenv) pi@RPI:~/flask_server $  deactivate
15.啟動虛擬環境
pi@RPI:~/flask_server $ source myenv/bin/activate

二.建置後端Flask 伺服器(請先確保你在虛擬環境內操作，否則 Flask 可能無法使用。)
1.先確認導航至項目資料夾
(myenv) pi@RPI:~/flask_server $ cd ~/flask_server
2.撰寫主程序腳本(創建伺服器及API)
(myenv) pi@RPI:~/flask_server $ nano app.py
3.在 nano 編輯器中，編輯 Python 代碼
4.app.py (編寫完後按x.y.enter)退出nano             #貼上附件app.py檔案內容
5.測試運行啟動 Flask伺服器
(myenv) pi@RPI:~/flask_server $ cd ~/flask_server   #確保在flask_server資料夾底下運行下列指令
(myenv) pi@RPI:~/flask_server $ python app.py       #如果成功，會顯示ip 端口等資訊
6.啟動成功後再一次退出伺服器
    按Ctrl+C
7.再次啟動 Flask伺服器(測試伺服器API)若無數據可以刪除資料庫再重新啟動 Flask伺服器
(myenv) pi@RPI:~/flask_server $ cd ~/flask_server   #確保在flask_server資料夾底下運行下列指令
(myenv) pi@RPI:~/flask_server $ python app.py       #如果成功，會顯示ip 端口等資訊
8.開啟CMD終端
  上傳溫度數據 允許Curl上傳的格式如下:
1.curl -X POST -d "T1=25.6&T2=26.4&T3=25.3&T4=60.3" http://192.168.2.90:5000/upload
2.curl -X POST -d "T1=25.6&T2=26.4&T3=&T4=" http://192.168.2.90:5000/upload
3.curl -X POST -d "T1=25.6" http://192.168.2.90:5000/upload
4.curl -X POST -d "T2=25.6" http://192.168.2.90:5000/upload
5.curl -X POST -d "T3=25.6" http://192.168.2.90:5000/upload
6.curl -X POST -d "T4=25.6" http://192.168.2.90:5000/upload

  獲取溫度數據
curl -X GET "http://192.168.2.90:5000/data?range=realTime"
  清除所有數據
curl -X POST http://你的樹莓派IP:5000/clear
  
三.建立 Systemd 服務(讓伺服器開機自動重啟)
1.按Ctrl+C退出伺服器
2.編寫Systemd文件
(myenv) pi@RPI3:~/flask_server $ sudo nano /etc/systemd/system/flask_server.service
3.貼上以下內容:

[Unit]
Description=Flask Temperature Server
After=network.target

[Service]
User=pi
WorkingDirectory=/home/pi/flask_server
ExecStart=/home/pi/flask_server/myenv/bin/python /home/pi/flask_server/app.py
Restart=always

[Install]
WantedBy=multi-user.target

#注意[Service]所載路徑是否相符,否則啟動會失敗

4.啟用Systemd 服務並啟動 Flask 伺服器
(myenv) pi@RPI3:~/flask_server $ sudo systemctl daemon-reload
(myenv) pi@RPI3:~/flask_server $ sudo systemctl enable flask_server
(myenv) pi@RPI3:~/flask_server $ sudo systemctl start flask_server
5.查看flask伺服器運行狀態
(myenv) pi@RPI3:~/flask_server $ sudo systemctl status flask_server
6.測試重啟
(myenv) pi@RPI3:~/flask_server $ sudo reboot
7.再次查看flask伺服器運行狀態
pi@RPI3:~ $ sudo systemctl status flask_server

四.創建前端網頁index.html 
1.導航至flask_server資料夾裡
pi@RPI:~/flask_server $ cd ~/flask_server
2.建立一個新的資料夾 static，用來存放網頁檔案
pi@RPI:~/flask_server $ mkdir static
4.導航至static資料夾裡
pi@RPI3:~/flask_server $ cd static
3.編寫index.html文件
pi@RPI3:~/flask_server/static $ nano index.html
4.貼上附件index.html代碼(注意修改成你的<樹梅派ip>:5000)
5.於瀏覽器中開啟網頁
http://<樹梅派ip>:5000

五.Arduino IDE模擬上傳數據
   ESP32.ino
   esp8266.ino

