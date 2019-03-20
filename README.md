# Passos para configurar o cluser Kubernetes

Nesse tutorial ensino como configuar um cluster Kubernetes com uma máquina Master e dois Nós.
Os comandos que devem ser executados com privilégios elevados são precedidos por  \#, os demais com $ no início da linha.

## Desativar o swapping

> \# swapoff -a

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

Nesse caso utilizamos a interface **enp0s8**.

### No Master

> echo 'auto enp0s8\n iface enp0s8 inet static\n address 192.168.0.2' >> /etc/network/interfaces

### No Nó 01

> echo 'auto enp0s8\n iface enp0s8 inet static\n address 192.168.0.3' >> /etc/network/interfaces

### No Nó 02

> echo 'auto enp0s8\n iface enp0s8 inet static\n address 192.168.0.4' >> /etc/network/interfaces


Ao final reiniciar a(s) máquina(s).

## Instalar as dependências

> \# apt-get install -y openssh-server docker.io kubelet kubeadm kubectl 

## Atualizar o arquivo de configuração do Kubernetes

Adicionar a linha a seguir após o último item Environment e antes do .rimeiro ExecStart.

> Environment=”cgroup-driver=systemd/cgroup-driver=cgroupfs”

> \# nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
