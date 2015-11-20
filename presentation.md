name: dockerizando-nodejs
class: center, middle, inverse

# Dockerizando uma aplicação NodeJS

### Nice subtitle

---

## Sobre mim

TODO: coloca foto!

- Consultor desenvolvedor na ThoughtWoks
- Conhece Docker a 2 anos

TODO: foto?

---

## Agenda

- O que é, por que e como instalar Docker?
- A aplicação node
- Dockerfile
- Build & run
- `docker-compose`

---

## O que é por que Docker?

- Isolamento: cgroup e namespaces
- Dependências dentro de um pacote (imagem)
- Densidade: um app por container, vários containars no mesmo servidor (Docker Host)
- Imagem como artefato de deployment
- Facilita escalar horizontalmente

---


## Instalação

OSX:

```sh
brew install docker docker-compose docker-machine
```

Ou Docker Toolbox: https://www.docker.com/docker-toolbox

.fixsize.center[ ![](imgs/docker-toolbox.png) ]

---

## A aplicação

- Um simples contador de visitas em NodeJS + Express
- Persistência em Redis
- https://github.com/labianchin/docker-nodejs-demo

.smaller[
```javascript
var express = require('express'),
    http = require('http'),
    redis = require('redis');

var client = redis.createClient(
  process.env.REDIS_1_PORT_6379_TCP_PORT || 6379,
  process.env.REDIS_1_PORT_6379_TCP_ADDR || '127.0.0.1'
);

var app = express();
app.get('/', function(req, res, next) {
  client.incr('visits', function(err, visits) {
    if(err) return next(err);
    res.send('Esta página foi visitada ' + visits + ' vezes!');
  });
});

http.createServer(app).listen(process.env.PORT || 3000, function() {
  console.log('Listening on port ' + (process.env.PORT || 3000));
});
```
]

---

## Criando um Dockerfile

```Dockerfile
FROM node:5.0.0

WORKDIR /app

ADD . /app
RUN npm install

ENV PORT=3000
EXPOSE 3000

CMD npm start
```

- Cada comando cria uma layer/camada
- Essas camadas são hierárquicas e cacheáveis

.hidden[
https://github.com/nodejs/docker-node/blob/2445743c1453941f787b0aa22cca51c62a1a3f09/5.0/Dockerfile
]

---

class: split-50

## Build

```sh
docker build -t my-app .
```

.fixsize2[
.column[
![](imgs/build1.png)
]
.column[
![](imgs/build2.png)
]
]

---

## Run

- Rodar redis linkado ao app

```sh
docker run --detach --name my-redis redis:latest
docker run -it -p 3000:3000 --link my-redis:redis_1 my-app
```

.fixsize.center[
![](imgs/run.png)
]


---

## docker-compose

Cansado de vários `docker run`?

```docker-compose.yml
web:
  build: .
  ports:
    - "3000:3000"
  links:
   - redis
redis:
  image: redis:latest
```

```sh
$ docker-compose up
```

---

## Fontes e Referências

- http://mherman.org/blog/2015/03/06/node-with-docker-continuous-integration-and-delivery/
- https://www.airpair.com/node.js/posts/getting-started-with-docker-for-the-nodejs-dev
- http://anandmanisankar.com/posts/docker-container-nginx-node-redis-example/
- http://www.slideshare.net/labianchin/verdades-do-docker

---

class: center, middle, inverse

## Perguntas?

---

## Dockerfile com cache


```Dockerfile
FROM node:5.0.0

WORKDIR /app

ADD package.json /app/
RUN npm install

ADD . /app

ENV PORT=3000
EXPOSE 3000

CMD npm start
```

---

## Build automatizado no Docker hub

https://hub.docker.com/r/labianchin/docker-nodejs-demo/builds/

---

## Volumes

```sh
$ docker volume create --name=myvolume
$ docker run -v myvolume:/data busybox sh -c "echo hello > /data/file.txt"
$ docker run -v myvolume:/data busybox sh -c "cat /data/file.txt"
```

```docker-compose.yml
web:
  build: .
  ports:
    - "3000:3000"
  links:
   - redis
redis:
  image: redis:latest
  volumes_from: myvolume
```


---

## Docker Machine

```sh
$ docker-machine create \
    --driver digitalocean \
    --digitalocean-access-token 0ab77166d407f479c6701652cee3a46830fef88b8199722b87821621736ab2d4 \
    staging
Creating SSH key...
Creating Digital Ocean droplet...
To see how to connect Docker to this machine, run: docker-machine env staging
```

---

## Boas práticas para imagens

- Um processo por container
- Minimize o número de layers: agrupar vários comandos no mesmo RUN
- Aproveite o build cache
- Prefira COPY ao invés de ADD

---

## Outros detalhes

- CI: build, test and package
- Em produção: entenda sobre mesos e/ou kubernetes
- Entenda shell scripting
- Cuidado com "yak shaving"

---

class: center, middle, inverse

## Fim
