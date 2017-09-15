# Server (transport Redis)
```
api:
  image: sstarcher/sensu
  command: api
  environment:
    REDIS_PASSWORD: 666666
    REDIS_RECONNECT_ON_ERROR: "true"
  links:
    - redis
    
server:
  image: sstarcher/sensu
  command: server
  environment:
    REDIS_PASSWORD: 666666
    REDIS_RECONNECT_ON_ERROR: "true"
  links:
    - redis
    - api
    
uchiwa:
  image: sstarcher/uchiwa
  environment:
      SENSU_DC_NAME: sensu
      SENSU_HOSTNAME: sensu
      SENSU_SERVICE_PORT: 4567
  links:
    - api:sensu
  ports:
    - '3001:3000'
    
redis:
  image: tutum/redis
  environment:
      REDIS_PASS: 666666
  ports:
    - '6379:6379'
```

# Client (transport Redis)
##### repo:
```
echo '[sensu]
name=sensu
baseurl=https://sensu.global.ssl.fastly.net/yum/$releasever/$basearch/
gpgcheck=0
enabled=1' | sudo tee /etc/yum.repos.d/sensu.repo
```

*/etc/sensu/config.json*:
```
{
  "client": {
    "name": "client-host01",
    "address": "server01",
    "subscriptions": [
      "ALL"
    ]
  }
}
```

*/etc/sensu/conf.d/redis.json*:
```
{
  "redis": {
    "host": "server01",
    "port": 6379,
    "password": "666666"
  }
}
```

*/etc/sensu/conf.d/transport.json*:
```
{
  "transport": {
    "name": "redis"
  }
}
```
