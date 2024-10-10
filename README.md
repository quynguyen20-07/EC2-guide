# Deploy NestJS Application to AWS EC2 with Docker

This guide provides step-by-step instructions to deploy a NestJS application to an Amazon EC2 instance (Amazon Linux 2023) using Docker and Docker Compose. It also includes instructions for version management, scaling strategies, and tag management.

## Prerequisites

- AWS Account
- EC2 Instance with Amazon Linux 2023
- Security Group configured to allow:
  - Port `8000` (or the port your app uses)
  - Port `5432` for PostgreSQL (if used)

---

## Step 1: Configure Docker Compose and Dockerfile

Ensure your project has the following configuration files:

### `docker-compose.yml`

```yaml
version: '3'

services:
  app:
    build:
      dockerfile: docker/Dockerfile.local
      context: .
    container_name: ${APP_NAME}-app
    env_file:
      - .env
    ports:
      - 8000:8000
    command: sh -c "npm run start:prod"
    volumes:
      - ./:/app
      - /app/node_modules
    depends_on:
      - db

  db:
    image: postgres:14.4-alpine
    container_name: roomsbooked-db
    environment:
      POSTGRES_DB: ${POSTGRES_DB}
      POSTGRES_USER: ${POSTGRES_USER}
      POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
    ports:
      - 5432:${DATABASE_PORT}
    volumes:
      - roomsbooked-db:/var/lib/postgresql/data
    restart: unless-stopped

volumes:
  roomsbooked-db:

```

### `Dockerfile`

```yaml
FROM node:20.18.0-alpine

WORKDIR /app

COPY package*.json /app/

RUN npm install --production

RUN apk update && apk upgrade && apk add --no-cache bash git

COPY . .

RUN npm run build

EXPOSE 8000

CMD ["node", "dist/main.js"]
```

## Step 2: Create and Configure EC2 Instance (Amazon Linux 2023)

- Create EC2 Instance:
  - Log in to the AWS Console and create an EC2 instance using Amazon Linux 2023.
- Configure Security Group:
    - Ensure the security group allows inbound traffic on ports:
        - 8000 for the NestJS app.
        - 5432 for PostgreSQL (if applicable).
- Associate Elastic IP (Optional):
    - Allocate and associate an Elastic IP to your instance for a static IP address. 

## Step 3: Install Docker and Docker Compose on EC2

### 1. SSH into EC2 Instance:

```bash
 ssh -i your-key.pem ec2-user@<EC2_IP_ADDRESS>
```

### 2. Update System Packages:

```bash
 sudo yum update -y
```

### 3. Install Docker:

```bash
sudo amazon-linux-extras enable docker
```

```bash
sudo yum install docker -y
```

### 4. Start and Enable Docker:

```bash
sudo systemctl start docker
```

```bash
sudo systemctl enable docker
```

### 5. Add EC2 User to Docker Group (to avoid using sudo with Docker commands):

```bash
sudo usermod -aG docker ec2-user
```

### 6. Install Docker Compose:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

### 7. Verify Docker and Docker Compose Installation:

```bash
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```

```bash
sudo chmod +x /usr/local/bin/docker-compose
```

## Step 4: Upload Your Application to EC2

### 1. Clone from GitHub/GitLab:

```bash
git clone https://github.com/your-repo.git
```

```bash
cd your-repo
```
### 2. Upload from local using scp:

```bash
scp -r ./your-project-directory ec2-user@<EC2_IP_ADDRESS>:/home/ec2-user/
```

```bash
cd your-repo
```


## Step 5: Build and Run with Docker Compose

### 1. Navigate to Your Project Directory:

```bash
cd /home/ec2-user/<project-directory>
```

### 2. Build and Run Docker Containers:

```bash
docker-compose up -d --build
```

### 3. Verify the Application:

```bash
Access your application by visiting http://<EC2_IP_ADDRESS>:8000.
```

## Step 6: Version Management (Optional)

### 1. Build Docker Image with a Version Tag:

```bash
docker build -t my-app:v1.0.0 .
```

### 2. Run the Image with Specific Version:

```bash
docker run -d -p 8000:8000 my-app:v1.0.0
```

To automate this with Docker Compose, update the docker-compose.yml file by adding a version tag in the app service definition:

```yaml
 
app:
  image: my-app:v1.0.0
  ...

```

## Step 7: Scaling the Application (Optional)
  
To scale your application, consider the following options:

### 1. Manual Scaling with Docker Compose:

- Increase the number of containers by using:
```bash
docker-compose up --scale app=3 -d
```

### 2. AWS Auto Scaling Groups:

- Set up an Auto Scaling Group on AWS to automatically increase or decrease EC2 instances based on CPU or memory usage.


### 3. Elastic Load Balancer (ELB):

- Distribute incoming traffic across multiple EC2 instances using AWS ELB for better load distribution.


## Step 8: Tagging Resources (Optional)

Amazon EC2 allows tagging of resources such as EC2 instances, volumes, and containers for better organization and tracking.

### 1. Add Tags to EC2 Instance:
- Install AWS CLI if not already installed:

```bash
sudo yum install awscli -y
```

### 2.  Add tags to your EC2 instance using the AWS CLI:

```bash
aws ec2 create-tags --resources <instance-id> --tags Key=Version,Value=1.0.0
```

Alternatively, you can add tags directly from the AWS Management Console by navigating to the Tags section of your EC2 instance.