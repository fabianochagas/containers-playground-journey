# Usando o Docker: criando contêineres MySQL e WordPress

Este tutorial irá guiar você na criação de dois contêineres Docker: um MySQL e um WordPress que usa o MySQL como banco de dados. Ambos os contêineres estarão na mesma "mini-rede" no Docker chamada wordpressnetwork. 

Ao invés de termos que instalar no nosso host um serviço completo de MySQL e outro serviço de Wordpress, vamos fazer isso tudo em contêiners. Isto é apenas um exemplo de como podemos lançar mão do uso de contêineres para facilitar a vida. Em muitos casos, as empresas não nos deixam instalar quase nada nos nossos laptops (por questões de segurança ou mesmo por pura encheção de saco de algum arquiteto de soluçoes que resolveu mandar uma meia-trava), entao, se tivermos apenas o docker instalado, já é um grande adianto. 

## Passo 1: Pré-requisito

Antes de começar, certifique-se de ter o Docker instalado em seu sistema. Você pode baixá-lo e instalá-lo a partir do site oficial do Docker (https://www.docker.com/get-started).

- [Instalando o Docker](https://docs.docker.com/engine/install/)

Certifique-se de que o Docker Desktop (para Windows e Mac) está rodando antes de continuar. 

O comando _Docker_ é cheio de parâmetros e com eles fazemos tudo o que precisamos e até mesmo por isso é hiper extenso. Aqui vamos nos ater apenas aos poucos comandos necessários para lançar os dois contêineres e assim termos o Wordpress funcionando.   


## Passo 1: Crie a rede no Docker 

O próximo passo é criar a rede Docker que os contêineres usarão. 
Este passo não é obrigatório, porém acho uma boa-prática para deixar numa mesma network (nesse caso podemos comparar a uma vNet) os contêineres que fazem parte do mesmo pacote de aplicaçoes. Veremos mais pra frente o que isso significa.
Para fazer isso, abra um terminal e execute o seguinte comando, a partir de qualquer diretório:

```
docker network create wordpressnetwork
```
Este comando criará uma rede Docker chamada wordpressnetwork. Lance o comando seguinte para listar quais _networks_ estão criadas em seu host (este seu computador local, pra ser mais exato. Mas já é legal começar a se acostumar com essa nomenclatura).

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
624d5dafd1f2   wordpressnetwork     bridge    local
0a0de3aeaaa0   none                null      local
```
## Passo 2: Crie uma imagem Docker do WordPress
A partir do diretório ./containers-playground-journey/docker/wordpress_mysql/dockerfiles, pelo terminal (linha de comando), execute o seguinte comando executará o build de uma imagem Docker, seguindo as instruções do arquivo ```wordpress.dockerfile```. </br>

```
docker build -f wordpress.dockerfile . -t wordpress-local:1.0.0
```

A saida sera semelhante ao que segue:
```
docker build -f wordpress.dockerfile . -t wordpress-local:1.0.0

[+] Building 1.6s (5/5) FINISHED
 => [internal] load build definition from wordpress.dockerfile                                                                         0.0s
 => => transferring dockerfile: 136B                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                      0.0s
 => => transferring context: 2B                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/wordpress:5.9-apache                                                                1.4s
 => CACHED [1/1] FROM docker.io/library/wordpress:5.9-apache@sha256:f9d68493ee98ea8f39e6e0fc2327b48e0b555ef0ec3fcc06b8d42cbc539c49a4   0.0s
 => exporting to image                                                                                                                 0.0s
 => => exporting layers                                                                                                                0.0s
 => => writing image sha256:f73319a04019577475b4318c36e547fa43967d4100bb7f2f54056a2e904d4136                                           0.0s
 => => naming to docker.io/library/mysql-local:1.0.0                                                                                   0.0s
```
Para listarmos as imagens que temos ja criadas (ou baixadas de um container-registry tipo Docker-Hub), elas serão listadas seguindo o comando: 
```
docker image ls 
REPOSITORY                      TAG       IMAGE ID       CREATED         SIZE
wordpress                       latest    8fec96b2307f   2 weeks ago     615MB
ghcr.io/linuxserver/duplicati   latest    57fc13401c85   4 weeks ago     844MB
kindest/node                    <none>    19d181509f30   3 months ago    928MB
wordpress-local                 1.0.0     f73319a04019   10 months ago   605MB
```
## Passo 2.1: Crie uma imagem Docker do MySQL
Semelhante à criaçao da imagem do WordPress, vamos criar uma imagem para o MySQL:</br>
```
docker build -f mysql.dockerfile . -t mysql-local:1.0.0
```

A saida sera semelhante ao que segue:
```
docker build -f mysql.dockerfile . -t mysql-local:1.0.0

[+] Building 0.3s (5/5) FINISHED
 => [internal] load build definition from mysql.dockerfile                                                                             0.0s
 => => transferring dockerfile: 124B                                                                                                   0.0s
 => [internal] load .dockerignore                                                                                                      0.0s
 => => transferring context: 2B                                                                                                        0.0s
 => [internal] load metadata for docker.io/library/mysql:latest                                                                        0.0s
 => [1/1] FROM docker.io/library/mysql:latest                                                                                          0.1s
 => exporting to image                                                                                                                 0.0s
 => => exporting layers                                                                                                                0.0s
 => => writing image sha256:5e06b29570aabe1cad774764c2f3c6e228a2d3ef0d02b79e626de1581d4906a2                                           0.0s
 => => naming to docker.io/library/mysql-local:1.0.0                                                                                   0.0s
 ```

## Passo 3: Crie o contêiner MySQL

O próximo passo é criar o contêiner MySQL, usando a imagem que acabamos de criar. Para fazer isso, execute o seguinte comando:
```
docker run --rm -d `
--name mysql `
--net wordpressnetwork `
-e MYSQL_DATABASE=exampledb `
-e MYSQL_USER=exampleuser `
-e MYSQL_PASSWORD=examplepassword `
-e MYSQL_RANDOM_ROOT_PASSWORD=1 `
-v ${PWD}/data:/var/lib/mysql `
mysql-local:1.0.0
```
### Observaçao:
Na linha de comando acima a gente tem o _backtick_ (`), que é apenas para continuar escrevendo o comando na linha seguinte e funciona em <u>WINDOWS</u>. Para <u>Linux/Mac</u>, troque o _backtick_ por uma barra invertida (\\).<br>


O comando _docker_ usado acima temos: 

[run](https://docs.docker.com/engine/reference/commandline/run/) - para criar e iniciar um novo contêiner a partir de uma imagem;

```--name``` - o nome do contêiner que será criado. Se não indicarmos o nome que queremos, o Docker vai sugerir um nome qualquer e certamente não é o que precisamos. Este é um passo importante, porque o Docker "gerencia" um mini-DNS interno de seus contêineres;

```--network``` - para informar ao Docker que este contêiner estará usando a rede interna que criamos;

```-e``` - é onde indicaremos as variáveis de ambiente que serão usadas pelo contêiner. Muitas aplicações se utilizam de variáveis de ambiente para "guiar" seu funcionamento. Neste exemplo, estamos informando ao MySQL que a variável ```MYSQL_ROOT_PASSWORD``` tem o valor ```senha123```

```-d``` - _Detached_ - quer dizer que o Docker vai iniciar o contêiner em background e vai informar o contêiner-ID (diferente do nome do contêiner)

```mysql-local:1.0.0``` - Neste ultimo parâmetro informamos qual é a imagem a partir da qual o contêiner será criado (mysql-local), e qual versão da imagem (1.0.0). Se esta imagem não estiver ainda sido baixada no seu host, o Docker vai fazer o _pull_ automagicamente. Este _pull_ é feito, por padrao do [Docker Hub](https://hub.docker.com/). Isto é importante para fazermos uso da exata versão da imagem que precisamos. Por exemplo: "na minha empresa usamos o MySQL versão 5.7.41", entao para esse caso usaríamos ```mysql:5.7.41```. Assim estaremos usando no nosso host a mesma versão do MySQL que está em uso na empresa, evitando incompatibilidade. Seria muito simples usarmos sempre a ultima versao disponível (```latest```), mas não é sempre assim que as empresas usam. 

```
docker run --rm -d `
--name mysql `
--net wordpressnetwork `
-e MYSQL_DATABASE=exampledb `
-e MYSQL_USER=exampleuser `
-e MYSQL_PASSWORD=examplepassword `
-e MYSQL_RANDOM_ROOT_PASSWORD=1 `
-v ${PWD}/data:/var/lib/mysql `
mysql-local:1.0.0

23720ffa2d6885966e1714a45cf77453223347192b1e8040133f7b167dc89f4a
```
Para saber quais contêineres estão em funcionamendo no host usamos o comando ```docker ps```:</br>

```
docker ps
CONTAINER ID   IMAGE               COMMAND                  CREATED         STATUS         PORTS                 NAMES
23720ffa2d68   mysql-local:1.0.0   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   3306/tcp, 33060/tcp   mysql
```
Este resultado nos mostra varias informaçoes sobre o contêiner: 
* ID do contêiner
* nome e versão da imagem
* Comando executado para iniciar o servidor (mais tarde falaremos mais sobre isso)
* portas expostas para acesso ao servidor (sim, temos agora um "servidor" MySQL rodando em contêiner)
* status da funcionamento do contêiner 
* a quanto tempo o contêiner está no ar

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
</br>

## Passo 4: Crie o contêiner WordPress

O próximo passo é criar o contêiner WordPress, usando a imagem que criamos. Para fazer isso, execute o seguinte comando:
```
docker run -d `
--rm `
-p 80:80 `
--name wordpress `
--net wordpressnetwork `
-e WORDPRESS_DB_HOST=mysql `
-e WORDPRESS_DB_USER=exampleuser `
-e WORDPRESS_DB_PASSWORD=examplepassword `
-e WORDPRESS_DB_NAME=exampledb `
wordpress-local:1.0.0
```
</br>

Liste os contêineres que estão rodando atualmente: </br>
```
docker ps
CONTAINER ID   IMAGE                   COMMAND                  CREATED              STATUS              PORTS                 NAMES
0f33e72b3fdc   wordpress-local:1.0.0   "docker-entrypoint.s…"   About a minute ago   Up About a minute   0.0.0.0:80->80/tcp    wordpress
23720ffa2d68   mysql-local:1.0.0       "docker-entrypoint.s…"   7 minutes ago        Up 7 minutes        3306/tcp, 33060/tcp   mysql
```
</br>

Note que durante a criação do contêiner WordPress (wordpress-local:1.0.0) usamos variaveis de ambiente ( marcadas com o ```-e```) que referenciam o serviço MySQL do contêiner criado anteriormente. 

</br>

## Passo 5 (e conclusão):
</br>

Pelo retorno do comando ```docker ps``` podemos ver que o contêiner wordpress-local:1.0.0 tem a porta 80 aberta. 
```
0.0.0.0:80->80/tcp
```
Esta notação diz que externamente a porta 80 esta recebendo conexões e direcionando internamente para a porta 80 do contêiner. 

Abra o navegador e na barra de endereços abra ``` localhost:80```, e a pagina inicial de configuração do WordPress abrirá, inicialmente pedindo para se escolher a lingua a usar. Após esta seleção, outra pagina abrirá para se concluir a instalação do WordPress.

</br>


Vimos que com estes dois contêineres conseguimos iniciar uma aplicação completamente stand-alone, com persistência de dados via banco de dados MySQL que tem um volume montado no file-system do host. </br>

Se quisermos parar os dois contêineres, escrevemos o comando: 
</br>

```
docker stop wordpress mysql
```
E com o comando ```docker ps``` confirmamos que os contêineres foram removidos. 
Caso queira reiniciar os contêineres, siga novamente os passos 3 e 4. Como os dados são persistidos no host, tudo o que foi feito no WordPress será recuperado normalmente, tal qual paramos e reiniciamos um serviço qualquer. 

Espero que tenhamos aprendido algo hoje.
Fiquem à vontade para compartilhar esta repo. 
