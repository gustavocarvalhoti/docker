# Comandos Docker

## Geral 
```
#Hypervisor     -> Virtualiza os recursos de hardware, utilizar até 90% do hardware
#Docker         -> Evolução das VMs (Ele nao precisa de sistema operacional, Divide tudo do SO) -> Gerencia containers
#Docker Compose -> Orquestrador de containers
#Docker Swarm   -> Multiplos Dockers em Cluster
#Docker Hub     -> Repositório https://hub.docker.com/
#Docker Machine -> Intalar e configurar hosts virtuais
#Container      -> Igual servidores -> Virtualização -> Contem sua application -> Pode limitar o uso de hardware -> Quando remove a camada de dados vai junto
#images         -> Uma imagem pode ser composta de várias imagens, se você já tem ele compartilha automaticamente
#Volumes        -> Onde salva os dados do container
#Docker hosts   -> Camada onde salva as novas informações do container (salva os volumes)
#Montando um Dockerfile -> Criar uma imagem a partir da nossa aplicação,
O Dockerfile define os comandos para executar instalações complexas e com características específicas.

***************************************************************************************************
#Comandos básicos
docker version
docker run hello-world <- Cria o container hello-world, pegou no Docker Hub e executou mostrando a msg e encerrou
docker run -it ubuntu <- Cria o container e entra no terminal dele
docker run ubuntu echo 'Oi mundo!' -> Executa o ubunto e passa um comando do linux para ele
docker run -d dockersamples/static-site -> Roda sem travar o terminal
docker run -d -P dockersamples/static-site -> O P conecta a porta interna do Docker com a externa (local aleatória)
docker run -d -P --name site-estatico dockersamples/static-site --> Dando nome aos bois
docker run -d -p 12345:80 --name site-estatico-com-porta dockersamples/static-site --> Escolhendo a porta (Local>12345:80<Docker)
docker run -d -P -e AUTHOR="Gustavo Carvalho" dockersamples/static-site --> Setando variável de ambiente -e AUTHOR="Gustavo Carvalho"
docker run -v "/var/www" ubuntu -> Cria o volume na pasta /var/www -> a minha pasta local fica em /var/lib/docker/volumes/
docker run -it -v "/home/gustavo/Documents/Docker-volumes:/var/www" ubuntu -> Local:Container (Quando escreve em 1 escreve no outro)

docker run -d -p 8080:3000 -v "/home/gustavo/Documents/Docker-volumes/volume-exemplo:/var/www" -w "/var/www" node npm start
-> Descrevendo
docker run -d -> Não trava o terminal
-p 8080:3000 <- Local:Container
-v "/home/gustavo/Documents/Docker-volumes/volume-exemplo:/var/www" <- Volume (Local:Container)
-w "/var/www" node npm start <- Executar o comando nessa pasta

docker run -d -p 8080:3000 -v "$(pwd):/var/www" -w "/var/www" node npm start -> Pega o caminho da sua pasta local, que vc está

docker ps -> Ativos (Rodando)
docker ps -q -> Lista o ID dos containers rodando
docker ps -a -> Exibe todos os que já foram criados
docker start -a -i cd4d4c377361 -> Inicia e entra no container ubuntu (-a = attach)
docker start --help -> Ver opções
docker exec -it 10d9eec16eb9 bash -> Executa o ubunto e entra nele para exec comandos
docker stop 2fe9b8ad4194
docker stop -t 0 2fe9b8ad4194 -> Stop sem esperar
docker stop $(docker ps -qa) -> Para todos os containers
docker rm -f dc583019693 -> Remove container (-f remove rapidão)
docker rmi hello-world -> Remove a imagem
docker container prune -> Remove todos os containers stopped (TOP)
docker system prune -a -> Remove os volumes sem uso e um monte de coisa
docker images -> Lista as imagens
docker port 9bdf413f5f0f -> Lista a porta utilizada pelo container
docker inspect container-> Vê se deu certo o container
docker history hello-world -> Lista as camadas da imagem
```

## Criando banco de dados
```
### ORACLE
https://hub.docker.com/r/wnameless/oracle-xe-11g/
docker pull wnameless/oracle-xe-11g
#1521 exporta a porta
docker run -d -p 49160:22 -p 1521:1521 -e ORACLE_ALLOW_REMOTE=true wnameless/oracle-xe-11g
# Entrar no container
ssh root@localhost -p 49160
password: admin
# Entrar no banco de dados
sqlplus
username: system
password: oracle

### MySQL
docker run -p 3306:3306 --name database-mysql -e MYSQL_ROOT_PASSWORD=root -d mysql:latest
```

## Wordpress
```
https://easyengine.io/docs/install/
apt-get install wget
wget -qO ee rt.cx/ee && bash ee
ee site create dev.credicitrus.com.br --wp
WordPress admin user : Gustavo
WordPress admin user plassword : 5BomViySQlqZ6FC

docker start 10d9eec16eb9
docker exec -it 10d9eec16eb9 bash
/etc/init.d/php5.6-fpm start
/etc/init.d/mysql start
/etc/init.d/nginx start
```

