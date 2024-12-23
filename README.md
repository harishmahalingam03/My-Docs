## Dockerfile for Frontend application

~~~~
FRONTEND DOCKERFILE

# Use the official Node.js image as the base image
FROM node:20.14.0

# Set the working directory in the container
WORKDIR /app

# Copy package.json and package-lock.json (if available)
COPY package*.json ./

# Install the dependencies
RUN npm install --legacy-peer-deps

# Copy the rest of your application code
COPY . .

# Expose the port your app runs on
EXPOSE 80

# Start the application
CMD ["npm", "run", "dev", "--", "-p", "80"]
~~~~
## Install docker compose , docker 
~~~~
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
 
~~~~

## Install docker after updating your local repo
~~~~
sudo apt update
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
~~~~
## Install AWS Cli before install cli install unzip to your instance
~~~~
sudo apt install unzip -y
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
~~~~
## Docker-compose file to run aura-app, aura-frontend, aura-db service
~~~~
version: '3.8'

services:
  aura-db:
    image: 270514764245.dkr.ecr.us-east-1.amazonaws.com/aura-postgres:latest
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.mylabel == db-instance
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: password
      POSTGRES_DB: hrms
      PGDATA: /data
    networks:
      - aura-network
    volumes:
      - /home/backup:/data
    ports:
      - "5432:5432"

  aura-app:
    image: 270514764245.dkr.ecr.us-east-1.amazonaws.com/aura-app:latest
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.mylabel == web-instance
    networks:
      - aura-network
    ports:
      - "8080:8080"
    depends_on:
      - aura-db  # aura-app starts after aura-db

  aura-frontend:
    image: 270514764245.dkr.ecr.us-east-1.amazonaws.com/aura-frontend:latest
    deploy:
      replicas: 1
      placement:
        constraints:
          - node.labels.mylabel == web-instance
    networks:
      - aura-network
    ports:
      - "80:80"
    depends_on:
      - aura-app  # aura-frontend starts after aura-app

networks:
  aura-network:
    driver: overlay
~~~~

## Commad to deploy as stack
~~~~
docker stack deploy -c docker-compose.yml --detach aura-stack
~~~~
## Only docker swarm service command 
## aura-app:
~~~~
docker service create \
  --name aura-app \
  --network aura-network \
  --constraint 'node.labels.mylabel == web-instance' \
  --publish published=8080,target=8080 \
  --replicas 1 \
  270514764245.dkr.ecr.us-east-1.amazonaws.com/aura-app:latest
~~~~
## aura-frontend:
~~~~
docker service create \
  --name aura-frontend \
  --network aura-network \
  --constraint 'node.labels.mylabel == web-instance' \
  --publish published=80,target=80 \
  --replicas 1 \
270514764245.dkr.ecr.us-east-1.amazonaws.com/aura-frontend:latest
~~~~
## aura-db:
~~~~
docker service create \
  --name aura-db \
  --network aura-network \
  --constraint 'node.labels.mylabel == db-instance' \
  --env POSTGRES_USER=postgres \
  --env POSTGRES_PASSWORD=password \
  --env POSTGRES_DB=hrms \
  --env PGDATA='/data' \
  --mount type=bind,source=/home/backup,target=/data \
  --publish published=5432,target=5432 \
  --replicas 1 \
  270514764245.dkr.ecr.us-east-1.amazonaws.com/aura-postgres:latest
~~~~

## To view logs of specific service
~~~~
docker service logs aura-stack_aura-db
~~~~
![image](https://github.com/user-attachments/assets/d0418d6e-4dec-4f0e-b2bc-b66b01bab53e)

## Command to Inspect the Stack deployed 
~~~~
docker stack ps aura-stack
~~~~
![image](https://github.com/user-attachments/assets/3ee6b5cb-b971-448f-a8fa-a234c350c40d)

## 
