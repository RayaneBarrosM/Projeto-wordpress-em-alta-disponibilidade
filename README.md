# Projeto Wordpress em alta-disponibilidade

## 💭Objetivos

- inplantar a plataforma Wordpress na nuvem AWS
- garantir alta disponibilidade, escalabilidade e resiliencia
- armazenar dados em banco relacional SQL

## Tecnologias utilizadas
- Maquina Windows 10
- AMI Amazon/Linux
 
# 🏹Passo a passo
- VPC
- RDS
- EFS
- Launch Template
- ASG e ALB

# Configurando a Rede
## 1) VPC
1. Procure por VPC
2. clique em `criar VPC`
3. Selecione a opção "VPC e muito mais
<img width="1322" height="848" alt="image" src="https://github.com/user-attachments/assets/69e73b92-2f35-4b60-bd4a-44a0f9a1ff81" />

. clique em `Criar VPC`

-----------------------------------------manual----------------------------
## 2) Sub-redes
1. Na barra de pesquisa procure por VPC
2. Crie a VPC
3. vá em Sub-redes e clique em <ins>`Criar sub-rede`</ins>
4. Adicione a vpc
5. Em configurações nomeie, adicone a zona 1a, e o bloco CIDR
6. Repita este processo até ter 6 subredes

<img width="887" height="295" alt="image" src="https://github.com/user-attachments/assets/dbf538ef-b729-4480-8dd1-a700cce448a1" />

❗<ins>Devem ser criadas 6 sub-redes que devem ser divididas entre duas AZ(Zonas de **disponibilidade**) e em publicas ou privadas, as privadas serão divididas entre o rds e a aplicação</ins>

| Sub-nets  | Quantidade | Publica | Privada |
| ------------- | ------------- | ------------- | ------------- |
| sub-net zona 1a  | 3 | 1 | 2 |
| sub-net zona 1b  | 3 | 1 | 2 |

## 3) Criando Internet Gateway
O Internet Gateway(IGW) é usado para tráfego **PÚBLICO** (entrada e saída).
1. Na barra lateral escolha internet Gateway e clique no botão `Criar Gateway da Internet`
2. nomeio
3. Após criado associe a sua VPC 

<img width="988" height="259" alt="image" src="https://github.com/user-attachments/assets/8b07ce27-25c4-43f2-8191-3e47ec2e3c8d" />

## 4) Criando NAT Gateway
O NAT gataway permiti o tráfego de saída de internet de instâncias em uma sub-rede privada.
1. Escolha NAT Gateway e clique no botão `criar gateway NAT`
2. nomei-lo, escolha uma sub-rede publica, definia a conectvidade como publica e aloque um IP elastico(caso não tenha ele ira criar automaticamente ao clicar em alocar IP elastico).
3. para este projeto será necessario criar dois nat gateways, um para dara zona 

<img width="616" height="370" alt="image" src="https://github.com/user-attachments/assets/515aeed8-57b6-44e5-84a4-5c6e8ba5c81d" />

## 4 Criando tabela de rotas
A tabela de rotas serve como controlador de tráfego para sua nuvem privada virtual (VPC).

1. Procure na lateral por tabela de rotas e clique em `criar`
2. nomeia conforme a conectividade e indique a VPC
3. crie 3 tabelas(duas privadas e uma publica)
4. Va para a aba rotas
5. clique em `Editar rotas`
6. Nas tabelas privadas adicione **Destino**: 0.0.0.0/0 **Alvo**: IP-natgateway
7. Na tabela publica adicione **Destino**: 0.0.0.0/0 **Alvo**: IP-internet_gateway
8. Va para a aba Associação de sub-rede e clique em `Editar associação de sub-rede`
9. selecione as que que voce separou para o tipo de conectividade da tabela

<img width="1124" height="493" alt="image" src="https://github.com/user-attachments/assets/b34f8cf4-36b7-47c1-843c-a17e9e1d8810" />

