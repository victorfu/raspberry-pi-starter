Installation Guide of Rasberry Pi 3 Applications
===


## Installation of Nodejs and Node-red

Remove all existing packages
```sh
$ sudo apt-get remove nodered
$ sudo apt-get remove nodejs nodejs-legacy
$ sudo apt-get remove npm
```

Install nvm
```sh
$ curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.1/install.sh | bash
```

You may need to execute code as a privileged user (`root`) using `sudo` in order to access to GPIO or privileged ports (`80`, `443`). Without this you may see an error like this:

```sh
$ sudo node
sudo: node: command not found
```

On way to fix this is by creating an alias, usually in `~/.profile`:

```
alias sudo='sudo env PATH=$PATH:$NVM_BIN'
```
Source: https://github.com/creationix/nvm/issues/43#issuecomment-139739406
Now you can run node using `sudo`.

Install nodejs
```sh
$ nvm install 6.11.1
```

Install pm2
```sh
$ npm install -g pm2
```

Install node-red
```sh
$ npm install -g --unsafe-perm node-red
```

Check the location of node-red binary
```sh
$ whereis node-red
node-red: /home/pi/.nvm/versions/node/v6.11.1/bin/node-red
```

Start node-red by pm2
```sh
$ pm2 start /home/pi/.nvm/versions/node/v6.11.1/bin/node-red -- -v
```

Tell PM2 to run on boot

PM2 is able to generate and configure a startup script suitable for the platform it is being run on.
Run these commands and follow the instructions it provides:
```sh
$ pm2 save
$ pm2 startup
```

## Installation of Node-red security
```sh
$ npm install -g node-red-admin
$ node-red-admin hash-pw
```
This will prompt you for a password and give you the bcrypt encrypted version of your password. Copy this string for later.

```sh
$ vim ~/.node-red/settings.js
```

In this file, you should find an option called adminAuth that is commented out. It looks something like this:
```sh
adminAuth: {
    type: "credentials",
    users: [{
        username: "admin",
        password: "$2a$08$zZWtXTja0fB1pzD4sHCMyOCMYz2Z6dNbM6tl8sJogENOMcxWV9DN.",
        permissions: "*"
    }]
}
```
Uncomment this, paste your bcrypt string in the password field

## Installation of PCSC

Instal the PCSC C-libraries
```sh
$ sudo apt-get install libtool
$ sudo apt-get install libusb-1.0-0-dev
$ sudo apt-get install flex
$ sudo apt-get install automake
$ cd /usr/src
$ sudo apt-get install libudev-dev
$ sudo git clone git://anonscm.debian.org/pcsclite/PCSC.git
$ cd PCSC
$ sudo ./bootstrap
$ sudo ./configure
$ sudo make
$ sudo make install
```

Instal the CCID C-libraries

Edit the file CCID/readers/supported_readers.txt, add the line:
0x1FC9:0x0102:InfoThink IT-102MU Reader
```sh
$ cd /usr/src
$ sudo git clone --recursive git://anonscm.debian.org/pcsclite/CCID.git
$ cd CCID
$ sudo ./bootstrap
$ sudo ./configure
$ sudo make
$ sudo make install
$ sudo cp src/92_pcscd_ccid.rules /etc/udev/rules.d
```

Test the pcsc & ccid installation
```sh
$ sudo killall pcscd -9
$ sudo LIBCCID_ifdLogLevel=0x000F pcscd --foreground --debug --apdu --color
```

Start pcscd at boot
```sh
$ sudo systemctl enable pcscd.service
```

Install pcsclite nodejs wrapper
```sh
$ sudo apt-get install libpcsclite1 libpcsclite-dev
$ cd ~/.node-red
$ npm install buffertools
$ npm install pcsclite
```

Install apdu2pcsc node-red node
```sh
$ cd ~/.node-red
$ npm install node-red-contrib-apdu2pcsc
```

##Reading NFC (Near Field Communication)
Details of ATR String

```sh
ATR = 3B 8F 80 01 80 4F 0C A0 00 00 03 06 03 00 03 00 00 00 00 68
```

