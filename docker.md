# Docker

Para iniciar um contêiner em Docker podemos usar o seguinte comando:
```
# docker run debian
```
Através desse comando, o Docker vai iniciar um contêiner baseado em uma imagem, pacote de configurações modelo para contêineres do Docker, do Debian.
Quando iniciamos um contêiner com a imagem do Debian e usamos o CentOS como distribuição, esse contêiner deve usar o Kernel do nosso sistema CentOS e insere os binários e bibliotecas que caracterizam o Debian.

Verifique se o contêiner está em execução através do comando a seguir:
```
# docker ps
```
ou
```
# docker container ls
```

Ops, aparentemente não há nenhum contêiner em execução.
Verifique com a opção -a dos comandos acima se o contêiner que iniciamos aparece parado.
```
# docker ps -a
```
ou
```
# docker container ls -a
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
7c45a979f10c        debian              "bash"              16 minutes ago      Exited (0) 16 minutes ago                       focused_meitner
```

Repare na saída do comando, ele está apresentando informações relevantes sobre o que foi definido para o contêiner que iniciamos:
* Em CONTAINER ID temos a identificação do contêiner pelos 12 números do seu hash, podemos interagir com esse contêiner a partir desse valor ou pelo nome.
* IMAGE define a imagem base do contêiner que iniciamos, nesse caso: Debian.
* Em COMMAND temos o comando iniciado com o contêiner, que deve manter um processo persistente, também pode ser um script em alguns casos.
* CREATED indica quanto tempo faz que o contêiner foi criado.
* STATUS apresenta o estado do contêiner, nesse caso parado há 16 minutos.
* PORTS só apresenta valor se há algum serviço em execução no contêiner que usa alguma porta de rede e essa porta está exposta para acesso na rede do contêiner.
* NAME indica o nome do contêiner, nesse caso, como não definimos nenhum nome, recebemos um aleatoriamente que é composto de um adjetivo e o nome de uma personalidade.

Nosso contêiner com o Docker não está em execução pois o processo bash não mantém persistência e logo é desligado. Vamos tentar iniciá-lo com o bash e manter a persistência dele através de novos parâmetros do Docker:
```
# docker run -dti debian
e2f54cfe860d840be5d82b6a4c40e404d566c8782f6ce4bb51c10e4c942ec2dc
```

* O parâmetro -d, ou --detach, executa o contêiner em background;
* O -t, ou --tty, gera e aloca um pseudo-TTY ao contêiner;
* Já o parâmetro -i, ou --interactive, nesse caso, permite a interação com o TTY aberto através do -t.


Verifique se o nosso contêiner realmente se manteve em execução.

```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e2f54cfe860d        debian              "bash"              7 seconds ago       Up 7 seconds                            zen_sinoussi
```

Agora sim! :-)

Vamos acessar o processo em execução do nosso contêiner, o bash, através do seguinte comando:
```
# docker attach zen_sinoussi
root@e2f54cfe860d:/#
```
*Não se esqueça de substituir o nome do contêiner para o que foi gerado no seu computador.

Pronto!
Repare que sua PS1 já mudou e agora identifica o usuário root e o ID do contêiner.
Você já está interagindo com o processo bash do contêiner e pode fazer o que quiser dentro dele, nada vai interagir na sua máquina virtual.
Vamos fazer a instalação de um pacote?
```
root@e2f54cfe860d:/# apt update && apt install procps -y
```

O pacote procps contém uma série de binários usados para fazer diagnósticos em relação à processos do sistema. Através do ps, verifique os processos em execução no contêiner:
```
root@e2f54cfe860d:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.1  0.1  18120  2004 pts/0    Ss   15:44   0:00 bash
root       257  0.0  0.1  36628  1524 pts/0    R+   15:45   0:00 ps aux
```
Repare que existem apenas 2 processos em execução no contêiner, o bash do qual estamos conectados e o próprio processo do comando ps que puxou essas informações - essa é uma das ideias do uso de contêineres: isolar os processos do sistema operacional.

Vamos sair do contêiner e verificar a quantidade de processos que nós temos no nosso sistema operacional.
```
root@e2f54cfe860d:/# exit

# ps aux
```
Observe que a lista é longa, diferente do que encontramos no contêiner.

