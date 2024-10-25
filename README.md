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