What does this hexidecimal string of number means?

`0x3B`, TS, Direction convention

`0x8F`, T0

`0x80`, TD1, Higher nibble 8 means: no TA2, TB2, TC2, only TD2 is following. Lower nibble 0 means T=0

`0x01`, TD2, Higher nibble 8 means: no TA3, TB3, TC3, only TD3 is following. Lower nibble 1 means T=1

`0x80`, T1, Category indicator byte, 80 means A status indicator may be present in an optional COMPACT-TLV data object.

`0x4F`, Tk, Application identifier Presence Indicator

`0x0C`, Length, Length = 12 data byte (from next byte to check sum byte)

`0xA0 0x00 0x00 0x03 0x06`, RID, PC/SC Workgroup

`0x03`, Standard	ISO14443A, part 3

`0x00 0x03`, Card Name, Mifare Ultralight

`00 01` Mifare 1K

`00 02` Mifare 4K

`00 03` Mifare Ultralight

`00 26` Mifare Mini

`F0 04` Topaz and Jewel

`F0 11` Felica 212K

`F0 12` Felica 424K

`FF` [SAK] undefined

`0x00 0x00 0x00 0x00`, RFU, RFU # 00 00 00 00

`0x68`, TCK, Check Sum. Ex-OR of all the bytes T0 to Tk


##APDU Commands
Read tag ID
```sh
FF CA 00 00 04
```

Read binary Page 0x04, 4 bytes
```sh
FF B0 00 04 04
```

Read binary Page 0x04, 16 bytes
```sh
FF B0 00 04 10
```

## Installation of Mqtt Broker
Check https://github.com/mcollina/mosca and install the following lib if using embedded mode:
```sh
$ sudo apt-get install make gcc g++
$ sudo apt-get install libzmq3 libzmq3-dev
```

## Installation of SQLite
```sh
$ sudo apt-get install sqlite3
$ cd ~/.node-red
$ npm install node-red-node-sqlite
```

## Installation of MySQL
```sh
$ sudo apt-get install mysql-server --fix-missing
$ sudo apt-get install mysql-client
$ cd ~/.node-red
$ npm install node-red-node-mysql
```

First time to configure the password for database
```sh
sudo mysql_secure_installation
```

```sh
sudo mysql -u root -p
```

If you have this problem: `Error: ER_NOT_SUPPORTED_AUTH_MODE: Client does not support authentication protocol requested by server; consider upgrading MariaDB client`, just use the following instructions to fix the problem.
```sh
use mysql;
update user set authentication_string=password(''), plugin='mysql_native_password' where user='root';
```
https://stackoverflow.com/questions/2101694/how-to-set-root-password-to-null/36234358#36234358


If you need to connect to the remote database via root account, check the following.
```sh
 GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;
 FLUSH PRIVILEGES;
```

In `/etc/mysql/my.cnf`, change the following line
```sh
bind-address = 127.0.0.1
```
to
```sh
bind-address = *
```

## Installation of InfluxDB
```sh
$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.2.4_armhf.deb
$ sudo dpkg -i influxdb_1.2.4_armhf.deb
$ cd ~/.node-red
$ npm install node-red-contrib-influxdb
```

## Installation of Mongodb
```sh
$ sudo apt-get install mongodb
$ cd ~/.node-red
$ npm i node-red-node-mongodb
```

## Configuration of Nginx proxy
```sh
upstream nodejs_upstream {
    server 127.0.0.1:1880;
    keepalive 64;
}

server {
    location / {
            proxy_pass         http://nodejs_upstream;
            proxy_redirect     off;
            proxy_set_header   Host              $http_host;
            proxy_set_header   X-Real-IP         $remote_addr;
            proxy_set_header   X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header   X-Forwarded-Proto $scheme;
            proxy_set_header   X-NginX-Proxy     true;

            proxy_http_version 1.1;
            proxy_set_header Upgrade $http_upgrade;
            proxy_set_header Connection "upgrade";
            proxy_max_temp_file_size 0;
            proxy_read_timeout 240s;
    }
    
    error_page 500 502 503 504 /500.html;
    location = /500.html {
        root /var/www/html;
    }
}

```

