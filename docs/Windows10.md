# Quick testing on windows 10

## Windows 10 Home Version - using Docker

### Add usb filters
- Install [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)
  - On Windows 10 home, docker toolbox ins needed. See or create a new section for windows 10 professional
- Install [Android Studio](https://developer.android.com/studio/index.html) Or the stanadalone android command line tools for adbd if that is all you need
  - I could not identify a way to pass through windows USB devices into docker, so I found it easy to do a quick test using adbd on Windows.
- Install some method of port redirection so you can forward from the docker interface to localhost for adb
  - Windows adb daemon does not properly listen on all ports
  - Alternatively, use the command to run adb in non-daemon mode `adb -a nodaemon server`
  - I used Bash for windows 10 and run this command
    - `socat tcp-l:55037,reuseaddr,fork tcp:localhost:5037`
- ( Optional ) Install [RethinkDB for windows](https://rethinkdb.com/docs/install/windows/)
  - Or use the RethinkDB docker image as mentions in configuration.  The rest of the guide will show using rethink in docker with stf
  
### Configuration
  I use this docker-compose.yml, if you are using RethinkDB for Windows, you can remove that section, and set the rethink DB port local variable
  
  ````yaml
  
  rethinkdb:
    image: rethinkdb:2.3
    ports:
      - "8080:8080"
      - "28015:28015"
      - "29015:29015"
    restart: always
    volumes:
      - "/srv/rethinkdb:/data"
    command: "rethinkdb --bind all --cache-size 2048"
  
  
  
  stf-local:
    image: openstf/stf
    links:
      - rethinkdb
    ports:
      - "7100:7100"
      - "7110:7110"
      - "7120:7120"
      - "7400-7500:7400-7500"
    restart: always
    environment:
      - LOCAL_ADB_HOST=somehost
    extra_hosts:
      - somehost:192.168.2.15
    command: stf local --public-ip 192.168.99.100 --provider-min-port 7400 --provider-max-port 7500 --adb-host somehost --adb-port 55037
  ````
  
