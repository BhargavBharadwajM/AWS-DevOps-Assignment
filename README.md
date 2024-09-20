
# AWS-DevOps Assignment: Chess Game Application Architecture

## Architecture Overview

### Frontend (Next.js)
- **Deployment**: Deployed on **Amazon ECS**.
- **Static Assets**: Stored in **Amazon S3** and delivered via **Amazon CloudFront** for optimized performance through a CDN.

### Backend (NestJS & WebSockets)
- **NestJS Backend**: Containerized and deployed on **Amazon ECS** using **AWS Fargate** for scalable compute power.
- **WebSockets**: Managed using **Amazon API Gateway WebSocket API** for efficient real-time communication and easier scalability.

### Database (PostgreSQL)
- **Amazon RDS (PostgreSQL)**: Stores relational data (game history, player profiles, etc.) with **multi-AZ failover** for high availability.

### Redis Caching (ElastiCache)
- **Amazon ElastiCache (Redis)**: Used for fast access to session data, game states, and leaderboards.

---

## Flow of the Application

### Frontend (Next.js) Flow
1. **Client Request**: The user accesses the chess game web application via a browser.
2. **CloudFront**: Static assets (HTML, CSS, JS) are served from **Amazon CloudFront**, ensuring low-latency content delivery across the globe.
3. **ALB Routing**: Requests that require server-side rendering or API calls (handled by NestJS) are routed through **Amazon ALB** to the ECS containers running the backend.

### Backend (NestJS) Flow
1. **WebSockets**: When a user initiates or joins a game, **WebSockets** are established via **Amazon API Gateway WebSocket API**.
2. **Real-time Communication**: Game states and moves are communicated in real-time between players using WebSockets managed by **NestJS** on ECS.
3. **Database Interaction**: Persistent game states are stored in **Amazon RDS (PostgreSQL)**, while **ElastiCache (Redis)** handles real-time session data for faster read/write operations.

---

## Bash Scripts Integration

1. **200 Status Code Monitoring**: A bash script monitors HTTP 200 status codes and provides insights into the health of the application. This data is exposed via an API endpoint in the Next.js app.
2. **AWS Service Listing**: Another bash script lists the AWS services related to the project in the specified region. This script is accessible via a custom endpoint, giving real-time insight into resources in use.

---

## Security Design

### VPC (Virtual Private Cloud)
- All components are deployed within a **VPC** to control access and traffic flow between resources.
- **Private Subnets**: **RDS (PostgreSQL)** and **ElastiCache (Redis)** are placed in private subnets to restrict direct access from the internet.
- **Public Subnets**: The **ALB** is placed in public subnets to route external traffic. ECS containers running the Next.js and NestJS backend are attached to these subnets, but communicate internally via the VPC.

### Application Load Balancer (ALB)
- The **ALB** routes traffic securely to ECS containers.
- **HTTPS** is enforced, ensuring encrypted communication between the clients and backend services.
- **Sticky Sessions**: Enabled for WebSocket connections to allow long-lived, real-time connections without disruption.

### AWS WAF (Web Application Firewall)
- **WAF** is integrated with ALB to protect against common web threats like SQL injection, XSS, and DDoS attacks.
- Custom rules monitor and block malicious traffic before it reaches the backend services, ensuring the application's security.

### Security Groups
- **Tight Control**: Security groups are configured to allow only necessary traffic:
  - ALB allows only HTTPS (port 443) traffic from the internet.
  - ECS instances only accept traffic from the ALB security group.
  - RDS and Redis only accept traffic from ECS instances within the VPC.

### IAM Policies & Secrets Management
- **IAM Roles and Policies**: Enforces least privilege access by assigning appropriate IAM roles to ECS tasks, RDS instances, and Lambda functions.
- **Secrets Manager**: Sensitive data (e.g., database credentials) is stored securely in **AWS Secrets Manager** and dynamically injected into ECS containers without hardcoding.

### Logging and Monitoring
- **Amazon CloudWatch**: Captures logs from ECS containers, ALB, and API Gateway. Alarms trigger notifications when thresholds are breached (e.g., high error rates).
- **AWS CloudTrail**: Logs all API requests for traceability and auditability, ensuring secure administrative actions on AWS resources.

---

## WebSockets and Real-time Gameplay

### Amazon API Gateway WebSocket API
- **Scalable WebSockets**: Manages WebSocket connections automatically, supporting millions of concurrent players.
- **Message Broadcasting**: Relays real-time game data between players (e.g., moves, game states), ensuring time-sensitive interactions.
- **Connection Scaling**: Automatically scales to manage WebSocket connections as demand increases, abstracting away the complexity of managing long-lived connections.

---

## Conclusion

This architecture ensures scalability, cost-efficiency, and high security. By leveraging AWS services like **CloudFront**, **ECS**, **ALB**, **API Gateway**, **RDS**, **ElastiCache**, and **WAF**, the chess game application can handle real-time gameplay with minimal latency, robust security, and seamless user experience.

---

### Additional Features
- Bash scripts for monitoring HTTP status codes and listing AWS services are integrated into the application and can be accessed via custom endpoints.
- **VPC** ensures network isolation and secure traffic control.
- **CloudWatch** and **CloudTrail** provide detailed monitoring and logging for enhanced security and operational insight.

---

This documentation outlines the core AWS architecture and security considerations for the chess game application, ensuring optimal performance, scalability, and security.

