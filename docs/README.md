# BettaFish Documentation

Welcome to the comprehensive documentation for BettaFish (微舆), an innovative multi-agent public opinion analysis system. This documentation is designed to help first-time users understand, install, and work with the system.

## 📚 Documentation Structure

### For First-Time Users

Start here if you're new to BettaFish:

1. **[System Overview](01-SYSTEM-OVERVIEW.md)** - Start Here!
   - What is BettaFish?
   - Key features and benefits
   - High-level architecture
   - Use cases and examples
   - Technology stack

2. **[Installation Guide](02-INSTALLATION-GUIDE.md)** - Get Up and Running
   - System requirements
   - Quick start with Docker (recommended)
   - Manual installation steps
   - Configuration guide
   - Verification and troubleshooting

### For Developers

Deep dive into the technical architecture:

3. **[Architecture Deep Dive](03-ARCHITECTURE.md)** - Technical Architecture
   - System design principles
   - Component architecture with diagrams
   - Agent architecture patterns
   - Data flow and communication
   - Scalability and performance
   - Security considerations

4. **[API Reference](04-API-REFERENCE.md)** - API Documentation
   - Flask main application APIs
   - Agent APIs (Query, Media, Insight)
   - Tool APIs (Search, Database, Sentiment)
   - WebSocket APIs
   - Error handling
   - Code examples

5. **[Database Schema](05-DATABASE-SCHEMA.md)** - Data Structures
   - Database design principles
   - Complete table schemas
   - Relationships and indexes
   - Query examples
   - Performance optimization
   - Backup and recovery

6. **[Workflow Guide](06-WORKFLOW.md)** - Understanding Data Flow
   - Complete analysis workflow
   - Agent execution flows
   - Forum collaboration mechanism
   - State management
   - Logging and monitoring

## 🚀 Quick Links

