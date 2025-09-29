# Projeto Wordpress em alta-disponibilidade

## 💭Objetivos

- inplantar a plataforma Wordpress na nuvem AWS
- garantir alta disponibilidade, escalabilidade e resiliencia
- armazenar dados em banco relacional SQL

## Tecnologias utilizadas
- Maquina Windows 10
- AMI Amazon/Linux
 
## 1) VPC

1. Procure por VPC
2. clique em `criar VPC`
3. Selecione a opção "VPC e muito mais
4. nomeie, escolha 2 zonas de disponibilidade, 2 sub-redes publicas e 4 sub-redes privadas
5. clique em `Criar VPC`
<img width="1322" height="848" alt="image" src="https://github.com/user-attachments/assets/6e026b77-2838-4c89-8529-a1454a153913" />

## Subnets

1. Escolha 2 sub-redes uma em cada AZ e renomeie como subnet-rds

## Tabela de rotas

As tabelas de rotas conjunto de regras que determinam o trafego

1. As tabelas foram criadas automaticamente, então verifique se o internet gateway na tabela publica e se dois natgataways foram criados para duas tabelas privadas
2. Selecione a tabela private1
3. Após isso va na aba `Associação de subrede`
4. Clique em `editar associação`
5. Selecione as subredes privadas que fazem parte da mesma zona
6. Refaça o processo com a tabela private2

<img width="686" height="370" alt="image" src="https://github.com/user-attachments/assets/52aee927-50b8-4190-a729-1f9eced35ae5" />

# 2) Security Group

Para continuar o projeto devem ser feitos securitys groups para o Bastion Host, ALB, EC2, RDS e EFS.
**Regras de entras do SG-Bastion:**

| Tipo | Origem |
| --- | --- |
| SSH | IP |
| **Regras de entras do SG-Load-Balancer:** |  |
| Tipo | Origem |
| ------------- | ------------- |
| HTTP | 0.0.0.0/0 |
| **Regras de entras do SG-EC2:** |  |
| Tipo | Origem |
| ------------- | ------------- |
| HTTP | SG-ALB |
| SSH | SG-Bastion |
| Todo trafego | IP |
| **Regras de entras do SG-RDS:** |  |
| Tipo | Origem |
| ------------- | ------------- |
| MySQL/Aurora | SG-EC2 |
| **Regras de entras do SG-EFS:** |  |
| Tipo | Origem |
| ------------- | ------------- |
| NFS | SG-EC2 |

# 3) Configurando Banco de dados(RDS)

1. Na barra de busca pesquise por RDS
2. Procure na barra lateral por `Grupos de sub-redes`
3. Clique em `Criar grupo de sub-redes de banco de dados`
4. Em zona de disponibilidade escolha `us-east-a` e `us-east-b` e selecione as subnets privadas separadas anteriormente para o rds
5. Clique em `Criar`
6. Procure na barra lateral por banco de dados
7. clique em `criar banco de dados`
8. Escolha criação padrão
9. Em opções de mecanismo escolha MYSQL versão 8.0.42
10. Escolha modelo de nivel gratuito single-AZ(para fins educativos)
11. de um nome para o banco, coloque seu nome de usuario(admin) e escolha a opção autogerenciada
12. escolha a instancia db.t3.micro
13. Escolha Grupo de segurança SG-RDS
14. Clique em `criar banco de dados`

<img width="818" height="65" alt="image" src="https://github.com/user-attachments/assets/69bc7151-1f46-49a3-a6ec-ad45ebab4998" />

## 4) Criando EFS

1. Acesse o Console do EFS
2. Clique em `Criar sistema de arquivos` e `Personalizar`
3. nomeie e passe para apro´xima pagina
4. Foram criado um **Mount Target** em cada AZ da sua VPC
5. Verifique se os IDs são das sub-nets rds
6. clique em `Criar`

## 4) Launch Template

1. crie um User data

