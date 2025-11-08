# API Reference

## Table of Contents
- [Introduction](#introduction)
- [Flask Main Application APIs](#flask-main-application-apis)
- [Agent APIs](#agent-apis)
- [Tool APIs](#tool-apis)
- [Database APIs](#database-apis)
- [Utility APIs](#utility-apis)
- [Error Handling](#error-handling)
- [Rate Limiting](#rate-limiting)

## Introduction

BettaFish provides multiple API layers for different use cases:

1. **Flask REST APIs**: HTTP endpoints for the main application
2. **Agent APIs**: Internal APIs exposed by each agent
3. **Tool APIs**: Specialized tools used by agents
4. **Database APIs**: Data access layer

## Flask Main Application APIs

Base URL: `http://localhost:5000`

### System Management

#### Start System
Initialize all agents and services.

```http
POST /api/system/start
Content-Type: application/json
```

**Response**:
```json
{
    "success": true,
    "message": "系统启动成功",
    "logs": [
        "Insight Agent已初始化",
        "Media Agent已初始化",
        "Query Agent已初始化",
        "ForumEngine启动完成"
    ]
}
```

#### Get System Status
Check if the system is running.

```http
GET /api/system/status
```

**Response**:
```json
{
    "success": true,
    "started": true,
    "starting": false
}
```

### Configuration Management

#### Get Configuration
Retrieve current configuration values.

```http
GET /api/config
```

**Response**:
```json
{
    "success": true,
    "config": {
        "HOST": "0.0.0.0",
        "PORT": "5000",
        "DB_HOST": "localhost",
        "DB_PORT": "5432",
        "INSIGHT_ENGINE_API_KEY": "sk-***",
        "INSIGHT_ENGINE_MODEL_NAME": "kimi-k2-0711-preview",
        // ... more config
    }
}
```

#### Update Configuration
Update configuration values (writes to .env file).

```http
POST /api/config
Content-Type: application/json

{
    "DB_HOST": "new_host",
    "INSIGHT_ENGINE_MODEL_NAME": "new-model"
}
```

**Response**:
```json
{
    "success": true,
    "config": {
        // Updated configuration
    }
}
```

### Application Control

#### Get Application Status
Get status of all agents.

```http
GET /api/status
```

**Response**:
```json
{
    "insight": {
        "status": "running",
        "port": 8501,
        "output_lines": 150
    },
    "media": {
        "status": "running",
        "port": 8502,
        "output_lines": 120
    },
    "query": {
        "status": "running",
        "port": 8503,
        "output_lines": 180
    },
    "forum": {
        "status": "running",
        "port": null,
        "output_lines": 45
    }
}
```

#### Start Individual Agent
Start a specific agent.

```http
GET /api/start/<app_name>
```

**Parameters**:
- `app_name`: One of `insight`, `media`, `query`, `forum`

**Response**:
```json
{
    "success": true,
    "message": "insight 应用启动中..."
}
```

#### Stop Individual Agent
Stop a specific agent.

```http
GET /api/stop/<app_name>
```

**Response**:
```json
{
    "success": true,
    "message": "insight 应用已停止"
}
```

#### Get Agent Output
Retrieve agent logs.

```http
GET /api/output/<app_name>
```

**Response**:
```json
{
    "success": true,
    "output": [
        "[12:34:56] Insight Agent initialized",
        "[12:34:57] Database connection established",
        "[12:34:58] Ready to process queries"
    ],
    "total_lines": 3
}
```

### Search & Analysis

#### Submit Search Query
Submit a query for analysis by all agents.

```http
POST /api/search
Content-Type: application/json

{
    "query": "公众对特斯拉的看法"
}
```

**Response**:
```json
{
    "success": true,
    "query": "公众对特斯拉的看法",
    "results": {
        "insight": {
            "success": true,
            "summary": "Database analysis shows...",
            "sentiment": {
                "positive": 0.65,
                "negative": 0.20,
                "neutral": 0.15
            }
        },
        "media": {
            "success": true,
            "summary": "Video analysis reveals...",
            "trending_topics": ["battery", "price", "design"]
        },
        "query": {
            "success": true,
            "summary": "News articles indicate...",
            "sources": 15
        }
    }
}
```

### Forum Management

#### Get Forum Log
Retrieve forum discussion log.

```http
GET /api/forum/log
```

**Response**:
```json
{
    "success": true,
    "log_lines": [
        "[12:34:56] [INSIGHT] Overall positive sentiment",
        "[12:34:57] [MEDIA] Videos show mixed reactions",
        "[12:34:58] [HOST] Interesting divergence..."
    ],
    "parsed_messages": [
        {
            "type": "agent",
            "sender": "INSIGHT Engine",
            "content": "Overall positive sentiment",
            "timestamp": "12:34:56",
            "source": "INSIGHT"
        }
    ],
    "total_lines": 3
}
```

#### Start Forum Monitoring
Manually start forum monitoring.

```http
GET /api/forum/start
```

**Response**:
```json
{
    "success": true,
    "message": "ForumEngine论坛已启动"
}
```

#### Stop Forum Monitoring
Manually stop forum monitoring.

```http
GET /api/forum/stop
```

**Response**:
```json
{
    "success": true,
    "message": "ForumEngine论坛已停止"
}
```

### Report Generation

#### Generate Report
Generate final HTML report from agent results.

```http
POST /api/report/generate
Content-Type: application/json

{
    "query": "特斯拉品牌声誉分析",
    "agent_results": {
        "insight": {...},
        "media": {...},
        "query": {...}
    },
    "template": "brand_reputation"
}
```

**Response**:
```json
{
    "success": true,
    "report_id": "report_20250115_123456",
    "report_path": "final_reports/report_20250115_123456.html",
    "report_url": "/reports/report_20250115_123456.html"
}
```

## Agent APIs

Each agent exposes APIs through their Streamlit interface.

### QueryEngine API

Base URL: `http://localhost:8603`

#### Search
Execute web search.

```http
POST /api/search
Content-Type: application/json

{
    "query": "Tesla electric vehicles",
    "tools": ["news", "web"],
    "max_results": 15
}
```

**Response**:
```json
{
    "success": true,
    "query": "Tesla electric vehicles",
    "results": {
        "news": [
            {
                "title": "Tesla Announces New Model",
                "url": "https://...",
                "score": 0.95,
                "date": "2025-01-15",
                "summary": "..."
            }
        ],
        "web": [...]
    },
    "summary": "Analysis of search results...",
    "metadata": {
        "total_results": 25,
        "processing_time": 3.2
    }
}
```

### MediaEngine API

Base URL: `http://localhost:8602`

#### Analyze Media
Analyze images or videos.

```http
POST /api/analyze
Content-Type: application/json

{
    "query": "iPhone 15 reviews",
    "media_types": ["image", "video"],
    "platforms": ["douyin", "xiaohongshu"]
}
```

**Response**:
```json
{
    "success": true,
    "results": {
        "videos": [
            {
                "url": "https://...",
                "platform": "douyin",
                "transcript": "...",
                "sentiment": 0.7,
                "key_points": ["camera quality", "battery life"]
            }
        ],
        "images": [...]
    },
    "summary": "Multimodal analysis shows..."
}
```

### InsightEngine API

Base URL: `http://localhost:8601`

#### Query Database
Query the private database.

```http
POST /api/query
Content-Type: application/json

{
    "query": "iPhone手机评价",
    "date_range": {
        "start": "2025-01-01",
        "end": "2025-01-15"
    },
    "platforms": ["weibo", "xiaohongshu"],
    "include_comments": true,
    "max_results": 200
}
```

**Response**:
```json
{
    "success": true,
    "results": {
        "posts": [
            {
                "id": "12345",
                "platform": "weibo",
                "content": "...",
                "date": "2025-01-10",
                "likes": 1500,
                "comments_count": 230
            }
        ],
        "comments": [...],
        "sentiment_analysis": {
            "overall": 0.65,
            "positive": 650,
            "negative": 200,
            "neutral": 150
        },
        "trending_topics": ["价格", "摄像", "电池"],
        "statistics": {
            "total_posts": 1000,
            "total_comments": 5000,
            "avg_sentiment": 0.65
        }
    },
    "summary": "Database analysis reveals..."
}
```

## Tool APIs

### Search Tools

#### TavilyNewsAgency

```python
from QueryEngine.tools import TavilyNewsAgency

agency = TavilyNewsAgency(api_key="your_key")

# News search
results = agency.news_search(
    query="Tesla",
    days=7,
    max_results=10
)

# Web search
results = agency.web_search(
    query="Tesla opinion",
    max_results=10
)

# QnA search
answer = agency.qna_search(
    query="What is Tesla's market share?"
)

# Extract content from URL
content = agency.extract_content(
    url="https://example.com/article"
)
```

**Response Format**:
```python
class TavilyResponse:
    success: bool
    results: List[Dict]
    error: Optional[str]
```

### Database Tools

#### MediaCrawlerDB

```python
from InsightEngine.tools import MediaCrawlerDB

db = MediaCrawlerDB()

# Search hot content
hot_posts = db.search_hot_content(
    date_range=("2025-01-01", "2025-01-15"),
    platform="weibo",
    limit=100
)

# Search topic globally
results = db.search_topic_globally(
    keyword="特斯拉",
    tables=["weibo_posts", "xiaohongshu_posts"],
    limit=200
)

# Search topic by date
results = db.search_topic_by_date(
    keyword="iPhone",
    date="2025-01-15",
    tables=["all"],
    limit=100
)

# Get comments for topic
comments = db.get_comments_for_topic(
    post_id="12345",
    limit=500
)

# Search on specific platform
results = db.search_topic_on_platform(
    keyword="电动车",
    platform="weibo",
    date_range=("2025-01-01", "2025-01-15"),
    limit=200
)
```

**Response Format**:
```python
class DBResponse:
    success: bool
    results: List[Dict]
    count: int
    error: Optional[str]
```

### Sentiment Analysis Tools

#### MultilingualSentimentAnalyzer

```python
from InsightEngine.tools import multilingual_sentiment_analyzer

# Analyze single text
result = multilingual_sentiment_analyzer.analyze(
    text="This product is amazing!",
    language="en"
)
# Returns: {"score": 0.92, "label": "positive", "confidence": 0.95}

# Batch analysis
texts = ["Great product!", "Terrible service", "It's okay"]
results = multilingual_sentiment_analyzer.batch_analyze(
    texts=texts,
    batch_size=32
)

# Supported languages (22 total):
# en, zh, es, fr, de, ja, ko, ar, ru, pt, it, nl, pl, tr, vi, th, sv, da, fi, no, cs, hu
```

**Response Format**:
```python
{
    "score": float,      # -1.0 to 1.0
    "label": str,        # "positive", "negative", "neutral"
    "confidence": float  # 0.0 to 1.0
}
```

#### Keyword Optimizer

```python
from InsightEngine.tools import keyword_optimizer

# Optimize keyword for better search
optimized = keyword_optimizer.optimize(
    keyword="苹果手机",
    language="zh"
)
# Returns: ["苹果手机", "iPhone", "Apple手机", "苹果"]

# With context
optimized = keyword_optimizer.optimize_with_context(
    keyword="特斯拉",
    context="分析电动汽车市场",
    max_variants=5
)
```

## Database APIs

### Database Schema Access

```python
from MindSpider.schema import db_manager

# Get connection
conn = db_manager.get_connection()

# Execute query
results = db_manager.execute_query(
    query="SELECT * FROM weibo_posts WHERE keyword = %s LIMIT 10",
    params=("Tesla",)
)

# Insert data
db_manager.insert_post(
    platform="weibo",
    content="...",
    author="...",
    post_id="12345",
    created_at="2025-01-15 12:00:00"
)

# Update data
db_manager.update_post(
    post_id="12345",
    sentiment_score=0.8
)

# Delete data
db_manager.delete_post(post_id="12345")
```

### ORM Models

```python
from MindSpider.schema.models import WeiboPost, XiaohongshuPost, Comment

# Query using ORM
posts = WeiboPost.query.filter(
    WeiboPost.keyword.contains("Tesla"),
    WeiboPost.created_at >= "2025-01-01"
).limit(100).all()

# Create new record
post = WeiboPost(
    post_id="12345",
    content="...",
    author="...",
    keyword="Tesla"
)
db.session.add(post)
db.session.commit()

# Update record
post = WeiboPost.query.get("12345")
post.sentiment_score = 0.8
db.session.commit()

# Delete record
db.session.delete(post)
db.session.commit()
```

## Utility APIs

### Logging

```python
from loguru import logger

# Configure logger
logger.add(
    "logs/app.log",
    rotation="500 MB",
    retention="10 days",
    level="INFO"
)

# Log messages
logger.info("Application started")
logger.debug("Debug information")
logger.warning("Warning message")
logger.error("Error occurred")
logger.exception("Exception with traceback")
```

### Configuration

```python
from config import settings, reload_settings

# Access configuration
api_key = settings.INSIGHT_ENGINE_API_KEY
model = settings.INSIGHT_ENGINE_MODEL_NAME

# Reload configuration
reload_settings()
```

### Retry Helper

```python
from utils.retry_helper import retry_on_failure

@retry_on_failure(max_retries=3, delay=1.0)
def make_api_call():
    # This will retry up to 3 times on failure
    response = requests.get("https://api.example.com")
    return response.json()
```

### Forum Reader

```python
from utils.forum_reader import read_forum_log, get_latest_host_speech

# Read entire forum log
messages = read_forum_log()

# Get latest moderator speech
latest = get_latest_host_speech()
# Returns: {"timestamp": "12:34:56", "content": "..."}

# Get messages from specific agent
insight_messages = read_forum_log(source="INSIGHT")
```

## Error Handling

### Standard Error Response

All APIs return errors in this format:

```json
{
    "success": false,
    "message": "Error description",
    "error_code": "ERROR_CODE",
    "details": {
        // Additional error details
    }
}
```

### Common Error Codes

- `INVALID_REQUEST`: Missing or invalid parameters
- `AUTH_ERROR`: API key invalid or missing
- `DB_ERROR`: Database connection or query failed
- `LLM_ERROR`: LLM API call failed
- `TIMEOUT_ERROR`: Request timed out
- `NOT_FOUND`: Resource not found
- `RATE_LIMIT`: Rate limit exceeded

### Example Error Handling

```python
import requests

try:
    response = requests.post(
        "http://localhost:5000/api/search",
        json={"query": "Tesla"},
        timeout=30
    )
    response.raise_for_status()
    result = response.json()
    
    if not result.get("success"):
        print(f"Error: {result.get('message')}")
    else:
        print(f"Results: {result}")
        
except requests.exceptions.Timeout:
    print("Request timed out")
except requests.exceptions.RequestException as e:
    print(f"Request failed: {e}")
```

## Rate Limiting

### Default Limits

- **Search API**: 10 requests per minute per IP
- **Configuration API**: 5 updates per minute
- **Report Generation**: 3 requests per minute

### Rate Limit Headers

```http
X-RateLimit-Limit: 10
X-RateLimit-Remaining: 5
X-RateLimit-Reset: 1610000000
```

### Rate Limit Error Response

```json
{
    "success": false,
    "message": "Rate limit exceeded",
    "error_code": "RATE_LIMIT",
    "details": {
        "limit": 10,
        "reset_at": "2025-01-15T12:35:00Z"
    }
}
```

## WebSocket API

For real-time updates, BettaFish uses Socket.IO.

### Connect to WebSocket

```javascript
const socket = io('http://localhost:5000');

socket.on('connect', () => {
    console.log('Connected to server');
});
```

### Events

#### console_output
Real-time agent console output.

```javascript
socket.on('console_output', (data) => {
    console.log(`[${data.app}] ${data.line}`);
});
```

#### forum_message
Real-time forum messages.

```javascript
socket.on('forum_message', (data) => {
    console.log(`${data.sender}: ${data.content}`);
});
```

#### status_update
Agent status changes.

```javascript
socket.on('status_update', (data) => {
    console.log('Agent status:', data);
});
```

## Code Examples

### Complete Search Flow

```python
import requests
import time

# 1. Start system
response = requests.post("http://localhost:5000/api/system/start")
print(response.json())

# Wait for initialization
time.sleep(10)

# 2. Submit search query
response = requests.post(
    "http://localhost:5000/api/search",
    json={"query": "特斯拉公众舆情分析"}
)
result = response.json()

# 3. Monitor forum discussion
response = requests.get("http://localhost:5000/api/forum/log")
forum_log = response.json()
print("Forum discussion:", forum_log['parsed_messages'])

# 4. Generate report
response = requests.post(
    "http://localhost:5000/api/report/generate",
    json={
        "query": "特斯拉公众舆情分析",
        "agent_results": result['results']
    }
)
report = response.json()
print(f"Report generated: {report['report_path']}")
```

### Custom Agent Integration

```python
from QueryEngine.agent import DeepSearchAgent
from config import settings

# Create custom agent
agent = DeepSearchAgent(config=settings)

# Execute search
results = agent.execute_search("Tesla public opinion")

# Access results
print("Summary:", results['summary'])
print("Sources:", len(results['sources']))
print("Sentiment:", results['sentiment'])
```

## Next Steps

- **[Database Schema](05-DATABASE-SCHEMA.md)**: Understand data structures
- **[Development Guide](07-DEVELOPMENT-GUIDE.md)**: Build custom integrations
- **[Troubleshooting](09-TROUBLESHOOTING.md)**: Debug common issues
