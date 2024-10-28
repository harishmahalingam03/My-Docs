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
