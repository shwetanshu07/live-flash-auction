# 🎯 Live Flash Auction Platform

> **Note:** This is my personal fork of a group project completed as part of the Cloud Computing capstone at NYU.
> Original repository: [live-flash-auction](https://github.com/Tj-Github30/live-flash-auction).
> My primary contributions: Real Time Bidding Mechanism, Dynamic Timer Countdown, AWS infrastructure Setup and API Configuration

## Overview
This project is a highly concurrent, event-driven Live Flash Auction system designed to manage real-time bidding, synchronized countdown timers, anti-sniping logic and automated notifications. Deployed on an **AWS EKS cluster** the platform is decoupled into specialized microservices to ensure high availability and independent scaling.

## Key Features
- **Atomic Bidding Engine**: Implemented Read-Compare-Write logic using server side Lua scripts in Redis to process concurrent bids atomically with O(1) complexity, propagating updates instantly across all connected clients without database locks on the hot path.
- **Distributed Real Time Communication**: Used Redis Pub/Sub to broadcast WebSocket events (bids, chat messages, timer ticks) across the distributed Kubernetes cluster, enabling realtime updates and a live chat room for auction participants without shared pod memory.
- **Synchronized Timers**: Used a centralized Timer Service which ensures that every client is synced and prevents client side timer manipulation.
- **Data Persistence**: Used an asynchronous persistence pipeline (SQS + Lambda) to write bid histories and final results to PostgreSQL/DynamoDB, keeping the bidding engine responsive and non-blocking.
- **Secure Passwordless Auth**: Used AWS Cognito Custom Auth flows for OTP based registration and login.
- **Automated Sniping Prevention**: Integrated logic within the Bid Processing service that detects and prevents late seconds bidding anomalies.
- **Instant Notifications**: Winners are notified immediately via email (SES) and on screen alerts.

## Demo & Presentation
- [Demo Youtube Video](https://www.youtube.com/watch?v=yf7C636RsY4)
- [Presentation](https://docs.google.com/presentation/d/1boE4evdzk-UhxoZZ3doOs1HZDF_qqqV7Cd1rvZ-Ieks/edit?usp=sharing)

### Some Screenshots

#### Passwordless Auth
<p align="center">
  <img src="https://github.com/user-attachments/assets/af4c63bb-5ecb-4a7a-acf2-080d7bcf3e7b" width="45%" />
  <img src="https://github.com/user-attachments/assets/c853a676-cfce-47f9-8e6a-bab542196acb" width="45%" /> 
</p>

#### Buyer and Seller Dashboards
<p align="center">
  <img src="https://github.com/user-attachments/assets/f2e4fe4a-79ca-4958-92d6-f751eb3dc324" width="45%" />
  <img src="https://github.com/user-attachments/assets/2bd1ed2d-28fe-461b-bca7-598f1eeb1e96" width="45%" /> 
</p>

#### Real Time Bidding Interface

##### Participants View

###### Participant 1
<p align="center">
  <img src="https://github.com/user-attachments/assets/6911ce37-b8e0-422b-a89e-714800ebdd17" width="90%" />
</p>

###### Participant 2
<p align="center">
  <img width="90%" src="https://github.com/user-attachments/assets/c123d088-ce18-49a1-8adb-4125c804b5f0"  />
</p>

*Note: All participants can see the bid history and if they have been outbid or they lead the bid*

##### Host View
<p align="center">
<img width="90%" alt="host" src="https://github.com/user-attachments/assets/87183a02-aa92-4cbc-9841-6d10c3065ecd" />
</p>


## Team Members & Contribution
| Functionality | [Tejaswini](https://github.com/Tj-Github30)  | [Komal](https://github.com/komal-b) | [Frank](https://github.com/frank2002) | [Shwetanshu](https://github.com/shwetanshu07) |
| :--- | :---: | :---: | :---: | :---: |
| **Real-Time Bidding Mechanism** |✔|✔||✔|
| **Dynamic Timer Countdown** |||✔|✔|
| **Auction Infrastructure Spin-up** |✔|✔||✔|
| **Leaderboard** |✔||✔||
| **Notifications** ||✔|✔||
| **Sniping Prevention & Rate Limiting** |✔||✔||
| **API Configuration** || ✔ | ✔ |✔|
| **Frontend (UI)** | ✔ | ✔ ||✔|
| **Backend & Deployment (Devops)** | ✔ | ✔ | ✔ | |

## Architecture

### Backend Services (EKS)
- **Auction Management Service** (Port 8000) - CRUD operations, auction lifecycle management
- **Bid Processing Service** (Port 8002) - Atomic bid validation, anti-snipe logic
- **WebSocket Service** (Port 8001) - Real-time updates, chat, participant tracking
- **Timer Service** (Port 8003) - Countdown management, auction end detection

### AWS Infrastructure
- **EKS Cluster**: `live-auction-eks-cluster` (Kubernetes 1.31)
- **RDS PostgreSQL**: Primary database for auctions and users
- **ElastiCache Redis**: Real-time state management and pub/sub
- **DynamoDB**: Bid history persistence
- **SQS**: Message queues for async processing
- **Lambda**: Bid persistence and notification handlers
- **Cognito**: User authentication and authorization
- **ALB**: Application Load Balancer for service exposure

## Quick Start

### Prerequisites
- AWS CLI configured with appropriate credentials
- kubectl configured for EKS cluster
- Docker Desktop installed
- Node.js 18+ installed

### Key Documentation
1. **[ENTIRE_PHASE_GUIDELINES.md](./ENTIRE_PHASE_GUIDELINES.md)** - Step-by-step AWS setup guide
2. **[backend/README.md](./backend/README.md)** - Backend architecture and setup
3. **[frontend/README.md](./frontend/README.md)** - Frontend setup instructions

## Project Structure

```
live-flash-auction/
├── backend/                    # Backend microservices
│   ├── auction-management-service/
│   ├── bid-processing-service/
│   ├── websocket-service/
│   ├── timer-service/
│   ├── shared/                 # Shared code and utilities
│   ├── lambdas/                # Lambda functions
│   └── k8s/                     # Kubernetes manifests
├── frontend/                    # React + TypeScript frontend
├── infra/                       # Infrastructure as code
├── scripts/                     # Deployment and utility scripts
└── README.md                   # This file
```

## Local Setup

1) Clone
```bash
git clone https://github.com/Tj-Github30/live-flash-auction
cd live-flash-auction
```

2) Backend

Follow backend/README.md

3) Frontend

Follow frontend/README.md

Frontend environment variables

Create frontend/.env:
```bash
VITE_API_BASE_URL=<ALB_URL_OR_LOCAL_API_BASE>
VITE_COGNITO_CLIENT_ID=<your-client-id>
VITE_COGNITO_REGION=us-east-1
```

## Deployment

Start here: ENTIRE_PHASE_GUIDELINES.md

### Redeploy Services

**All Services**:
```bash
./redeploy-all-services.sh
```

**Individual Services**:
- `./redeploy-auction-service.sh`
- `./redeploy-bid-processor-service.sh`
- `./redeploy-websocket-service.sh`

### Verify AWS Resources
```bash
./verify-aws-resources.sh
```


## Monitoring & Logs

### Check Service Logs
```bash
# Timer service
kubectl logs -l app=timer -n default --tail=50

# Auction management
kubectl logs -l app=auction-management -n default --tail=50

# WebSocket service
kubectl logs -l app=websocket -n default --tail=50

# Bid processing
kubectl logs -l app=bid-processing -n default --tail=50
```

### Check Lambda Logs
```bash
aws logs tail /aws/lambda/auction-notifications-lambda --since 30m
aws logs tail /aws/lambda/bid-persistence-lambda --since 30m
```

## Troubleshooting

### Services Not Responding
1. Check pod status: `kubectl get pods -n default`
2. Check pod logs: `kubectl logs <pod-name> -n default`
3. Check service endpoints: `kubectl get endpoints -n default`
4. Verify ALB: `kubectl get ingress -n default`

### Frontend Connection Issues
1. Verify ALB URL: `kubectl get ingress -n default`
2. Check CORS configuration in backend services
3. Verify API endpoints: `curl http://<ALB_URL>/api/auctions`
4. Check browser console for errors

### Notification Issues
1. Check timer service logs for "Successfully enqueued notification message"
2. Check Lambda logs for notification processing
3. Verify SQS queue: `aws sqs get-queue-attributes --queue-url <QUEUE_URL>`
4. Verify SES email configuration



## Environment Variables

### Backend (Kubernetes Secrets)
All backend services use Kubernetes secrets (`auction-secrets`). See `backend/k8s/` for configuration.

### Frontend (.env)
Create `frontend/.env` with:
```bash
VITE_API_BASE_URL=<ALB_URL>
VITE_COGNITO_CLIENT_ID=<your-client-id>
VITE_COGNITO_REGION=us-east-1
```

## License

This project is part of a Cloud Computing course assignment.

---



