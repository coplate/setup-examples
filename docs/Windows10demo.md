# Quick testing on windows 10
## Background 

I had to set this up on a windows 10 laptop, becasue that was all I had to demo to my management, here are the two ways I set it up.
I did it in bash downloading all the dependencies first, becasue I didn't understand dockers networking the first time.  Then I took more time to learn that, and ran it in docker.


## Windows 10 Home Version - using Bash for Windows 10

### Setup
0. Install Android tools ( studio or command line ) I already did this in windows GUI, not in bash.

- I followed this guide but did it on windows.   I have recreated the guide in case it is removed from gist: https://gist.github.com/yasuyk/bbaa28568c497d95ff92566efc4ecc3c
  - Windows apt installed node 4.2.6, so I needed to use the `ALLOW_OUTDATED_DEPENDENCIES=1` to run STF for quick test


1. Add RethinkDB key

        source /etc/lsb-release && echo "deb http://download.rethinkdb.com/apt $DISTRIB_CODENAME main" | sudo tee /etc/apt/sources.list.d/rethinkdb.list
        wget -qO- https://download.rethinkdb.com/apt/pubkey.gpg | sudo apt-key add -

2. Install packages

        sudo apt-get update && sudo apt-get install -y git nodejs nodejs-legacy npm rethinkdb android-tools-adb python autoconf automake libtool build-essential ninja-build libzmq3-dev libprotobuf-dev git graphicsmagick yasm stow

3. Install additional packages via npm

        sudo npm install -g bower karma gulp

4. Install ZeroMQ

        cd ~/Downloads && wget http://download.zeromq.org/zeromq-4.1.2.tar.gz && tar -zxvf zeromq-4.1.2.tar.gz && cd zeromq-4.1.2 && ./configure --without-libsodium --prefix=/usr/local/stow/zeromq-4.1.2
        make && sudo make install && cd /usr/local/stow && sudo stow -vv zeromq-4.1.2

5. Install Google protobuf

        cd ~/Downloads && git clone https://github.com/google/protobuf.git && cd protobuf && ./autogen.sh && ./configure --prefix=/usr/local/stow/protobuf-`git rev-parse --short HEAD`
        make && sudo make install && cd /usr/local/stow && sudo stow -vv protobuf-*

6. Update library path

        sudo ldconfig

7. Install stf

        sudo npm install -g stf

## Run OpenSTF

1. Start required services

        rethinkdb &
        adb start-server

2. Start OpenSTF

        ALLOW_OUTDATED_DEPENDENCIES=1 stf local

or 
 
        ALLOW_OUTDATED_DEPENDENCIES=1 stf local --public-ip <ip address>

3. View in actions

    Go to htttp://<your_ip_address>:7100
    
## Windows 10 Home Version - using Docker

### Setup
- Install [Docker Toolbox](https://docs.docker.com/toolbox/toolbox_install_windows/)
  - On Windows 10 home, docker toolbox is needed. See or create a new section for windows 10 professional
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
      - LOCAL_ADB_HOST=localpc
    extra_hosts:
      - localpc:<<PUT YOUR PC IP HERE, not 127.0.0.1 >>
    command: stf local --public-ip <<PUT YOUR PUBLIC IP HERE, I 192.168.99.100 as that is the IP of teh docker image>> --provider-min-port 7400 --provider-max-port 7500 --adb-host localpc --adb-port 55037
  ````
  
