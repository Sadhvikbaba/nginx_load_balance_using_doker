﻿# Node.js Express Server with Nginx Load Balancing in Docker

## 📌 Project Overview
This project demonstrates how to deploy multiple **Node.js Express servers** using **Docker** and distribute traffic using **Nginx as a load balancer**. The setup includes three Node.js services running in separate Docker containers and an Nginx server that load balances incoming requests.

## 🏗️ Project Structure
```
├── Dockerfile
├── docker-compose.yaml
├── nginx.conf
├── server.js
├── package.json
├── index.html
└── README.md
```

## 🚀 Technologies Used
- **Node.js** - Backend framework
- **Express.js** - Web server
- **Docker** - Containerization
- **Docker Compose** - Managing multi-container applications
- **Nginx** - Load balancing

---
## 🛠️ Setup & Installation

### 1️⃣ Clone the Repository
```sh
 git clone https://github.com/Sadhvikbaba/nginx_load_balance_using_docker
 cd nginx_load_balance_using_docker
```

### 2️⃣ Create and Configure Docker Containers

#### 🐳 Dockerfile
This file defines how to containerize the Node.js application.
```dockerfile
FROM node:14
WORKDIR /app
COPY server.js .
COPY package.json .
COPY index.html .
RUN npm install
EXPOSE 3000
CMD ["node", "server.js"]
```

#### 🐳 Docker Compose File
The **docker-compose.yml** file runs three instances of the app (`app1`, `app2`, `app3`) on different ports (3001, 3002, 3003).
```yaml
version: '3'
services:
  app1:
    build: .
    environment:
      - APP_NAME=App1
    ports:
      - "3001:3000"

  app2:
    build: .
    environment:
      - APP_NAME=App2
    ports:
      - "3002:3000"

  app3:
    build: .
    environment:
      - APP_NAME=App3
    ports:
      - "3003:3000"
```

### 3️⃣ Configure Nginx as a Load Balancer
The **nginx.conf** file defines the load balancing configuration.
```nginx
worker_processes  1;

events {
    worker_connections  1024;
}

http {
    include       mime.types;
    upstream nodejs_cluster {
        least_conn;
        server 127.0.0.1:3001;
        server 127.0.0.1:3002;
        server 127.0.0.1:3003;
    }

    server {
        listen 8080;
        server_name localhost;

        location / {
            proxy_pass http://nodejs_cluster;
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```

### 4️⃣ Build and Run the Containers
Run the following command to build and start all services:
```sh
docker-compose up --build -d
```
- `-d` runs the containers in detached mode.
- `--build` ensures the containers are rebuilt if needed.

### 5️⃣ Test the Load Balancer
Open your browser and go to:
```
http://localhost:8080
```
Each refresh will send the request to a different Node.js instance.

---
## 📂 File Explanations

### `server.js`
A simple Express.js server that serves an HTML file and logs requests.
```javascript
const express = require('express');
const path = require('path');
const app = express();
const port = 3000;
const service = process.env.APP_NAME;

app.use('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'index.html'));
    console.log(`Request from ${service}`);
});

app.listen(port, () => {
    console.log(`${service} is running on port ${port}`);
});
```

### `index.html`
A simple HTML page to test the server response.
```html
<!DOCTYPE html>
<html>
<head>
    <title>Load Balanced Node App</title>
</head>
<body>
    <h1>Welcome to the Load Balanced App!</h1>
</body>
</html>
```

---
## 🛑 Stop and Remove Containers
To stop and remove all running containers:
```sh
docker-compose down
```

---
## 🎯 Conclusion
This setup ensures that incoming requests are distributed across multiple Node.js servers, improving performance and availability. The **Nginx load balancer** helps to efficiently manage traffic between the containers.

### 🔗 Next Steps
- Add **HTTPS support** to Nginx.
- Implement **container health checks**.
- Deploy to a **cloud provider**.

🚀 **Happy Coding!**

