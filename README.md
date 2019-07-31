目標
====
Rasberry Pi 3 model BにUbuntu MATEを入れ、デジタルマイク[SPH0645LM4H](https://www.switch-science.com/catalog/3207/)とI2S通信する。

インストール手順
================
0. 公式ページ(https://ubuntu-mate.org/download/ )からUbuntu MATEをダウンロードし、Rasberry Pi 3 model Bに書き込む。今回使用したのは、ubuntu-mate-18.04.2-beta1-desktop-armhf+raspi-ext4.img。

1. /boot/config.txtに以下を追加で書き込む。
```
dtparam=i2s=on
```
2. /etc/modulesに以下を追加で書き込む。
```
snd-bcm2835
```
3. リブートする。
```
sudo reboot
```
4. snd_bcm2835などのモジュールがロードされていることを確認。
```
lsmod | grep snd # snd_bcm2835 などが表示されるはず。
```
5. linux imageのアップデートおよびlinux headerのインストール。
```
sudo apt update
sudo apt install linux-image-raspi2 # 今回インストールしたバージョン：linux-image-raspi2/bionic-updates,bionic-security,now 4.15.0.1041.39 armhf
sudo apt install linux-headers-raspi2 # 今回インストールしたバージョン：linux-headers-raspi2/bionic-updates,bionic-security,now 4.15.0.1041.39 armhf
```
6. リブートする。
```
sudo reboot
```
7. i2sをコンパイルする準備ができていることを確認。
```
sudo mount -t debugfs debugs /sys/kernel/debug # mount: debugs already mounted or mount point busy. などと表示されるはず。
```
8. カーネルモジュールをmakeするためのパッケージをインストール。
```
git clone https://github.com/PaulCreaser/rpi-i2s-audio
cd rpi-i2s-audio
```
9. カーネルモジュールをmakeして作成し、カーネル本体にロードする。
```
sudo apt install make gcc git bc libncurses5-dev bison flex libssl-dev
make -C /lib/modules/$(uname -r )/build M=$(pwd) modules
sudo insmod my_loader.ko
```
10. audioグループに自分のユーザを追加する。
```
sudo adduser $USER audio
```
11. ユーザの追加を反映させるため、ログアウトしてログインする。
12. Rasberry Piをブートするたびに、今回作成したカーネルモジュールを自動でロードできるようにする。
```
cd rpi-i2s-audio
sudo cp my_loader.ko /lib/modules/$(uname -r)
echo 'my_loader' | sudo tee --append /etc/modules > /dev/null # 先ほど書いたsnd-bcm2835と改行されて書かれているか確認する。
sudo depmod -a
sudo modprobe my_loader
```
13. リブートして設定完了。
```
sudo reboot
```

テスト
======
・マイクデバイスの一覧を表示し、「snd_rpi_simple_card」があることを確認。
```
arecord -l
```
・arecordというコマンドから音声を保存する。ノートPCなどで再生できればテスト成功！
```
arecord -D plughw:1 -c2 -r 48000 -f S32_LE -t wav -V stereo -v file_stereo.wav
```
・pyaudioからも読める。
```
import pyaudio
p = pyaudio.PyAudio()
for index in range(p.get_device_count()):
    print(p.get_device_info_by_index(index)['name'])
    print('')
```


参考
====
- https://tomosoft.jp/design/?p=11471
- https://learn.adafruit.com/adafruit-i2s-mems-microphone-breakout/raspberry-pi-wiring-and-test

（これらのサイトの環境はRasbianだが自分の環境はUbuntu MATEなので、細かい違いがある）
