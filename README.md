# hastebin-docker
[hastebin](hastebin.com) server run using docker-compose. It uses alpine images for nodejs and redis to keep things small:

* `node:6.9-alpine` runs [hastebin-server](https://github.com/seejohnrun/haste-server) on Node.js `6.9`
* `redis:alpine` runs Redis backend
* it persists data in `redis` directory, which is attached as a volume to `redis:alpine`
* uses application data as a volume, which is attached as a volume to hastebin image
* can be run using [Rancher](https://github.com/rancher/rancher)

Setup:

1. clone this repo
2. get `hastebin-server' submodule:
  
  ```shell
  git submodule init
  git submodule update --recursive
  ```
  
3. copy `config.js` file from `hastebin-server` repo to main directory, 
because you will have to change it and it is, unfortunatly, tracked by git... it is ignored by this repo, though.
  
  ```shell
  cp haste-server/config.js .
  ```
  I don't want to keep copy of this file in repo, because it might change in the future.
  
4. edit `config.js` and replace redis host name with `redis`, as defined in `docker-compose.yml`. Finally it should look like this:
  
  ```yaml
  # .....
  
    "storage": {
    "type": "redis",
    "host": "redis",  # <- this used to be 0.0.0.0
    "port": 6379,
    "db": 2,
    "expire": 2592000
  },
  # ....
  ```
  
5. run whole thing by using either `docker-compose up` or following:

  ```shell
  docker-compose build
  docker-compose create
  docker-compose start
  ```
  or something similar, whatever your needs are.

6. you should be able to see something like this, when running `docker ps`:

  | CONTAINER ID | IMAGE | COMMAND | CREATED | STATUS | PORTS | NAMES |
  | --- | --- | --- | --- | --- | --- | --- |
  | 78728f03787a | hastebindocker_hastebin:latest | "/bin/sh -c 'npm inst" | 9 minutes ago | Up 9 minutes | 0.0.0.0:7777->7777/tcp | hastebindocker_hastebin_1 |
  | c61119e29014 | redis:alpine | "docker-entrypoint.sh" | 9 minutes ago | Up 9 minutes | 6379/tcp | hastebindocker_redis_1 |
  
7. hastebin can be accessed on: `your-docker-host-ip:7777` out of the box, but you might want to add 
some king of reverse proxy in front of it, i.e. using nginx:
 
 ```nginx
  server {
    listen 80 default_server;
    location / {
        proxy_set_header   X-Real-IP $remote_addr;
        proxy_set_header   Host      $http_host;
        proxy_pass         http://your-docker-host-ip:7777;
    }
  }
  ```
  
