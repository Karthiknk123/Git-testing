**N8N Installation and configuration using Docker**

1\) Install Docker in ubuntu system

-   Update system

sudo apt update && sudo apt upgrade -y

-   Install dependencies

sudo apt install -y apt-transport-https ca-certificates curl
software-properties-common gnupg

-   Add Docker\'s official GPG key

curl -fsSL https://download.docker.com/linux/ubuntu/gpg \| sudo gpg
\--dearmor -o /etc/apt/trusted.gpg.d/docker.gpg

-   Add Docker\'s official APT repo

echo \"deb \[arch=amd64\] https://download.docker.com/linux/ubuntu
\$(lsb_release -cs) stable\" \| \\

sudo tee /etc/apt/sources.list.d/docker.list \> /dev/null

-   Install Docker CE and Compose plugin

sudo apt update

sudo apt install -y docker-ce docker-ce-cli containerd.io
docker-compose-plugin

-   Enable and start Docker

sudo systemctl enable docker

sudo systemctl start docker

-   Test Docker

docker \--version

2\) Create n8n Project Directory and file

-   mkdir \~/n8n

-   cd \~/n8n

-   nano .env

N8N_HOST=https://\<URL\>

N8N_PORT=5678

N8N_PROTOCOL=https

\# This tells n8n it is served from a subpath

N8N_PATH=\<sub-path-name\>

\# These must also use the subpath

WEBHOOK_URL=https://\<URL\>/\<Sub-path-URL\>/

N8N_EDITOR_BASE_URL=https://\<URL\>/\<Sub-path-URL\>/

N8N_API_URL=https://\<URL\>/\<Sub-path-URL\>/

\# Authentication

N8N_BASIC_AUTH_ACTIVE=true

N8N_BASIC_AUTH_USER=admin

N8N_BASIC_AUTH_PASSWORD=m0N1Y\|\>d!bkX9\>\|

\# Cookie secure flag (should be true if using HTTPS)

N8N_SECURE_COOKIE=true

\# Database config

DB_TYPE=postgresdb

DB_POSTGRESDB_HOST=\<database-server-IP\>

DB_POSTGRESDB_PORT=5432

DB_POSTGRESDB_DATABASE=\<database-name\>

DB_POSTGRESDB_USER=\<database-username\>

DB_POSTGRESDB_PASSWORD=\<database-password\>

\# Runner

N8N_RUNNERS_ENABLED=true

NOTE: For the above database configuration, create an database with
username and password and provide the same details on above file setup

4\) Create Docker Compose File

-   nano docker-compose.yml

version: \"3.7\"

services:

n8n:

image: n8nio/n8n

ports:

\- \"5678:5678\"

env_file:

\- .env \# Load ALL variables from .env into the container

volumes:

\- n8n_data:/home/node/.n8n

\- /root/n8n/custom-nodes:/home/node/.n8n/custom

restart: unless-stopped

volumes:

n8n_data:

5\) Start n8n

docker compose up -d

6\) Check if it\'s running:

docker ps

7\) Check version of n8n inside docker

docker exec -it \<container_id\> n8n \--version

8\) If N8N is installed in application server and database is in
different server, check the connection for database inside the docker

**docker exec -it \<conatiner_id\> sh** #This will prompt to shell then
check the connectivity for database port

**nc -zv \<database_IP\> \<database_port\>** #Should show open

7\) **Docker Service & System**

sudo systemctl start docker \# Start Docker service

sudo systemctl stop docker \# Stop Docker service

sudo systemctl restart docker \# Restart Docker

sudo systemctl enable docker \# Enable Docker to start on boot

sudo systemctl status docker \# Check Docker service status

-   **Docker Containers**

docker ps \# List running containers

docker ps -a \# List all containers (including stopped)

docker start \<container_id\> \# Start a stopped container

docker stop \<container_id\> \# Stop a running container

docker restart \<container_id\> \# Restart a container

docker rm \<container_id\> \# Remove a stopped container

docker inspect \<container_id\> \# Detailed info about container (JSON)

docker stats \# Live resource usage of containers

-   **Docker Logs & Exec**

docker logs \<container_id\> \# Show logs

docker logs -f \<container_id\> \# Follow logs (like tail -f)

docker logs \--tail 100 \<container_id\> \# Show last 100 log lines

docker exec -it \<container_id\> sh \# Get shell inside container

docker exec -it \<container_id\> bash \# If bash is available

docker exec -it \<container_id\> \<command\> \# Run command inside
container

-   **Docker Images**

