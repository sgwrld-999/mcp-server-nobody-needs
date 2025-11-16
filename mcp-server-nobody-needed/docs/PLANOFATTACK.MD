## 1. Meeting Suffering Index API

**Title:** mcp-server-meeting-pain-calculator

**One-line description:** Calculates the "suffering quotient" of meetings based on duration, attendee count, and time of day—a metric nobody needs but everyone secretly wants.

**Why this makes a good interview project:**
• Demonstrates calendar API integration and OAuth2 flow implementation
• Shows complex calculation engine with configurable weight factors
• Highlights data aggregation and analytical reporting capabilities

**MVP feature list:**
• Meeting analysis with pain score calculation
• Integration with calendar systems (mock OAuth2)
• Team suffering leaderboards and rankings
• Historical suffering trends and predictions
• Configurable pain factors (Monday multiplier, lunch penalty)
• Automated suffering reports via email

**Optional stretch features:**
• Real-time suffering meter during ongoing meetings
• Meeting optimization suggestions to reduce suffering
• Suffering badges and achievements system
• Cross-organization suffering benchmarks

**High-level architecture:**
• FastAPI service with OAuth2 authentication flow
• FASTMCP integration for AI-powered meeting analysis
• Scheduled background jobs for report generation
• Redis-like in-memory store for session management
• MCP cloud deployment with auto-scaling based on meeting volume

**Pydantic schemas:**
• Meeting: id: UUID, duration_minutes: int, attendees: list[str], time: datetime
• SufferingScore: meeting_id: UUID, score: float, factors: dict[str, float]
• TeamStats: team_id: str, total_suffering: float, worst_meeting: UUID
• PainConfig: monday_multiplier: float, lunch_penalty: float, size_factor: float
• SufferingReport: period: str, total_score: float, meetings: list[Meeting]
• OptimizationSuggestion: meeting_id: UUID, recommendations: list[str]

**Core REST endpoints:**
• POST /auth/callback - OAuth2 callback - {code, state} → {access_token, user_id}
• POST /meeting/analyze - Analyze meeting - {meeting_data} → {suffering_score, breakdown}
• GET /team/{id}/leaderboard - Team rankings - {} → {rankings[], top_sufferers[]}
• GET /trends/predict - Predict suffering - {date_range} → {predictions[], confidence}
• PUT /config/factors - Update pain factors - {factors{}} → {applied_config}
• POST /report/schedule - Schedule reports - {frequency, recipients[]} → {schedule_id}
• GET /benchmark - Industry comparison - {} → {percentile, industry_average}
• POST /optimize - Get optimization tips - {meeting_ids[]} → {suggestions[]}

**Testing checklist:**
• Unit tests for suffering calculation with various factor combinations
• OAuth2 flow integration tests with mock providers
• Scheduled job execution and email delivery tests
• Performance tests for bulk meeting analysis (1000+ meetings)
• Data consistency tests for concurrent score updates
• FASTMCP tool registration and invocation tests

**One-week milestone plan:**
• Day 1: Implement OAuth2 flow, design suffering calculation algorithm
• Day 2: Build meeting analysis endpoints, create scoring engine
• Day 3: Add team features, leaderboards, and ranking systems
• Day 4: Implement reporting system, scheduled jobs, email integration
• Day 5: Add FASTMCP integration, configure MCP tools
• Day 6: Build optimization engine, add prediction features, write tests
• Day 7: Deploy to MCP cloud, add monitoring, create API documentation

**3 interview talking points:**
• Implemented secure OAuth2 flow supporting multiple calendar providers with token refresh
• Designed proprietary suffering algorithm considering 15+ weighted factors with ML-based calibration
• Built distributed job scheduler processing 100K+ meeting analyses daily with fault tolerance

----
## Implementation Details

### **1. OAuth2 & Calendar Integration Setup**

**Google OAuth2 Implementation:**
- Use Google Cloud Console to create OAuth2 credentials (Client ID and Secret)
- Configure redirect URIs for your FastAPI application (typically `http://localhost:8000/auth/callback`)
- Use the `authlib` library for OAuth2 flow - it handles the complexity of token exchange
- Store refresh tokens securely to maintain long-term access to calendar data
- Implement both authorization and callback endpoints in FastAPI

**Calendar Data Access:**
- After OAuth authentication, use Google Calendar API to fetch meeting data
- Store access tokens in session middleware (use `SessionMiddleware` from Starlette)
- Mock calendar integration for development using static JSON data initially
- Consider implementing Microsoft Graph API for Outlook calendar support later

### **2. Background Task Processing Architecture**

**Choose Your Approach:**

**Option A: FastAPI BackgroundTasks (Simpler)**
- Use for lightweight operations like single meeting analysis
- Built into FastAPI, no extra infrastructure needed
- Good for tasks under 30 seconds
- Add tasks with `background_tasks.add_task()` in your endpoints

**Option B: Celery + Redis (Production-ready)**
- Required for batch processing multiple meetings
- Install Redis locally or use Docker container
- Create Celery worker processes separate from FastAPI
- Use `celery.delay()` to queue tasks asynchronously
- Implement result backend for tracking job status

