# PosTech-ComandosDocker

## Intalação no windows
No powershell instalar o wsl com ```wsl --instal```, logo após podemos baixar o executavel no site oficial.

## Aula 1
- Para verificar a versão do docker: ```docker version```
- Para verificar os containers em execução: ```docker ps```
  - Parâmetro ```-a``` para listar os que estão parados
- Para executaremos nosso primeiro container Linux e interagiremos com ele via linha de comando:
  O parâmetro ```-it``` permite acessar o container. Ubuntu é a imagem do container que vamos usar, latest é a tag da versão e /bin/bash é o ponto de entrada que queremos, no caso, abrir o terminal bash: ```docker run -it ubuntu:latest /bin/bash```
  - Podemos usar o parâmetro ```--name <nome>``` para nomearmos o container a nossa maneira
  - Agora que abrimos o container, podemos visualizar a versão do sistema que estamos utilizando: ```cat /etc/os-release```
- Para parar um container: ```docker stop <containerId or name>```
- Para dar start em um container: ```docker start <containerId or name>```
- Para remover um container: ```docker rm <containerId or name>```
  - Podemos utilizar o parâmetro ```-f``` para forçar

## Aula 2
### Gerenciando imagens
- Para listar as imagens: ```docker images```
- Para remover uma imagem: ```docker image rm <imageId>```
 - Parâmetro ```-f``` para forçar - comando pode ser abreviado para ```docker rmi <imageId>```
- Para apagar toda imagem que não estiver sendo usada: ```docker image prune```
- Para baixar a imagem do repository: ```docker pull image:tag```
  - O nome da imagem e a tag podem ser encontradas no registry, no nosso caso, usamos o Docker Hub. Um outro comando que é bastante utilizado é o ```docker push```, que ao contrário do pull, serve para enviarmos uma imagem para o registry para que você utilize posteriormente, seja distribuindo de forma pública ou em um registry privado.

## Aula 3
### Dockerfile
Arquivo (sem extensão alguma) que cria a imagem
#### Comandos do arquievo
- FROM: Esse comando é obrigatório e é com ele que definimos a imagem base da nossa aplicação, por exemplo: ```FROM openjdk``` 
  - A imagem base openjdk é utilizada para executar aplicações JAVA, posto que ela retorna, além do sistema operacional que irá executar, a instalação da versão do Java que necessitamos. Uma atenção que sempre devemos ter aqui é verificar se já não há uma imagem mais próxima dos seus requisitos, como por exemplo a openjdk, e buscar sempre optar por imagens oficiais ou que passaram por testes de vulnerabilidade e segurança para evitar qualquer comprometimento da sua aplicação.
- RUN: Essa instrução também é executada no tempo de criação da imagem, que irá executar comandos da mesma forma que escrevemos comandos via terminal. Dessa forma, podemos instalar ou remover dependências no sistema, alterar configurações, permissões etc.
- CMD e ENTRYPOINT: Esses dois comandos têm a mesma função do comando RUN, que é executar um comando, mas elas são executadas no momento da criação do container e não da imagem. 
  - A diferença entre eles é que o comando CMD pode ser sobrescrito na hora de executar o container, ao passo que o comando ENTRYPOINT não permite que seja sobrescrito.
- EXPOSE: Esse comando tem como objetivo documentar no seu arquivo Dockerfile com qual porta aquele container terá comunicação. 
  - Vale destacar que, mesmo que não coloquemos esse comando no Dockerfile, é possível que seu container possa utilizar determinada porta quando usamos o parâmetro -p no comando Docker run.
- VOLUME: Essa instrução é muito importante para que, na criação da imagem, sejam gerados os metadados de que existe esse volume quando o container for executado. Mesmo assim precisamos realizar o mapeamento do volume no momento da execução, na pasta de destino máquina host, para que ela possa ser acessível. Apesar de podermos mapear qualquer caminho no momento de execução do container, é interessante também como documentação mapearmos as pastas/volumes que podem ser úteis na nossa imagem.
- WORKDIR O WORKDIR é onde iremos definir nosso local de execução dos comandos, principalmente dos ```CMD, RUN, ENTRYPOINT, ADD e COPY```. Esse caminho que definirmos também será o ponto de abertura do container quando executarmos ele no modo interativo.
- ADD e COPY: ADD copia arquivos remotos para alguma pasta na imagem e também pode copiar arquivos compactados (tar. gz) que serão descompactados na imagem automaticamente. Por questões de semântica, sempre utilize COPY para copiar arquivos e pastas locais e ADD para arquivos remotos ou compactados.

<br>

- Para compilar a imagem ```docker build . -t <nome da imagem>:<tag da image>```
- Para subir a um container dessa imagem ```docker run -p 80:80 teste-image:latest```
  - O parâmetro ```-p <porta host> <porta container>```  mapeia determinada porta do host com a porta do container.
  - O parâmetro ```-P``` vincula automaticamente alguma porta do host com alguma porta do container
  - Para ver portas vinculadas ```docker port <container id>```

