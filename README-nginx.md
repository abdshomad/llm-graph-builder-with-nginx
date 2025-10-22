# Docker Compose with Nginx Setup

This setup provides a single entry point through nginx that serves both the frontend and backend, hiding the backend from direct user access.

## Architecture

```
User → Nginx (Port 80) → Frontend (Port 8080)
                    → Backend (Port 8000)
```

## Key Features

- **Single Entry Point**: Users only access the application through port 80
- **API Proxying**: All `/api/*` requests are forwarded to the backend
- **Frontend Serving**: All other requests serve the React frontend
- **Security**: Backend is not directly accessible from outside
- **Performance**: Nginx handles compression, caching, and rate limiting

## Environment Setup

### 1. Create Environment Files

You need to create `.env` files for both backend and frontend:

```bash
# Copy the template files to create your environment files
cp backend-env-template backend/.env
cp frontend-env-template frontend/.env
```

### 2. Configure Backend Environment

Edit `backend/.env` and set the required values:

```bash
# Required API Keys
OPENAI_API_KEY="your-openai-api-key"
DIFFBOT_API_KEY="your-diffbot-api-key"

# Neo4j Database (if using external database)
NEO4J_URI="neo4j://your-neo4j-host:7687"
NEO4J_USERNAME="your-username"
NEO4J_PASSWORD="your-password"
```

### 3. Configure Frontend Environment

Edit `frontend/.env` and set the required values:

```bash
# Backend API URL - IMPORTANT: Keep as /api for nginx proxy
VITE_BACKEND_API_URL="/api"

# Your API keys (if needed for frontend features)
VITE_GOOGLE_CLIENT_ID="your-google-client-id"
```

## Usage

### Start the services:
```bash
docker-compose -f docker-compose-with-nginx.yml up -d
```

### Stop the services:
```bash
docker-compose -f docker-compose-with-nginx.yml down
```

### View logs:
```bash
docker-compose -f docker-compose-with-nginx.yml logs -f
```

## Configuration

### Frontend Configuration
The frontend is configured to use `/api` as the backend URL, which nginx will proxy to the backend service.

### Environment Variables
All environment variables from the original docker-compose.yml are preserved. The key change is:
- `VITE_BACKEND_API_URL` is set to `/api` instead of `http://localhost:8000`

### Nginx Configuration
The nginx configuration (`nginx.conf`) includes:
- API routing (`/api/*` → backend)
- WebSocket support (`/ws/*` → backend)
- Frontend serving (all other routes)
- Security headers
- Gzip compression
- Rate limiting
- Health checks

## Access Points

- **Application**: http://localhost (port 80)
- **Health Check**: http://localhost/health
- **Frontend Health**: http://localhost/health (proxied to frontend)

## Benefits

1. **Simplified Deployment**: Single port exposure
2. **Security**: Backend not directly accessible
3. **Performance**: Nginx handles optimization
4. **Scalability**: Easy to add load balancing later
5. **SSL Ready**: Easy to add SSL termination

## Troubleshooting

### Check service status:
```bash
docker-compose -f docker-compose-with-nginx.yml ps
```

### Check nginx logs:
```bash
docker logs nginx-proxy
```

### Test API endpoints:
```bash
curl http://localhost/api/health
```

### Rebuild services:
```bash
docker-compose -f docker-compose-with-nginx.yml up --build -d
```
