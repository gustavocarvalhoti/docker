# Comandos Docker

## Geral 
```
#Hypervisor     -> Virtualiza os recursos de hardware, utilizar até 90% do hardware
#Docker         -> Evolução das VMs (Ele não precisa de sistema operacional, Divide tudo do SO) -> Gerência containers
#Docker Compose -> Orquestrador de containers
#Docker Swarm   -> Múltiplos Dockers em Cluster
#Docker Hub     -> Repositório https://hub.docker.com/
#Docker Machine -> Instalar e configurar hosts virtuais
#Container      -> Igual servidores -> Virtualização -> Contém sua application -> Pode limitar o uso de hardware -> Quando remove a camada de dados vai junto
#images         -> Uma imagem pode ser composta de várias imagens, se você já tem ele compartilha automaticamente
#Volumes        -> Onde salvar os dados do container
#Docker hosts   -> Camada que salva as novas informações do container (salva os volumes)
#Montando um Dockerfile -> Criar uma imagem a partir da nossa aplicação,
O Dockerfile define os comandos para executar instalações complexas e com características específicas.

***************************************************************************************************
#Comandos básicos
docker version
docker run hello-world <- Cria o container hello-world, pegou no Docker Hub e executou mostrando a msg e encerrou
docker run -it ubuntu <- Cria o container e entra no terminal dele
docker run ubuntu echo 'Oi mundo!' -> Executa o ubuntu e passa um comando do linux para ele
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
docker exec -it 10d9eec16eb9 bash -> Executa o ubuntu e entra nele para exec comandos
docker stop 2fe9b8ad4194
docker stop -t 0 2fe9b8ad4194 -> Stop sem esperar
docker stop $(docker ps -qa) -> Para todos os containers
docker rm -f dc583019693 -> Para o container e remove (-f remove rapidão)
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

### PostgresSQL
docker run --name some-postgres -e POSTGRES_PASSWORD=root -d -p 5432:5432 postgres

### Mongo NoSQL
# Baixa a imagem
docker pull mongo
# Remove o container, se existir
docker stop node-mongoose
docker container rm node-mongoose
# Gera o container
docker run --name node-mongoose -p 27017:27017 -d mongo
```

## Wordpress
```
https://easyengine.io/docs/install/
apt-get install wget
wget -qO ee rt.cx/ee && bash ee
ee site create dev.credicitrus.com.br --wp
WordPress admin user : Gustavo
WordPress admin user password : 5BomViySQlqZ6FC

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
ENTRYPOINT ["npm","start"] <- Dá para usar assim tb
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
password
docker push gustavoc/node -> Subindo a imagem
```

## Networking no Docker - Como interligar diversos containers (MySQL, App, Cache)
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
#Docker Compose - Múltiplos containers
#Ele verifica o arquivo docker-compose.yml ("Ialmi") "Receita de bolo"

NGINX "Endnex" - Faz o load balancer, balanceamento de acessos para cada servidor lógico
NGINX "Endnex" - Cuida dos arquivos estáticos: CSS, JS, IMG

Mãos a obra, BD + 3 Servidores + NGINX
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
      - "80:80" <- Porta de dentro e fora
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
      - "mongodb" <- Sobe depois esse
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
#Docker Swarm facilita a criação e administração de um cluster de containers.
#Docker Cloud - Ele instala a máquina virtual na Amazon e sob os containers
```

## Treinamento Alura - Kubernetes: Pods, Services e ConfigMaps
```
Kubernetes:
Escalabilidade horizontal, gerencia Cluster de máquinas em paralelo executando os containers.
Orquestrador de containers. Ele sabe quando um container está em processamento alto e pode ativar outro.

Pods:
Capsula com um ou mais containers
Cda um tem um endereço IP:10.0.0.1 e os containers dentro tem as portas :8080, :9000.
Caso todos os conteiners falhem ele morre, nascem para serem substituidos.
Compartilham namespaces de rede e IPC
Podem compartilhar volumes

Services - SVC:
Abstração para expor aplicações executando em um ou mais pods.
IP's fixos para comunicação
DNS para um ou mais pods
Capas de fazer balanceamento de carga
SVC é dividido em:
ClusterIP    - Deixar o IP fixo ṕara comunicação interna dos Pods
NodePort     - Comunicação com o mundo externo, fora dos Pods, funciona como ClusterIP tb
LoadBalancer - 

