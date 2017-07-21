# Installation of Nodejs and Node-red

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

# Installation of Node-red security
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

# Installation of PCSC

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

# Installation of Mqtt Broker
Check https://github.com/mcollina/mosca and install the following lib if using embedded mode:
```sh
$ sudo apt-get install make gcc g++
$ sudo apt-get install libzmq3 libzmq3-dev
```

# Installation of SQLite
```sh
$ sudo apt-get install sqlite3
$ cd ~/.node-red
$ npm install node-red-node-sqlite
```

# Installation of MySQL
```sh
$ sudo apt-get install mysql-server --fix-missing
$ sudo apt-get install mysql-client
$ cd ~/.node-red
$ npm install node-red-node-sqlite
```

# Installation of InfluxDB
```sh
$ wget https://dl.influxdata.com/influxdb/releases/influxdb_1.2.4_armhf.deb
$ sudo dpkg -i influxdb_1.2.4_armhf.deb
$ cd ~/.node-red
$ npm install node-red-contrib-influxdb
```

# Configuration of Nginx proxy
```
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