```
#!/bin/bash
set -x

echo "- INICIANDO INSTALAÇÃO - "
echo "Data/Hora de Início: $(date)"

echo "[1/3] Instalando Docker, Docker Compose e Dependências ---"
sudo yum update -y
sudo yum install docker -y
sudo service docker start
sudo systemctl enable docker
sudo usermod -a -G docker ec2-user
echo "  -> Docker configurado e iniciado."

sudo curl -L "<https://github.com/docker/compose/releases/latest/download/docker-compose-$>(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
echo "  -> Docker Compose instalado."

sudo yum install -y amazon-efs-utils nfs-utils mysql
echo "  -> amazon-efs-utils e MySQL client instalados."

echo " [2/3] Configurando e Montando EFS ---"
sudo mkdir -p /my-compose
sudo mkdir -p /efs/wp-content
echo "  -> Diretórios criados."

# Configura variáveis
RDS_HOST="....amazonaws.com"
RDS_ADMIN_USER="..."
RDS_ADMIN_PASSWORD="..."
WP_DB_NAME="wordpress"
EFS_DNS="fs-029b5f412d959b9d1.efs.us-east-1.amazonaws.com"
echo "  -> Variáveis de RDS/EFS configuradas."

# Montagem do EFS
sudo mount -t nfs4 -o nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 $EFS_DNS:/ /efs/wp-content
echo "  -> EFS montado em /efs/wp-content."

# Configura montagem automática no fstab
echo "$EFS_DNS:/ /efs/wp-content nfs4 nfsvers=4.1,rsize=1048576,wsize=1048576,hard,timeo=600,retrans=2 0 0" | sudo tee -a /etc/fstab

echo "--- Configurando Permissões ---"
sudo chmod -R 755 /efs
sudo chown -R ec2-user:ec2-user /efs/wp-content
echo "  -> Permissões ajustadas."

echo " [3/3] Configurando DB e Iniciando WordPress ---"

# Cria banco de dados no RDS
mysql -h $RDS_HOST -u $RDS_ADMIN_USER -p$RDS_ADMIN_PASSWORD -e "CREATE DATABASE IF NOT EXISTS $WP_DB_NAME CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;"
echo "  -> Banco de dados criado/verificado no RDS."

# Cria o docker-compose.yml
cat > /my-compose/docker-compose.yml << EOF
version: '3.3'

services:
  wordpress:
    image: wordpress:latest
    restart: always
    ports:
      - "80:80"
    environment:
      WORDPRESS_DB_HOST: $RDS_HOST:3306
      WORDPRESS_DB_USER: $RDS_ADMIN_USER
      WORDPRESS_DB_PASSWORD: $RDS_ADMIN_PASSWORD
      WORDPRESS_DB_NAME: $WP_DB_NAME
    volumes:
      - /efs/wp-content:/var/www/html/wp-content
EOF
echo "  -> Arquivo docker-compose.yml criado."

echo "--- Iniciando WordPress ---"
cd /my-compose
sudo docker-compose up -d
echo "  -> Containers WordPress iniciados."

echo "=== ✅ INSTALAÇÃO CONCLUÍDA EM: $(date) ===" | sudo tee /var/log/wordpress-install.log

```

1. No console EC2 procure por modelo de execução
2. clique em `criar`
3. Dê um nome ao seu template.
4. Selecione a AMI Amazon
5. Escolha o tipo de instância `t2.micro`
6. Selecione a chave SSH usada anteriormente para acesso ou crie uma nova
7. Associe os SGs EC2
8. expanda a seção "Detalhes avançados" e cole o script

# 5) Criar o Application Load Balancer (ALB)

1. No console da EC2 procure por Load Balancer
2. clique em `Load Balancer`
3. Escolha `Application load balancer`
4. Mude para **internet facing**
5. Deixe **Grupos de IPs** desmarcado
6. Selecione suas subnets públicas
7. Escolha o security group SG-ALB
8. clique em `Criar grupo de destino`
9. nomeie e em **Configurações avançadas**
10. Porta da verificação: Porta de tráfego (Traffic port)
11. Salve e volte para o ALB
12. Atualize os grupos de destinos e selecione o que acabou de ser criado

# 6) Criar o Auto Scaling Group (ASG):

1. Clique em **Create Auto Scaling group**
2. Dê um nome ao seu ASG
3. Selecione o **Launch Template** que você acabou de criar
4. adicione a sua VPC e todas as subnets privadas -> próximo
5. Vincule o ASG ao seu Application Load Balancer(ALB) e escolha o destino
6. Passe para a próxima etapa
7. Em tamanho do grupo coloque **Capacidade desejada: 2**, **Capacidade desejada: 2** e **Capacidade desejada: 2**
8. Escolha Política de dimensionamento com monitoramento do objetivo e nomeie
9. clique em `criar`

# 6) Testando

1. Crie uma instancia para fazer a coneção com a instancia privada via Bastion Host
2. Utilize a mesma chave de acesso anterior, habilite IP publico e permita apenas trafego ssh
3. Abra o powershell ou Git Bash e rode o comando

```
  scp -i "KeyPair-04.pem" "KeyPair-04.pem" ec2-user@SEU_IP_PUBLICO_BASTION_HOST:/home/ec2-user/
  ssh -i "caminho-da-chave-de-acesso.pem" ec2-user@IP-instancia Bastion_host
  chmod 400chave-de-acesso.pem
  ssh -i "chave-de-acesso.pem" ec2-user@IP-privado-instancia-wordpress
  cat /var/log/wordpress-install.log #para ver os logs

```

**Caso de unhealth no grupo de destino**
verifique a internet com o comando `ping google.com`

## 7) Editando WordPress
Para saber mais sobre como editar o wordpress [Clique aqui](./Wordpress-Edição.md)

