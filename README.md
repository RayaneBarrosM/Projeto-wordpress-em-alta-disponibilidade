# Projeto Wordpress em alta-disponibilidade

## 💭Objetivos

- inplantar a plataforma Wordpress na nuvem AWS
- garantir alta disponibilidade, escalabilidade e resiliencia
- armazenar dados em banco relacional SQL

# 🏹Passo a passo

**## 1) instalar o dockerdescktop na maquina**

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

**## 2. Criação do Docker compose**
    Para utilizar o Wordpresse iremos baixa-lo por meio do Docker Compose
 
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

**## 3. Rodando a Imagem**
```
docker-compose up -d
```
<img width="401" height="82" alt="image" src="https://github.com/user-attachments/assets/dd2323af-7668-4cbb-aafd-845f0a673575" />


> [!IMPORTANT]
> Caso não consiga copiar ou não tenha conseguido fazer acesse [WordPress Image](https://hub.docker.com/_/wordpress) para informações mais detalhadas.


