# Vagrant
Para começar a usar o Vagrant e montar o laboratório do curso, criamos um diretório para armazenar o código e iniciarmos nosso primeiro Vagrantfile (arquivo que específica as características das máquinas virtuais que usaremos no curso).
```
$ mkdir infraagil
$ cd infraagil
$ vagrant init -m
```
O último comando cria justamente o arquivo Vagrantfile no diretório corrente, através do parâmetro -m definimos que este arquivo deve conter o mínimo de informações possível, do contrário deve apresentar uma série de comentários.

Verifique o conteúdo do Vagrantfile.
```
$ cat Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "base"
end
```
* A primeira linha define o inicio das configurações através da ferramenta Vagrant, o valor "2" entre parenteses específica a versão de compatibilidade com o software, o config entre pipes define como serão configuradas as características das máquinas virtuais.
* Na segunda linha definimos o nome da box da qual a máquina virtual criada irá se basear. Boxes são pacotes de sistema operacional que normalmente são configurados com características específicas para fazerem parte de algum ambiente, ou são genéricos, como os que vamos usar, que contém apenas o sistema operacional em seu estado mínimo.
* Por fim, através do end, encerramos o bloco de configurações iniciado na palavra do, da primeira linha.


Vamos alterar a identificação da box que vamos usar inserindo no lugar de base: centos/7. Para mais informações e para encontrar novos boxes, visite o site https://app.vagrantup.com/boxes/search.
```
$ vim Vagrantfile
Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
end
```
Pronto, agora estamos prontos para iniciar uma nova máquina virtual baseada no CentOS 7! Mas antes, vamos confirmar se a sintaxe do arquivo está correta para iniciarmos o sistema.
```
$ vagrant validate
Vagrantfile validated successfully.
```
Caso a mensagem acima tenha surgido significa que está tudo certo. Agora vamos verificar como o Vagrant vai chamar a máquina virtual e qual será o provisionador padrão que ele vai usar - a plataforma de virtualização, nós precisamos que seja o VirtualBox.
```
$ vagrant status
Current machine states:

default                   not created (virtualbox)

The environment has not yet been created. Run `vagrant up` to
create the environment. If a machine is not created, only the
default provider will be shown. So if a provider is not listed,
then the machine is not created for that environment.
```
Repare que o resultado do comando apresenta uma máquina virtual com o nome default e com o estado not created, assim como entre parenteses é definido que o virtualbox é a plataforma que irá manter esta máquina.

Vamos iniciar a máquina default (este procedimento pode demorar caso o box centos/7 não estiver baixado no nosso computador).
```
$ vagrant up
```
Ao fim do procedimento execute novamente o comando para verificar o estado da máquina pelo Vagrant.
```
$ vagrant status
Current machine states:

default                   running (virtualbox)

The VM is running. To stop this VM, you can run `vagrant halt` to
shut it down forcefully, or you can run `vagrant suspend` to simply
suspend the virtual machine. In either case, to restart it again,
simply run `vagrant up`.
```
Repare que agora a máquina virtual default tem o estado running e está pronta para podermos acessá-la.

Com o vagrant ssh podemos acessar a máquina virtual através do protocolo SSH, não é necessário se preocupar com as credenciais pois o Vagrant faz troca de chaves para agilizar esse passo. As chaves e outras características ficam no diretório .vagrant, junto ao Vagrantfile.
```
$ vagrant ssh default
```
Pronto!
Você já esta conectado(a) à máquina virtual com o CentOS 7. Repare que seu usuário inicial é vagrant mas com um sudo su você pode se elevar a root.

Para sair da conexão à máquina virtual execute o comando exit ou a combinação de teclas Control e D.
```
$ exit
```
De volta ao host hospedeiro, vamos mudar um pouco as características do nosso Vagrantfile para que possamos ligar mais máquinas através dessa configuração. Antes, vamos apagar essa máquina default, não vamos precisar dela...
```
$ vagrant destroy default -f
```
O Vagrantfile pode especificar uma série de características em relação as máquinas virtuais que ele inicia, dentre elas: quantidade de cpus e memória que a máquina vai ocupar de recursos do hospedeiro, endereço IP e hostname que ela vai configurar no sistema virtual, etc.

Veja no exemplo abaixo algumas características aplicadas no Vagrantfile para iniciar uma máquina virtual de nome vm.
```
Vagrant.configure("2") do |config|
     config.vm.define "vm" do |machine|
         machine.vm.hostname = "vm"
         machine .vm.box = "ubuntu/xenial64"
         machine.vm.network "private_network", ip: "192.168.99.10", dns: "8.8.8.8"
     config.vm.provider "virtualbox" do |machine|
         machine.name = "vm"
         machine.memory = "1024"
         machine.cpus = 1
     end
     end
end
```
Nesta configuração, define-se:

* machine.vm.hostname = "vm": O hostname do sistema da máquina virtual como vm.
* machine .vm.box = "ubuntu/xenial64": A box ubuntu/xenial64, onde xenial é o codinome da versão 16.04 do sistema Ubuntu.
* machine.vm.network: As características de configuração de rede, endereço IP e DNS padrão do sistema virtual.
* config.vm.provider "virtualbox" do |machine|: Inicio de configuração para o VirtualBox, definindo recursos para a máquina virtual.
* machine.name: Define o nome da máquina para o VirtualBox.
* machine.memory : Quantidade de memória reservada para esta máquina virtual.
* machine.cpus: Define a quantidade de cores que esta maquina virtual deve alocar.


