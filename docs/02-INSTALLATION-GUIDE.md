# Installation & Setup Guide

## Table of Contents
- [System Requirements](#system-requirements)
- [Quick Start with Docker](#quick-start-with-docker)
- [Manual Installation](#manual-installation)
- [Configuration](#configuration)
- [Database Setup](#database-setup)
- [Verification](#verification)
- [Troubleshooting](#troubleshooting)

## System Requirements

### Hardware Requirements
- **CPU**: 2+ cores recommended
- **RAM**: 4GB minimum, 8GB+ recommended
- **Storage**: 10GB+ free space (more if storing large amounts of crawled data)
- **Network**: Stable internet connection for API calls and crawling

### Software Requirements
- **Operating System**: Windows, Linux, or MacOS
- **Python**: 3.9 or higher (3.11 recommended)
- **Database**: PostgreSQL 12+ or MySQL 5.7+
- **Docker** (Optional): For containerized deployment
- **Git**: For cloning the repository

### API Keys Required
You will need API keys for the following services:

1. **LLM Providers** (at least one of each):
   - **Moonshot AI (Kimi)**: For Insight Agent - [Apply here](https://platform.moonshot.cn/)
   - **Gemini**: For Media and Report Agents - [Get via AIHubMix](https://aihubmix.com/?aff=8Ds9)
   - **DeepSeek**: For Query Agent - [Apply here](https://www.deepseek.com/)
   - **Qwen/SiliconFlow**: For Forum Moderator - [Apply here](https://cloud.siliconflow.cn/)

2. **Search APIs**:
   - **Tavily API**: For web search - [Apply here](https://www.tavily.com/)
   - **Bocha AI** (Optional): For Chinese web search - [Apply here](https://open.bochaai.com/)

## Quick Start with Docker

Docker provides the easiest way to get started with BettaFish.

### Step 1: Clone the Repository

```bash
git clone https://github.com/666ghj/BettaFish.git
cd BettaFish
```

### Step 2: Configure Environment Variables

1. Copy the example environment file:
```bash
cp .env.example .env
```

2. Edit the `.env` file with your configuration:
```bash
nano .env  # or use your preferred editor
```

**Required configurations:**

```env
# ====================== Database Configuration ======================
DB_HOST=db                    # Use 'db' for Docker, or your host for manual install
DB_PORT=5432                  # 5432 for PostgreSQL, 3306 for MySQL
DB_USER=bettafish
DB_PASSWORD=bettafish
DB_NAME=bettafish
DB_CHARSET=utf8mb4
DB_DIALECT=postgresql         # or 'mysql'

# ====================== LLM Configuration ======================
# Insight Agent (Recommended: Kimi)
INSIGHT_ENGINE_API_KEY=your_moonshot_api_key_here
INSIGHT_ENGINE_BASE_URL=https://api.moonshot.cn/v1
INSIGHT_ENGINE_MODEL_NAME=kimi-k2-0711-preview

# Media Agent (Recommended: Gemini)
MEDIA_ENGINE_API_KEY=your_gemini_api_key_here
MEDIA_ENGINE_BASE_URL=https://aihubmix.com/v1
MEDIA_ENGINE_MODEL_NAME=gemini-2.5-pro

# Query Agent (Recommended: DeepSeek)
QUERY_ENGINE_API_KEY=your_deepseek_api_key_here
QUERY_ENGINE_BASE_URL=https://api.deepseek.com
QUERY_ENGINE_MODEL_NAME=deepseek-reasoner

# Report Agent (Recommended: Gemini)
REPORT_ENGINE_API_KEY=your_gemini_api_key_here
REPORT_ENGINE_BASE_URL=https://aihubmix.com/v1
REPORT_ENGINE_MODEL_NAME=gemini-2.5-pro

# Forum Host (Recommended: Qwen)
FORUM_HOST_API_KEY=your_siliconflow_api_key_here
FORUM_HOST_BASE_URL=https://api.siliconflow.cn/v1
FORUM_HOST_MODEL_NAME=Qwen/Qwen3-235B-A22B-Instruct-2507

# Keyword Optimizer (Recommended: Qwen)
KEYWORD_OPTIMIZER_API_KEY=your_siliconflow_api_key_here
KEYWORD_OPTIMIZER_BASE_URL=https://api.siliconflow.cn/v1
KEYWORD_OPTIMIZER_MODEL_NAME=Qwen/Qwen3-30B-A3B-Instruct-2507

# ====================== Search API Configuration ======================
TAVILY_API_KEY=your_tavily_api_key_here
BOCHA_WEB_SEARCH_API_KEY=your_bocha_api_key_here  # Optional
```

### Step 3: Start Services

```bash
docker compose up -d
```

This will:
- Pull all required Docker images
- Start PostgreSQL database
- Start the BettaFish application
- Initialize the database schema

### Step 4: Access the Application

Open your browser and navigate to:
```
http://localhost:5000
```

You should see the BettaFish web interface.

### Step 5: Start the System

1. Click on the **"Configuration"** tab in the web interface
2. Verify your API keys are correctly set
3. Click **"Start System"** button
4. Wait for all agents to initialize (you'll see status updates)

### Docker Commands Reference

```bash
# Start services (detached mode)
docker compose up -d

# View logs
docker compose logs -f

# Stop services
docker compose down

# Restart services
docker compose restart

# Rebuild after code changes
docker compose up -d --build

# Remove all data (including database)
docker compose down -v
```

## Manual Installation

For development or if you prefer not to use Docker, follow these steps:

### Step 1: Install Python and Dependencies

#### 1.1 Create Virtual Environment

**Using Conda (Recommended):**
```bash
conda create -n bettafish python=3.11
conda activate bettafish
```

**Using venv:**
```bash
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate
```

**Using uv (Fastest):**
```bash
uv venv --python 3.11
source .venv/bin/activate  # On Windows: .venv\Scripts\activate
```

#### 1.2 Install Dependencies

```bash
# Basic installation
pip install -r requirements.txt

# Or with uv (much faster)
uv pip install -r requirements.txt
```

**Note**: If you don't want to install sentiment analysis models (they require ML libraries like PyTorch), you can comment out the "Machine Learning" section in `requirements.txt` before running the install command.

### Step 2: Install Playwright Browsers

Playwright is required for web crawling functionality:

```bash
playwright install chromium
```

### Step 3: Set Up Database

#### Option A: Local PostgreSQL

1. Install PostgreSQL (if not already installed):
   - **Ubuntu/Debian**: `sudo apt-get install postgresql postgresql-contrib`
   - **MacOS**: `brew install postgresql`
   - **Windows**: Download from [postgresql.org](https://www.postgresql.org/download/windows/)

2. Create database and user:
```sql
sudo -u postgres psql

CREATE DATABASE bettafish;
CREATE USER bettafish WITH PASSWORD 'your_secure_password';
GRANT ALL PRIVILEGES ON DATABASE bettafish TO bettafish;
\q
```

3. Update `.env` file:
```env
DB_DIALECT=postgresql
DB_HOST=localhost
DB_PORT=5432
DB_USER=bettafish
DB_PASSWORD=your_secure_password
DB_NAME=bettafish
```

#### Option B: Local MySQL

1. Install MySQL:
   - **Ubuntu/Debian**: `sudo apt-get install mysql-server`
   - **MacOS**: `brew install mysql`
   - **Windows**: Download from [mysql.com](https://dev.mysql.com/downloads/installer/)

2. Create database and user:
```sql
mysql -u root -p

CREATE DATABASE bettafish CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
CREATE USER 'bettafish'@'localhost' IDENTIFIED BY 'your_secure_password';
GRANT ALL PRIVILEGES ON bettafish.* TO 'bettafish'@'localhost';
FLUSH PRIVILEGES;
exit
```

3. Update `.env` file:
```env
DB_DIALECT=mysql
DB_HOST=localhost
DB_PORT=3306
DB_USER=bettafish
DB_PASSWORD=your_secure_password
DB_NAME=bettafish
DB_CHARSET=utf8mb4
```

#### Option C: Cloud Database (Recommended for Production)

The BettaFish team offers a free cloud database service with:
- 100,000+ daily real public opinion data
- Real-time updates
- Multi-dimensional tag classification
- High availability
- Professional support

**To apply**: Contact 670939375@qq.com

> **Note**: As of October 1, 2025, new applications are temporarily suspended for data compliance review.

### Step 4: Configure Environment Variables

Follow the same `.env` configuration steps as in the Docker section above.

### Step 5: Initialize Database

The database will be automatically initialized when you first run the application. Or you can manually initialize:

```bash
cd MindSpider
python main.py --setup
```

### Step 6: Start the Application

```bash
# Activate your virtual environment first
conda activate bettafish  # or your environment activation command

# Start the main application
python app.py
```

The application will start on `http://localhost:5000` by default.

### Step 7: Start Individual Agents (Optional)

If you want to run agents individually for testing:

```bash
# Start Insight Engine
streamlit run SingleEngineApp/insight_engine_streamlit_app.py --server.port 8501

# Start Media Engine
streamlit run SingleEngineApp/media_engine_streamlit_app.py --server.port 8502

# Start Query Engine
streamlit run SingleEngineApp/query_engine_streamlit_app.py --server.port 8503
```

## Configuration

### Environment Variables

All configuration is done through the `.env` file. Key sections:

#### Flask Server Configuration
```env
HOST=0.0.0.0                  # Allow external access, use 127.0.0.1 for local only
PORT=5000                     # Web interface port
```

#### Agent Configuration Parameters

You can fine-tune agent behavior by editing agent-specific config files:

**QueryEngine Configuration** (`QueryEngine/utils/config.py`):
```python
class Config:
    max_reflections = 2           # Number of reflection rounds
    max_search_results = 15       # Maximum search results to retrieve
    max_content_length = 8000     # Maximum content length to process
```

**MediaEngine Configuration** (`MediaEngine/utils/config.py`):
```python
class Config:
    comprehensive_search_limit = 10  # Comprehensive search limit
    web_search_limit = 15            # Web search limit
```

**InsightEngine Configuration** (`InsightEngine/utils/config.py`):
```python
class Config:
    default_search_topic_globally_limit = 200    # Global search limit
    default_get_comments_limit = 500             # Comment retrieval limit
    max_search_results_for_llm = 50              # Max results sent to LLM
```

### Custom LLM Providers

BettaFish supports any LLM provider that follows the OpenAI API format. To use a custom provider:

1. Update the `.env` file with your provider's details:
```env
INSIGHT_ENGINE_API_KEY=your_api_key
INSIGHT_ENGINE_BASE_URL=https://your-provider.com/v1
INSIGHT_ENGINE_MODEL_NAME=your-model-name
```

2. Example with SiliconFlow:
```env
QUERY_ENGINE_API_KEY=sk-xxxxx
QUERY_ENGINE_BASE_URL=https://api.siliconflow.cn/v1
QUERY_ENGINE_MODEL_NAME=Qwen/Qwen2.5-72B-Instruct
```

## Database Setup

### Automatic Initialization

When you start the application for the first time, it will automatically:
1. Create all required tables
2. Set up indexes
3. Initialize default data

### Manual Initialization

If you need to manually initialize or reset the database:

```bash
cd MindSpider
python main.py --setup
```

### Database Schema

The system uses several tables to store:
- **Crawled Posts**: Social media posts from various platforms
- **Comments**: User comments on posts
- **Topics**: Extracted trending topics
- **Hot Rankings**: Trending content rankings
- **Sentiment Analysis**: Stored sentiment scores

For detailed schema information, see [Database Schema Documentation](05-DATABASE-SCHEMA.md).

## Verification

### 1. Check Service Status

After starting, verify all services are running:

```bash
# For Docker
docker compose ps

# For manual install, check processes
ps aux | grep streamlit
ps aux | grep python
```

### 2. Check Web Interface

1. Navigate to `http://localhost:5000`
2. You should see the BettaFish main interface
3. Try the "Configuration" tab to verify API keys are loaded

### 3. Check Database Connection

```python
# Test database connection
python -c "from config import settings; print(f'Database: {settings.DB_DIALECT}://{settings.DB_HOST}:{settings.DB_PORT}/{settings.DB_NAME}')"
```

### 4. Run a Test Query

1. In the web interface, click "Start System"
2. Wait for all agents to initialize
3. Enter a simple query like: "What's the public opinion on AI?"
4. Click "Search" and verify results appear

### 5. Check Logs

Logs are stored in the `logs/` directory:
```bash
ls logs/
# Should show: insight.log, media.log, query.log, forum.log

# View recent logs
tail -f logs/insight.log
```

## Troubleshooting

### Common Issues

#### Issue 1: Database Connection Failed

**Symptoms**: 
- Error: "Could not connect to database"
- Application fails to start

**Solutions**:
1. Verify database is running:
   ```bash
   # PostgreSQL
   sudo systemctl status postgresql
   
   # MySQL
   sudo systemctl status mysql
   ```

2. Check credentials in `.env` match your database setup

3. Verify network connectivity:
   ```bash
   # Test PostgreSQL connection
   psql -h localhost -U bettafish -d bettafish
   
   # Test MySQL connection
   mysql -h localhost -u bettafish -p bettafish
   ```

4. Check firewall rules allow database connections

#### Issue 2: API Key Errors

**Symptoms**:
- Error: "Invalid API key"
- Agents fail to start

**Solutions**:
1. Verify API keys are correctly set in `.env`
2. Check for extra spaces or quotes around keys
3. Confirm API keys are active and not expired
4. Test API key directly:
   ```python
   from openai import OpenAI
   client = OpenAI(api_key="your_key", base_url="https://api.moonshot.cn/v1")
   response = client.chat.completions.create(
       model="kimi-k2-0711-preview",
       messages=[{"role": "user", "content": "Hello"}]
   )
   print(response.choices[0].message.content)
   ```

#### Issue 3: Port Already in Use

**Symptoms**:
- Error: "Address already in use"
- Application won't start

**Solutions**:
1. Check what's using the port:
   ```bash
   # Linux/Mac
   lsof -i :5000
   
   # Windows
   netstat -ano | findstr :5000
   ```

2. Kill the process or change port in `.env`:
   ```env
   PORT=5001
   ```

#### Issue 4: Playwright Browser Not Found

**Symptoms**:
- Error: "Executable doesn't exist"
- Crawler fails

**Solutions**:
1. Install Playwright browsers:
   ```bash
   playwright install chromium
   ```

2. If issues persist, try:
   ```bash
   playwright install-deps
   playwright install chromium
   ```

#### Issue 5: Streamlit Apps Won't Start

**Symptoms**:
- Agents show "starting" but never reach "running"
- Timeout errors

**Solutions**:
1. Check Streamlit is installed:
   ```bash
   streamlit --version
   ```

2. Try starting manually to see errors:
   ```bash
   streamlit run SingleEngineApp/insight_engine_streamlit_app.py --server.port 8501
   ```

3. Check for port conflicts
4. Review `logs/insight.log` for detailed errors

#### Issue 6: Out of Memory

**Symptoms**:
- Application crashes
- System becomes unresponsive

**Solutions**:
1. Increase available RAM
2. Reduce batch sizes in agent configs
3. Limit concurrent operations:
   - Reduce `max_search_results`
   - Decrease `default_get_comments_limit`

#### Issue 7: Slow Performance

**Symptoms**:
- Queries take very long
- System is sluggish

**Solutions**:
1. Check database indexes are created:
   ```sql
   \d+ posts  -- PostgreSQL
   SHOW INDEX FROM posts;  -- MySQL
   ```

2. Reduce search limits in config files
3. Use faster LLM models (e.g., smaller Qwen models)
4. Enable database query caching

### Getting Help

If you encounter issues not covered here:

1. **Check Logs**: Always check `logs/` directory first
2. **Search Issues**: [GitHub Issues](https://github.com/666ghj/BettaFish/issues)
3. **Ask Community**: [GitHub Discussions](https://github.com/666ghj/BettaFish/discussions)
4. **Read FAQ**: [Common Questions](https://github.com/666ghj/BettaFish/issues/185)
5. **Contact Support**: Email 670939375@qq.com

### Debug Mode

To run in debug mode for more verbose output:

```python
# In app.py, change:
socketio.run(app, host=HOST, port=PORT, debug=True)  # Enable debug
```

Or set environment variable:
```bash
export FLASK_DEBUG=1
python app.py
```

## Next Steps

Now that you have BettaFish installed:

1. **[Understand the Architecture](03-ARCHITECTURE.md)**: Learn how components interact
2. **[Explore the Workflow](06-WORKFLOW.md)**: Understand data flow
3. **[Run the Crawler](08-CRAWLER-GUIDE.md)**: Start collecting data
4. **[Customize Reports](07-DEVELOPMENT-GUIDE.md#custom-report-templates)**: Create custom templates
5. **[API Integration](04-API-REFERENCE.md)**: Integrate with your applications

## Maintenance

### Regular Tasks

1. **Update Dependencies** (monthly):
   ```bash
   pip install -r requirements.txt --upgrade
   ```

2. **Clean Old Logs** (weekly):
   ```bash
   find logs/ -name "*.log" -mtime +7 -delete
   ```

3. **Database Backup** (daily):
   ```bash
   # PostgreSQL
   pg_dump -U bettafish bettafish > backup_$(date +%Y%m%d).sql
   
   # MySQL
   mysqldump -u bettafish -p bettafish > backup_$(date +%Y%m%d).sql
   ```

4. **Monitor Disk Space**:
   ```bash
   df -h
   du -sh logs/
   ```

### Security Best Practices

1. **Never commit `.env` file** to version control
2. **Use strong database passwords**
3. **Rotate API keys regularly**
4. **Keep Python and dependencies updated**
5. **Use HTTPS in production** (configure reverse proxy)
6. **Limit database access** to localhost if possible
7. **Monitor API usage** to detect anomalies