Verifique novamente se o contêiner está em execução.
```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
```
Aparentemente não né...
Isso porque, ao sair do contêiner, fechamos também o processo bash que mantinha aquele contêiner em execução. Podemos iniciar novamente o contêiner usando o parâmetro start do binário docker:
```
# docker start zen_sinoussi

# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e2f54cfe860d        debian              "bash"              11 minutes ago       Up 1 second                            zen_sinoussi
```
Vamos acessar o contêiner novamente, verificar seus processos e sair de forma que não feche o processo bash em execução.
```
# docker attach zen_sinoussi
root@e2f54cfe860d:/# ps aux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  18120  1892 pts/0    Ss   15:56   0:00 bash
root         6  0.0  0.1  36628  1528 pts/0    R+   15:59   0:00 ps aux
```
Para sair sem fechar o bash use a combinação de teclas Control + p  Control + q.
```
root@e2f54cfe860d:/# read escape sequence
```
Observe se realmente funcionou:
```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
e2f54cfe860d        debian              "bash"              13 minutes ago       Up 2 minutes                            zen_sinoussi
```
Deu certo! :-D

Pare e apague o contêiner através dos seguintes comandos:

Para parar:
```
# docker stop zen_sinoussi
```
Para apagar:
```
# docker rm zen_sinoussi
```
Ao apagar o contêiner não podemos listá-lo através do docker ps -a ou iniciá-lo novamente.

Agora vamos trabalhar com outras imagens do Docker, você pode encontrar centenas delas no repositório oficial do Docker, o Hub, acesse https://hub.docker.com/. Por padrão, todas as imagens que usamos são baixadas do Docker Hub, inclusive a do Debian que usamos até agora.

Para listar as imagens do Docker que já temos no nosso computador, usamos o seguinte comando:
```
# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
debian              latest              d508d16c64cd        2 weeks ago         101MB
```
* As informações apresentadas descrevem, primeiramente o nome da imagem;
* Em TAG, a versão da imagem que baixamos, latest referencia a última versão disponível no repositório;
* O IMAGE ID é o ID da imagem representado por um hash;
* CREATED indica quando a imagem foi lançada ou atualizada;
* SIZE referencia o tamanho da imagem em relação ao armazenamento do sistema.


Vamos baixar a imagem httpd, referente ao servidor web Apache, a partir do Docker Hub através da nossa linha de comando:
```
# docker pull httpd
```
Se você tentar executar um contêiner com uma imagem que não esteja no seu sistema, não se preocupe: o Docker automaticamente procura uma imagem com o mesmo nome que você referenciou no Docker Hub, sem necessidade de que você execute manualmente o docker pull para isso.

Verifique se a httpd está na lista de imagens locais no seu sistema:
```
# docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              d3a13ec4a0f1        13 days ago         132MB
debian              latest              d508d16c64cd        2 weeks ago         101MB
```
Dessa vez vamos usar a imagem httpd, expor a porta 80 da aplicação web que deve se iniciar e dar o nome de apache ao contêiner.
```
# docker run -d -p 80:80 --name apache httpd
```
Verifique se o contêiner está no ar e se é possível acessar a página http://192.168.16.10/ do navegador. Certamente você vai encontrar a página padrão do Apache, com a tradicional mensagem: 'It works!'.

No parâmetro -p, ou --publish, indicamos primeiramente a porta no nosso sistema operacional que irá corresponder à porta do serviço no contêiner, que é descrita logo em seguida.
```
# docker ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS              PORTS               NAMES
ea508e0a06ec        httpd               "httpd-foreground"   7 seconds ago       Up 5 seconds        0.0.0.0:80->80/tcp   apache
```
Repare que o comando usado dessa vez para manter o contêiner é httpd-foreground, este comando não é interativo, logo, em caso de acessá-lo através do parâmetro attach, não teremos nada além de registros de acesso à página padrão do Apache. Se for necessário acessar o contêiner para interação, o ideal é executar um novo processo dentro do contêiner com o bash.
```
# docker exec -ti apache bash
root@ea508e0a06ec:/usr/local/apache2#
```
Pronto, agora estamos dentro do contêiner!
Se sairmos desse processo com o convencional exit, não corremos o risco de parar o contêiner pois o processo que o mantém é o httpd-foreground.

Mesmo assim, vamos parar este contêiner e apagá-lo, podemos fazer isso tudo em um único comando:
```
# docker rm -f apache
```
Agora vamos iniciá-lo de forma diferente: personalizando a página que será apresentada no navegador. Primeiro, vamos criar um arquivo em HTML simples:
```
# vim index.html
<h1><marquee>Alright!</marquee></h1>
```
Para indicar que o arquivo index.html vai para dentro do contêiner na sua inicialização, precisamos usar o parâmetro -v, ou --volume, referenciando o arquivo index.html, a origem, e o diretório ou arquivo que ele vai substituir no contêiner, o destino (/usr/local/apache2/htdocs/)- separados por dois pontos ":".
```
# docker run -d -p 80:80 --name apache -v index.html:/usr/local/apache2/htdocs/ httpd
```
Verifique se agora a página no navegador através do endereço http://192.168.16.10/, é apresentada a palavra "Alright!" num texto em movimento.
Legal, não é?!
