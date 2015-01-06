#Intel Edisonのメモ書き

【注】個人的なメモ書きを公開しています．不正確な情報もあると思いますので，自己責任でお願いします．(2014/01/07)

ロボットは小型化が求められますが，無線LANを搭載していながら，驚異的に小さなコンピュータが現れました．遠隔操作ロボットのブレークスルーになりそうです．
時間がかかっても良ければ，画像処理もこの中でできそうです．  
http://www.intel.co.jp/content/www/jp/ja/do-it-yourself/edison.html

ただし，A/Dコンバータなどを搭載していませんので，その点では制御用モジュールというより，知能化モジュールと読んだほうが良さそうなユニットです．
なお，これらが必要な場合は，Arduinoのように使用できるGellieoボードもあります．  
http://www.intel.co.jp/content/www/jp/ja/do-it-yourself/galileo-maker-quark-board.html

---

##１）用意したEdison
Intel Edison Breakout Board Kit  
http://akizukidenshi.com/catalog/g/gM-08570/

##２）パソコンにドライバを入れる
Windows Driver setup 1.0.0を予めインストール  
https://communities.intel.com/docs/DOC-23242

##３）準備
MicroUSBケーブル２本でパソコンと接続，１本は（電源＋通信），１本はターミナル用（115200bps）  
Edison起動開始，ターミナルには起動の様子が表示される．  
ちなみに，ターミナルソフトにはTeratermを使用．

##４）立ち上がったところで早速OSの入れ替え
普段使用しているUbuntuとできるだけ環境を一緒にしたいので，標準で入っているYoctoLinuxからubilinuxに変更する．  
以下からダウンロードできる．  
http://www.emutexlabs.com/ubilinux  
これを展開しておく

##５）展開したフォルダの中にdfu-util.exeを入れる．
ソースコードはこちら  
http://dfu-util.sourceforge.net/  
コンパイル済みのものも探すと見つかりますが，自己責任で

##６）OSの入れ替え
あとの手続きは，こちらを参照  
http://www.emutexlabs.com/ubilinux/29-ubilinux/218-ubilinux-installation-instructions-for-intel-edison  
ざっと述べると，

コマンドプロンプトを立ち上げて，toFlashフォルダに移動して，以下を入力

    flashall.bat

USBケーブル（電源側）を抜いて，再度差し込む  
コメントが流れるので，それに従いしばらく待つ.

##７）ターミナルの方がログイン画面になるので，以下を入力

    ubilinux logoin: root  
    password: edison

これで，いつも使い慣れたDebianの環境となる．  
ただし，Edisonの特徴であるPWM,GPIO,SPI通信などハードウェアを直接操作するライブラリは，まだ使用できない．

ちなみに，リアルタイム制御を行うことをひとつの目的としているが，ダウンロードしたUbilinuxのカーネルにはどうもリアルタイム性を高めるためのパッチをもともと当ててあるようである．（推測を含む）

必要に応じてカーネルをビルドして，入れ替えることもできる．

##８）無線LANの設定
ターミナルから，以下を入力

    vi /etc/network/interfaces

各無線LANの環境に応じて変更

サンプル(WEP)

    auto wlan0  
    iface wlan0 inet dhcp
      # For WPA
      #wpa-ssid Emutex
      #wpa-psk passphrase
      # For WEP
      wireless-essid 【ESSID】
      wireless-mode open
      wireless-key s:【パスワード】

【】は環境に応じて変更

##９）無線LANの起動

    ifup wlan0

pingなどを使用して，無線LANに接続されたかを確認します．

##１０）mraaライブラリのインストール
edisonにはハードウェアを操作するためのAPIが用意されており，その解説はこちら
http://iotdk.intel.com/docs/master/mraa/index.html

これをインストールして使えるようにする．

ターミナルから以下を入力

    apt-get install python2.7-dev git cmake swig ntp
    git clone https://github.com/intel-iot-devkit/mraa.git
    cd mraa
    mkdir build
    cd build
    cmake .. -DBUILDSWIGNODE=OFF -DCMAKE_INSTALL_PREFIX:PATH=/usr
    make
    make install
    make clean

##１１）サンプルをコンパイル

    cd ../examples
    gcc -o helloedison helloedison.c -lmraa
    ./helloedison

他にもサンプルがありますので，ここでI/O関連の使い方が分かると思います．


##■参考　Edisonの情報

１）Intelのページ  
http://www.intel.co.jp/content/www/jp/ja/do-it-yourself/edison.html

２）スイッチサイエンスのページ  
http://trac.switch-science.com/wiki/IntelEdison

３）書き込めなくなったときのツール  
http://sourceforge.net/projects/xfstk/
