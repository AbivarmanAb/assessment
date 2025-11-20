Create docker-compose.yml


      version: '3.9'
      
      services:
        jenkins:
          image: jenkins/jenkins:lts
          container_name: jenkins
          user: root
          ports:
            - "8080:8080"
            - "50000:50000"
          volumes:
            - jenkins_home:/var/jenkins_home
          healthcheck:
            test: ["CMD", "curl", "-f", "http://localhost:8080/login"]
            interval: 30s
            timeout: 10s
            retries: 5
      
        redis:
          image: redis:7
          container_name: redis
          ports:
            - "6379:6379"
          command: ["redis-server", "--appendonly", "yes"]
          volumes:
            - redis_data:/data
          healthcheck:
            test: ["CMD", "redis-cli", "ping"]
            interval: 10s
            timeout: 5s
            retries: 3
      
        sample-app:
          build: ./sample-app
          container_name: sample-app
          depends_on:
            - redis
          environment:
            - REDIS_HOST=redis
          ports:
            - "3000:3000"
          healthcheck:
            test: ["CMD", "curl", "-f", "http://localhost:3000/"]
            interval: 10s
            timeout: 5s
            retries: 5
      
        nginx:
          image: nginx:latest
          container_name: nginx
          depends_on:
            - sample-app
          ports:
            - "80:80"
          volumes:
            - ./nginx.conf:/etc/nginx/nginx.conf:ro
          healthcheck:
            test: ["CMD", "curl", "-f", "http://localhost/"]
            interval: 15s
            timeout: 5s
            retries: 5
      
      volumes:
        jenkins_home:
        redis_data:

  Setup Script (setup.sh)

    #!/bin/bash

      echo "==> Checking Docker installation..."
      if ! command -v docker &> /dev/null
      then
          echo "Docker is NOT installed. Install Docker first."
          exit 1
      fi
      
      echo "==> Checking Docker Compose..."
      if ! command -v docker compose &> /dev/null
      then
          echo "Docker Compose not found. Install Docker Compose v2."
          exit 1
      fi
      
      echo "==> Creating required directories..."
      mkdir -p sample-app
      
      echo "==> Starting all services using Docker Compose..."
      docker compose up -d --build
      
      echo "==> Waiting for containers to become healthy..."
      sleep 10
      
      echo "==> Containers Status:"
      docker ps --format "table {{.Names}}\t{{.Status}}\t{{.Ports}}"
      
      echo "==> Showing Jenkins initial password..."
      docker exec jenkins cat /var/jenkins_home/secrets/initialAdminPassword
      
      echo "==> Logs Available:"
      echo "  - Jenkins: docker logs -f jenkins"
      echo "  - Redis: docker logs -f redis"
      echo "  - Sample App: docker logs -f sample-app"
      echo "  - Nginx: docker logs -f nginx"
