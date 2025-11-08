# BettaFish System Overview

## Table of Contents
- [Introduction](#introduction)
- [What is BettaFish?](#what-is-bettafish)
- [Key Features](#key-features)
- [System Architecture](#system-architecture)
- [Core Components](#core-components)
- [Technology Stack](#technology-stack)
- [Use Cases](#use-cases)

## Introduction

Welcome to BettaFish (微舆), an innovative multi-agent public opinion analysis system built from scratch. This documentation is designed for first-time users to understand the system's architecture, components, and workflows.

**"BettaFish"** (Betta is a small yet combative and beautiful fish) symbolizes the project's philosophy: **"small but powerful, fearless of challenges"**.

## What is BettaFish?

BettaFish is an AI-powered public opinion analysis platform that:

1. **Breaks Information Cocoons**: Helps users see beyond their filtered information bubbles
2. **Restores Original Public Sentiment**: Provides authentic, unfiltered public opinion data
3. **Predicts Future Trends**: Uses AI to forecast sentiment and opinion trajectories
4. **Assists Decision-Making**: Provides actionable insights for businesses and researchers

### How It Works (User Perspective)

From a user's perspective, using BettaFish is incredibly simple:

1. **Ask a Question**: Like chatting with an assistant, you simply ask about what you want to analyze
   - Example: "What's the public opinion about Wuhan University?"
   - Example: "Analyze the sentiment around the new iPhone release"

2. **Automatic Analysis**: The system automatically:
   - Searches 30+ social media platforms (Weibo, Xiaohongshu, TikTok, Kuaishou, etc.)
   - Analyzes millions of comments and posts
   - Extracts sentiment, trends, and key topics
   - Generates comprehensive reports

3. **Get Insights**: Receive a detailed HTML report with:
   - Sentiment analysis
   - Key topics and trends
   - Timeline of events
   - Predictive insights
   - Actionable recommendations

## Key Features

### 1. AI-Driven Comprehensive Monitoring
- **24/7 AI Crawler Clusters**: Continuously monitor 10+ major social media platforms
- **Real-time Data Capture**: Capture trending content and drill down to user comments
- **Multi-Platform Coverage**: Weibo, Xiaohongshu (Little Red Book), TikTok, Kuaishou, and more

### 2. Composite Analysis Engine Beyond LLM
- **5 Specialized AI Agents**: Each with unique tools and thinking patterns
- **Fine-tuned Models**: Custom-trained models for sentiment analysis
- **Statistical Models**: Traditional ML methods for robust analysis
- **Multi-Model Collaboration**: Ensures depth, accuracy, and multi-dimensional perspective

### 3. Powerful Multimodal Capabilities
- **Video Analysis**: Deep analysis of TikTok, Kuaishou short videos
- **Image Understanding**: Extract information from images and memes
- **Structured Data Extraction**: Parse weather cards, calendars, stock info from search engines

### 4. Agent "Forum" Collaboration Mechanism
- **Unique Toolsets**: Each agent has specialized tools for their domain
- **Debate Moderator**: An AI moderator facilitates agent discussions
- **Chain-of-Thought Collision**: Agents debate and refine ideas through "forum" discussion
- **Collective Intelligence**: Produces higher-quality insights than single-model approaches

### 5. Seamless Public-Private Data Integration
- **Public Opinion Analysis**: Analyze data from public social media
- **Private Database Support**: Integrate your internal business databases
- **Secure Interfaces**: High-security APIs for data integration
- **Vertical Business Insights**: Combine external trends with internal data

### 6. Lightweight & Highly Extensible Framework
- **Pure Python**: Clean, modular design
- **One-Click Deployment**: Easy setup with Docker or manual installation
- **Easy Customization**: Add custom models and business logic effortlessly
- **Rapid Expansion**: Quickly adapt to new use cases

## System Architecture

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                        User Interface                        │
│                      (Flask Web App)                         │
└────────────────────────┬────────────────────────────────────┘
                         │
                         │ User Query
                         ↓
┌─────────────────────────────────────────────────────────────┐
│                    Main Application (app.py)                 │
│  ┌──────────────────────────────────────────────────────┐  │
│  │              Parallel Agent Orchestration             │  │
│  └──────────────────────────────────────────────────────┘  │
└───┬──────────────────┬──────────────────┬──────────────────┘
    │                  │                  │
    │                  │                  │
    ↓                  ↓                  ↓
┌────────┐      ┌──────────┐      ┌─────────────┐
│ Query  │      │  Media   │      │   Insight   │
│ Agent  │      │  Agent   │      │   Agent     │
└────────┘      └──────────┘      └─────────────┘
    │                  │                  │
    │                  │                  │
    │ Web Search       │ Multimodal       │ Database
    │ (Tavily)         │ Analysis         │ Mining
    │                  │ (Images/Video)   │ (MySQL/PG)
    │                  │                  │
    └─────────┬────────┴──────────┬───────┘
              │                   │
              ↓                   ↓
    ┌──────────────────────────────────┐
    │      ForumEngine (Moderator)     │
    │   Facilitates Agent Discussion   │
    └──────────────┬───────────────────┘
                   │
                   │ Collective Analysis
                   ↓
         ┌──────────────────┐
         │   Report Agent   │
         │ (Report Engine)  │
         └──────────────────┘
                   │
                   │ Final Report
                   ↓
         ┌──────────────────┐
         │   HTML Report    │
         │   (Delivered)    │
         └──────────────────┘
```

### Complete Analysis Workflow

| Step | Phase Name | Main Operations | Components | Loop? |
|------|------------|-----------------|------------|-------|
| 1 | User Query | Flask receives user question | Flask App | No |
| 2 | Parallel Launch | 3 Agents start simultaneously | Query, Media, Insight | No |
| 3 | Preliminary Analysis | Each agent does initial research | Agents + Tools | No |
| 4 | Strategy Formulation | Agents plan detailed research | Agent Decision Modules | No |
| 5-N | **Iterative Phase** | **Forum + Deep Research** | **All Agents + Forum** | **Multi-round** |
| 5.1 | Deep Research | Specialized search guided by moderator | Agents + Reflection + Forum | Each loop |
| 5.2 | Forum Collaboration | ForumEngine monitors and summarizes | ForumEngine + LLM Moderator | Each loop |
| 5.3 | Integration | Agents adjust based on discussion | Agents + forum_reader | Each loop |
| N+1 | Result Integration | Report Agent collects all findings | Report Agent | No |
| N+2 | Report Generation | Multi-round report creation | Report Agent + Templates | No |

## Core Components

### 1. QueryEngine (Query Agent)
**Purpose**: Web search and news aggregation

- **Role**: Search domestic and international news sources
- **Tools**: Tavily API (6 search tools)
- **Capabilities**:
  - News search
  - Web search
  - Topic discovery
  - Trend identification
- **Output**: Structured search results with relevance scores

### 2. MediaEngine (Media Agent)
**Purpose**: Multimodal content analysis

- **Role**: Analyze images, videos, and rich media content
- **Tools**: Image analysis, video transcription, multimodal LLMs
- **Capabilities**:
  - Short video analysis (TikTok, Kuaishou)
  - Image understanding
  - Structured card extraction (weather, stock, calendar)
  - Visual sentiment analysis
- **Output**: Multimodal insights and extracted information

### 3. InsightEngine (Insight Agent)
**Purpose**: Private database mining

- **Role**: Deep analysis of proprietary public opinion databases
- **Tools**: 
  - MediaCrawlerDB (5 database query tools)
  - Sentiment analyzer (supports 22 languages)
  - Keyword optimizer (Qwen-powered)
- **Capabilities**:
  - Database querying
  - Sentiment analysis
  - Keyword extraction
  - Statistical analysis
- **Output**: Data-driven insights from internal databases

### 4. ReportEngine (Report Agent)
**Purpose**: Intelligent report generation

- **Role**: Create comprehensive, professional reports
- **Tools**: Template engine, multi-round generation
- **Capabilities**:
  - Template selection (matches report type to content)
  - Multi-round refinement
  - HTML generation
  - Style customization
- **Output**: Beautiful HTML reports

### 5. ForumEngine
**Purpose**: Agent collaboration and debate

- **Role**: Facilitate inter-agent communication
- **Components**:
  - `monitor.py`: Log monitoring and forum management
  - `llm_host.py`: LLM-powered debate moderator
- **Mechanism**:
  - Monitors agent communications
  - Detects key summaries and findings
  - Generates moderator commentary
  - Facilitates chain-of-thought discussions
- **Output**: Enhanced collective intelligence through debate

### 6. MindSpider
**Purpose**: Data collection and crawling

- **Role**: Gather data from social media platforms
- **Components**:
  - `BroadTopicExtraction`: Extract trending topics
  - `DeepSentimentCrawling`: Deep crawl for sentiment data
  - `MediaCrawler`: Core crawler engine
- **Supported Platforms**:
  - Weibo (微博)
  - Xiaohongshu (小红书)
  - TikTok (抖音)
  - Kuaishou (快手)
  - And more...
- **Output**: Structured data stored in PostgreSQL/MySQL

### 7. SentimentAnalysisModel
**Purpose**: Emotion and sentiment analysis

- **Available Models**:
  1. **WeiboMultilingualSentiment** (Recommended): Supports 22 languages
  2. **WeiboSentiment_SmallQwen**: Small-parameter Qwen3 fine-tuning
  3. **WeiboSentiment_Finetuned**: BERT/GPT-2 fine-tuned models
  4. **WeiboSentiment_MachineLearning**: Traditional ML (SVM, Naive Bayes, XGBoost, LSTM)

- **Capabilities**:
  - Positive/Negative/Neutral classification
  - Confidence scores
  - Multilingual support
  - Batch processing

## Technology Stack

### Backend
- **Language**: Python 3.9+
- **Web Framework**: Flask + SocketIO
- **Database**: PostgreSQL / MySQL
- **ORM**: SQLAlchemy
- **Task Queue**: Threading (for agent parallelization)

### AI/ML
- **LLM Providers**: 
  - Kimi (Moonshot AI) - For Insight Agent
  - Gemini - For Media and Report Agents
  - DeepSeek - For Query Agent
  - Qwen - For Forum Moderator and Keyword Optimizer
- **Fine-tuned Models**:
  - BERT (Chinese)
  - GPT-2 with LoRA
  - Qwen3 (small parameter)
- **Traditional ML**: scikit-learn, XGBoost

### Frontend
- **UI**: HTML/CSS/JavaScript
- **Real-time Communication**: Socket.IO
- **Individual Agent UIs**: Streamlit

### Infrastructure
- **Crawler**: Playwright (browser automation)
- **Search API**: Tavily, Bocha AI
- **Containerization**: Docker + Docker Compose
- **Logging**: Loguru

## Use Cases

### 1. Brand Reputation Monitoring
Monitor what people are saying about your brand across social media platforms.

**Example Query**: "Analyze public opinion about Apple iPhone 15"

**Output**:
- Overall sentiment (positive/negative/neutral percentages)
- Key discussion topics
- Emerging issues or complaints
- Competitor comparisons
- Trend over time

### 2. Social Event Analysis
Understand public reaction to social events, policies, or news.

**Example Query**: "Public opinion on the new education policy"

**Output**:
- Stakeholder sentiment (parents, teachers, students)
- Main concerns and support areas
- Geographic sentiment distribution
- Influential voices and key opinion leaders

### 3. Product Launch Analysis
Analyze reception of new product launches.

**Example Query**: "Analyze sentiment around Tesla Model Y launch in China"

**Output**:
- Initial reactions
- Feature discussions
- Price perception
- Comparison with competitors
- Purchase intent signals

### 4. Crisis Management
Monitor and respond to PR crises in real-time.

**Example Query**: "Analyze the customer service complaint trending on Weibo"

**Output**:
- Spread velocity and reach
- Sentiment intensity
- Key influencers amplifying the issue
- Recommended response strategies

### 5. Market Research
Understand market trends and consumer preferences.

**Example Query**: "What are people saying about electric vehicles in 2024?"

**Output**:
- Popular brands and models
- Common concerns (range, charging, price)
- Feature preferences
- Emerging trends

### 6. Academic Research
Conduct research on social media behavior and public opinion.

**Example Query**: "Study sentiment evolution around COVID-19 vaccines"

**Output**:
- Temporal sentiment changes
- Demographic differences
- Misinformation patterns
- Trust in institutions

## Benefits Over Traditional Methods

### Compared to Manual Analysis
- **Speed**: Minutes vs. weeks of manual analysis
- **Scale**: Millions of posts vs. hundreds
- **Objectivity**: AI-driven vs. human bias
- **Real-time**: Live monitoring vs. delayed reports

### Compared to Simple Dashboards
- **Intelligence**: AI insights vs. raw metrics
- **Context**: Understands nuance and sarcasm
- **Multimodal**: Analyzes text, images, and videos
- **Predictive**: Forecasts trends vs. reports past

### Compared to Single-Model Systems
- **Depth**: Multiple specialized agents vs. one generalist
- **Debate**: Agents challenge each other's findings
- **Robustness**: Cross-validation from multiple perspectives
- **Quality**: Higher-quality collective intelligence

## Next Steps

Now that you understand what BettaFish is and how it works, proceed to:

1. **[Installation Guide](02-INSTALLATION-GUIDE.md)**: Set up BettaFish on your system
2. **[Architecture Deep Dive](03-ARCHITECTURE.md)**: Understand the technical architecture
3. **[API Reference](04-API-REFERENCE.md)**: Learn about available APIs
4. **[Development Guide](07-DEVELOPMENT-GUIDE.md)**: Customize and extend the system

## Getting Help

- **Issues**: [GitHub Issues](https://github.com/666ghj/BettaFish/issues)
- **Discussions**: [GitHub Discussions](https://github.com/666ghj/BettaFish/discussions)
- **Email**: 670939375@qq.com
- **FAQ**: [GitHub FAQ](https://github.com/666ghj/BettaFish/issues/185)
