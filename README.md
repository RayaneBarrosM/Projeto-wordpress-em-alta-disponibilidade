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

# Configurando a Rede (VPC)
## 1) Sub-redes
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

## 2) Criando Internet Gateway
O Internet Gateway(IGW) é usado para tráfego **PÚBLICO** (entrada e saída).
1. Na barra lateral escolha internet Gateway e clique no botão `Criar Gateway da Internet`
2. nomeio
3. Após criado associe a sua VPC 

<img width="988" height="259" alt="image" src="https://github.com/user-attachments/assets/8b07ce27-25c4-43f2-8191-3e47ec2e3c8d" />

## 3) Criando NAT Gateway
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

# Configurando Banco de dados(RDS)
1. Na barra de busca pesquise por RDS
2. clique em `criar banco de dados`
3. Escolha criação padrão
4. Em opções de mecanismo escolha MYSQL
5. Escolha modelo de nivel gratuito(single-AZ)
6. Em Grupo de segurança de VPC clique em `Criar novo`
7. nomeio como  **GPseguranca-RDS**
8. Clique em `criar banco de dados`
9. Vá até o serviço EC2 -> Security Groups
10. Localize o grupo  `GPseguranca-RDSVPC` que foi criado e selecioneo
11. Na aba "Regras de entrada", clique em "Editar regras de entrada"
12. Escolha `Tipo: MYSQL/Aurora` `Origim: sg-IP_EC2` (Isso garante que APENAS as instâncias EC2 poderão acessar a porta do banco de dados.)
13.  Clique em `Salvar regras`
14.  Verifique se a rota foi criada
    
<img width="818" height="65" alt="image" src="https://github.com/user-attachments/assets/69bc7151-1f46-49a3-a6ec-ad45ebab4998" />

## 3) Criando EFS

1.primeiro crie um grupo de segurança para o EC2

1. **Acesse o Console do EFS:**
2. Clique em **"Criar sistema de arquivos"**
3. Após a criação, clique no ID do seu EFS
4. Vá para a aba **"Rede",** o EFS tentou criar um *Mount Target* em cada AZ da sua VPC.
5. Clique em atualizar → após carregar clique em `gerenciar`
6. Escolha o **Security Group das suas instâncias EC2 em ambos** *Mount Target*
7. Clique em `Salvar`
8. Vá em security groups no do console EC2
9. clique em criar, nomeie e em regras de entrada enconha `Tipo: NFS` `Origem: sg-EC2`

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

2. No console EC2 procure por modelo de execução
3. clique em `criar`
4. Dê um nome ao seu template.
5. **AMI:** Selecione uma AMI que você já tenha configurado, ou o Amazon
6. **Instance type:** Escolha o tipo de instância `t2.micro`
- **Key pair:** Selecione a chave SSH usada anteriormente para acesso
- **Security groups:** Associe os SGs EC2
- **Em User data** e cole o script

# 4) Criar o Auto Scaling Group (ASG):
1. Clique em **Create Auto Scaling group**.
2. Dê um nome ao seu ASG.
3. Selecione o **Launch Template** que você acabou de criar.
4. Configure as sub-redes privadas para o ASG adicionando a sua VPC e todas as subnets privadas→proximo.
5. Vincule o ASG ao seu **Application Load Balancer (ALB)**
6. Na aba **Anexar a um novo balanceador de carga escolha Application Load Balancer, nomeie, interface-facing, escolha duas sub-nets publicas e crie um novo grupo de destino**
7. Passe para a próxima etapa
8. Escolha Política de dimensionamento com monitoramento do objetivo e nomeie
9. clique em `criar`

# 5) Criar o Application Load Balancer (ALB)
1. O ALB distribuirá o tráfego entre as instâncias do ASG.
2. No console da AWS, vá para **EC2 > Load Balancers**.
3. verifique se seu Loadbalancer foi criado
4. vá para grupos de destino(target group)
5. verifique se o grupo foi criado
6. Entre na aba destino
7. verifique o `Health Check` ou “status de integridade ”para verificar a saúde das instâncias.
8. Se o status estiver `healthy`, isso significa que o ALB consegue se conectar à instância!

# 6) Testando
1. crie uma instancia para fazer a coneção com a instancia via Bastion Host
2. utilize a mesma chave de acesso anterior, habilite IP publico e permita apenas trafego ssh
3. crie um security group para ele
```
# Regras de entrada:
SSH | TCP | 22 | Seu-IP/32
# Regras de saida:
SSH | TCP | 22 | wordpress-ec2-sg (para acessar as instâncias WordPress)
```
4. Vá para o **Security Group das suas instâncias do WordPress** (o Security Group que o seu Launch Template está usando).
5. Exclua a regra de entrada que permite o tráfego SSH (porta 22).
6. Adicione novamente
7. **Em origem** coloque o **ID do Security Group do Bastion Host**
8. abra o powershell e rode o comando
  ```
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

