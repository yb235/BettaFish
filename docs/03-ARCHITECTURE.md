# Architecture Deep Dive

## Table of Contents
- [Introduction](#introduction)
- [System Design Principles](#system-design-principles)
- [Component Architecture](#component-architecture)
- [Agent Architecture](#agent-architecture)
- [Data Flow](#data-flow)
- [Communication Patterns](#communication-patterns)
- [State Management](#state-management)
- [Scalability & Performance](#scalability--performance)

## Introduction

BettaFish is built on a **multi-agent collaborative architecture** where specialized AI agents work together to provide comprehensive public opinion analysis. This document provides a technical deep dive into how the system is architected.

## System Design Principles

### 1. Multi-Agent Collaboration
Instead of relying on a single monolithic AI model, BettaFish employs multiple specialized agents that:
- Have domain-specific expertise
- Use specialized tools and data sources
- Debate and refine ideas through forum discussions
- Produce collective intelligence superior to single-model approaches

### 2. Modularity & Extensibility
The system is designed with:
- **Loose Coupling**: Components interact through well-defined interfaces
- **Plugin Architecture**: Easy to add new agents, tools, or data sources
- **Configuration-Driven**: Behavior controlled through environment variables and config files

### 3. Separation of Concerns
Clear separation between:
- **Data Collection** (MindSpider): Crawling and storage
- **Analysis** (Agents): Intelligence and insights
- **Presentation** (ReportEngine): Report generation
- **Orchestration** (app.py): Coordination and workflow

### 4. Asynchronous Processing
- Agents run in parallel for faster results
- Non-blocking I/O for web requests
- Background tasks for long-running operations

## Component Architecture

### High-Level Component Diagram

```
┌─────────────────────────────────────────────────────────────────┐
│                        Presentation Layer                        │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐   │
│  │ Flask Web UI │  │  Streamlit   │  │  REST API          │   │
│  │ (app.py)     │  │  UIs         │  │  (Flask endpoints) │   │
│  └──────────────┘  └──────────────┘  └────────────────────┘   │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────┴────────────────────────────────────┐
│                      Application Layer                           │
│  ┌──────────────────────────────────────────────────────────┐  │
│  │           Multi-Agent Orchestration System                │  │
│  │                                                            │  │
│  │  ┌─────────────┐  ┌─────────────┐  ┌──────────────┐    │  │
│  │  │ QueryEngine │  │MediaEngine  │  │InsightEngine │    │  │
│  │  │   (Agent)   │  │  (Agent)    │  │   (Agent)    │    │  │
│  │  └─────────────┘  └─────────────┘  └──────────────┘    │  │
│  │                                                            │  │
│  │  ┌──────────────────────────────────────────────┐        │  │
│  │  │          ForumEngine (Moderator)              │        │  │
│  │  │  - Log Monitor                                │        │  │
│  │  │  - LLM Host (Debate Moderator)               │        │  │
│  │  └──────────────────────────────────────────────┘        │  │
│  │                                                            │  │
│  │  ┌──────────────────────────────────────────────┐        │  │
│  │  │          ReportEngine (Agent)                 │        │  │
│  │  │  - Template Selection                         │        │  │
│  │  │  - Multi-round Generation                     │        │  │
│  │  └──────────────────────────────────────────────┘        │  │
│  └──────────────────────────────────────────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────┴────────────────────────────────────┐
│                        Service Layer                             │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────────────┐  │
│  │  LLM Clients │  │   Search    │  │  Sentiment Analysis  │  │
│  │  - OpenAI    │  │   Services  │  │  - Multilingual      │  │
│  │  Compatible  │  │  - Tavily   │  │  - BERT/GPT-2        │  │
│  │              │  │  - Bocha    │  │  - ML Models         │  │
│  └──────────────┘  └─────────────┘  └──────────────────────┘  │
└────────────────────────────┬────────────────────────────────────┘
                             │
┌────────────────────────────┴────────────────────────────────────┐
│                         Data Layer                               │
│  ┌──────────────┐  ┌─────────────┐  ┌──────────────────────┐  │
│  │  PostgreSQL  │  │    MySQL    │  │   MindSpider         │  │
│  │  /MySQL      │  │             │  │   Crawler            │  │
│  │  Database    │  │             │  │   - Topic Extraction │  │
│  │              │  │             │  │   - Deep Crawling    │  │
│  └──────────────┘  └─────────────┘  └──────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Component Responsibilities

#### 1. Flask Application (app.py)
**Purpose**: Main application orchestrator

**Responsibilities**:
- Start/stop agents (Streamlit subprocesses)
- Manage ForumEngine monitoring
- Serve web interface
- Handle API requests
- Real-time communication via SocketIO
- Configuration management

**Key Code Structure**:
```python
# Main orchestration
def initialize_system_components():
    # Start Streamlit apps for each agent
    # Start ForumEngine monitor
    # Initialize ReportEngine
    
# Flask routes
@app.route('/api/search', methods=['POST'])
def search():
    # Dispatch query to agents
    # Collect results
    # Return aggregated response
```

#### 2. Agent Base Architecture
**Purpose**: Common structure for all agents

**Components**:
- **agent.py**: Main agent logic
- **llms/**: LLM client wrapper (OpenAI-compatible)
- **nodes/**: Processing nodes (modular pipeline stages)
- **tools/**: Domain-specific tools
- **state/**: State management
- **prompts/**: Prompt templates
- **utils/**: Helper functions and configuration

**Node-Based Pipeline**:
```
Query → ReportStructureNode → FirstSearchNode → FirstSummaryNode
                                      ↓
                               ReflectionNode → ReflectionSummaryNode
                                      ↓
                              ReportFormattingNode → Final Output
```

#### 3. ForumEngine
**Purpose**: Facilitate agent collaboration

**Components**:

**monitor.py** - Log Monitoring:
```python
class LogMonitor:
    def __init__(self, log_dir="logs"):
        # Monitor insight.log, media.log, query.log
        self.monitored_logs = {
            'insight': 'logs/insight.log',
            'media': 'logs/media.log', 
            'query': 'logs/query.log'
        }
    
    def monitor_logs(self):
        # Watch for SummaryNode outputs
        # Parse agent findings
        # Trigger host speech when threshold reached
```

**llm_host.py** - Debate Moderator:
```python
def generate_host_speech(agent_speeches):
    # Analyze agent findings
    # Generate moderator commentary
    # Guide further discussion
    # Write to forum.log
```

**Communication Flow**:
1. Agents write summaries to their log files
2. Monitor detects SummaryNode outputs
3. Aggregates agent speeches
4. LLM Host generates moderator commentary
5. Agents read forum.log via forum_reader tool
6. Agents adjust strategies based on forum discussion

#### 4. ReportEngine
**Purpose**: Generate professional reports

**Architecture**:
```
User Query + Agent Results
         ↓
Template Selection Node
    (Chooses appropriate template)
         ↓
Content Generation Node
    (Multi-round refinement)
         ↓
HTML Generation Node
    (Applies styling, creates final HTML)
         ↓
Final Report (HTML)
```

**Template Library**:
- Social event analysis
- Brand reputation monitoring
- Product launch analysis
- Crisis management
- Market research
- Academic research

## Agent Architecture

### Common Agent Structure

Each agent follows this pattern:

```python
class DeepSearchAgent:
    def __init__(self, config=None):
        # 1. Load configuration
        self.config = config or settings
        
        # 2. Initialize LLM client
        self.llm_client = LLMClient(
            api_key=self.config.API_KEY,
            base_url=self.config.BASE_URL,
            model_name=self.config.MODEL_NAME
        )
        
        # 3. Initialize tools
        self.search_agency = SearchTools()
        self.sentiment_analyzer = SentimentAnalyzer()
        
        # 4. Initialize processing nodes
        self._initialize_nodes()
        
        # 5. Initialize state
        self.state = State()
    
    def _initialize_nodes(self):
        # Create processing pipeline
        self.first_search_node = FirstSearchNode(self.llm_client)
        self.reflection_node = ReflectionNode(self.llm_client)
        self.summary_node = SummaryNode(self.llm_client)
        # ... more nodes
    
    def execute_search(self, query: str):
        # Main execution flow
        # 1. Structure planning
        # 2. Initial search
        # 3. Reflection and refinement
        # 4. Final summary
```

### QueryEngine (Query Agent)

**Specialization**: Web search and news aggregation

**Tools**:
```python
class TavilyNewsAgency:
    def news_search(self, query, days=7, max_results=10)
    def web_search(self, query, max_results=10)
    def qna_search(self, query)
    def topic_extraction(self, query)
    def extract_content(self, url)
    def comprehensive_search(self, query)
```

**Workflow**:
1. **Query Analysis**: Understand user intent
2. **Tool Selection**: Choose appropriate Tavily tools
3. **Parallel Search**: Execute multiple searches
4. **Result Aggregation**: Combine and deduplicate
5. **Relevance Scoring**: Rank by relevance
6. **Summary Generation**: Create coherent summary

**Example Search**:
```python
# User query: "Public opinion on Tesla"
agent = DeepSearchAgent()

# 1. Structure planning
structure = agent.first_search_node.execute(state)

# 2. Execute searches
news_results = agent.search_agency.news_search("Tesla", days=7)
web_results = agent.search_agency.web_search("Tesla opinion")

# 3. Reflection
refined_queries = agent.reflection_node.execute(state)

# 4. Final summary
summary = agent.summary_node.execute(state)
```

### MediaEngine (Media Agent)

**Specialization**: Multimodal content analysis

**Tools**:
```python
class MediaSearchAgency:
    def image_search(self, query)
    def video_search(self, query)
    def extract_video_transcript(self, video_url)
    def analyze_image(self, image_url)
    def extract_structured_cards(self, html)
```

**Capabilities**:
- Video transcription and analysis (TikTok, Kuaishou)
- Image understanding and OCR
- Structured data extraction (weather, stocks, calendar)
- Visual sentiment analysis

**Workflow for Video Analysis**:
```python
# 1. Search for videos
videos = agent.search_agency.video_search("iPhone 15 review")

# 2. Extract transcripts
for video in videos:
    transcript = agent.search_agency.extract_video_transcript(video.url)
    
    # 3. Analyze content
    analysis = agent.llm_client.analyze_multimodal(
        text=transcript,
        image=video.thumbnail
    )
    
    # 4. Extract sentiment
    sentiment = agent.sentiment_analyzer.analyze(transcript)
```

### InsightEngine (Insight Agent)

**Specialization**: Private database mining

**Tools**:
```python
class MediaCrawlerDB:
    def search_hot_content(self, date_range, platform, limit)
    def search_topic_globally(self, keyword, tables, limit)
    def search_topic_by_date(self, keyword, date, tables, limit)
    def get_comments_for_topic(self, post_id, limit)
    def search_topic_on_platform(self, keyword, platform, date, limit)
```

**Unique Features**:
- **Keyword Optimizer**: Uses Qwen to optimize SQL search keywords
- **Sentiment Analyzer**: Multilingual (22 languages) sentiment analysis
- **Statistical Analysis**: Aggregate statistics and trends

**Workflow for Database Analysis**:
```python
# 1. Optimize keyword
optimized_keyword = keyword_optimizer.optimize("苹果手机")
# Result: ["苹果手机", "iPhone", "Apple手机", "苹果"]

# 2. Search across tables
results = db.search_topic_globally(
    keyword=optimized_keyword,
    tables=['weibo_posts', 'xiaohongshu_posts', 'douyin_videos'],
    limit=200
)

# 3. Get comments for top posts
for post in results[:10]:
    comments = db.get_comments_for_topic(post.id, limit=500)
    
    # 4. Analyze sentiment
    sentiments = sentiment_analyzer.batch_analyze(comments)
    
    # 5. Aggregate statistics
    stats = {
        'positive': len([s for s in sentiments if s > 0.5]),
        'negative': len([s for s in sentiments if s < -0.5]),
        'neutral': len([s for s in sentiments if -0.5 <= s <= 0.5])
    }
```

## Data Flow

### End-to-End Query Flow

```
┌─────────────┐
│ User Query  │
│ "Analyze    │
│  Tesla PR"  │
└──────┬──────┘
       │
       ↓
┌──────────────────────────────────────────┐
│   Flask App (app.py)                     │
│   - Receives query via POST /api/search  │
│   - Stores query in state                │
└──────┬───────────────────────────────────┘
       │
       ↓ [Parallel Launch]
       │
       ├─────────────┬─────────────┬─────────────┐
       ↓             ↓             ↓             ↓
┌──────────┐  ┌──────────┐  ┌──────────┐  ┌──────────┐
│  Query   │  │  Media   │  │ Insight  │  │  Forum   │
│  Agent   │  │  Agent   │  │  Agent   │  │  Engine  │
└────┬─────┘  └────┬─────┘  └────┬─────┘  └────┬─────┘
     │             │             │             │
     │ Web Search  │ Multimodal  │ Database    │ Monitor
     │ (Tavily)    │ Analysis    │ Query       │ Agents
     │             │             │             │
     ↓             ↓             ↓             │
  Results       Results       Results          │
     │             │             │             │
     └─────────────┴─────────────┴──────┬──────┘
                                       │
                   Write summaries to logs
                                       │
                                       ↓
                              ┌──────────────┐
                              │ ForumEngine  │
                              │ - Parse logs │
                              │ - Generate   │
                              │   moderator  │
                              │   speech     │
                              └───────┬──────┘
                                       │
                   Agents read forum.log
                                       │
                   ↓───────────────────┼───────────────────↓
               Refine             Refine              Refine
               strategy          strategy             strategy
                   │                 │                   │
                   ↓                 ↓                   ↓
            [Reflection Loop - Multiple Rounds]
                   │                 │                   │
                   └─────────────────┴───────────────────┘
                                       │
                              Final summaries
                                       │
                                       ↓
                              ┌──────────────┐
                              │Report Agent  │
                              │- Collect all │
                              │  results     │
                              │- Select      │
                              │  template    │
                              │- Generate    │
                              │  HTML report │
                              └───────┬──────┘
                                       │
                                       ↓
                              ┌──────────────┐
                              │ Final Report │
                              │  (HTML)      │
                              └──────────────┘
```

### State Management Flow

Each agent maintains state through execution:

```python
class State:
    def __init__(self):
        self.query = ""                    # Original user query
        self.report_structure = []         # Planned sections
        self.search_results = {}           # Raw search results
        self.summaries = []                # Generated summaries
        self.reflections = []              # Reflection outputs
        self.final_report = ""             # Final formatted report
        self.metadata = {}                 # Additional metadata
        
    def add_search_results(self, tool_name, results):
        self.search_results[tool_name] = results
        
    def add_summary(self, summary):
        self.summaries.append(summary)
```

### Log-Based Communication

Agents communicate through structured log files:

**Agent Log Format** (e.g., `logs/insight.log`):
```
[HH:MM:SS] InsightEngine.nodes.summary_node - INFO - Summary generated:
{
    "section": "Public Sentiment Analysis",
    "findings": [
        "Overall positive sentiment: 65%",
        "Key concern: Battery life",
        "Trending topic: Camera quality"
    ],
    "confidence": 0.87,
    "sources": 1250
}
```

**Forum Log Format** (`logs/forum.log`):
```
[HH:MM:SS] [INSIGHT] Overall positive sentiment: 65%, key concern: battery life
[HH:MM:SS] [MEDIA] Video reviews show mixed reactions to camera
[HH:MM:SS] [QUERY] News articles highlight pricing controversy
[HH:MM:SS] [HOST] Interesting divergence: database shows positive sentiment, but media analysis reveals concerns. Query Agent, can you investigate the pricing controversy more deeply?
```

## Communication Patterns

### 1. Request-Response Pattern
Used for: API calls, database queries

```python
# Agent → Tool
results = agent.search_agency.news_search(query="Tesla")

# Agent → LLM
response = agent.llm_client.chat_completion(messages=[...])
```

### 2. Pub-Sub Pattern
Used for: Forum communication

```python
# Agent publishes to log
logger.info(f"Summary: {summary_json}")

# ForumEngine subscribes
monitor.detect_summary_in_logs()

# ForumEngine publishes to forum.log
forum_logger.info(f"[HOST] {moderator_speech}")

# Agents subscribe via forum_reader
forum_content = read_forum_log()
```

### 3. Pipeline Pattern
Used for: Node-based processing

```python
# Sequential processing
state = State(query="...")
state = first_search_node.execute(state)
state = reflection_node.execute(state)
state = summary_node.execute(state)
state = formatting_node.execute(state)
```

## Scalability & Performance

### Current Architecture

**Concurrency**: 
- 3 agents run in parallel (Query, Media, Insight)
- Each agent is a separate Streamlit process
- ForumEngine runs as a background thread

**Bottlenecks**:
1. **LLM API Rate Limits**: Each agent makes multiple API calls
2. **Database Queries**: Can be slow for large datasets
3. **Sequential Node Processing**: Within each agent, nodes run sequentially

### Optimization Strategies

#### 1. Caching
```python
# Cache LLM responses
@lru_cache(maxsize=1000)
def cached_llm_call(prompt, model):
    return llm_client.chat_completion(prompt, model)

# Cache database queries
@ttl_cache(maxsize=500, ttl=3600)
def cached_db_query(keyword):
    return db.search_topic_globally(keyword)
```

#### 2. Batch Processing
```python
# Batch sentiment analysis
sentiments = sentiment_analyzer.batch_analyze(
    texts=comments,
    batch_size=32
)
```

#### 3. Asynchronous Execution
```python
# Async search
async def parallel_search(queries):
    tasks = [
        search_agency.news_search(q)
        for q in queries
    ]
    return await asyncio.gather(*tasks)
```

#### 4. Database Optimization
```sql
-- Add indexes for common queries
CREATE INDEX idx_posts_keyword ON posts(keyword);
CREATE INDEX idx_posts_date ON posts(created_at);
CREATE INDEX idx_posts_platform ON posts(platform);

-- Composite index for common queries
CREATE INDEX idx_posts_composite ON posts(keyword, created_at, platform);
```

### Horizontal Scaling

For high-load scenarios:

```
┌─────────────────────────────────────────┐
│        Load Balancer (nginx)             │
└─────────────┬───────────────────────────┘
              │
       ┌──────┴──────┐
       │             │
       ↓             ↓
┌─────────────┐ ┌─────────────┐
│  Flask      │ │  Flask      │
│  Instance 1 │ │  Instance 2 │
└──────┬──────┘ └──────┬──────┘
       │               │
       └───────┬───────┘
               │
               ↓
     ┌──────────────────┐
     │   Task Queue     │
     │   (Celery/RQ)    │
     └────────┬─────────┘
              │
       ┌──────┴──────┐
       │             │
       ↓             ↓
┌───────────┐  ┌───────────┐
│  Worker 1 │  │  Worker 2 │
│  Agents   │  │  Agents   │
└───────────┘  └───────────┘
       │             │
       └──────┬──────┘
              ↓
     ┌──────────────┐
     │   Database   │
     │   (Primary)  │
     └───────┬──────┘
             │
      ┌──────┴──────┐
      │             │
      ↓             ↓
┌───────────┐ ┌───────────┐
│  Replica  │ │  Replica  │
│  (Read)   │ │  (Read)   │
└───────────┘ └───────────┘
```

## Security Considerations

### 1. API Key Management
- Store in `.env` file (never commit)
- Use environment variables
- Rotate keys regularly
- Monitor usage for anomalies

### 2. Database Security
- Use parameterized queries (prevent SQL injection)
- Limit database user permissions
- Enable SSL/TLS for connections
- Regular backups

### 3. Input Validation
```python
def validate_query(query: str) -> bool:
    # Length check
    if len(query) > 1000:
        return False
    
    # SQL injection patterns
    sql_patterns = ['DROP', 'DELETE', '--', ';']
    if any(pattern in query.upper() for pattern in sql_patterns):
        return False
    
    return True
```

### 4. Rate Limiting
```python
from flask_limiter import Limiter

limiter = Limiter(
    app,
    key_func=lambda: request.remote_addr,
    default_limits=["100 per hour"]
)

@app.route('/api/search', methods=['POST'])
@limiter.limit("10 per minute")
def search():
    # Handle search
```

## Next Steps

- **[Workflow Guide](06-WORKFLOW.md)**: Understand step-by-step execution
- **[API Reference](04-API-REFERENCE.md)**: Explore available APIs
- **[Development Guide](07-DEVELOPMENT-GUIDE.md)**: Customize and extend

## Further Reading

- **Node-Based Architecture**: Learn about processing nodes in each agent
- **Prompt Engineering**: See how prompts are structured in `prompts/prompts.py`
- **Tool Development**: Create custom tools for agents
- **Forum Mechanism**: Deep dive into agent collaboration