docker images \# List images

docker rmi \<image_id\> \# Remove image

docker pull \<image_name\> \# Download image

docker build -t myimage . \# Build image from Dockerfile

docker history \<image_id\> \# Show layers of image

-   **Docker Volumes & Networks**

docker volume ls \# List volumes

docker volume inspect \<volume\> \# Inspect a volume

docker volume rm \<volume\> \# Remove a volume

docker network ls \# List networks

docker network inspect \<network\> \# Inspect network

docker network rm \<network\> \# Remove network

-   **Docker Compose**

docker compose up -d \# Start services in background

docker compose down \# Stop & remove containers, networks

docker compose restart \# Restart all services

docker compose logs -f \# Follow logs of all services

docker compose ps \# Show status of services

docker compose exec \<svc\> sh \# Exec into a service container

8\) Configure the NGINX for N8N with subpath name and without subpath
name (SSL Required)

Below mentioned without subpath name

server {

server_name \<Sub-Domain\>;

access_log /var/log/nginx/n8n-api-access.log;

error_log /var/log/nginx/n8n-api-error.log;

location / {

proxy_set_header X-Forwarded-Host \$host;

proxy_set_header X-Forwarded-Server \$host;

proxy_set_header X-Forwarded-For \$proxy_add_x_forwarded_for;

proxy_set_header X-Forwarded-Proto \$scheme;

proxy_pass http://\<Server-IP\>:5678;

proxy_http_version 1.1;

proxy_set_header Upgrade \$http_upgrade;

proxy_set_header Connection \'upgrade\';

proxy_set_header Host \$host;

proxy_cache_bypass \$http_upgrade;

}

}

9\) Below mentioned with Subpath name (Add the location above SSL
configuration file)

\# n8n served under /sflow/

location /sflow/ {

rewrite \^/sflow/(.\*)\$ /\$1 break;

proxy_pass http://\<Server-IP\>:5678;

proxy_http_version 1.1;

proxy_set_header Upgrade \$http_upgrade;

proxy_set_header Connection \'upgrade\';

proxy_set_header Host \$host;

proxy_cache_bypass \$http_upgrade;

proxy_set_header X-Forwarded-For \$remote_addr;

proxy_set_header X-Forwarded-Proto \$scheme;

proxy_set_header X-Forwarded-Host \$host;

}

10\) Check the workflow created has been appeared inside the created
postgresql database

-   **List the workflow**

sudo -u postgres psql -d \<database_name\> -c \'SELECT id, name,
\"createdAt\" FROM workflow_entity ORDER BY \"createdAt\" DESC LIMIT
5;\'

-   **Delete the workflow**

sudo -u postgres psql -d \<database_name\> -c \"DELETE FROM
workflow_entity WHERE name = \'Workflow_name\';\"

11\) Install Ollama

-   Update your server

sudo apt update && sudo apt upgrade -y

-   Install Ollama

curl -fsSL https://ollama.com/install.sh \| sh

-   Start and enable Ollama service

sudo systemctl enable ollama

sudo systemctl enable ollama

-   Check status:

systemctl status ollama

-   Verify it's running

Ollama listens on port **11434** by default

curl http://localhost:11434/api/tags

-   Pull a model

ollama pull llama3.2

11\) To connect the postgres via docker , run the below command (replace
username , database name and IP)

sudo docker run -it \--rm postgres:12-alpine psql -h 172.31.32.2 -U
mgvcl_n8n -d mgvcl_n8n_prod

12\) If 2FA login issue faced, re-set the 2FA and delete the user and
re-create the admin account and other user where all the workflow will
be safe.

Connect to postgresql psql and connect to N8N database and run below
command

First run below command

**UPDATE \"public\".\"user\"**

**SET \"settings\" = (settings::jsonb - \'twoFactorAuth\')::json**

**WHERE \"email\" = \'\_\_\_\_\';**

Then run below command\
\
**UPDATE \"public\".\"user\"**

**SET "mfaEnabled" = false**

**WHERE \"email\" = \'\_\_\_\_';**

13\) After above working fine, and after login if you faced to set-up
another 2FA again run the below command and then try to re-genarate 2FA

**UPDATE \"public\".\"user\"**

**SET**

**\"settings\" = (settings::jsonb - \'twoFactorAuth\')::json,**

**\"mfaEnabled\" = false,**

**\"mfaSecret\" = NULL,**

**\"mfaRecoveryCodes\" = NULL**

**WHERE \"email\" = \'\_\_\_\_\';**
