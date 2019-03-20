# Passos para configurar o cluser Kubernetes

Nesse tutorial ensino como configuar um cluster Kubernetes com uma máquina Master e dois Nós.
Os comandos que devem ser executados com privilégios elevados são precedidos por  \#, os demais com $ no início da linha.

## Desativar o swapping

> \# swapoff -a

> \# sed -i '/swap/s/^/#/g' /etc/fstab

OU

> \# nano /etc/fstab

Comentar com # a linha tipo Swap

## Atualizar o nome do host

> \# echo 'kmaster' > /etc/hostname

> \# echo 'knode' > /etc/hostname


## Atualizar os arquivos de host de Master e Nós

Master, Nó 01 e Nó 02

> \# sed -i '1 i192.168.0.2 kmaster' /etc/hosts

> \# sed -i '1 i192.168.0.3 knode01' /etc/hosts

> \# sed -i '1 i192.168.0.4 knode02' /etc/hosts

 O comando **sed -i '1 i'** /arquivo insere uma linha no início de um arquivo, nesse caso o mapeamento do ip e nome de host no arquivo **hosts**.


## Configurar IP estático nas máquinas.

Nesse caso utilizamos a interface **enp0s8**. Usamos o comando printf em vez do echo para poder inserir quebra de linhas (\n).



### No Master

> printf '\nauto enp0s8\niface enp0s8 inet static\naddress 192.168.0.2\n' >> /etc/network/interfaces

### No Nó 01

> printf '\nauto enp0s8\niface enp0s8 inet static\naddress 192.168.0.3\n' >> /etc/network/interfaces

### No Nó 02

> printf '\nauto enp0s8\niface enp0s8 inet static\naddress 192.168.0.4\n' >> /etc/network/interfaces


Ao final reiniciar a(s) máquina(s).

## Instalar as dependências

> \# apt-get update && apt-get install -y apt-transport-https curl

> \# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

> \# printf 'deb http://apt.kubernetes.io/ kubernetes-xenial main\n' > /etc/apt/sources.list.d/kubernetes.list

> \# apt-get update

> \# apt-get install -y openssh-server docker.io kubelet kubeadm kubectl 

## Atualizar o arquivo de configuração do Kubernetes

Adicionar a linha a seguir após o último item Environment e antes do .rimeiro ExecStart.

> Environment=”cgroup-driver=systemd/cgroup-driver=cgroupfs”

> \# nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