**Implementation Steps:**
1. Start with BackgroundTasks for MVP
2. Migrate to Celery when you need job persistence and scheduling
3. Use APScheduler for scheduled report generation (daily/weekly summaries)

### **3. Caching Strategy with Redis**

**Setup Redis Caching:**
- Use `fastapi-cache` or `aiocache` libraries for decorator-based caching
- Implement TTL (Time To Live) for different data types:
  - Meeting data: 1 hour TTL
  - Suffering scores: 24 hours TTL
  - Team leaderboards: 5 minutes TTL

**Caching Pattern:**
```
1. Check Redis for cached result using key pattern: "meeting:{meeting_id}"
2. If miss, calculate suffering score
3. Store in Redis with appropriate TTL
4. Return result
```

**Use Redis for:**
- Session storage (OAuth tokens)
- Calculated suffering scores
- Team aggregation data
- Rate limiting counters

### **4. FastAPI Application Structure**

**Project Layout:**
```
meeting-suffering-index/
├── app/
│   ├── api/
│   │   ├── auth.py (OAuth endpoints)
│   │   ├── meetings.py (Analysis endpoints)
│   │   ├── reports.py (Reporting endpoints)
│   ├── core/
│   │   ├── config.py (Environment variables)
│   │   ├── oauth_flow.py (OAuth setup)
│   │   ├── suffering_calculator.py (Core logic)
│   ├── models/
│   │   ├── meeting.py (Pydantic schemas)
│   │   ├── score.py (Score models)
│   ├── services/
│   │   ├── calendar.py (Calendar integration)
│   │   ├── cache.py (Redis client)
│   │   ├── background.py (Task management)
│   └── main.py (FastAPI app initialization)
├── worker/
│   └── celery_app.py (If using Celery)
├── .env
├── requirements.txt
└── Dockerfile
```

### **5. FASTMCP Integration**

**Converting FastAPI to MCP:**
1. Install FASTMCP: `pip install fastmcp`
2. Create MCP server from FastAPI app:
   ```python
   from fastmcp import FastMCP
   mcp = FastMCP.from_fastapi(app)
   ```
3. This automatically exposes your REST endpoints as MCP tools
4. LLMs can then discover and use your meeting analysis capabilities

**MCP Tool Mapping:**
- GET endpoints → MCP Resources
- POST/PUT/DELETE → MCP Tools
- Configure RouteMap for custom mapping if needed

### **6. Deployment Strategy**

**Local Development:**
1. Use Docker Compose to run FastAPI, Redis, and Celery together
2. Create docker-compose.yml with three services
3. Use environment variables for configuration

**MCP Cloud Deployment:**
1. Push your code to GitHub repository
2. Create FastMCP Cloud account (free for personal use)
3. Connect GitHub repo to FastMCP Cloud
4. Set entry point as `main.py:mcp`
5. FastMCP Cloud handles SSL, scaling, and authentication

**Alternative: Google Cloud Run:**
1. Create Dockerfile for your FastAPI app
2. Deploy using `gcloud run deploy`
3. Use Cloud Run proxy for authenticated access
4. Supports auto-scaling and pay-per-use pricing

### **7. Testing Strategy**

**Test Components:**
1. **Unit Tests**: Test suffering calculation logic independently
2. **Integration Tests**: Test OAuth flow with mock providers
3. **API Tests**: Use FastAPI TestClient for endpoint testing
4. **Load Tests**: Use Locust or Apache Bench for performance testing
5. **MCP Tests**: Use FastMCP Client to test tool invocations

### **8. Development Workflow**

**Week 1 Breakdown:**
- **Day 1-2**: Set up OAuth2, create basic FastAPI structure, implement mock calendar data
- **Day 3**: Build suffering calculation engine, add caching layer
- **Day 4**: Implement background tasks, add team features
- **Day 5**: Integrate FASTMCP, test MCP tool access
- **Day 6**: Add reporting, scheduling, comprehensive testing
- **Day 7**: Deploy to MCP Cloud or Cloud Run, documentation

### **9. Key Libraries & Tools**

**Essential Dependencies:**
- `fastapi` - Web framework
- `fastmcp` - MCP integration
- `authlib` - OAuth2 implementation
- `redis` or `aiocache` - Caching
- `celery` (optional) - Background tasks
- `pydantic` - Data validation
- `httpx` - Async HTTP client
- `uvicorn` - ASGI server

### **10. Important Considerations**

**Security:**
- Never store OAuth tokens in code
- Use environment variables for secrets
- Implement rate limiting on all endpoints
- Validate all input data with Pydantic

**Performance:**
- Start with in-memory caching, add Redis when needed
- Use async/await for all I/O operations
- Implement pagination for large result sets
- Consider using connection pooling for Redis

**Monitoring:**
- Add logging at key points
- Implement health check endpoints
- Monitor Redis memory usage
- Track API response times

This approach gives you a production-ready MCP server that can analyze meeting suffering while being accessible to AI agents through the MCP protocol. Start simple with BackgroundTasks and in-memory caching, then scale up to Celery and Redis as needed.