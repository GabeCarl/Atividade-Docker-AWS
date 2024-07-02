# Atividade-Docker-AWS

## Criação do EC2

- VPC (projeto-vpc) Criada com 2 sub-redes privadas em us-east-1a, us-east-2b

- Criado um EC2 Ubuntu, t3.small, atribuido a VPC **"projeto-vpc"** e sub-rede **"us-east-1a"**

- Adicionado um script em User Data para instalação e configuração do docker em tempo de inicialização de sistema:

```
#!/bin/bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```
## Criação do RDS

- Foi criado um database RDS Mysql 8.0.35
  - Nome: wp-database-mysql
  - Endpoint: wp-database-mysql.cbmgc8cwcv47.us-east-1.rds.amazonaws.com
  - Porta: 3306

- O Banco foi conectado ao EC2 e se mantém as regras do SG padrão do AWS

## Criação do Contêiner

- Foi instalado o docker-compose-v2 para rodar um arquivo .yml para inicialização automática de um contêiner wordpress

- Foi feito um arquivo .yml **"docker-compose.yml"**:

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
      - WORDPRESS_DB_HOST=wp-database-mysql.cbmgc8cwcv47.us-east-1.rds.amazonaws.com
      - WORDPRESS_DB_USER=admin
      - WORDPRESS_DB_PASSWORD=(senha)
      - WORDPRESS_DB_NAME=mysql
    volumes:
      - /wordpress:/var/www/html
```

- Ao executar o arquivo com docker-compose o wordpress é iniciado corretamente com as devidas configurações
