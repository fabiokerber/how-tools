Para instalar o ansible em um sistema CentOS 7, primeiramente é necessário instalar o pacote com repositórios adicionais da RedHat, a fim de obter a última versão estável do software, através do seguinte comando:

```
# yum install epel-release -y
```

A seguir, basta instalar o Ansible.

```
# yum install ansible -y
```

Pronto!
Com o Ansible já instalado, vamos entender como funciona a estrutura de diretórios da ferramenta, acesse o diretório /etc/ansible e liste seu conteúdo.

```
# cd /etc/ansible
# ls
ansible.cfg  hosts  roles
```

Repare que existem 2 arquivos e 1 diretório vazio.

O arquivo ansible.cfg armazena elementos de configuração do Ansible, se aplicam as formas como seu dispositivo deve se conectar aos sistemas gerenciados pela ferramenta e até definições de diretórios padrões de armazenamento de códigos.


O arquivo hosts é o inventário, nele deve-se armazenar a lista de dispositivos que o Ansible pode interagir, originalmente este arquivo apenas possuí comentários com instruções de como montar este inventário.


O diretório roles mantém códigos construídos ou obtidos pela comunidade que, por propósito, devem satisfazer, de forma independente, as necessidades que o fizeram ser escrito. Usamos o comando ansible-galaxy para interagir com esses códigos.


Antes de começar a programar para o nosso laboratório, vamos interagir através do Ansible com o sistema onde ele está instalado através dos comandos Ad-Hoc.
Primeiro, vamos instalar o vim usando o seguinte comando:

```
# ansible localhost -m package -a "name=vim state=present"
```

O comando ansible chama o método ad-hoc do Ansible.
localhost indica os dispositivos que você vai interagir com as próximas instruções (com exceção de localhost e 127.0.0.1 é necessário que o dispositivo esteja registrado no inventário do arquivo hosts do Ansible ou equivalente).
-m indica o módulo que será usado para interagir com ações nos dispositivos listados anteriormente, o módulo package faz interações em pacotes no sistema).
-a define os argumentos do módulo selecionado, nesse caso o nome e estado do pacote vim no sistema operacional.


Pronto. Após a execução desse comando o pacote vim já deve estar disponível para o sistema operacional!
Você acabou de usar o método de interação ad-hoc do Ansible, esse método é sempre chamado através do comando ansible, com ele é possível fazer interações rápidas no seu ou em outros dispositivos sem a necessidade de desenvolver códigos para as instruções.

Agora vamos começar a desenvolver códigos para o Ansible executar ações no nosso dispositivo, equivalente ao que fizemos através do ad-hoc, no entanto com as instruções através de um arquivo que chamamos de playbook.

Primeiro, vamos criar um diretório para armazenar o nosso código.

```
# mkdir playbooks
# cd playbooks
```

As instruções do primeiro código definem uma instalação de pacotes também, dessa vez o pacote a ser instalado se chama wget.

```
# vim primeiro.yml
---
- hosts: localhost
  tasks:
  - package:
      name: wget
      state: present
```

No inicio da lista, o parâmetro hosts define em quais dispositivos esse código vai interagir.
Tasks define o inicio das instruções baseadas em módulos.
Package é o módulo genérico para instalação de pacotes, através dele é possível instalar pacotes em sistemas baseados em RedHat ou Debian.
Name, dentro da lista iniciada no módulo yum, define o nome do pacote a ser instalado.
State, também dentro da lista iniciada no módulo yum, define o estado do pacote no sistema, nesse caso garantindo que ele esteja instalado (do contrário inserir absent, para desinstalar).


Para executar as instruções do arquivo, basta chamá-lo através do binário ansible-playbook.

```
# ansible-playbook primeiro.yml
```

Pronto! O pacote htop já deve estar disponível no seu sistema.

Agora vamos começar a preparar o Ansible para interagir em outros dispositivos, antes de qualquer coisa preparamos seu inventário.

```
# cd /etc/ansible
# vim hosts

[lab]
192.168.16.[10:12]
```

Para nosso ambiente, criamos um grupo de hosts no inventário chamado lab que é um codinome para referenciar todas as máquinas do laboratório que, nesse caso, identificamo-las através do IP usando um intervalo com expressão regular para definir na mesma linha os endereços.

Após configurar o intervalo, voltamos ao arquivo de playbook e mudamos o valor do parâmetro hosts para o host ou grupo de hosts definidos no inventário.

```
# vim /etc/ansible/playbooks/primeiro.yml
---
- hosts: lab
  tasks:
  - package:
      name: htop
      state: present
```

Pronto!
Vamos executar a playbook e verificar se tudo funcionou bem.

