# Workflow & Data Flow Guide

## Table of Contents
- [Introduction](#introduction)
- [Complete Analysis Workflow](#complete-analysis-workflow)
- [Agent Workflows](#agent-workflows)
- [Forum Collaboration Flow](#forum-collaboration-flow)
- [Report Generation Flow](#report-generation-flow)
- [Data Processing Pipeline](#data-processing-pipeline)
- [State Management](#state-management)

## Introduction

This document explains how data flows through the BettaFish system, from user query to final report. Understanding these workflows will help you:
- Debug issues more effectively
- Customize the system behavior
- Optimize performance
- Build integrations

## Complete Analysis Workflow

### Step-by-Step Execution

#### Phase 1: System Initialization (One-time)

```
User clicks "Start System"
         ↓
┌────────────────────────────────────┐
│ app.py: initialize_system_components()
│ 1. Stop any existing ForumEngine
│ 2. Start 3 Streamlit apps (Insight, Media, Query)
│ 3. Start ForumEngine monitor thread
│ 4. Initialize ReportEngine
└────────────────────────────────────┘
         ↓
All agents ready and waiting for queries
```

**Timeline**: ~30 seconds
**Logged to**: `logs/insight.log`, `logs/media.log`, `logs/query.log`

#### Phase 2: Query Submission

```
User enters query: "Analyze Tesla brand reputation"
         ↓
┌────────────────────────────────────┐
│ POST /api/search
│ {
│   "query": "Analyze Tesla brand reputation"
│ }
└────────────────────────────────────┘
         ↓
Flask app dispatches to 3 agent APIs
```

**Timeline**: <1 second
**State**: Query stored in session

#### Phase 3: Parallel Agent Execution (5-15 minutes)

```
         Query dispatched
              ↓
     ┌────────┴────────┐
     │                 │
     ↓                 ↓
QueryAgent      MediaAgent      InsightAgent
     │                 │                │
     │ [Each agent executes independently]
     │                 │                │
     ↓                 ↓                ↓
```

**Each agent follows this internal workflow:**

```
1. ReportStructureNode
   - Analyze query
   - Plan report sections
   - Identify research areas
   ↓ [5-10 seconds]
   
2. FirstSearchNode
   - Execute initial searches
   - Use agent-specific tools
   - Gather preliminary data
   ↓ [30-60 seconds]
   
3. FirstSummaryNode
   - Analyze initial results
   - Generate section summaries
   - Log findings to agent log
   ↓ [20-30 seconds]
   
4. ReflectionNode (Loop: 2-3 times)
   - Review current findings
   - Identify gaps
   - Read forum.log for other agents' insights
   - Formulate new search queries
   ↓ [15-20 seconds per reflection]
   
5. Deeper Searches
   - Execute refined queries
   - Fill knowledge gaps
   - Cross-reference with forum
   ↓ [30-45 seconds]
   
6. ReflectionSummaryNode
   - Analyze all gathered data
   - Generate comprehensive summaries
   - Log final findings
   ↓ [20-30 seconds]
   
7. ReportFormattingNode
   - Format results
   - Structure for report
   - Return to main app
   ↓ [10 seconds]
```

**Timeline**: 5-15 minutes per agent (parallel execution)
**Logged to**: Individual agent logs

#### Phase 4: Forum Collaboration (Concurrent with Phase 3)

```
ForumEngine Monitor (Background Thread)
         ↓
┌────────────────────────────────────────────┐
│ Continuously monitors:                      │
│ - logs/insight.log                          │
│ - logs/media.log                            │
│ - logs/query.log                            │
└────────────────┬───────────────────────────┘
                 │
     Detects SummaryNode outputs
                 ↓
┌────────────────────────────────────────────┐
│ Parse and extract key findings:            │
│ [12:34:56] [INSIGHT] Positive sentiment 65% │
│ [12:34:57] [MEDIA] Video concerns about X   │
│ [12:34:58] [QUERY] News highlights Y        │
└────────────────┬───────────────────────────┘
                 │
     Buffer accumulates 5 agent messages
                 ↓
┌────────────────────────────────────────────┐
│ LLM Host generates moderator speech:       │
│ "Interesting divergence between database   │
│  and media analysis. Query Agent, please   │
│  investigate the pricing controversy..."    │
└────────────────┬───────────────────────────┘
                 │
     Written to logs/forum.log
                 ↓
┌────────────────────────────────────────────┐
│ Agents read forum.log via forum_reader     │
│ - Adjust search strategies                 │
│ - Investigate suggested areas              │
│ - Cross-validate findings                  │
└────────────────────────────────────────────┘
```

**Timeline**: Throughout Phase 3
**Logged to**: `logs/forum.log`

#### Phase 5: Result Aggregation

```
All 3 agents complete
         ↓
┌────────────────────────────────────────┐
│ Flask app collects results:            │
│ {                                       │
│   "insight": {                          │
│     "summary": "...",                   │
│     "sentiment": {...},                 │
│     "statistics": {...}                 │
│   },                                    │
│   "media": {                            │
│     "summary": "...",                   │
│     "videos_analyzed": 50,              │
│     "key_topics": [...]                 │
│   },                                    │
│   "query": {                            │
│     "summary": "...",                   │
│     "sources": 15,                      │
│     "articles": [...]                   │
│   }                                     │
│ }                                       │
└────────────────┬───────────────────────┘
                 │
     Return to frontend
```

**Timeline**: <1 second
**State**: Results stored in session

#### Phase 6: Report Generation (2-5 minutes)

```
User clicks "Generate Report" (or automatic)
         ↓
┌────────────────────────────────────────┐
│ POST /api/report/generate               │
│ {                                       │
│   "query": "...",                       │
│   "agent_results": {...},               │
│   "template": "auto"                    │
│ }                                       │
└────────────────┬───────────────────────┘
                 │
         ReportEngine processes
                 ↓
┌────────────────────────────────────────┐
│ 1. Template Selection                  │
│    - Analyze query and results          │
│    - Choose appropriate template        │
│    - Load template structure            │
└────────────────┬───────────────────────┘
                 ↓ [10 seconds]
┌────────────────────────────────────────┐
│ 2. Content Generation (Multi-round)    │
│    Round 1: Generate initial content    │
│    Round 2: Refine and improve          │
│    Round 3: Final polish                │
└────────────────┬───────────────────────┘
                 ↓ [60-120 seconds]
┌────────────────────────────────────────┐
│ 3. HTML Generation                      │
│    - Apply CSS styling                  │
│    - Add charts and visualizations      │
│    - Insert tables and lists            │
│    - Format citations                   │
└────────────────┬───────────────────────┘
                 ↓ [30 seconds]
┌────────────────────────────────────────┐
│ 4. Save and Return                      │
│    - Save to final_reports/             │
│    - Return report URL                  │
│    - Log completion                     │
└────────────────────────────────────────┘
```

**Timeline**: 2-5 minutes
**Output**: `final_reports/report_YYYYMMDD_HHMMSS.html`

### Total Execution Time

- **Minimum**: 7 minutes (simple query, fast APIs)
- **Typical**: 10-15 minutes (standard query)
- **Maximum**: 20-25 minutes (complex query, slow APIs, multiple reflections)

## Agent Workflows

### QueryEngine Detailed Workflow

```
Input: User query "Tesla public opinion"
         ↓
┌─────────────────────────────────────────────────┐
│ ReportStructureNode                             │
│ ┌─────────────────────────────────────────────┐ │
│ │ LLM Prompt:                                 │ │
│ │ "Given the query 'Tesla public opinion',   │ │
│ │  what sections should the report have?"    │ │
│ └─────────────────────────────────────────────┘ │
│ Output:                                         │
│ [                                               │
│   "Recent News Coverage",                       │
│   "Social Media Sentiment",                     │
│   "Key Discussion Topics",                      │
│   "Controversy Analysis"                        │
│ ]                                               │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ FirstSearchNode                                 │
│ For each section:                               │
│   1. Generate search queries                    │
│   2. Execute Tavily tools                       │
│      - news_search("Tesla")                     │
│      - web_search("Tesla public opinion")       │
│      - comprehensive_search("Tesla sentiment")  │
│   3. Collect and deduplicate results            │
└────────────────┬────────────────────────────────┘
                 ↓
Results: 45 articles, 20 web pages
         ↓
┌─────────────────────────────────────────────────┐
│ FirstSummaryNode                                │
│ For each section:                               │
│   LLM analyzes relevant results                 │
│   Generates section summary                     │
│   Logs to logs/query.log                        │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ ReflectionNode (Round 1)                        │
│ ┌─────────────────────────────────────────────┐ │
│ │ LLM Prompt:                                 │ │
│ │ "Review these findings. What's missing?"   │ │
│ │ Current summaries: [...]                    │ │
│ │ Forum insights: [reads forum.log]          │ │
│ └─────────────────────────────────────────────┘ │
│ Output:                                         │
│ [                                               │
│   "Need more recent news (last 48 hours)",     │
│   "Missing user reviews from forums",          │
│   "Should investigate pricing controversy"     │
│ ]                                               │
└────────────────┬────────────────────────────────┘
                 ↓
Execute refined searches...
         ↓
ReflectionNode (Round 2)
         ↓
Final summaries generated
         ↓
┌─────────────────────────────────────────────────┐
│ ReportFormattingNode                            │
│ - Structure all summaries                       │
│ - Add metadata (sources, timestamps)            │
│ - Format for final report                       │
└────────────────┬────────────────────────────────┘
                 ↓
Return results to main app
```

### InsightEngine Detailed Workflow

```
Input: User query "Tesla brand reputation"
         ↓
┌─────────────────────────────────────────────────┐
│ Keyword Optimization                            │
│ ┌─────────────────────────────────────────────┐ │
│ │ Qwen LLM optimizes keyword:                 │ │
│ │ Input: "Tesla"                              │ │
│ │ Output: ["Tesla", "特斯拉", "Model 3",       │ │
│ │          "Model Y", "Elon Musk"]            │ │
│ └─────────────────────────────────────────────┘ │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ FirstSearchNode                                 │
│ Execute database queries:                       │
│                                                 │
│ 1. search_hot_content()                         │
│    - Hot rankings for Tesla                     │
│    - Platform: Weibo, XHS, Douyin              │
│    - Date: Last 7 days                          │
│    Result: 156 hot posts                        │
│                                                 │
│ 2. search_topic_globally()                      │
│    - All tables                                 │
│    - Keywords: optimized list                   │
│    - Limit: 200 per table                       │
│    Result: 843 posts                            │
│                                                 │
│ 3. get_comments_for_topic()                     │
│    - Top 20 posts by engagement                 │
│    - 500 comments each                          │
│    Result: 8,234 comments                       │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ Sentiment Analysis                              │
│ Process 8,234 comments in batches:              │
│                                                 │
│ multilingual_sentiment_analyzer.batch_analyze(  │
│     texts=comments,                             │
│     batch_size=32,                              │
│     language="auto"                             │
│ )                                               │
│                                                 │
│ Results:                                        │
│ - Positive: 5,352 (65%)                         │
│ - Negative: 1,647 (20%)                         │
│ - Neutral: 1,235 (15%)                          │
│ - Avg score: 0.45                               │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ FirstSummaryNode                                │
│ LLM analyzes:                                   │
│ - Sentiment distribution                        │
│ - Key topics from posts                         │
│ - Trending concerns                             │
│ - Platform differences                          │
│                                                 │
│ Logs findings to logs/insight.log               │
└────────────────┬────────────────────────────────┘
                 ↓
Reflection loops (2-3 rounds)
- Read forum.log for other agents' findings
- Identify data gaps
- Execute targeted searches
- Re-analyze with new context
         ↓
Final comprehensive database analysis
```

### MediaEngine Detailed Workflow

```
Input: User query "Tesla brand reputation"
         ↓
┌─────────────────────────────────────────────────┐
│ FirstSearchNode                                 │
│ Execute multimodal searches:                    │
│                                                 │
│ 1. Video search (TikTok/Douyin, Kuaishou)      │
│    Query: "Tesla review"                        │
│    Results: 87 videos found                     │
│                                                 │
│ 2. Image search                                 │
│    Query: "Tesla car"                           │
│    Results: 145 images found                    │
│                                                 │
│ 3. Structured card extraction                   │
│    - Stock prices                               │
│    - Company info cards                         │
│    - Feature comparisons                        │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ Video Processing                                │
│ For top 20 videos by views:                     │
│                                                 │
│ 1. Extract transcript                           │
│    - Speech-to-text AI                          │
│    - Chinese/English support                    │
│                                                 │
│ 2. Analyze visual content                       │
│    - Multimodal LLM (Gemini)                    │
│    - Frame analysis                             │
│    - Detect objects and scenes                  │
│                                                 │
│ 3. Sentiment analysis                           │
│    - Transcript sentiment                       │
│    - Visual sentiment                           │
│    - Combine scores                             │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ Image Processing                                │
│ For top 30 images by engagement:                │
│                                                 │
│ 1. OCR extraction                               │
│    - Extract text from images                   │
│    - Memes, infographics                        │
│                                                 │
│ 2. Visual understanding                         │
│    - Scene detection                            │
│    - Product features visible                   │
│    - Sentiment from visuals                     │
└────────────────┬────────────────────────────────┘
                 ↓
┌─────────────────────────────────────────────────┐
│ FirstSummaryNode                                │
│ Multimodal insights:                            │
│ - Video sentiment trends                        │
│ - Common visual themes                          │
│ - User concerns from videos                     │
│ - Comparison with competitors                   │
│                                                 │
│ Logs to logs/media.log                          │
└────────────────┬────────────────────────────────┘
                 ↓
Reflection and refinement
         ↓
Final multimodal analysis
```

## Forum Collaboration Flow

### How Agents Communicate

```
Time: 12:34:50
┌─────────────────────────────────────┐
│ Insight Agent completes first       │
│ summary, logs to insight.log:       │
│                                     │
│ [12:34:50] FirstSummaryNode - INFO │
│ Summary: Database shows 65%         │
│ positive sentiment, but concerns    │
│ about battery life trending up      │
└─────────────────────────────────────┘
         ↓
Time: 12:34:52
┌─────────────────────────────────────┐
│ ForumEngine detects summary in      │
│ insight.log, extracts and logs to   │
│ forum.log:                          │
│                                     │
│ [12:34:52] [INSIGHT] Database shows │
│ 65% positive sentiment, battery     │
│ concerns trending                   │
└─────────────────────────────────────┘
         ↓
Time: 12:34:55
┌─────────────────────────────────────┐
│ Query Agent completes first summary │
│ logs to query.log:                  │
│                                     │
│ [12:34:55] FirstSummaryNode - INFO │
│ Summary: News articles highlight    │
│ pricing controversy and production  │
│ delays                              │
└─────────────────────────────────────┘
         ↓
Time: 12:34:57
┌─────────────────────────────────────┐
│ ForumEngine extracts and logs:      │
│                                     │
│ [12:34:57] [QUERY] News articles    │
│ highlight pricing controversy       │
└─────────────────────────────────────┘
         ↓
Time: 12:35:00
┌─────────────────────────────────────┐
│ Media Agent completes first summary │
│                                     │
│ [12:35:00] FirstSummaryNode - INFO │
│ Summary: Videos show mixed reactions│
│ Camera praised, price criticized    │
└─────────────────────────────────────┘
         ↓
Time: 12:35:02
┌─────────────────────────────────────┐
│ ForumEngine has 5+ messages,        │
│ triggers LLM Host moderator:        │
│                                     │
│ [12:35:02] [HOST] Interesting       │
│ pattern emerging: database positive │
│ (65%), but news and videos show     │
│ pricing concerns. Query Agent,      │
│ investigate pricing in detail.      │
│ Media Agent, analyze more price-    │
│ focused videos. Insight Agent,      │
│ check if pricing sentiment changed  │
│ over time.                          │
└─────────────────────────────────────┘
         ↓
Time: 12:35:10
┌─────────────────────────────────────┐
│ Query Agent enters ReflectionNode,  │
│ reads forum.log, sees moderator     │
│ suggestion, adjusts strategy:       │
│                                     │
│ New search: "Tesla pricing          │
│ controversy 2025"                   │
└─────────────────────────────────────┘
         ↓
All agents adjust strategies based on forum discussion
         ↓
Produces higher quality, cross-validated results
```

## Data Processing Pipeline

### Inside Each Node

#### Example: FirstSummaryNode

```
Input: 
- State object with search_results
- Report structure sections
         ↓
┌─────────────────────────────────────────┐
│ 1. Group Results by Section            │
│    For section "Public Sentiment":      │
│    - Filter relevant results            │
│    - Extract key information            │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ 2. Prepare LLM Prompt                   │
│    Template:                            │
│    """                                  │
│    You are analyzing public sentiment.  │
│    Here are the results:                │
│    {formatted_results}                  │
│                                         │
│    Please provide:                      │
│    1. Overall sentiment                 │
│    2. Key trends                        │
│    3. Important findings                │
│    """                                  │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ 3. Call LLM                             │
│    response = llm_client.chat(          │
│        messages=[                       │
│            {"role": "system",           │
│             "content": system_prompt},  │
│            {"role": "user",             │
│             "content": user_prompt}     │
│        ],                               │
│        temperature=0.3                  │
│    )                                    │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ 4. Parse Response                       │
│    summary = response.content           │
│    metadata = {                         │
│        "section": "Public Sentiment",   │
│        "sources": 45,                   │
│        "confidence": 0.87               │
│    }                                    │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ 5. Log to Agent Log                     │
│    logger.info(f"Summary: {summary}")   │
└────────────────┬────────────────────────┘
                 ↓
┌─────────────────────────────────────────┐
│ 6. Update State                         │
│    state.summaries.append({             │
│        "section": "Public Sentiment",   │
│        "content": summary,              │
│        "metadata": metadata             │
│    })                                   │
└────────────────┬────────────────────────┘
                 ↓
Return updated state
```

## State Management

### State Object Structure

```python
class State:
    def __init__(self):
        # Input
        self.query: str = ""
        
        # Planning
        self.report_structure: List[str] = []
        
        # Search results
        self.search_results: Dict[str, List[Dict]] = {}
        # {
        #     "news_search": [...],
        #     "web_search": [...],
        #     "db_query": [...]
        # }
        
        # Analysis
        self.summaries: List[Dict] = []
        # [
        #     {
        #         "section": "Public Sentiment",
        #         "content": "...",
        #         "metadata": {...}
        #     }
        # ]
        
        # Reflection
        self.reflections: List[Dict] = []
        # [
        #     {
        #         "round": 1,
        #         "gaps": ["..."],
        #         "new_queries": ["..."]
        #     }
        # ]
        
        # Final output
        self.final_report: str = ""
        
        # Metadata
        self.metadata: Dict = {
            "start_time": None,
            "end_time": None,
            "total_sources": 0,
            "llm_calls": 0
        }
```

### State Transitions

```
Empty State
    ↓
[ReportStructureNode]
    ↓
State with report_structure
    ↓
[FirstSearchNode]
    ↓
State with search_results
    ↓
[FirstSummaryNode]
    ↓
State with initial summaries
    ↓
[ReflectionNode]
    ↓
State with reflections and new queries
    ↓
[Additional searches...]
    ↓
State with enhanced search_results
    ↓
[ReflectionSummaryNode]
    ↓
State with refined summaries
    ↓
[ReportFormattingNode]
    ↓
State with final_report
    ↓
Return to caller
```

## Logging Flow

### Log Hierarchy

```
Main Application (app.py)
└─ logs/app.log
   - System events
   - Agent lifecycle
   - API requests

Insight Agent (InsightEngine)
└─ logs/insight.log
   - Database queries
   - Sentiment analysis
   - Node executions
   
Media Agent (MediaEngine)
└─ logs/media.log
   - Video processing
   - Image analysis
   - Multimodal results

Query Agent (QueryEngine)
└─ logs/query.log
   - Web searches
   - News searches
   - Result parsing

Forum Engine
└─ logs/forum.log
   - Agent communications
   - Moderator speeches
   - Collaboration events
```

### What Gets Logged

```python
# Node start
logger.info(f"FirstSearchNode starting for query: {query}")

# Tool execution
logger.debug(f"Executing tool: news_search with params: {params}")

# Results
logger.info(f"Found {len(results)} results")

# LLM calls
logger.debug(f"LLM call with {len(messages)} messages")

# Summaries (Important for Forum)
logger.info(f"Summary: {summary_text}")

# Errors
logger.error(f"Search failed: {error}")
logger.exception("Exception occurred")

# Performance
logger.info(f"Node completed in {duration}s")
```

## Next Steps

- **[API Reference](04-API-REFERENCE.md)**: Integrate with workflows
- **[Development Guide](07-DEVELOPMENT-GUIDE.md)**: Customize workflows
- **[Troubleshooting](09-TROUBLESHOOTING.md)**: Debug workflow issues
