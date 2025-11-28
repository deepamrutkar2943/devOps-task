# frontend/Dockerfile
FROM node:20-alpine

WORKDIR /usr/src/app

# Install Angular CLI
RUN npm install -g @angular/cli@15

# Install dependencies
COPY package*.json ./
RUN npm install

# Copy source
COPY . .

EXPOSE 8081

# Angular dev server inside the container
CMD ["ng", "serve", "--host", "0.0.0.0", "--port", "8081"]