##  Montando um Dockerfile - Criar uma imagem do aplicativo
```
### Criar um arquivo Dockerfile na pasta que vc quer -> Colocar o arquivo na pasta que vc quer montar
FROM node:latest <- baseada o node, da para escolher a versão
MAINTAINER Gustavo Carvalho da Silva <- Quem criou
COPY . /var/www <- Copia todos os arquivos locais para a pasta do container
WORKDIR /var/www <- Executar o comando nessa pasta
RUN npm install <- Ao Criar o container ele executa o comando
ENTRYPOINT npm start <- Executa ao iniciar o container
ENTRYPOINT ["npm","start"] <- Da para usar assim tb
EXPOSE 3000 <- O container utiliza a porta 3000

#Ficou assim:
FROM node:latest
MAINTAINER Gustavo Carvalho da Silva
COPY . /var/www
WORKDIR /var/www
RUN npm install
ENTRYPOINT npm start
ENTRYPOINT ["npm","start"]
EXPOSE 3000

#Agora precisa compilar
cd /home/gustavo/Documents/Docker-volumes/DockerFile
docker build -f Dockerfile -t gustavoc/node . -> O ponto é porque está na pasta local
docker build -t gustavocarvalhoti/node . -> Se o nome é Dockerfile não precisa do -f

#Verificar se deu certo
docker images
gustavoc/node       latest              2aafc3f1afd2        2 minutes ago       673MB
node                latest              4885ab8871c2        2 days ago          673MB

#Criar o container
docker run -d -p 8080:3000 gustavocarvalhoti/node
docker run -d -p 3000:3000 gustavocarvalhoti/node

#Setando variaveis de ambiente
ENV NODE_ENV=production
ENV PORT=3000
#Da para colocar assim tb
EXPOSE $PORT

***************************************************************************************************
#Enviando minha imagem para o Docker hub
#Feito o cadastro digite no terminal
docker login
gustavocarvalhoti
12152015
docker push gustavoc/node -> Subindo a imagem
```

## Networking no Docker - Como inteligar diversos container (MySQL, App, Cache)
```
docker run -it ubuntu
docker inspect 94da
NetworkSettings -> Networks -> bridge (Ele inicia na default)
hostname -i -> No container do Ubuntu digite, descobre o IP
#A img vem apenas com o default, #Update, install iputils-ping, -y (Autoriza)
apt-get update && apt-get install -y iputils-ping
#A rede virtual foi criada pelo Docker randomicamente os IPs

#Criando a minha própria rede
docker network create --driver bridge minha-rede
docker network ls -> lista a rede
docker run -it --name meu-container-de-ubuntu --network minha-rede ubuntu
apt-get update && apt-get install -y iputils-ping
docker run -it --name segundo-ubuntu --network minha-rede ubuntu
#Agora que eu criei a rede eu posso dar um ping por nome TOP, uso isso para acessar o BD
ping segundo-ubuntu (Posso fazer isso para redes que eu criei, para achar por nome)
(Não precisa do IP \o/)

***************************************************************************************************
#Conectando App + BD em containe diferentes
docker run -d --name meu-mongo --network minha-rede mongo
docker pull douglasq/alura-books:cap05
docker run --network minha-rede -d -p 8080:3000 douglasq/alura-books:cap05
http://localhost:8080/seed -> Cria a massa
http://localhost:8080 -> Verifica se deu certo
Done (Top \o/)

***************************************************************************************************
#Docker Compose - Multiplos containers
#Ele verifica o arquivo docker-compose.yml ("Ialmi") "Receita de bolo"

NGINX "Endnex" - Faz o load balancer, balanceamento de acessos para cada servidor logico
NGINX "Endnex" - Cuida dos arquivos staticos, CSS, JS, IMG

Mãos a obra, BD + 3Servidores + NGINX
cd /home/gustavo/Documents/Docker-volumes/alura-docker-cap06
Criei o arquivo docker-compose.yml
version: '3'
services:
  nginx:
    build:
      dockerfile: ./docker/nginx.dockerfile
      context: .
    image: gustavocarvalhoti/nginx
    container_name: nginx
    ports:
      - "80:80" <- Porta de dentor e fora
    networks:
      - production-network <- O traço - significa que é um ArrayList
    depends_on:
      - "node1"
      - "node2"
      - "node3"
  mongodb:
    image: mongo <- Usa img default
    networks:
      - production-network
  node1:
    build:
      dockerfile: ./docker/alura-books.dockerfile
      context: .
    image: gustavocarvalhoti/alura-books
    container_name: alura-books1
    ports:
      - "3000"
    networks:
      - production-network
    depends_on:
      - "mongodb" <- Sobe depois desse
  node2:
    build:
      dockerfile: ./docker/alura-books.dockerfile
      context: .
    image: gustavocarvalhoti/alura-books
    container_name: alura-books2
    ports:
      - "3000"
    networks:
      - production-network
    depends_on:
      - "mongodb"
  node3:
    build:
      dockerfile: ./docker/alura-books.dockerfile
      context: .
    image: gustavocarvalhoti/alura-books
    container_name: alura-books3
    ports:
      - "3000"
    networks:
      - production-network
    depends_on:
      - "mongodb"
networks:
  production-network:
    driver: bridge

#Fazendo a build
docker-compose build

#Iniciando os containers
docker-compose up
docker-compose up -d

#Testando
localhost:80/seed
localhost:80

#Lista os serviços rodando
docker-compose ps

#Stop
docker-compose stop

#Stop e remove (Cuidado)
docker-compose down

#Verificar se um container consegue falar com o outro
docker exec -it alura-books1 ping alura-books2

#Instalando o docker-compose
sudo curl -L https://github.com/docker/compose/releases/download/1.15.0/docker-compose-`uname -s`-`uname -m` -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose

***************************************************************************************************
#Docker Swarm facilita a criação e administração de um cluster de containers.

***************************************************************************************************
#Docker Cloud - Ele instala a maquina virtual na Amazon e sob os containers
```