```html
<!DOCTYPE html>
<html>
	<head>
		<meta http-equiv="refresh" content="3;url=http://localhost" />
	</head>
<body>

<div class="center">
<div class="lds-css ng-scope">
  <div style="width:100%;height:100%" class="lds-facebook">
    <div></div>
    <div></div>
    <div></div>
</div>

  <style type="text/css">
  @keyframes lds-facebook_1 {
  0% {
    top: 36px;
    height: 128px;
  }
  50% {
    top: 60px;
    height: 80px;
  }
  100% {
    top: 60px;
    height: 80px;
  }
  }
  @-webkit-keyframes lds-facebook_1 {
    0% {
      top: 36px;
      height: 128px;
    }
    50% {
      top: 60px;
      height: 80px;
    }
    100% {
      top: 60px;
      height: 80px;
    }
  }
  @keyframes lds-facebook_2 {
    0% {
      top: 41.99999999999999px;
      height: 116.00000000000001px;
    }
    50% {
      top: 60px;
      height: 80px;
    }
    100% {
      top: 60px;
      height: 80px;
    }
  }
  @-webkit-keyframes lds-facebook_2 {
    0% {
      top: 41.99999999999999px;
      height: 116.00000000000001px;
    }
    50% {
      top: 60px;
      height: 80px;
    }
    100% {
      top: 60px;
      height: 80px;
    }
  }
  @keyframes lds-facebook_3 {
    0% {
      top: 48px;
      height: 104px;
    }
    50% {
      top: 60px;
      height: 80px;
    }
    100% {
      top: 60px;
      height: 80px;
    }
  }
  @-webkit-keyframes lds-facebook_3 {
    0% {
      top: 48px;
      height: 104px;
    }
    50% {
      top: 60px;
      height: 80px;
    }
    100% {
      top: 60px;
      height: 80px;
    }
  }
  .lds-facebook {
    position: relative;
  }
  .lds-facebook div {
    position: absolute;
    width: 30px;
  }
  .lds-facebook div:nth-child(1) {
    left: 35px;
    background: #3be8b0;
    -webkit-animation: lds-facebook_1 1s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    animation: lds-facebook_1 1s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    -webkit-animation-delay: -0.2s;
    animation-delay: -0.2s;
  }
  .lds-facebook div:nth-child(2) {
    left: 85px;
    background: #1aafd0;
    -webkit-animation: lds-facebook_2 1s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    animation: lds-facebook_2 1s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    -webkit-animation-delay: -0.1s;
    animation-delay: -0.1s;
  }
  .lds-facebook div:nth-child(3) {
    left: 135px;
    background: #6a67ce;
    -webkit-animation: lds-facebook_3 1s cubic-bezier(0, 0.5, 0.5, 1) infinite;
    animation: lds-facebook_3 1s cubic-bezier(0, 0.5, 0.5, 1) infinite;
  }
  .lds-facebook {
    width: 200px !important;
    height: 200px !important;
    -webkit-transform: translate(-100px, -100px) scale(1) translate(100px, 100px);
    transform: translate(-100px, -100px) scale(1) translate(100px, 100px);
  }
  .center {
    display: flex;
    justify-content: center;
    align-items: center;
    height: 95vh;
    overflow-y: hidden;
    overflow-x: hidden
  }
  </style></div>
</div>

</body>
</html>
```

Another redirect script:
```js
<script type="text/javascript">
    /*<![CDATA[*/
    var TimerVal = 20;
    function CountDown(){
	setTimeout( "CountDown()", 1000  );
	TimerVal=TimerVal-1;
	if (TimerVal<0) { TimerVal=0;
	    location.reload(true);
	    //window.location.href = "http://www.xaluan.com";
	}
    }
    CountDown();
    /*]]>*/
</script>
```

## Change Splash Screen
The splash screen is a PNG file at `/usr/share/plymouth/themes/pix/splash.png`. Replace the PNG to change the splash screen.