Ao final teremos: 
| Tabelas  | Quantidade subnets |
| ------------- | ------------- |
| Privada  | 2 |
| Privada  | 2 |
| Publico  | 2 | 
# Security Group
Para continuar o projeto devem ser feitos securitys groups para o Bastion Host, ALB, EC2, RDS e EFS.
**Regras de entras do SG-Bastion:**
| Tipo  | Origem |
| ------------- | ------------- |
| SSH  | IP |
**Regras de entras do SG-Load-Balancer:**
| Tipo  | Origem |
| ------------- | ------------- |
| HTTP  | 0.0.0.0/0 |
**Regras de entras do SG-EC2:**
| Tipo  | Origem |
| ------------- | ------------- |
| HTTP  | SG-ALB |
| SSH  | SG-Bastion |
| Todo trafego  | IP |
**Regras de entras do SG-RDS:**
| Tipo  | Origem |
| ------------- | ------------- |
| MySQL/Aurora | SG-EC2 |
**Regras de entras do SG-EFS:**
| Tipo  | Origem |
| ------------- | ------------- |
| NFS  | SG-EC2 |


# Configurando Banco de dados(RDS)
1. Na barra de busca pesquise por RDS
2. na lateral clique em grupos de sub-redes
3. verifique se `wordpress-db-subnet-group` foi criado automaticamente
4. caso tenha sido clique em `Editar subredes`
5. Escolha as zonas us-east-a e us-east-b e as subnets separadas para o rds
    <img width="642" height="680" alt="image" src="https://github.com/user-attachments/assets/7ec7f94c-43b0-4862-b2c9-bb0091b713e0" />
6. Va para Banco de dados
7. clique em `criar banco de dados`
8. Escolha criação padrão
9. Em opções de mecanismo escolha MYSQL versão 8.0.42
10. Escolha modelo de nivel gratuito single-AZ(apenas para estudo)
11. escolha autogerenciada e coloque a senha
12. Escolha a instancia `db.t3.micro`, escolha a sub-rede criada e o grupo de segurança SG-RDS
13. Clique em `criar banco de dados`
    
<img width="818" height="65" alt="image" src="https://github.com/user-attachments/assets/69bc7151-1f46-49a3-a6ec-ad45ebab4998" />

## 3) Criando EFS

1. **Acesse o Console do EFS:**
2. Clique em `Criar sistema de arquivos` e em `Personalizado`
3. nomeie e aberte o botão `Proximo`
4. No *Mount Target* verifique se o ID das sub-redes são das separadas para o rds
5. Clique em `próximo` e depois em `criar`
## 4) Launch Template

1. crie um User data

```
#!/bin/bash
# Script de user-data para instalar WordPress via Docker Compose

# Redirecionamento simples para o log de instalação
# TUDO que for impresso daqui para frente vai para este arquivo.
exec > /var/log/wordpress-install.log 2>&1

echo "=== 🚀 INICIANDO INSTALAÇÃO AUTOMATIZADA DO WORDPRESS ==="
echo "Data/Hora: $(date)"

# ... O restante do seu script (com yum e systemctl) vem aqui ...
echo "=== 🚀 INICIANDO INSTALAÇÃO AUTOMATIZADA DO WORDPRESS ==="
echo "Data/Hora: $(date)"
# [1/4] Atualizar sistema e instalar Docker e cliente EFS
echo "[1/4] 📦 Atualizando sistema e instalando dependências..."
sudo yum update -y
sudo yum install -y docker amazon-efs-utils
echo "✅ Pacotes instalados"

# [2/4] Configurar e iniciar o Docker
echo "[2/4] 🐳 Configurando e iniciando o Docker..."
sudo service docker start
sudo usermod -a -G docker ec2-user
sudo systemctl enable docker
echo "✅ Docker configurado e iniciado"

# [3/4] Configurar e montar o EFS
echo "[3/4] 💾 Configurando e montando EFS..."
sudo mkdir -p /mnt/efs/wordpress
# SUBSTITUA o ID do EFS: fs-05d0213523a66f295
sudo mount -t efs -o tls fs-05d0213523a66f295:/ /mnt/efs
sudo chown -R ec2-user:ec2-user /mnt/efs
echo "✅ EFS montado"

# [4/4] Criar o arquivo docker-compose.yml e iniciar o WordPress
echo "[4/4] 📁 Criando o arquivo docker-compose.yml e iniciando containers..."
sudo mkdir -p /home/ec2-user/wordpress-project
cd /home/ec2-user/wordpress-project
sudo chown -R ec2-user:ec2-user /home/ec2-user/wordpress-project
sudo cat > docker-compose.yml << 'EOF'
version: '3.8'
services:
  wordpress:
    image: wordpress:latest
    container_name: wordpress
    restart: always
    ports:
      - "80:80"
    environment:
      # SUBSTITUA os valores abaixo com os seus do RDS
      WORDPRESS_DB_HOST: "database-2.ckhgqequqtkn.us-east-1.rds.amazonaws.com"
      WORDPRESS_DB_USER: "admin"
      WORDPRESS_DB_PASSWORD: "sbdpJ448!"
      WORDPRESS_DB_NAME: "database-2"
    volumes:
      # Mapeia o diretório do EFS para o contêiner
      - /mnt/efs/wordpress:/var/www/html
    networks:
      - wordpress_network
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost"]
      interval: 30s
      timeout: 10s
      retries: 5
networks:
  wordpress_network:
    driver: bridge
EOF
echo "✅ Arquivo docker-compose.yml criado"
sleep 5 # Esperar para garantir que o arquivo seja escrito
sudo docker compose up -d
echo "✅ Instalação finalizada"
```

