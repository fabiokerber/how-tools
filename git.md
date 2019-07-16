# Git

Modos de configuração (por prioridade):

* --local - no repositório, configurado no .git/config
* --global - no usuário
* --system - grava no sistema, configurado no /etc/gitconfig

Comandos úteis:

* git config --list 
* git show - detalha as informações do último commit
* git blame arq00  - mostra quem fez a última alteração em um arquivo

* git reset (1) - Remove o arquivo do staging area
* git reset HEAD^/HEAD~1 (2) - Reseta um commit, o HEAD^ indica o último commit (para mais de um HEAD^^), o HEAD~num indica o número de commits a resetar.
* git checkout - volta o arquivo para o conteúdo do último commit, antes de adiciona-lo no staging area
* git revert - Reverte um commit
* git reflog - Indica as alterações nos ponteiros (referencias)