ConfigMaps:
Armazena as variáveis de ambiente para utilizarmos nos Pods

Master: Control Plane
Gerencia o cluster                      - api - recebe e executa - rest
Mantem e atualiza o estado desejado     - c-m - controler manager
Recebe a executa novos comandos         - sched - definir aonde o pod será executado e etcd - armazena dados vitais no banco chave valor
Node:   Nodes
Executa as aplicações (Pods).           - kubelet - executar pod dentro dos nodes, kube-proxy - comunicação entre os nodes

api - Ela cria um pod, edita replica set (RS), lê os dados de um deploy e deleta volumes, 
Kubectl  - Ferramenta para se comunicar com a api para Criar, Ler, Atualizar e Remover dados do Cluster via a api rest.
Minikube - Cria um cluster com um ambiente virtualizado para testarmos
```

### Install Kubernetes
```
https://kubernetes.io/docs/tasks/tools/install-kubectl/
sudo apt-get install curl -y
curl -LO "https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl

https://kubernetes.io/docs/tasks/tools/install-minikube/
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 \ && chmod +x minikube
sudo mkdir -p /usr/local/bin/
sudo install minikube /usr/local/bin/

Baixar o Virtual Box
https://www.virtualbox.org/wiki/Linux_Downloads
sudo dpkg -i virtualbox-6.1_6.1.12-139181_Ubuntu_bionic_amd64.deb 
Utilizaremos o driver de virtualização dele
```

### Commands
```
minikube start --vm-driver=virtualbox       <- Criar/Iniciar o Cluster
Criando um Pod
kubectl run nginx-pod --image=nginx:latest  <- Imagem que vai baixar, nginx-pod - nome do pod
kubectl get pods                            <- Verificando
kubectl get pods --watch                    <- Tempo real
kubectl get pods -o wide                    <- Exibe mais informações do Pod, inclusive IP
kubectl describe pod nginx-pod              <- Informações do Pod
kubectl edit pod nginx-pod                  <- Editar informações do Pod
https://cursos.alura.com.br/course/kubernetes-pods-services-configmap/task/79659
kubectl delete pod nginx-pod                <- Deletar pod, ao Deletar o IP muda
kubectl delete pods --all                   <- Kill all
kubectl delete svc --all
kubectl delete configmap --all
kubectl get configmap                       <- Ver os configmaps
kubectl describe configmap database-configmap

#### Executar o arquivo para criar o Pod
cd examples/
kubectl apply -f ./first-pod.yaml           <- Create
kubectl delete -f ./first-pod.yaml          <- Delete
https://kubeyaml.com/ <- Validador de yaml 

kubectl exec -it news-portal -- bash        <- Entrar no container dentro do Pod

kubectl describe pod news-portal
Name:         news-portal
Namespace:    default
Priority:     0
Node:         minikube/192.168.99.100
Status:       Running
IP:           172.17.0.3                    <- IP do Pod para comunicar dentro do Cluster

ClusterIP - svc-pod-2 - interno
kubectl apply -f ./svc-pod-2.yaml           <- Criar
kubectl get svc -o wide                     <- Ler os services
kubectl exec -it pod-1 -- bash
curl 10.104.118.88:80
Para o service encontrar o Pod precisa add a label nop service
selector: app: second-pod                    <- Label que ele pesquisa
ports: - port: 80                            <- E roda nessa porta
No Pod vc precisa adicionar a label tb
labels: app: second-pod                      <- Agora ele sabe o IP para comunicação dos Pods
ports: - port: 80                            <- Entrada e container 80
ports: - port: 9000 targetPort: 80           <- Entrada 9000 e container 80

NodePort - svc-pod-1 - externo
kubectl get nodes -o wide                   <- Para descobrir o IP no Linux, no Windowns ele reconhece como localhost
INTERNAL-IP -> 192.168.99.100
kubectl get svc
PORT(S) - NodePort -> 80:30893/TCP
Agora vc consegue acessar no Navegador
http://192.168.99.100:30893/

Variáveis de ambiente no MySQL
kubectl exec -it database -- bash
mysql -u root -p
q1w2e3r4

kubectl exec -it backend -- bash
Verificar as variaveis de ambiente utilizadas em bancodedados.php

https://kubernetes.io/docs/concepts/services-networking/service/#nodeport

http://192.168.99.100:30000     <- Ler as noticias
http://192.168.99.100:30001     <- Cadastrar as noticias
admin
admin

```