```
# ansible-playbook /etc/ansible/playbooks/primeiro.yml
PLAY [all] *********************************************************************

TASK [Gathering Facts] *********************************************************
The authenticity of host '192.168.16.11 (192.168.16.11)' can't be established.
ECDSA key fingerprint is SHA256:FOULFd85SQRZ//hMmO5dlhBNORAugZFwqn3Z1hpJxL0.
ECDSA key fingerprint is MD5:0a:30:df:cd:3d:00:4a:96:ef:43:de:c5:99:d0:9c:a0.
Are you sure you want to continue connecting (yes/no)? The authenticity of host '192.168.16.12 (192.168.16.12)' can't be established.
ECDSA key fingerprint is SHA256:TWkne3G2Pk3qkCMUq0YkJvoSa9OLtTTDIanKlfx0KgY.
ECDSA key fingerprint is MD5:2c:99:0b:5e:d0:4f:20:c2:70:8c:03:d2:65:7f:28:6d.
Are you sure you want to continue connecting (yes/no)? The authenticity of host '192.168.16.10 (192.168.16.10)' can't be established.
ECDSA key fingerprint is SHA256:wm8CL5b1r1DbAKeo5YPiGI3y4ulV/uodTlbeX++5aHY.
ECDSA key fingerprint is MD5:76:64:19:54:50:23:e8:9a:91:25:4d:bc:c2:ce:12:38.
Are you sure you want to continue connecting (yes/no)?
```

Vish!
Certamente essa não era a saída que nós esperávamos :-(
Ao tentar acessar as outras máquinas virtuais do laboratório, como padrão do SSH, é necessário confirmar o registro do fingerprint do sistema... Mas imagina ter que aceitar o fingerprint de centenas de dispositivos! Para não precisar fazer isso, vamos até o arquivo de configuração do Ansible e alterar a diretiva que fará com que para todos os fingerprints sejam aceitos.

Basta acessar o arquivo, remover o comentário do parâmetro host_key_checking e manter seu valor em False para evitar a checagem dos fingerprints.

```
# vim ansible.cfg
host_key_checking = False
```

Teste novamente a execução da playbook.

```
# ansible-playbook /etc/ansible/playbooks/primeiro.yml
```

Agora vai!

A partir de agora vamos começar a trabalhar com Roles, padrões de códigos do Ansible que interagem com arquivos de variáveis, templates, etc.

As roles, por padrão ficam dispostas no diretório /etc/ansible/roles e podemos interagir com elas através do binário ansible-galaxy.

```
# cd /etc/ansible/roles
# ansible-galaxy init web
- web was created successfully
```

Através do comando ansible-galaxy usamos o parâmetro init para definir que será construída uma role com o nome que segue: web. O resultado do comando para criação da role é simples: ele cria toda estrutura de diretórios para organização do código Ansible, acesse o diretório web e repare.

```
# cd web
# ls
README.md  defaults  files  handlers  meta  tasks  templates  tests  vars
```

O arquivo README.md deve conter informações sobre o desenvolvimento da role e instruções de como executá-la.
O diretório defaults deve manter arquivos com variáveis padrões para a execução da role.
O diretório files mantém arquivos que podem ser transferidos durante a execução da role.
Em handlers podemos inserir tarefas repetitivas que serão executadas com a role para que possamos referenciá-las no código de desenvolvimento de tasks sempre que for necessário.
Em meta armazenamos metadados que podem ser úteis para publicação do código no portal do Ansible Galaxy.
As tasks são as tarefas que agiram nos computadores remotos.
Templates, assim como files, armazena arquivos que podem ser transferidos durante a execução da role, no entanto, diferente de files, em templates as variáveis contidas no arquivo são calculadas baseadas na execução vigente.
Tests define os procedimentos de teste de aderência das ações aplicadas na role.
Já em vars definimos as variáveis usadas nas roles, essas com mais prioridade do que as definidas em default.


Antes de continuar, vamos usar o portal Ansible Galaxy para procurar e usar roles disponibilizadas por outros usuários. Através do navegador, acesse galaxy.ansible.com e clique em Search para buscar códigos desenvolvidos pela comunidade, as roles devem aparecer em ordem de popularidade - repare que o usuário geerlingguy tem muitas das roles mais populares do portal.

Vamos usar a caixa de pesquisa para procurar por alguma role que nos ajude a automatizar a instalação do Docker nos nossos dispositivos. A role recomendada foi desenvolvida pelo usuário geerlingguy (https://galaxy.ansible.com/geerlingguy/docker) e tem compatibilidade com diversas distribuições de Linux.
