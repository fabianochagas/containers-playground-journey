# Usando o Docker: criando contêineres MySQL e WordPress

Este tutorial irá guiar você na criação de dois contêineres Docker: um MySQL e um WordPress que usa o MySQL como banco de dados. Ambos os contêineres estarão na mesma "mini-rede" no Docker chamada mydockernetwork. 

Ao invés de termos que instalar no nosso host um serviço completo de MySQL e outro serviço de Wordpress, vamos fazer isso tudo em contêiners. Isto é apenas um exemplo de como podemos lançar mão do uso de contêineres para facilitar a vida. Em muitos casos, as empresas não nos deixam instalar quase nada nos nossos laptops (por questões de segurança ou mesmo por pura encheção de saco de algum arquiteto de soluçoes que resolveu mandar uma meia-trava), entao, se tivermos apenas o docker instalado, já é um grande adianto. 

## Passo 1: Pré-requisito

Antes de começar, certifique-se de ter o Docker instalado em seu sistema. Você pode baixá-lo e instalá-lo a partir do site oficial do Docker (https://www.docker.com/get-started).

- [Instalando o Docker](https://docs.docker.com/engine/install/)

Certifique-se de que o Docker Desktop (para Windows e Mac) está rodando antes de continuar. 

O comando _Docker_ é cheio de parâmetros e com eles fazemos tudo o que precisamos e até mesmo por isso é hiper extenso. Aqui vamos nos ater apenas aos poucos comandos necessários para lançar os dois contêineres e assim termos o Wordpress funcionando.   

## Passo 2: Crie a rede no Docker 

O próximo passo é criar a rede Docker que os contêineres usarão. Para fazer isso, abra um terminal e execute o seguinte comando, a partir de qualquer diretório:

```
docker network create mydockernetwork
```
Este comando criará uma rede Docker chamada mydockernetwork. Lance o comando seguinte para listar quais _networks_ estão criadas em seu host (este seu computador local, pra ser mais exato. Mas já é legal começar a se acostumar com essa nomenclatura).

```
docker network ls
```
o resultado sera semelhante a esse:
```
docker network ls
NETWORK ID     NAME                DRIVER    SCOPE
07f16799bec9   bridge              bridge    local
fb5d5a10c748   host                host      local
c1be246ad7db   kind                bridge    local
624d5dafd1f2   mydockernetwork     bridge    local
0a0de3aeaaa0   none                null      local
```

## Passo 3: Crie o contêiner MySQL

O próximo passo é criar o contêiner MySQL. Para fazer isso, execute o seguinte comando:
```
docker run --name mysql-container --network mydockernetwork -e MYSQL_ROOT_PASSWORD=senha123 -d mysql:latest
```

O comando _docker_ usado acima temos: 

[run](https://docs.docker.com/engine/reference/commandline/run/) - para criar e iniciar um novo contêiner a partir de uma imagem;

```--name``` - o nome do contêiner que será criado. Se não indicarmos o nome que queremos, o Docker vai sugerir um nome qualquer e certamente não é o que precisamos. Este é um passo importante, porque o Docker "gerencia" um mini-DNS interno de seus contêineres;

```--network``` - para informar ao Docker que este contêiner estará usando a rede interna que criamos;

```-e``` - é onde indicaremos as variáveis de ambiente que serão usadas pelo contêiner. Muitas aplicações se utilizam de variáveis de ambiente para "guiar" seu funcionamento. Neste exemplo, estamos informando ao MySQL que a variável ```MYSQL_ROOT_PASSWORD``` tem o valor ```senha123```

```-d``` - _Detached_ - quer dizer que o Docker vai iniciar o contêiner em background e vai informar o contêiner-ID (diferente do nome do contêiner)

```mysql:latest``` - Neste ultimo parâmetro informamos qual é a imagem a partir da qual o contêiner será criado (mysql), e qual versão da imagem (latest). Se esta imagem não estiver ainda sido baixada no seu host, o Docker vai fazer o _pull_ automagicamente. Este _pull_ é feito, por padrao do [Docker Hub](https://hub.docker.com/). Isto é importante para fazermos uso da exata versão da imagem que precisamos. Por exemplo: "na minha empresa usamos o MySQL versão 5.7.41", entao para esse caso usaríamos ```mysql:5.7.41```. Assim estaremos usando no nosso host a mesma versão do MySQL que está em uso na empresa, evitando incompatibilidade. Seria muito simples usarmos sempre a ultima versao disponível (```latest```), mas não é sempre assim que as empresas usam. 

```
docker run --name mysql-container --network mydockernetwork -e MYSQL_ROOT_PASSWORD=senha123 -d mysql:latest
Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
767a87c58327: Downloading [============>                                      ]  11.42MB/44.56MB
cbd6d17e71a0: Download complete
9b17ad003fbc: Download complete
410b54c19b6b: Download complete
c6192cec9415: Download complete
f7be351756ff: Download complete
ae2d1ab519ee: Downloading [=========>                                         ]  10.23MB/56.23MB
119cfaa7dea0: Download complete
7176b3cc6ba1: Downloading [==========================================>        ]  42.19MB/50.19MB
2eb39e909e2b: Waiting
e935886e1025: Waiting
```
Ao final, teremos algo semelhante a isso: 
```
docker run --name mysql-container --network mydockernetwork -e MYSQL_ROOT_PASSWORD=senha123 -d mysql:latest
Unable to find image 'mysql:latest' locally
latest: Pulling from library/mysql
767a87c58327: Pull complete
cbd6d17e71a0: Pull complete
9b17ad003fbc: Pull complete
410b54c19b6b: Pull complete
c6192cec9415: Pull complete
f7be351756ff: Pull complete
ae2d1ab519ee: Pull complete
119cfaa7dea0: Pull complete
7176b3cc6ba1: Pull complete
2eb39e909e2b: Pull complete
e935886e1025: Pull complete
Digest: sha256:2596158f73606ba571e1af29a9c32bec5dc021a2495e4a70d194a9b49664f4d9
Status: Downloaded newer image for mysql:latest
9ab80579a5c56fe989decc238bbed045275d24e56c770a5e5a48fafe29606065
```

Para saber quais contêineres estão em funcionamendo no host usamos o comando "docker ps":
```
docker ps
CONTAINER ID   IMAGE          COMMAND                  CREATED         STATUS         PORTS                 NAMES
9ab80579a5c5   mysql:latest   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   3306/tcp, 33060/tcp   mysql-container
```
Este resultado nos mostra varias informaçoes sobre o contêiner: 
* ID do contêiner
* nome e versão da imagem
* Comando executado para iniciar o servidor (mais tarde falaremos mais sobre isso)
* portas expostas para acesso ao servidor (sim, temos agora um "servidor" MySQL rodando em contêiner)
* status da funcionamento do contêiner 
* tempo de "vida" do contêiner

Para lermos o log do contêiner, usamos o comando ```docker log```:</br>
```
docker logs mysql-container
2023-03-20 01:28:26+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.32-1.el8 started.
2023-03-20 01:28:26+00:00 [Note] [Entrypoint]: Switching to dedicated user 'mysql'
2023-03-20 01:28:26+00:00 [Note] [Entrypoint]: Entrypoint script for MySQL Server 8.0.32-1.el8 started.
2023-03-20 01:28:27+00:00 [Note] [Entrypoint]: Initializing database files
2023-03-20T01:28:27.118435Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
2023-03-20T01:28:27.119316Z 0 [System] [MY-013169] [Server] /usr/sbin/mysqld (mysqld 8.0.32) initializing of server in progress as process 81
2023-03-20T01:28:27.133495Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2023-03-20T01:28:27.837147Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2023-03-20T01:28:29.555170Z 6 [Warning] [MY-010453] [Server] root@localhost is created with an empty password ! Please consider switching off the --initialize-insecure option.
2023-03-20 01:28:33+00:00 [Note] [Entrypoint]: Database files initialized
2023-03-20 01:28:33+00:00 [Note] [Entrypoint]: Starting temporary server
2023-03-20T01:28:34.071653Z 0 [Warning] [MY-011068] [Server] The syntax '--skip-host-cache' is deprecated and will be removed in a future release. Please use SET GLOBAL host_cache_size=0 instead.
2023-03-20T01:28:34.076467Z 0 [System] [MY-010116] [Server] /usr/sbin/mysqld (mysqld 8.0.32) starting as process 124
2023-03-20T01:28:34.128660Z 1 [System] [MY-013576] [InnoDB] InnoDB initialization has started.
2023-03-20T01:28:34.590890Z 1 [System] [MY-013577] [InnoDB] InnoDB initialization has ended.
2023-03-20T01:28:35.025712Z 0 [Warning] [MY-010068] [Server] CA certificate ca.pem is self signed.
2023-03-20T01:28:35.025793Z 0 [System] [MY-013602] [Server] Channel mysql_main configured to support TLS. Encrypted connections are now supported for this channel.
2023-03-20T01:28:35.029418Z 0 [Warning] [MY-011810] [Server] Insecure configuration for --pid-file: Location '/var/run/mysqld' in the path is accessible to all OS users. Consider choosing a different directory.
```


