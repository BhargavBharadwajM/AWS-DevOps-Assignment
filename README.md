# AWS-DevOps-Assignment

*Architecture Overview
Frontend (Next.js)
Deployed on Amazon ECS.
Static assets are stored in Amazon S3 and delivered via Amazon CloudFront to improve performance through a CDN.
Backend (NestJS & WebSockets)
NestJS backend is containerized and deployed on Amazon ECS using AWS Fargate for scalable compute power.
WebSockets are managed using Amazon API Gateway WebSocket API for efficient real-time communication and easier scalability.
Database (PostgreSQL)
Amazon RDS (PostgreSQL) is used to store relational data (game history, player profiles, etc.), with multi-AZ failover for high availability.
Redis Caching (ElastiCache)
Amazon ElastiCache (Redis) is used for fast access to session data, game states, and leaderboards.
*Flow of the Application
Frontend (Next.js) Flow
Client Request: The user accesses the chess game web application via a browser.
CloudFront: The static assets (HTML, CSS, JS) are served from Amazon CloudFront. This ensures low-latency content delivery across the globe.
ALB Routing: Requests that require server-side rendering or API calls (handled by NestJS) are routed to Amazon ALB, which forwards the traffic to the ECS containers running the backend.
Backend (NestJS) Flow
WebSockets: When a user initiates or joins a game, WebSockets are established through Amazon API Gateway WebSocket API.
Real-time Communication: Game states and moves are communicated in real-time between players using WebSockets, managed by NestJS on ECS.
Database Interaction: Persistent game states are stored in RDS (PostgreSQL), while ElastiCache (Redis) handles real-time session data for faster read/write operations.
Bash Scripts Integration
200 Status Code Monitoring: A bash script monitors HTTP 200 status codes and provides insights into the health of the application, which can be viewed via an API endpoint exposed by the Next.js app.
AWS Service Listing: Another bash script lists the AWS services related to the project in the specified region, providing real-time insight into resources in use. This script is also accessible via a custom endpoint.
*Security Design
VPC (Virtual Private Cloud)
All components are deployed within a VPC to control access and traffic flow between resources.
Private Subnets: RDS (PostgreSQL), ElastiCache (Redis) are placed in private subnets to restrict direct access from the internet.
Public Subnets: The ALB is placed in public subnets to route external traffic. ECS containers running the Next.js and NestJS backend are attached to these subnets but still communicate internally through the VPC.
Application Load Balancer (ALB)
The ALB routes traffic securely to the ECS containers.
HTTPS is enforced, ensuring encrypted communication between the clients and the backend services.
Sticky sessions are enabled for WebSocket connections, allowing real-time communication through long-lived connections without disruption.
AWS WAF (Web Application Firewall)
WAF is integrated with ALB to provide protection against common web threats like SQL injection, XSS, and DDoS attacks.
Rules are created to monitor and block malicious traffic before it reaches the backend services, ensuring the application remains secure.
Security Groups
Tight Control: Security groups are configured to allow only necessary traffic:
ALB allows only HTTPS (port 443) traffic from the internet.
ECS instances (backend services) only accept traffic from the ALB security group.
RDS and Redis only accept traffic from ECS instances within the VPC.
IAM Policies & Secrets Management
IAM Roles and Policies:
Least privilege access is enforced by assigning the appropriate IAM roles to ECS tasks, RDS instances, and Lambda functions.
ECS tasks have specific permissions to access RDS, ElastiCache, S3, and Secrets Manager.
AWS Secrets Manager: All sensitive data (such as database credentials and API keys) are stored securely in AWS Secrets Manager. Secrets are dynamically injected into ECS containers without hardcoding.
Logging and Monitoring
Amazon CloudWatch: CloudWatch logs capture all requests and events from ECS containers, ALB, and API Gateway. CloudWatch Alarms are configured to trigger notifications in case of threshold breaches (e.g., high error rates or latency).
CloudTrail: AWS CloudTrail logs all API requests, ensuring traceability and auditability for any administrative actions taken on the AWS resources.
*WebSockets and Real-time Game Play
Amazon API Gateway WebSocket API: Scales WebSocket connections automatically, handling millions of concurrent players.
Message Broadcasting: WebSockets are used to relay real-time game data between players (e.g., moves, game states) and handle time-sensitive interactions.
Connection Scaling: Unlike traditional WebSocket setups, the API Gateway abstracts away connection management, scaling automatically as demand increases.
