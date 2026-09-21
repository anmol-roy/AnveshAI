# Frontend-Backend Integration Setup Guide

## Overview
This guide explains how to connect the Next.js frontend to the Python FastAPI backend for the IP-SAKTI Sahayak application.

## Prerequisites
- Python 3.11+ with virtual environment
- Node.js 18+ 
- Both frontend and backend codebases

## Backend Setup

### 1. Start the Backend Server
The backend should be running on `http://127.0.0.1:8000`:

```bash
cd server
.venv/Scripts/python.exe run_server.py
```

The server will start with the following endpoints:
- `GET /` - Health check
- `GET /health` - Health check  
- `POST /query` - Legal Q&A with citations
- `POST /analyze-invention` - Invention analysis
- `POST /patentability-check` - Patentability analysis
- `POST /ask` - Unified routed Q&A
- `POST /analyze-formulation` - Formulation classification and ABS compliance
- `POST /multilingual-query` - Multilingual Q&A

### 2. Verify Backend is Running
```bash
curl http://127.0.0.1:8000/
```

Expected response:
```json
{
  "status": "ok",
  "service": "IP-SAKTI Sahayak",
  "version": "6.0.0"
}
```

## Frontend Setup

### 1. Configure Environment Variables
Create or update `client/.env.local` with:

```env
# Backend API Configuration
NEXT_PUBLIC_API_BASE_URL=http://127.0.0.1:8000

# Clerk Authentication (required)
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY=your_clerk_publishable_key_here
```

**Important:** The `.env.local` file is git-ignored and should never be committed.

### 2. Install Frontend Dependencies
```bash
cd client
npm install
```

### 3. Start the Frontend Development Server
```bash
npm run dev
```

The frontend will typically run on `http://localhost:3000`

## Integration Points

### 1. API Client
The frontend uses a centralized API client at `client/lib/api-client.ts` that:
- Provides TypeScript interfaces matching backend Pydantic models
- Handles HTTP requests to the backend
- Manages error handling and response parsing
- Exports a singleton `apiClient` instance

### 2. Connected Pages

#### Ask Page (`/ask/[chatId]`)
- **Backend endpoint:** `POST /ask`
- **Purpose:** Unified routed Q&A with automatic IP-type detection
- **Features:**
  - General legal questions
  - Formulation analysis integration
  - Multi-turn conversations
  - Evidence citations and confidence levels

#### Analysis Page (`/analysis`)
- **Backend endpoint:** `POST /analyze-formulation`  
- **Purpose:** Formulation classification and ABS compliance analysis
- **Features:**
  - Ingredient extraction
  - Traditional knowledge detection
  - ABS assessment
  - Prior art search

### 3. Error Handling
The integration includes comprehensive error handling:
- Network errors (backend unreachable)
- API errors (invalid requests, server errors)
- User-friendly error messages
- Retry functionality

## Testing the Integration

### 1. Test Backend Health
```bash
curl http://127.0.0.1:8000/health
```

### 2. Test Simple Query
Using the frontend:
1. Navigate to `/ask`
2. Enter a question like "What is Section 3(p) of the Patents Act?"
3. Submit and verify the response includes citations

### 3. Test Formulation Analysis
Using the frontend:
1. Navigate to `/analysis`
2. Fill in formulation details (e.g., "Neem and turmeric skin balm")
3. Add ingredients (e.g., "Neem", "Turmeric")
4. Submit and verify the analysis results

## Troubleshooting

### Backend Connection Issues
- **Error:** "Network error: Unable to connect to the backend server"
- **Solution:** Verify backend is running on correct port (8000)
- **Check:** `NEXT_PUBLIC_API_BASE_URL` in frontend `.env.local`

### CORS Issues
- **Error:** Browser console shows CORS errors
- **Solution:** Backend has CORS middleware configured with `allow_origins=["*"]`
- **Check:** Verify backend middleware configuration in `server/src/api/main.py`

### API Response Format Mismatches
- **Error:** "Failed to parse API response"
- **Solution:** Check backend response matches expected Pydantic models
- **Debug:** Check browser network tab for actual API responses

### Environment Variables Not Loading
- **Error:** API calls fail with undefined base URL
- **Solution:** Restart frontend development server after changing `.env.local`
- **Check:** Variables must start with `NEXT_PUBLIC_` to be available in browser

## Production Considerations

### 1. Backend URL
In production, update `NEXT_PUBLIC_API_BASE_URL` to your production backend URL.

### 2. Authentication
- Implement proper authentication between frontend and backend
- Add API keys or JWT tokens as needed
- Secure the backend endpoints

### 3. Rate Limiting
- Add rate limiting to backend endpoints
- Implement request throttling on frontend

### 4. Error Monitoring
- Add error tracking (e.g., Sentry)
- Monitor API response times and error rates
- Set up alerts for backend failures

## Architecture Notes

The integration follows the architecture described in `ARCHITECTURE_NOTES.md`:
- Frontend acts as UI shell
- Backend handles all AI processing and retrieval
- API layer provides clean separation of concerns
- TypeScript types ensure type safety across the boundary

## Next Steps

1. Configure your Clerk authentication keys
2. Test all major API endpoints
3. Implement additional error handling as needed
4. Add loading states for better UX
5. Set up monitoring and logging
6. Configure production deployment
