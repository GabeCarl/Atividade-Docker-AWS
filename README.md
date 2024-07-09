# Atividade-Docker-AWS

## Criação do EC2

- VPC `projeto-vpc` Criada com 2 sub-redes privadas em `us-east-1a` `us-east-2b`

- Criado um EC2 Ubuntu, t3.small, atribuido a VPC `projeto-vpc` e sub-rede `us-east-1a`

- Adicionado um script em User Data para instalação e configuração do docker em tempo de inicialização de sistema:

  Ubuntu:

```
#!/bin/bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```

Amazon Linux:
```
#!/bin/bash
sudo yum update
sudo yum install docker -y
sudo systemctl start docker
sudo systemctl enable docker
```

## Criação do RDS

- Foi criado um database RDS Mysql 8.0.35
  - Nome: wp-database-mysql
  - Endpoint: wp-mysql.cbmgc8cwcv47.us-east-1.rds.amazonaws.com
  - Porta: 3306

- O Banco foi conectado ao EC2 e se mantém as regras do SG padrão do AWS

## Criação do Contêiner

- Foi instalado o docker-compose para rodar um arquivo .yml para inicialização automática de um contêiner wordpress
  - Ubuntu: ```docker-compose-v2```
 
  - Amazon Linux:
  
```
sudo curl -L "https://github.com/docker/compose/releases/latest/download/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- Foi feito um arquivo .yml `docker-compose.yml`:

```
version: '3.8'

name: wordpress
services:

  wordpress:
    image: wordpress:latest
    ports:
      - 80:80
    restart: always
    environment:
      - WORDPRESS_DB_HOST=wp-mysql.cbmgc8cwcv47.us-east-1.rds.amazonaws.com
      - WORDPRESS_DB_USER=admin
      - WORDPRESS_DB_PASSWORD=admin123
      - WORDPRESS_DB_NAME=wp_mysql
    volumes:
      - efs/wordpress:/var/www/html
```

- Ao executar o arquivo com docker-compose o wordpress é iniciado corretamente com as devidas configurações

## Criação do EFS

- Foi criado um EFS
  - Nome: EFS-Atividade-Docker

- Abrir a porta padrão do NFS(TCP) no SG

- Criado o Diretório /efs para montagem do EFS

- Instalação no nfs-common/nfs-utils para configurar a montagem do lado do client

- Instalar o "amazon-nfs-utils" no Amazon Linux / Ubuntu: segue tutorial para instalação do "amazon-efs-utils" no ubuntu, através do GitHub da AWS 
```
$ sudo apt-get update
$ sudo apt-get -y install git binutils rustc cargo pkg-config libssl-dev
$ git clone https://github.com/aws/efs-utils
$ cd efs-utils
$ ./build-deb.sh
$ sudo apt-get -y install ./build/amazon-efs-utils*deb
```

- Foi usado o comando `mount -t efs -o tls fs-010063368b4a811a5:/ efs` para montar o nfs

- Criado um diretório `/efs/wordpress` para guardar os dados públicos do wordpress

- Foi adicionado uma linha `fs-010063368b4a811a5:/ /efs efs _netdev,noresvport,tls 0 0` no fstab para montagem na inicialização do efs

- Alterar o campo `volumes` do `docker-compose.yml` para `/efs/wordpress` para direcionar os arquivos do wordpress para o efs

## Criação EC2 privada

- Foi Criado um NAT geteway para que as máquina privadas tenha acesso a internet

- Foi Criado 2 EC2 `t3.small / Amazon Linux` em sub-nets privadas em 2 AZs `east-1a` e `east-1b`

-  Uma instância pública foi usada para fazer o bastion host nas instâncias privadas

- Foi refeito todas as configurações acima

  ## Criação Load Balancer

- Foi criado um load balancer classic selecionando 2 AZs `east-1a` e `east-1b` para redirecionamento

- Necessário colocar 1 sub-rede pública para cada AZ selecionada

- Foi Apontado 2 EC2 para o LB redirecionar, cada uma em 1 AZ

##Criação Auto Scaling

- Foi Criado um modelo usando de base as atuis EC2

- Configurado a VPC e Sub-redes privadas em `east-1a` e `east-1b`

- Definido Auto Scaling para no mínimo 0 e máximo 2 instâncias, sendo desejado 1

- Configurado uma política de escalabilidade para adicionar instâncias quando CPU atingir 90% de uso
