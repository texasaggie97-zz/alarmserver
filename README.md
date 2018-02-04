# AlarmServer

This is the alarmserver.py portion of @LXXero's SmartThings and DSC alarm panels [integration](https://github.com/LXXero/DSCAlarm).

I am putting this script into a Docker container for ease of use and portability. My VM that was running it died, so I have used
this opportunity to learn about Docker.

## Changes from LXXero/DSCAlarm
* `alarmserver_logger` rewritten using Python logging. This allows the use of log rotation.
* Add the ability to read some of the sensitive information from the environement instead of the config file. This allows me to put the config file on GitHub without worrying about having my alarm code on the Internet for all to see.

## To create and publish the container:
```
docker build -t alarmserver . ; docker tag alarmserver texasaggie97/alarmserver:latest ; docker push texasaggie97/alarmserver
```

## RancherOS

I am using RancherOS to host and manage Docker running in a lightweight VM on FreeNAS. Here are the docker-compose.yml and rancher-compose.yml
to easily recreate the container.

### docker-compose.yml:
```
version: '2'
volumes:
  logs:
    external: true
    driver: rancher-nfs

services:
  alarmserver:
    image: texasaggie97/alarmserver
    environment:
      ALARMCODE: "1234"
      CALLBACKURL_BASE: "https://graph.api.smartthings.com/api/smartapps/installations"
      CALLBACKURL_APP_ID: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
      CALLBACKURL_ACCESS_TOKEN: "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
      LOGFILE: "/logs/docker-alarmserver.log"
      ENVISALINKHOST: "192.168.0.149"
    stdin_open: true
    working_dir: /alarmserver
    volumes:
    - logs:/logs
    tty: true
    ports:
    - 8111:8111/tcp
    command:
    - python
    - alarmserver.py
    labels:
      io.rancher.container.pull_image: always

  hover-dns-updater:
    image: texasaggie97/hover-dns-updater
    environment:
      USERNAME: "username"
      PASSWORD: "password"
    stdin_open: true
    working_dir: /hover-dns-updater
    volumes:
    - logs:/logs
    tty: true
    command:
    - python
    - hover-dns-updater.py
    - --service
    labels:
      io.rancher.container.pull_image: always
```

### rancher-compose.yml:
```
version: '2'
services:
  alarmserver:
    scale: 1
    start_on_create: true
  hover-dns-updater:
    scale: 1
    start_on_create: true
```


From LXXero's readme:
### Alarmserver Setup

1. First, edit 'alarmserver.cfg' and add in the OAuth/Access Code information to the callback_url_app_id and callbackurl_access_token values,
   and adjust your zones/partitions at the bottom of the file. If you're upgrading, be sure to update the list of callback event codes to match
   the upstream config example. Leaving them at the defaults is likely what you already want.

2. The alarm panels and zone devices get created/deleted automatically when alarmserver starts. Ensure all your zones and partitions are properly
   defined as per the included alarmserver.cfg example.

4. Fire up the AlarmServer. Your devices should get created in smartthings, and you should start seeing events pushed to them within a few moments
   on your smart phone.

## Thanks!
Thanks goes out to the following people, without their previous work none of this would have been possible:
* juggie
* Ethomasii
* blacktirion
* Rob Fisher <robfish@att.net>
* Carlos Santiago <carloss66@gmail.com>
* JTT <aesystems@gmail.com>
* Donny K <donnyk+envisalink@gmail.com>
* Leaberry <leaberry@gmail.com>
* Kent Holloway <drizit@gmail.com>
* Matt Martz <matt.martz@gmail.com>

And for Dim and Dimmer, Geko / Statusbits

