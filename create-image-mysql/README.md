# Create image on the docker hub

````
cd create-image-mysql
docker login
docker build -t gustavocarvalhoti/luizalabs-db:1 .
docker push gustavocarvalhoti/luizalabs-db:1
````

# Create image and use with a volume

````
Create a docker-compose.yml in you project
For start use: docker-compose up
Verify this example on the https://github.com/gustavocarvalhoti/quarkus

version: '3'

services:
    luizalabs-api:
        image: quarkus/luizalabs-api:8
        build:
            context: ./
            dockerfile: src/main/docker/Dockerfile.${QUARKUS_MODE:-jvm}
        environment:
            QUARKUS_DATASOURCE_URL: jdbc:mysql://mysql-luizalabs:3306/luizalabs?autoReconnect=true
        networks:
            - mysql-db
        ports:
            - 8080:8080
        depends_on:
            - mysql-luizalabs

    mysql-luizalabs:
        image: gustavocarvalhoti/luizalabs-db:3
        environment:
            - MYSQL_ROOT_PASSWORD=root
            - MYSQL_PASSWORD=root
        volumes:
            - mysql.luizalabs:/var/lib/mysql
        ports:
            - 3306:3306
        networks:
            - mysql-db

networks:
    mysql-db:
        driver: bridge

volumes:
    mysql.luizalabs:
````