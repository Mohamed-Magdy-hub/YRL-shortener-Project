# Product Requirements Document (PRD) — URL Shortener Application

## 1. Overview
A simple, production-ready URL shortener web application that allows users to shorten long URLs, redirect using short codes, and optionally view basic statistics.  
The purpose is to provide a fully working application that will be deployed using a complete DevOps pipeline (Docker, Jenkins, Kubernetes, Terraform).

## 2. Business Requirements

### 2.1 Problem
Users want to shorten long URLs into compact, shareable links.

### 2.2 Target Users
- Public users  
- DevOps team testing CI/CD workflows

### 2.3 Value
- Simple and fast URL shortening  
- Easy to integrate with automated deployment workflows  

### 2.4 Non-Goals
To keep the project simple & deployment-friendly:  
- No user accounts  
- No advanced analytics  
- No microservices  
- No billing or subscriptions  

## 3. Functional Requirements

### 3.1 Core Features

#### 1) Shorten URL
- **Endpoint:** `POST /api/shorten`  
- **Body:** `{ "url": "https://..." }`  
- **Response:** `{ "shortCode": "xyz12a" }`

#### 2) Redirect
- **Endpoint:** `GET /{shortCode}`  
- Redirects to the original URL  
- Increments click counter

#### 3) Get All URLs (optional admin mode)
- **Endpoint:** `GET /api/urls`

#### 4) Delete URL (optional)

#### 5) Basic Stats (optional)
- Click count  
- CreatedAt timestamp  

## 4. Technical Requirements

### 4.1 Architecture
- Monolithic application  
- **Backend:** Node.js (Express)  
- **Frontend:** Simple HTML/Bootstrap or lightweight React  
- **Database:** SQLite (local) → PostgreSQL (production optional)  
- Containerized using Docker  
- Exposed port: **3000**

### 4.2 Database Schema
**urls table**

| field         | type          |
|---------------|---------------|
| id            | int (pk)      |
| original_url  | text          |
| short_code    | varchar(8)    |
| clicks        | int           |
| created_at    | timestamp     |

## 5. Non-Functional Requirements
- Response time < 200ms  
- Input validation & sanitization  
- Basic logging  
- Error handling  
- Stable REST API  
- Lightweight dependencies  

## 6. Deliverables
- Complete working backend  
- Working frontend form  
- SQLite schema  
- Dockerfile  
- Jenkinsfile  
- Kubernetes manifests  
- README with instructions
