# Atividade-Docker-AWS

## Criação do EC2

- VPC (projeto-vpc) Criada com 2 sub-redes privadas em us-east-1a, us-east-2b

- Criado um EC2 Ubuntu, t3.small, atribuido a VPC **"projeto-vpc"** e sub-rede **"us-east-1a"**

- Adicionado um script em dados de usuario para instalação e configuração do docker em tempo de inicialização de sistema:

```
#!/bin/bash
sudo apt update -y
sudo apt install docker.io -y
sudo systemctl start docker
sudo systemctl enable docker
```
