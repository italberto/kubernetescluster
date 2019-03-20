# Passos para configurar o cluser Kubernetes
##### Autor: italberto@gmail.com

Nesse tutorial ensino como configuar um cluster Kubernetes com uma máquina Master e dois Nós.
Os comandos que devem ser executados com privilégios elevados são precedidos por  \#, os demais com $ no início da linha.

**OBS:** *Caso esteja usando máquinas virtuais, garanta que as máquinas usadas tenham pelo menos dois núcleos reservados, pois 2 é o número de núcleos mínimo necessário para iniciar o cluster.*

## Desativar o swapping

	# swapoff -a

	# sed -i '/swap/s/^/#/g' /etc/fstab

OU

	# nano /etc/fstab

Comentar com # a linha tipo Swap

## Atualizar o nome do host

	# echo 'kmaster' > /etc/hostname

	# echo 'knode' > /etc/hostname


## Atualizar os arquivos de host de Master e Nós

Master, Nó 01 e Nó 02

	# sed -i '1 i192.168.0.2 kmaster' /etc/hosts

	# sed -i '1 i192.168.0.3 knode01' /etc/hosts

	# sed -i '1 i192.168.0.4 knode02' /etc/hosts

 O comando **sed -i '1 i'** /arquivo insere uma linha no início de um arquivo, nesse caso o mapeamento do ip e nome de host no arquivo **hosts**.


## Configurar IP estático nas máquinas.

Nesse caso utilizamos a interface **enp0s8**. Usamos o comando printf em vez do echo para poder inserir quebra de linhas (\n).



### No Master

	printf '\nauto enp0s8\niface enp0s8 inet static\naddress 192.168.0.2\n' >> /etc/network/interfaces

### No Nó 01

	printf '\nauto enp0s8\niface enp0s8 inet static\naddress 192.168.0.3\n' >> /etc/network/interfaces

### No Nó 02

	printf '\nauto enp0s8\niface enp0s8 inet static\naddress 192.168.0.4\n' >> /etc/network/interfaces


Ao final reiniciar a(s) máquina(s).

## Instalar as dependências

	# apt-get update && apt-get install -y apt-transport-https curl

	# curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -

	# printf 'deb http://apt.kubernetes.io/ kubernetes-xenial main\n' > /etc/apt/sources.list.d/kubernetes.list

	# apt-get update

	# apt-get install -y openssh-server docker.io kubelet kubeadm kubectl 

## Atualizar o arquivo de configuração do Kubernetes

Adicionar a linha a seguir após o último item Environment e antes do primeiro ExecStart.

	Environment="cgroup-driver=systemd/cgroup-driver=cgroupfs"

	# nano /etc/systemd/system/kubelet.service.d/10-kubeadm.conf


# Configurações no Master (Kmaster)

## Iniciando o cluster

	# kubeadm init --apiserver-advertise-address=192.168.0.2 --pod-network-cidr=192.168.0.0/16

Pode levar alguns minutos para tudo ficar pronto.

O comando irá sugerir dois passos que devem ser seguidos para completar a configuração do cluster. 
Copie e salve a linha que apresenta os tokens para que um nó ingresse, futuramente, no cluster.

	kubeadm join 192.168.0.2:6443 --token: xxx --discovery-token-ca-cert-hash xxx

	$ mkdir -p $HOME/.kube
	$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
	$ sudo chown $(id -u):$(id -g) $HOME/.kube/config

Para verificar se o kubernetes está rodando, execute:

	$ kubectl get pods -o wide --all-namespaces

Se tudo estiver certo até aqui, apenas um (ou dois) pod estará no estado Pending, o de DNS (coredns). Para fazê-lo funcionar, execute a seguinte linha:

	$ kubectl apply -f https://docs.projectcalico.org/v3.0/getting-started/kubernetes/installation/hosted/kubeadm/1.7/calico.yaml 

Agora é só instalar o dashboard.

	$ kubectl create -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

## Iniciando o dashboard

	$ kubectl proxy &

Execute os seguintes comandos para criar uma conta para o dashboard no namespace default e adicionar as regras para a conta dashboard.

$ kubectl create serviceaccount dashboard -n default.

	$ kubectl create clusterrolebinding dashboard-admin -n default \
	--clusterrole=cluster-admin \
	--serviceaccount=default:dashboard

E execute também:

	$ kubectl get secret $(kubectl get serviceaccount dashboard -o jsonpath="{.secrets[0].name}") -o jsonpath="{.data.token}" | base64 --decode

Guarde co código gerado em um arquivo e em seguida abra  no browser o endereço: http://localhost:8001.
Na tela que aparecer, escolha a opção de autenticação por Token. E copie no campo o token gerado no passo anterior.


# Configurações no(s) Nó(s) (Knode)