## Configure Kiosk Mode
Edit the autostart file:
```sh
vim ~/.config/lxsession/LXDE-pi/autostart
```

Add the following commands:
```sh
@lxpanel --profile LXDE-pi
@pcmanfm --desktop --profile LXDE-pi
@xset s off
@xset -dpms
@xset s noblank
@sed -i 's/"exited_cleanly": false/"exited_cleanly": true/' ~/.config/chromium-browser Default/Preferences
@chromium-browser --noerrdialogs --kiosk http://localhost --incognito --disable-translate
```

## Rotate the Pi screen
Edit `/boot/config.txt` and add the following line:
```sh
lcd_rotate=2
```
## Disable touch screen
Edit `/boot/config.txt` and add the following line:
```sh
disable_touchscreen=1
```

## Configure DS3231 RTC
Enable I2C pins.
```sh
sudo raspi-config
```

Detect I2C address
```sh
sudo i2cdetect -y 1
```

Found there is an I2C device at 0x68. Write its address into i2c-adapter to let system know a new device detected.
```sh
sudo echo ds3231 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
```

Execute i2cdetect again and make sure the address is changed to UU.
```sh
sudo i2cdetect -y 1
```

Adjust clock from Internet or manual input
```sh
sudo service ntp start
```
```sh
sudo ntpdate 0.tw.pool.ntp.org

```
```sh
date -s "2016-11-23 08:30:25"
```

Read clock from hwclock
```sh
sudo hwclock -r
```

Write clock to hwclock
```sh
sudo hwclock -w
```

Configure hwclock as Linux system time
```sh
sudo hwclock -s
```

Check all time information
```sh
timedatectl
```

Execute RTC hardware clock on boot. Put the following commands in `/etc/rc.local`.
```sh
echo ds3231 0x68 > /sys/class/i2c-adapter/i2c-1/new_device
sudo hwclock -s
```

## Set up WiFi on your Raspberry Pi from SD card
You'll have to locate the boot directory, on my Mac it's in /Volumes/boot.
```sh
cd /Volumes/boot
```

Add wpa_supplicant.conf file
For Raspbian Jessie:
```sh
network={
    ssid="NETWORK_NAME"
    psk="PASSWORD"
    key_mgmt=WPA-PSK
}
```

For Raspbian Stretch (and newer versions of RetroPie):
```sh
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
network={
    ssid="NETWORK_NAME"
    psk="PASSWORD"
    key_mgmt=WPA-PSK
}
```

## Hide mouse curcor
```sh
$ sudo apt-get install unclutter
$ vim ~/.config/lxsession/LXDE-pi/autostart
```

Add the following line:
```sh
@unclutter -idle 0.1
```

## Disable RPi logo
Edit `/boot/cmdline.txt` add the following line:
```sh
logo.nologo quiet splash
```

## Control WiFi
Enable/Disable WiFi interface.
```sh
$ sudo ip link set dev wlan0 up
$ sudo ip link set dev wlan0 down
```

Request dynamic IP from DHCP for WiFi.
```sh
$ sudo dhclient wlan0
```

Configure static IP address and default route.
```sh
$ sudo ip addr add 192.168.1.14/24 dev wlan0
$ sudo ip link set dev wlan0 up
$ sudo ip route add default via 192.168.1.1
```
These commands configure your interface but these changes will not survive a reboot, since the information is not stored anyhwere. 

To configure a interface permanently you'll need to edit the interfaces file, `/etc/network/interfaces`.
```sh
$ sudo vi /etc/network/interfaces
```
```sh
## To configure a dynamic IP address
auto wlan0
iface wlan0 inet dhcp

## Or configure a static IP
auto wlan0
iface wlan0 inet static
  address 192.168.1.14
  gateway 192.168.1.1
  netmask 255.255.255.0
  network 192.168.1.0
  broadcast 192.168.1.255
```

For these settings to take effect you need to restart your networking services.
```sh
$ sudo /etc/init.d/networking restart
```
