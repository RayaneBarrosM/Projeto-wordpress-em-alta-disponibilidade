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

❗<ins>Devem ser criadas 6 sub-redes que devem ser divididas entre duas AZ(Zonas de **disponibilidade**) e em publicas ou privadas</ins>

| Sub-nets  | Quantidade | Publica | Privada |
| ------------- | ------------- |------------- |
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
| Tabelas  | Quantidade subnets | Tabelas  | Quantidade subnets |
| ------------- | ------------- | ------------- | ------------- |
| Privada  | 2 | Privada  | 2 | Privada  | 2 |
| Privada  | 2 | Privada  | 2 | Privada  | 2 |
| Publico  | 2 | Privada  | 2 | Privada  | 2 |





## 2) Editando WordPress
Para saber mais sobre como editar o wordpress [Clique aqui](./Wordpress-Edição.md)
## 3) Criando Banco de dados
Na barra de pesquisa procure por RDS
Clique em Criar banco de dados
Criação padrão
na seção de configuração escolha o db MySQL

