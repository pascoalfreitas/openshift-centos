# Portas necessárias para o Cluster Openshift

Abra as portas na rede que serão usadas para configurar o Cluster Openshift.
```
22 			TCP
53 or 8053		TCP/UDP
80 or 443		TCP
1936			TCP
4001			TCP
2379 and 2380		TCP
4789			UDP
8443			TCP
10250			TCP
```
Para obter mais detalhes sobre as portas do Openshift, consulte este link https://docs.openshift.com/container-platform/3.11/install/prerequisites.html


# Install Openshift 3.11 on CentOS
Provisione 'n' número de nós. Neste exemplo, criei 3 nós CentOS na AWS. Você precisa fazer as alterações necessárias para poder fazer o login no CentOS com a conta de usuário 'root'.
**Nota: Isso não é recomendado em ambiente de produção**

Habilitar acesso root em /etc/ssh/sshd_config file. Remova o comentário abaixo da configuração no arquivo. Faça login com o usuário *centos* e execute estes comandos.

```
$ sudo vi /etc/ssh/sshd_config
PermitRootLogin yes    	#Uncomment this line in /etc/ssh/sshd_config and save it.
	 
$ sudo systemctl restart sshd
```

Por padrão, o IAM não permite o login com o usuário * root * em instâncias do EC2, mas você pode usar as authroized_keys do usuário *centos  para acessá-lo.

```
$ sudo -s
$ cp /home/centos/.ssh/authorized_keys /root/.ssh/authorized_keys
```

Agora você está pronto para fazer o login com o usuário *root* com as mesmas chaves privadas que você usou para o usuário *centos*. Da mesma forma, habilite o acesso root em cada nó que fará parte do Cluster Openshift.

# Install git and checkout openshift-centos project on the *Master* node

```
$ yum install -y git
$ git clone https://github.com/pascoalfreitas/openshift-centos.git
$ cd openshift-centos
```

**Nota:** Se houver erro de "caracteres inválidos" ao executar qualquer comando de script, execute o comando sed para remover caracteres inválidos, por exemplo:
```
$ sed -i -e 's/\r$//' install-tools.sh
```


Não se esqueça de dar permissão de executável primeiro, se ainda não tiver sido concedida.
*install-tools.sh* arquivo instala todos os pré-requisitos necessários para configurar o Openshift 3.11 no CentOS. Certifique-se de estar conectado com o usuário root.

```
$ chmod +x install-tools.sh
$ ./install-tools.sh
```

Copie *install-tools.sh* de um nó para outro para que você possa executar os pré-requisitos de instalação em todos os nós. Substitua *host_ip* pelo endereço IP ou nome do host do seu nó.
```
#To ssh from one node to another using .pem file
$ scp -i keypair.pem install-tools.sh  root@host_ip:~/
$ ssh -i Openshift-keypair.pem root@host_ip
$ ./install-tools.sh
```


## Update DOMAIN_NAME in the inventory file

Você pode querer configurar um novo **nome de domínio**, você pode substituí-lo. No meu caso, o nome de domínio que usei é **aruntechhub.xyz** que adquiri em um provedor de DNS.
*Existem vários arquivos de inventário no repositório. Você pode escolher um, personalizá-lo de acordo com suas necessidades e renomeá-lo com* **inventory.ini**. 
**O mesmo arquivo de inventário será usado para instalar o cluster Openshift.** 
Substitua seu nome de domínio pelas seguintes propriedades no arquivo de inventário:
```
openshift_public_hostname=console.your_domain_name
openshift_master_default_subdomain=apps.your_domain_name
```

Assim que todas as alterações forem feitas, você está pronto para instalar o Openshift. Execute os seguintes comandos no nó mestre:
```
$ chmod +x install-openshift.sh
$ ./install-openshift.sh
```

Se todos os trabalhos ansible forem bem-sucedidos, você pode acessar o painel Openshift em **https://console.your_domain_name**
