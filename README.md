# Projeto Wordpress em alta-disponibilidade

## 💭Objetivos

- inplantar a plataforma Wordpress na nuvem AWS
- garantir alta disponibilidade, escalabilidade e resiliencia
- armazenar dados em banco relacional SQL

# 🏹Passo a passo
## 1) Configuração de Rede (VPC)
Na barra de pesquisa procure por VPC
vá em Sub-redes e clique em <ins>`Criar sub-rede`</ins>
<img width="928" height="282" alt="image" src="https://github.com/user-attachments/assets/d89b27b8-4dd0-4e11-bb4a-00be4f2fa484" />

<img width="897" height="681" alt="image" src="https://github.com/user-attachments/assets/0610340a-b44c-4336-a4be-c2c448d822a9" />
<img width="887" height="295" alt="image" src="https://github.com/user-attachments/assets/dbf538ef-b729-4480-8dd1-a700cce448a1" />

❗Devem ser criadas 6 sub-redes e devem ser utilizadas duas AZ(Zonas de **disponibilidade**)

## 2) Criando Internet Gateway
O Internet Gateway(IGW) é usado para tráfego **PÚBLICO** (entrada e saída).

Na barra lateral escolha internet Gateway e clique no botão `Criar Gateway da Internet`

![image.png](attachment:89a2b6bb-fa34-4497-9838-f05bc6fdac49:image.png)

Após criar voce deve associa-lo a sua VPC

![image.png](attachment:07334935-1e9b-44b7-b960-6ead67b20e68:image.png)

## 4) Criando tabela de rotas

Para criar a tabela de rotas é preciso nomea-la e indicar a VPC

![image.png](attachment:a0dc66ce-34f9-4d94-afd8-ab87c84203be:image.png)

##3 Criando NAT Gateway

Para isso é necessario nomea-lo, escolher a sub-rede, definir a conectvidade da subnet e adicionar um IP elastico, caso  não tenha ele ira criar automaticamente ao clicar em alocar IP elastico. 

![image.png](attachment:c048dca4-b0e6-4d0c-b6be-4ee080408f1c:image.png)
-----------------------------------
## 1) instalar o Docker Desktop na maquina

1. Acesse https://docs.docker.com/desktop/ 
2. Escolha a instalação com base no seu SO
3. Selecione Docker Desktop for Windows - x86_64

## 2)Criando Container WordPress

**1. Crie uma pasta para armazenar seu projeto**
    
`````
    cd Documents
    
    mkdir PjWordPress 
    
    cd PjWordPress 
    

`````

**2. Criação do Docker compose**

    Para utilizar o Wordpresse iremos baixá-lo por meio do Docker Compose:
 
```
    services:
    
      wordpress:
        image: wordpress
        restart: always
        ports:
          - 8080:80 //porta padrão do wordpress
        environment:
          WORDPRESS_DB_HOST: db
          WORDPRESS_DB_USER: exampleuser
          WORDPRESS_DB_PASSWORD: examplepass
          WORDPRESS_DB_NAME: exampledb
        volumes:
          - wordpress:/var/www/html //diretorio onde será salvo as ações feitas no documento
    
      db:
        image: mysql:8.0  //versão utilizada
        restart: always
        environment:
          MYSQL_DATABASE: exampledb
          MYSQL_USER: exampleuser
          MYSQL_PASSWORD: examplepass
          MYSQL_RANDOM_ROOT_PASSWORD: '1'
        volumes:
          - db:/var/lib/mysql
    
    volumes:
      wordpress:
      db:
    
 ```
 <sub>As informações devem ser alteradas conforme as suas necessidades</sub>

**3. Rodando a Imagem**
```
docker-compose up -d
```
<img width="401" height="82" alt="image" src="https://github.com/user-attachments/assets/dd2323af-7668-4cbb-aafd-845f0a673575" />


> [!IMPORTANT]
> Caso não consiga copiar ou não tenha conseguido fazer acesse [WordPress Image](https://hub.docker.com/_/wordpress) para informações mais detalhadas.

## 2) Editando WordPress
Para saber mais sobre como editar o wordpress [Clique aqui](./Wordpress-Edição.md)
## 3) Criando Banco de dados
Na barra de pesquisa procure por RDS
Clique em Criar banco de dados
Criação padrão
na seção de configuração escolha o db MySQL