1. No console EC2 procure por modelo de execução
2. clique em `criar`
3. Dê um nome ao seu template.
4. Selecione a **AMI Amazon**
6. **Instance type:** Escolha o tipo de instância `t2.micro`
7. **Key pair:** Selecione a chave SSH usada anteriormente ou crie um novo
8. **Security groups:** Associe os SGs EC2
9. **Em User data** e cole o script
10. Expanda a seção "Detalhes avançados" no final cole o script no user data
12. clique em `criar modelo`

# 4) Criar o Application Load Balancer (ALB)
1. O ALB distribuirá o tráfego entre as instâncias do ASG.
2. No console da AWS, vá para **EC2 > Load Balancers**
3. 
   
# 5) Criar o Auto Scaling Group (ASG):
1. Clique em **Create Auto Scaling group**.
2. Dê um nome ao seu ASG.
3. Selecione o **Launch Template** que você acabou de criar.
4. Configure as sub-redes privadas para o ASG adicionando a sua VPC e todas as subnets privadas→proximo.
5. Vincule o ASG ao seu **Application Load Balancer (ALB)**
6. Na aba **Anexar a um novo balanceador de carga escolha Application Load Balancer, nomeie, interface-facing, escolha duas sub-nets publicas e crie um novo grupo de destino**
7. Passe para a próxima etapa
8. Escolha Política de dimensionamento com monitoramento do objetivo e nomeie
9. clique em `criar`



# 6) Testando
1. crie uma instancia para fazer a coneção com a instancia via Bastion Host
2. utilize a mesma chave de acesso anterior, habilite IP publico e permita apenas trafego ssh
3. utilize o security group `SG-Bastion`
4. abra o powershell ou Git Bash e rode o comando
  ```
  scp -i "KeyPair-04.pem" "KeyPair-04.pem" ec2-user@<IP_PUBLICO_DO_BASTION_HOST>:/home/ec2-user/
  ssh -i "caminho-da-chave-de-acesso.pem" ec2-user@IP-instancia Bastion_host
  chmod 400chave-de-acesso.pem
  ssh -i "chave-de-acesso.pem" ec2-user@IP-privado-instancia-wordpress
  ```
para ver os logs digite `cat /var/log/wordpress-install.log`

**Casos de unhealth**
verifique se foram criados dois nat gateways para cada região, ambos estão com tipo de conectividade publica e IPs publicos e privados, além disso tenho duas tabelas de rotas privadas cada uma recebendo duas subnets privadas com as rotas no padrão 172.31.0.0/16	local
0.0.0.0/0	nat-xxxxxxxxxxxxx (Seu NAT Gateway ID)
-----------------------------------
## 2) Editando WordPress
Para saber mais sobre como editar o wordpress [Clique aqui](./Wordpress-Edição.md)

