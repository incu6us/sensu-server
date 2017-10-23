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
  build: uchiwa/
  restart: on-failure
  links:
    - api:sensu
  volumes:
    - ./uchiwa/config/uchiwa.json:/etc/uchiwa/config.json
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

# Server customization
### Install custom plugins inside container
```
/opt/sensu/embedded/bin/gem sources --remove https://rubygems.org/
/opt/sensu/embedded/bin/gem sources --add http://rubygems.org/
/opt/sensu/embedded/bin/gem install sensu-plugins-slack
```

### Slack handler config
*/etc/sensu/handlers/slack-handler.json*:
```
{
  "handlers": {
    "slack": {
      "type": "pipe",
      "command": "handler-slack.rb",
      "severites": ["ok", "critical", "unknown", "warning"]
      }
    },
    "slack": {
      "webhook_url": "https://hooks.slack.com/services/HIDDEN/SECRET",
      "template" : "",
      "dashboard": "http://HOST:PORT/#/client/$DataCenter/"
  }
}
```

##### make slack as a default notification handler
*/etc/sensu/handlers/default-handlers.json*:
```
{
  "handlers": {
    "default": {
      "type": "set",
      "handlers": [
        "slack"
      ]
    }
  }
}
```