Ao verificar o estado da máquina baseado nessa nova configuração, observe que ele já apresenta o nome definido no Vagrantfile.
```
$ vagrant status
Current machine states:

vm                        not created (virtualbox)

The environment has not yet been created. Run `vagrant up` to
create the environment. If a machine is not created, only the
default provider will be shown. So if a provider is not listed,
then the machine is not created for that environment.
```
Para trabalhar com a configuração de múltiplas máquinas no mesmo Vagrantfile, basta repetir algumas configurações alterando os valores desejados, observe no exemplo a seguir:
```
Vagrant.configure("2") do |config|
     config.vm.define "vm" do |machine|
         machine.vm.hostname = "vm"
         machine .vm.box = "ubuntu/xenial64"
         machine.vm.network "private_network", ip: "192.168.99.10", dns: "8.8.8.8"
     config.vm.provider "virtualbox" do |machine|
         machine.name = "vm"
         machine.memory = "1024"
         machine.cpus = 1
     end
     end
     config.vm.define "web" do |server|
         server.vm.hostname = "vm"
         server .vm.box = "centos/7"
         server.vm.network "private_network", ip: "192.168.99.11", dns: "8.8.8.8"
     config.vm.provider "virtualbox" do |server|
         server.name = "vm"
         server.memory = "1024"
         server.cpus = 1
     end
     end
end
```
Pronto! Repare que apenas estendemos o código com as mesmas características e apenas alteramos o nome da máquina, a box, o endereço IP e a quantidade de memória para a nova máquina virtual. Repare a saída do vagrant status neste caso:
```
$ vagrant status
Current machine states:

vm                        not created (virtualbox)
web                       not created (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
Legal!
Mas não é assim que vamos montar nosso laboratório, vamos fazer de uma forma mais prática e deixando nosso código menor no arquivo Vagrantfile com o auxílio de um arquivo YAML para definir as características das máquinas virtuais e um loop de configuração para agilizar o processo.

Primeiro vamos montar um arquivo que podemos chamar de machines.yml com algumas definições para as máquinas virtuais:
```
$ vim machines.yml
- name: vm
  cpus: 1
  memory: 3072
  hostname: vm
  ip: 192.168.99.10
  sistema: ubuntu/xenial64

- name: web
  cpus: 1
  memory: 1024
  hostname: web
  ip: 192.168.99.11
  sistema: centos/7
```
Vamos preparar o Vagrantfile para interagir com os valores definidos nos blocos do aquivo machines.yml, repare como vai ficar:
```
$ vim Vagrantfile

require 'yaml'
yaml = YAML.load_file("machines.yml")

Vagrant.configure("2") do |config|
  yaml.each do |server|
    config.vm.define server["name"] do |srv|
      srv.vm.box = server["sistema"]
      srv.vm.network "private_network", ip: server["ip"]
      srv.vm.hostname = server["hostname"]
      srv.vm.provider "virtualbox" do |vb|
        vb.name = server["name"]
        vb.memory = server["memory"]
        vb.cpus = server["cpus"]
      end
    end
  end
end
```
* Agora, o que mudou foi que importamos a funcionalidade de interpretação de arquivos YAML através da primeira linha e na linha seguinte carregamos o arquivo machines.yml para o código do Vagrant.
* No inicio da configuração, repare que existe uma linha com yaml.each abrindo outras configurações através de server, isso faz com que para cada bloco do YAML as configurações seguintes carreguem e substituam valores pelos descritos e referenciados pelo nome server.
* No resto do arquivo todas as definições e características das máquinas virtuais foram substituídas pela referencia da variável server, que puxa valores equivalentes aos campos de mesmo nome descritos no machines.yml.

Vamos ver como o Vagrant apresenta o estado dessas configurações, logo depois de verificar a sintaxe do Vagrantfile:
```
$ vagrant validate
Vagrantfile validated successfully.

$ vagrant status
Current machine states:

vm                        running (virtualbox)
web                       not created (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
Não vamos iniciar as máquinas virtuais ainda pois, para nosso laboratório, vamos identificar as máquinas de outra forma. Altere o arquivo machines.yml com as seguintes características:
```
$ vim machines.yml
---
- name: tools
  cpus: 1
  memory: 1024
  hostname: tools.lab.com
  ip: 192.168.16.10
  sistema: centos/7

- name: vm1
  cpus: 1
  memory: 1024
  hostname: vm1.lab.com
  ip: 192.168.16.11
  sistema: centos/7

- name: vm2
  cpus: 1
  memory: 1024
  hostname: vm2.lab.com
  ip: 192.168.16.12
  sistema: centos/7

- name: vm3
  cpus: 1
  memory: 1024
  hostname: vm3.lab.com
  ip: 192.168.16.13
  sistema: ubuntu/bionic64
```
Pronto, agora vamos iniciar o ambiente virtual!
```
$ vagrant up
```
Por fim, vamos verificar se as máquinas puderem iniciar através da opção status:
```
$ vagrant status
Current machine states:

tools                     running (virtualbox)
vm1                       running (virtualbox)
vm2                       running (virtualbox)
vm3                       running (virtualbox)

This environment represents multiple VMs. The VMs are all listed
above with their current state. For more information about a specific
VM, run `vagrant status NAME`.
```
