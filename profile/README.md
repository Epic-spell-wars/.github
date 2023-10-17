Docker-compose to run it all.

```
version: "3.9"

services:
  ui:
    build: "ui-service/"
    environment:
      - "REACT_APP_API_URL=http://localhost:8080/api"
      - "REACT_APP_SOCKET_URL=http://localhost:8888"
    ports:
      - "3000:80"
    depends_on:
      - "gateway"

  gateway:
    build: "api-service/"
    environment:
      - SERVER_PORT=8080
      - EUREKA_ZONE=http://localhost:8080/eureka
    ports:
      - "8080:8080"

  auth:
    build: "auth-service/"
    environment:
      - "MONGODB_URI=mongodb://mongo"
      - "MONGODB_DB=auth-service"
      - "EUREKA_ZONE=http://gateway:8080/eureka"
      - "EUREKA_ENABLE=true"
      - "SENDGRID_API_KEY=APIKEY"
      - "CLOUDAMQP_URL=amqp://guest:guest@event-bus/my_vhost"
    depends_on:
      - "mongo"
      - "gateway"
      - "event-bus"

  game:
    build: "game-service/"
    environment:
      - "MONGODB_URI=mongodb://mongo"
      - "MONGODB_DB=game-service"
      - "EUREKA_ENABLE=true"
      - "EUREKA_ZONE=http://gateway:8080/eureka"
    depends_on:
      - "gateway"
      - "mongo"

  player:
    build: "player-service/"
    environment:
      - "MONGODB_URI=mongodb://mongo"
      - "MONGODB_DB=player-service"
      - "EUREKA_ENABLE=true"
      - "EUREKA_ZONE=http://gateway:8080/eureka"
      - "CLOUDAMQP_URL=amqp://guest:guest@event-bus/my_vhost"
    depends_on:
      - "gateway"
      - "mongo"

  lobby:
    build: "lobby-service/"
    environment:
      - "EUREKA_ENABLE=true"
      - "EUREKA_HOST=gateway"
      - "EUREKA_PORT=8080"
      - "MONGODB_URI=mongodb://mongo"
      - "MONGODB_DB=lobby-service"
      - "PORT=3000"
      - "ENABLE_REDIS=true"
      - "REDIS_URL=redis://redis:6379"
    ports:
      - "8888:3000"
    depends_on:
      - "gateway"
      - "mongo"
      - "game"

  mongo:
    image: mongo
    restart: always
    ports:
      - "27017:27017"

  event-bus:
    image: rabbitmq:3-management-alpine
    restart: always
    environment:
      - "RABBITMQ_DEFAULT_VHOST=my_vhost"
    ports:
      - "9090:15672"
      - "5671:5671"
      - "5672:5672"

  redis:
    image: redis
    restart: always
    ports:
      - "6379:6379"
        
  bot:
    build: "bot-service/"
    restart: always
    environment:
      - "SOCKET_URL=http://localhost:8092/"


```