Exemplo de arquivo dockerfile:
```
FROM ubuntu:22.10
RUN apt-get update
RUN apt-get install nginx -y

VOLUME ["/var/www/html"]

WORKDIR "/var/www/html/"

COPY "index.html" "index.html"

ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]

EXPOSE 80
```

### Docker Compose
Ferramenta de orquestração de contêineres que permite definir, configurar e executar vários containers Docker

Exemplo de arquivo compose (.yml):
```
services:
  db:
    # We use a mariadb image which supports both amd64 & arm64 architecture
    image: mariadb:10.6.4-focal
    # If you really want to use MySQL, uncomment the following line
    #image: mysql:8.0.27
    command: '--default-authentication-plugin=mysql_native_password'
    volumes:
      - db_data:/var/lib/mysql
    restart: always
    environment:
      - MYSQL_ROOT_PASSWORD=somewordpress
      - MYSQL_DATABASE=wordpress
      - MYSQL_USER=wordpress
      - MYSQL_PASSWORD=wordpress
    expose:
      - 3306
      - 33060
  wordpress:
    image: wordpress:latest
    volumes:
      - wp_data:/var/www/html
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=db
      - WORDPRESS_DB_USER=wordpress
      - WORDPRESS_DB_PASSWORD=wordpress
      - WORDPRESS_DB_NAME=wordpress
volumes:
  db_data:
  wp_data:
```
- O nome padrão do arquivo é docker-compose.yml e ele deve ser colocado na pasta raiz do nosso container. Para executá-lo, basta rodar o comando: ```docker compose up -d```
- Caso você precise especificar um arquivo de nome diferente, podemos fazer isso utilizando a flag ```-f``` no comando docker compose: ```$ docker compose -f FILENAME.yml up -d```
  - O parâmetro -d serve para não ficarmos com o terminal preso na execução dos containers
- Para parar os containers, utilizamos o comando down e podemos executar sem passar o nome do arquivo, caso estejamos na mesma pasta do arquivo docker-compose.yml ou usando o a flag -f para especificar qual arquivo de composse queremos desligar: ```docker compose down```

## Aula 4
### Docker hub
O Docker Hub, como falamos anteriormente, é o registry fornecido pela Docker que permite aos desenvolvedores compartilhar e gerenciar imagens de contêiner Docker. Ele funciona como um repositório centralizado de imagens de container Docker, permitindo que os usuários compartilhem e baixem imagens facilmente.

- Para logar o terminal no Docker Hub: ```docker login -u <usuario>```

Para poder subir uma imagem no registry devemos ter uma tag no padrão <seu usuario>/<image>:<tag>
Podemos pegar uma imagem fazer uma cópia dela e colocar em uma outra tag

- ```docker tag <imagem local>:<tag da imagem> <seu usuario>/<imagem>:<tag>``` - exemplo: ```docker tag teste-image:latest biel2001/teste-image:latest```

Para subir a imagem

- ```docker push <image>``` - exemplo: ```docker push biel2001/teste-image:latest```

### Persistência de Dados
Para não perdemos os dados do container alguma das soluções são

- Pegar um diretório do host e mapear com um diretório do container: ```docker run -it -v <diretório máquina>:<diretório container> ubuntu:latest /bin/bash``` or ```--mount type=bind, source= <diretório>, target= <diretório container>```
- Por volumes (administrado pelo docker): ```docker run -it -v <nome volume>:<diretório container> ubuntu:latest /bin/bash``` or ```--mount source= <nome volume>, target= <diretório container>```
  - Podemos ver os volumes com ```docker volume ls```
  - Podemos criar volumes com ```docker volume create <nome>```

## Aula 5
### Redes (foco em bridge) - dois containers se comunicando
O docker por padrão ja nos gera 3 tipos de rede, sendo a padrão a bridge.
Isso significa que ao rodarmos um container ele já tem uma rede bridge.
Podemos verificar isso por meio de um ```docker inspect <container id>```

- Para listar as redes ```docker network ls```

Porém a dessa maneira para nos comunicarmos com outro container seria necessário utilizar o IP.
Tendo em vista de que os containers podem cair e alterar o número de ip essa não é a melhor forma de acesso.
A melhor maneira é criando a nossa própria rede!

- Para criar uma rede (bridge) ```docker network create --driver bridge <nome da rede>```
- Para deletar uma rede ```docker network rm <id>```
- Para vincular um container a nossa rede ```docker run -it --name ubunto1 --network <nome rede> ubunto bash```

Com isso, ao subir outro container especificando esta mesma rede podemos dar um ping, por exemplo, por meio do nome do container.

#### Pratica de redes
```
docker pull mongo:4.4.6
docker pull aluradocker/alura-books:1.0
docker network create --driver bridge minha-bridge
docker run -d --network minha-bridge --name meu-mongo mongo:4.4.6
docker run -d --network minha-bridge --name alurabooks -p 3000:3000 aluradocker/alura-books:1.0
```
Com isso se acessarmos http://localhost:3000 veremos a aplicação no ar.
Na aplicação temos um endpoint que salva determinados livros ele pode ser acessado com http://localhost:3000/seed
agora se voltarmos a página inicial teremos os livros listados.
Testamos assim o vinculo de dois containers.
