# CensoEscolarFrontend-loadbalancer

## 1. Criar a network
docker network create webnet


## 2. Subir os containers
docker run -itd --network webnet --name loadbalancer -p 82:80 nginx:1.29.3-alpine

docker run -itd --network webnet --name node1 -v CAMINHO/DO/BUILD:/usr/share/nginx/html nginx:1.29.3-alpine
docker run -itd --network webnet --name node2 -v CAMINHO/DO/BUILD:/usr/share/nginx/html nginx:1.29.3-alpine
docker run -itd --network webnet --name node3 -v CAMINHO/DO/BUILD:/usr/share/nginx/html nginx:1.29.3-alpine
docker run -itd --network webnet --name node4 -v CAMINHO/DO/BUILD:/usr/share/nginx/html nginx:1.29.3-alpine
docker run -itd --network webnet --name node5 -v CAMINHO/DO/BUILD:/usr/share/nginx/html nginx:1.29.3-alpine


## 3. Configurar o loadbalancer
docker exec -it loadbalancer sh
cd /etc/nginx/conf.d
nano default.conf

## 3.1 Edite o arquivo dessa forma
upstream webfront {
  server node1:80;
  server node2:80;
  server node3:80;
  server node4:80;
  server node5:80;
}

server {
  listen 80;
  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_pass http://webfront;
  }
}

## 3.2 No mesmo arquivo, configure os nós dessa forma
server {
  listen 80;
  server_name _;
  root /usr/share/nginx/html;
  index index.html;

  location / {
    try_files $uri /index.html;
  }

  location /static/ {
    expires 1y;
    add_header Cache-Control "public, immutable";
  }
}

## 4. Reinicie os containers
docker restart loadbalancer node1 node2 node3 node4 node5

## 5. Acesse a aplicação em
http://localhost:82



