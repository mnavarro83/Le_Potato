# Assumptions:

a) Base Raspbian install on Le Potato SOC (AML-S905X-CC)

b) User with sudo privileges.

# Steps followed:

## Klipper

1) Enable SSH and start it (yes that is what --now does)

```
sudo systemctl enable ssh --now
```

2) Install dfu-util: 

```
sudo apt install dfu-util
```

3) Install Klipper dependencies: 

```
sudo apt install virtualenv python-dev libffi-dev build-essential libncurses-dev libusb-dev avrdude gcc-avr binutils-avr avr-libc stm32flash dfu-util libnewlib-arm-none-eabi gcc-arm-none-eabi binutils-arm-none-eabi libusb-1.0-0
```

4) Clone Klipper repo: 

```
git clone https://github.com/KevinOConnor/klipper
```

5) Enable virtual environment and install Python dependencies: 

```
cd ~
virtualenv -p python2 ./klippy-env
./klippy-env/bin/pip install -r ./klipper/scripts/klippy-requirements.txt
```

6) Edit file /etc/systemd/system/klipper.service. Change "mnavarro" to your user in all applicable entries:

```
[Unit]
Description=Starts Klipper and provides klippy Unix Domain Socket API
Documentation=https://www.klipper3d.org/
After=network.target
Before=moonraker.service
Wants=udev.target

[Install]
Alias=klippy
WantedBy=multi-user.target

[Service]
Environment=KLIPPER_CONFIG=/home/mnavarro/klipper_config/printer.cfg
Environment=KLIPPER_LOG=/home/mnavarro/klipper_logs/klippy.log
Environment=KLIPPER_SOCKET=/tmp/klippy_uds
Type=simple
User=mnavarro
RemainAfterExit=yes
ExecStart= /home/mnavarro/klippy-env/bin/python /home/mnavarro/klipper/klippy/klippy.py ${KLIPPER_CONFIG} -l ${KLIPPER_LOG} -a ${KLIPPER_SOCKET}
Restart=always
RestartSec=10
```

7) Enable Klipper (creates paths):

```
sudo systemctl enable klipper.service
```

8) Create directories:

```
mkdir ~/klipper_config
mkdir ~/klipper_logs
mkdir ~/gcode_files
touch ~/klipper_config/printer.cfg
```

9) Import suitable configuration into printer.cfg and start klipper

```
vim ~/klipper_config/printer.conf
sudo systemctl start klipper
```

## Moonraker:

1) More dependencies:

```
sudo apt install python3-virtualenv python3-dev libopenjp2-7 python3-libgpiod curl libcurl4-openssl-dev libssl-dev liblmdb-dev libsodium-dev zlib1g-dev libjpeg-dev
```

2) Clone Moonraker:

```
git clone https://github.com/Arksine/moonraker.git
```

3) More Python dependencies:

```
cd ~
virtualenv -p python3 ./moonraker-env
./moonraker-env/bin/pip install -r ./moonraker/scripts/moonraker-requirements.txt
```

4) Edit configuration file ~/klipper_config/moonraker.conf. Following is a BASIC config (not sharing my exact config)

```
[server]
host: 0.0.0.0
port: 7125
enable_debug_logging: False
config_path: ~/klipper_config
log_path: ~/klipper_logs

[authorization]
cors_domains:
    https://my.mainsail.xyz
    http://my.mainsail.xyz
    http://*.local
    http://*.lan
trusted_clients:
    10.0.0.0/8
    127.0.0.0/8
    169.254.0.0/16
    172.16.0.0/12
    192.168.0.0/16
    FE80::/10
    ::1/128

# enables partial support of Octoprint API
[octoprint_compat]

# enables moonraker to track and store print history.
[history]

# this enables moonraker's update manager
[update_manager]

[update_manager mainsail]
type: web
repo: mainsail-crew/mainsail
path: ~/mainsail
```

5) Modify system unit file /etc/systemd/system/moonraker.service, change mnavarro to your username in all instances:

