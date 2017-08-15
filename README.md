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

Install nodejs
```sh
$ nvm install 6.10.1
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
node-red: /home/pi/.nvm/versions/node/v6.10.1/bin/node-red
```

Start node-red by pm2
```sh
$ pm2 start /home/pi/.nvm/versions/node/v6.10.1/bin/node-red -- -v
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
$ npm install node-red-node-sqlite
```

## Installation of InfluxDB
```sh
$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.2.4_armhf.deb
$ sudo dpkg -i influxdb_1.2.4_armhf.deb
$ cd ~/.node-red
$ npm install node-red-contrib-influxdb
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
}

```

## Change Splash Screen
The splash screen is a PNG file at `/usr/share/plymouth/themes/pix/splash.png`. Replace the PNG to change the splash screen.

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