### Getting Started
- [What is BettaFish?](01-SYSTEM-OVERVIEW.md#what-is-bettafish)
- [Quick Start with Docker](02-INSTALLATION-GUIDE.md#quick-start-with-docker)
- [System Requirements](02-INSTALLATION-GUIDE.md#system-requirements)

### Common Tasks
- [Configure API Keys](02-INSTALLATION-GUIDE.md#configuration)
- [Start the System](02-INSTALLATION-GUIDE.md#verification)
- [Submit a Query](04-API-REFERENCE.md#submit-search-query)
- [Generate Reports](04-API-REFERENCE.md#generate-report)

### Troubleshooting
- [Common Issues](02-INSTALLATION-GUIDE.md#troubleshooting)
- [Database Problems](05-DATABASE-SCHEMA.md#backup--recovery)
- [Performance Optimization](03-ARCHITECTURE.md#scalability--performance)

### Development
- [Architecture Overview](03-ARCHITECTURE.md#component-architecture)
- [Extend Agents](03-ARCHITECTURE.md#agent-architecture)
- [Custom Tools](04-API-REFERENCE.md#tool-apis)
- [Database Queries](05-DATABASE-SCHEMA.md#query-examples)

## 📖 Learning Path

### Path 1: User → Analyst (Non-Technical)
Perfect for business users, analysts, and researchers who want to use BettaFish for public opinion analysis.

```
Start → [System Overview] → [Installation - Docker Section] → Use the System!
```

**Time**: 30 minutes to get started

**What you'll learn**:
- What BettaFish can do for you
- How to install and configure
- How to run analysis queries
- How to interpret results

### Path 2: Developer → Integrator (Technical)
For developers who want to integrate BettaFish into their applications.

```
Start → [System Overview] → [Installation] → [API Reference] → Build Integration
```

**Time**: 2-3 hours

**What you'll learn**:
- Complete system architecture
- REST API endpoints
- How to make API calls
- Error handling
- Building custom integrations

### Path 3: Developer → Contributor (Advanced)
For developers who want to extend or modify BettaFish.

```
Start → [All Docs] → Understand System → Make Modifications
```

**Time**: 1-2 days

**What you'll learn**:
- Deep system architecture
- Agent internals
- Database schema
- Data flow and workflows
- How to add new agents
- How to create custom tools
- Performance optimization

## 🎯 Use Case Examples

### Example 1: Brand Monitoring

**Goal**: Monitor what people say about your brand on social media

**Steps**:
1. Install BettaFish using [Docker Guide](02-INSTALLATION-GUIDE.md#quick-start-with-docker)
2. Configure API keys
3. Submit query: "Analyze public opinion about [Your Brand]"
4. Review comprehensive report with:
   - Sentiment analysis
   - Key discussion topics
   - Trending concerns
   - Competitor comparisons

**Documents**: [System Overview](01-SYSTEM-OVERVIEW.md#use-cases) → [Installation](02-INSTALLATION-GUIDE.md) → [API Reference](04-API-REFERENCE.md#submit-search-query)

### Example 2: Product Launch Analysis

**Goal**: Understand market reception of a new product

**Steps**:
1. Run BettaFish crawler to collect recent data
2. Query: "Public sentiment about [Product Name] launch"
3. Agents analyze:
   - News coverage (Query Agent)
   - Video reviews (Media Agent)
   - Social media comments (Insight Agent)
4. Generate report with insights

**Documents**: [Workflow Guide](06-WORKFLOW.md) → [Database Schema](05-DATABASE-SCHEMA.md)

### Example 3: Custom Integration

**Goal**: Integrate BettaFish analysis into your application

**Steps**:
1. Set up BettaFish backend
2. Use REST APIs to submit queries programmatically
3. Receive structured JSON results
4. Display in your application UI

**Code Example**:
```python
import requests

# Submit analysis request
response = requests.post(
    "http://localhost:5000/api/search",
    json={"query": "Tesla brand sentiment"}
)

results = response.json()
# Use results in your application
```

**Documents**: [API Reference](04-API-REFERENCE.md) → [Architecture](03-ARCHITECTURE.md)

## 🔍 Key Concepts

### Multi-Agent Architecture
BettaFish uses multiple specialized AI agents that work together:
- **Query Agent**: Searches news and web sources
- **Media Agent**: Analyzes videos and images
- **Insight Agent**: Mines private databases
- **Forum Engine**: Facilitates agent collaboration
- **Report Agent**: Generates final reports

[Learn more →](01-SYSTEM-OVERVIEW.md#core-components)

### Forum Collaboration
A unique mechanism where agents:
- Share findings through log files
- An AI moderator facilitates discussion
- Agents adjust strategies based on peers' insights
- Produces collective intelligence

[Learn more →](06-WORKFLOW.md#forum-collaboration-flow)

### Node-Based Processing
Each agent uses a pipeline of processing nodes:
- ReportStructureNode: Plans the analysis
- SearchNode: Gathers data
- SummaryNode: Analyzes findings
- ReflectionNode: Identifies gaps
- FormattingNode: Structures output

[Learn more →](03-ARCHITECTURE.md#agent-base-architecture)

## 📊 System Capabilities

### Data Sources
- **Social Media**: Weibo, Xiaohongshu, TikTok, Kuaishou, Bilibili
- **News**: Major news outlets (via Tavily API)
- **Web**: General web search
- **Videos**: Short video platforms
- **Images**: Image sharing platforms

### Analysis Types
- **Sentiment Analysis**: 22 languages supported
- **Topic Extraction**: AI-powered topic identification
- **Trend Analysis**: Time-series sentiment tracking
- **Multimodal**: Text, image, and video analysis
- **Statistical**: Aggregated metrics and insights

### Output Formats
- **HTML Reports**: Beautiful, styled reports
- **JSON API**: Structured data for integrations
- **Real-time**: WebSocket updates during analysis
- **Database**: Store results for later analysis

## 🛠️ Technical Stack

### Backend
- **Python 3.9+**: Core language
- **Flask**: Web framework
- **PostgreSQL/MySQL**: Database
- **Streamlit**: Agent UIs

### AI/ML
- **LLM Providers**: Kimi, Gemini, DeepSeek, Qwen
- **Fine-tuned Models**: BERT, GPT-2, Qwen3
- **Traditional ML**: SVM, Naive Bayes, XGBoost

### Infrastructure
- **Docker**: Containerization
- **Playwright**: Web crawling
- **Tavily/Bocha**: Search APIs
- **Socket.IO**: Real-time communication

[Full details →](01-SYSTEM-OVERVIEW.md#technology-stack)

## 🤝 Getting Help

### Official Resources
- **GitHub Issues**: [Report bugs](https://github.com/666ghj/BettaFish/issues)
- **Discussions**: [Ask questions](https://github.com/666ghj/BettaFish/discussions)
- **FAQ**: [Common questions](https://github.com/666ghj/BettaFish/issues/185)
- **Email**: 670939375@qq.com

### Community
- **Linux.do Discussion**: [Active thread](https://linux.do/t/topic/1009280)
- **Documentation**: You're reading it!

### Before Asking for Help

1. **Check the docs**: Search this documentation
2. **Check logs**: Look in `logs/` directory
3. **Search issues**: Someone may have had the same problem
4. **Provide details**: Error messages, logs, configuration

## 📈 What's Next?

### System Improvements
The BettaFish team is working on:
- **Prediction Models**: Time-series forecasting of sentiment trends
- **Graph Neural Networks**: Relationship analysis between topics
- **Multimodal Fusion**: Better integration of text, image, and video
- **Real-time Analysis**: Live monitoring of social media

### Documentation Improvements
Help us improve these docs:
- **Examples**: More real-world use cases
- **Tutorials**: Step-by-step guides
- **Videos**: Screen recordings and walkthroughs
- **Translations**: Documentation in more languages

## 📝 Document Conventions

Throughout this documentation:

- `code` - Inline code, commands, file names
- **Bold** - Important terms, emphasis
- *Italic* - File paths, placeholders
- > Blockquotes - Important notes
- `[Link text](url)` - Cross-references

### Code Blocks

```bash
# Shell commands (run in terminal)
$ command --flag value
```

```python
# Python code examples
def example():
    pass
```

```sql
-- SQL queries
SELECT * FROM table;
```

```json
// JSON examples
{
  "key": "value"
}
```

## 🙏 Credits

### Main Contributors
BettaFish is developed by a dedicated team. See the [contributor graph](https://github.com/666ghj/BettaFish/graphs/contributors).

### Sponsors
- **LLM API**: [AIHubMix](https://aihubmix.com/?aff=8Ds9), [302.AI](https://share.302.ai/P66Qe3)
- **Compute**: LionCC, VibeCodingAPI

### Open Source Libraries
BettaFish builds on many excellent open-source projects:
- Flask, Streamlit, PostgreSQL/MySQL
- PyTorch, Transformers, scikit-learn
- Playwright, Requests, Loguru
- And many more!

## 📄 License

BettaFish is licensed under the [GPL-2.0 License](../LICENSE).

## ⚠️ Disclaimer

**Important**: BettaFish is for educational and research purposes only:
- Not for commercial use without permission
- Comply with robots.txt and platform ToS
- Respect data privacy and regulations
- Use responsibly and ethically

[Full disclaimer →](01-SYSTEM-OVERVIEW.md#disclaimer)

---

## 🎉 Ready to Start?

1. **New Users**: Start with [System Overview](01-SYSTEM-OVERVIEW.md)
2. **Quick Start**: Jump to [Docker Installation](02-INSTALLATION-GUIDE.md#quick-start-with-docker)
3. **Developers**: Review [Architecture](03-ARCHITECTURE.md) and [APIs](04-API-REFERENCE.md)

**Questions?** Check the [FAQ](https://github.com/666ghj/BettaFish/issues/185) or [ask the community](https://github.com/666ghj/BettaFish/discussions).

---

*Last Updated: January 2025*
*Documentation Version: 1.0.0*
*BettaFish Version: 1.0.0*