```
[Unit]
Description=Moonraker provides Web API for klipper
Documentation=https://moonraker.readthedocs.io/en/latest/
After=network.target klipper.service

[Install]
WantedBy=multi-user.target

[Service]
Environment=MOONRAKER_CONFIG=/home/mnavarro/klipper_config/moonraker.conf
Environment=MOONRAKER_LOG=/home/mnavarro/klipper_logs/moonraker.log
Type=simple
User=mnavarro
RemainAfterExit=yes
ExecStart=/home/mnavarro/moonraker-env/bin/python /home/mnavarro/moonraker/moonraker/moonraker.py -c ${MOONRAKER_CONFIG} -l ${MOONRAKER_LOG}
Restart=always
RestartSec=10
```

6) Update policy kit:

```
sudo apt install packagekit
~/moonraker/scripts/install-moonraker.sh -r
cd ~/moonraker/scripts  
./set-policykit-rules.sh  
sudo service moonraker restart
```

7) Test:

```
http://<printer-ip>:7125/printer/info
```

## Mainsail

1) Install nginx

```
sudo apt install nginx
```

2) Create necessary configuration files:

```
sudo touch /etc/nginx/sites-available/mainsail
sudo touch /etc/nginx/conf.d/upstreams.conf
sudo touch /etc/nginx/conf.d/common_vars.conf
```

3) In /etc/nginx/sites-available/mainsail

```
# /etc/nginx/sites-available/mainsail

server {
    listen 80 default_server;

    access_log /var/log/nginx/mainsail-access.log;
    error_log /var/log/nginx/mainsail-error.log;

    # disable this section on smaller hardware like a pi zero
    gzip on;
    gzip_vary on;
    gzip_proxied any;
    gzip_proxied expired no-cache no-store private auth;
    gzip_comp_level 4;
    gzip_buffers 16 8k;
    gzip_http_version 1.1;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/x-javascript application/json application/xml;

    # web_path from mainsail static files
    root /home/pi/mainsail;

    index index.html;
    server_name _;

    # disable max upload size checks
    client_max_body_size 0;

    # disable proxy request buffering
    proxy_request_buffering off;

    location / {
        try_files $uri $uri/ /index.html;
    }

    location = /index.html {
        add_header Cache-Control "no-store, no-cache, must-revalidate";
    }

    location /websocket {
        proxy_pass http://apiserver/websocket;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection $connection_upgrade;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_read_timeout 86400;
    }

    location ~ ^/(printer|api|access|machine|server)/ {
        proxy_pass http://apiserver$request_uri;
        proxy_set_header Host $http_host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Scheme $scheme;
    }
	
    location /webcam/ {
        proxy_pass http://mjpgstreamer1/;
    }

    location /webcam2/ {
        proxy_pass http://mjpgstreamer2/;
    }

    location /webcam3/ {
        proxy_pass http://mjpgstreamer3/;
    }

    location /webcam4/ {
        proxy_pass http://mjpgstreamer4/;
    }
}
```

4) In /etc/nginx/conf.d/common_vars.conf:

```
# /etc/nginx/conf.d/common_vars.conf

map $http_upgrade $connection_upgrade {
    default upgrade;
    '' close;
}
```

5) In /etc/nginx/conf.d/upstreams.conf

```
# /etc/nginx/conf.d/upstreams.conf

upstream apiserver {
    ip_hash;
    server 127.0.0.1:7125;
}

upstream mjpgstreamer1 {
    ip_hash;
    server 127.0.0.1:8080;
}

upstream mjpgstreamer2 {
    ip_hash;
    server 127.0.0.1:8081;
}

upstream mjpgstreamer3 {
    ip_hash;
    server 127.0.0.1:8082;
}

upstream mjpgstreamer4 {
    ip_hash;
    server 127.0.0.1:8083;
}
```

6) Clone and install Mainsail:

```
cd ~/mainsail
wget -q -O mainsail.zip https://github.com/mainsail-crew/mainsail/releases/latest/download/mainsail.zip && unzip mainsail.zip && rm mainsail.zip
```

That's it, should do it. Macros and mainsail.conf will still need modification very likely. Further configuration available at: https://docs.mainsail.xyz/setup/manual-setup/mainsail


