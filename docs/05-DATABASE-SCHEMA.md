# Database Schema Documentation

## Table of Contents
- [Introduction](#introduction)
- [Database Design](#database-design)
- [Tables Overview](#tables-overview)
- [Detailed Schema](#detailed-schema)
- [Relationships](#relationships)
- [Indexes](#indexes)
- [Query Examples](#query-examples)

## Introduction

BettaFish uses a relational database (PostgreSQL or MySQL) to store crawled social media data, analysis results, and system metadata. The schema is designed to handle high-volume data from multiple platforms while maintaining query performance.

## Database Design

### Design Principles

1. **Platform Agnostic**: Separate tables for each platform to accommodate platform-specific fields
2. **Denormalization**: Some data is denormalized for query performance
3. **Indexing**: Strategic indexes on frequently queried columns
4. **Partitioning**: Date-based partitioning for large tables (optional)
5. **UTF-8**: Full UTF-8 support (utf8mb4) for emoji and special characters

### Supported Platforms

- **Weibo** (微博)
- **Xiaohongshu** (小红书)
- **Douyin** (抖音/TikTok)
- **Kuaishou** (快手)
- **Bilibili** (哔哩哔哩)
- **And more...**

## Tables Overview

### Core Tables

| Table Name | Description | Row Count (Typical) |
|------------|-------------|---------------------|
| `weibo_posts` | Weibo posts | 1M+ |
| `weibo_comments` | Comments on Weibo posts | 10M+ |
| `xiaohongshu_posts` | Xiaohongshu posts | 500K+ |
| `xiaohongshu_comments` | Comments on XHS posts | 5M+ |
| `douyin_videos` | Douyin/TikTok videos | 300K+ |
| `douyin_comments` | Comments on Douyin videos | 3M+ |
| `kuaishou_videos` | Kuaishou videos | 200K+ |
| `kuaishou_comments` | Comments on Kuaishou videos | 2M+ |
| `hot_rankings` | Platform hot rankings | 10K+ |
| `topics` | Extracted topics | 50K+ |
| `keywords` | Keyword management | 1K+ |
| `crawl_history` | Crawling history | 100K+ |

## Detailed Schema

### weibo_posts

Stores Weibo posts (微博文章).

```sql
CREATE TABLE weibo_posts (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    post_id VARCHAR(100) UNIQUE NOT NULL,
    
    -- Content
    content TEXT NOT NULL,
    title VARCHAR(500),
    
    -- Author information
    author VARCHAR(100) NOT NULL,
    author_id VARCHAR(100),
    author_avatar VARCHAR(500),
    
    -- Metadata
    keyword VARCHAR(200),
    topic VARCHAR(200),
    platform VARCHAR(50) DEFAULT 'weibo',
    
    -- Engagement metrics
    likes_count INTEGER DEFAULT 0,
    comments_count INTEGER DEFAULT 0,
    shares_count INTEGER DEFAULT 0,
    views_count INTEGER DEFAULT 0,
    
    -- Media
    images TEXT[],                    -- Array of image URLs
    videos TEXT[],                    -- Array of video URLs
    
    -- Location
    location VARCHAR(200),
    
    -- Timestamps
    post_date TIMESTAMP,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Analysis results
    sentiment_score FLOAT,            -- -1.0 to 1.0
    sentiment_label VARCHAR(20),      -- positive, negative, neutral
    sentiment_confidence FLOAT,       -- 0.0 to 1.0
    
    -- Source information
    source_url VARCHAR(1000),
    source_type VARCHAR(50),
    
    -- Status
    is_deleted BOOLEAN DEFAULT FALSE,
    is_verified BOOLEAN DEFAULT FALSE,
    
    -- Indexes
    INDEX idx_post_id (post_id),
    INDEX idx_keyword (keyword),
    INDEX idx_topic (topic),
    INDEX idx_author (author),
    INDEX idx_post_date (post_date),
    INDEX idx_crawled_at (crawled_at),
    INDEX idx_sentiment_label (sentiment_label),
    INDEX idx_composite (keyword, post_date),
    
    -- Full-text search (PostgreSQL)
    -- CREATE INDEX idx_content_fulltext ON weibo_posts USING gin(to_tsvector('chinese', content));
);
```

### weibo_comments

Stores comments on Weibo posts.

```sql
CREATE TABLE weibo_comments (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    comment_id VARCHAR(100) UNIQUE NOT NULL,
    
    -- Foreign key
    post_id VARCHAR(100) NOT NULL,
    parent_comment_id VARCHAR(100),    -- For nested comments
    
    -- Content
    content TEXT NOT NULL,
    
    -- Author
    author VARCHAR(100) NOT NULL,
    author_id VARCHAR(100),
    
    -- Engagement
    likes_count INTEGER DEFAULT 0,
    
    -- Timestamps
    comment_date TIMESTAMP,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Analysis
    sentiment_score FLOAT,
    sentiment_label VARCHAR(20),
    sentiment_confidence FLOAT,
    
    -- Status
    is_deleted BOOLEAN DEFAULT FALSE,
    
    -- Indexes
    INDEX idx_comment_id (comment_id),
    INDEX idx_post_id (post_id),
    INDEX idx_author (author),
    INDEX idx_comment_date (comment_date),
    INDEX idx_sentiment (sentiment_label),
    
    -- Foreign key constraint
    FOREIGN KEY (post_id) REFERENCES weibo_posts(post_id) ON DELETE CASCADE
);
```

### xiaohongshu_posts

Stores Xiaohongshu (Little Red Book) posts.

```sql
CREATE TABLE xiaohongshu_posts (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    post_id VARCHAR(100) UNIQUE NOT NULL,
    
    -- Content
    content TEXT NOT NULL,
    title VARCHAR(500),
    
    -- Author
    author VARCHAR(100) NOT NULL,
    author_id VARCHAR(100),
    
    -- Metadata
    keyword VARCHAR(200),
    topic VARCHAR(200),
    platform VARCHAR(50) DEFAULT 'xiaohongshu',
    
    -- Engagement
    likes_count INTEGER DEFAULT 0,
    comments_count INTEGER DEFAULT 0,
    shares_count INTEGER DEFAULT 0,
    favorites_count INTEGER DEFAULT 0,
    
    -- Media
    images TEXT[],
    videos TEXT[],
    
    -- Tags
    tags TEXT[],
    
    -- Location
    location VARCHAR(200),
    
    -- Timestamps
    post_date TIMESTAMP,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Analysis
    sentiment_score FLOAT,
    sentiment_label VARCHAR(20),
    sentiment_confidence FLOAT,
    
    -- Product information (XHS is shopping-focused)
    product_name VARCHAR(300),
    product_price DECIMAL(10, 2),
    product_brand VARCHAR(200),
    
    -- Source
    source_url VARCHAR(1000),
    
    -- Status
    is_deleted BOOLEAN DEFAULT FALSE,
    
    -- Indexes
    INDEX idx_post_id (post_id),
    INDEX idx_keyword (keyword),
    INDEX idx_post_date (post_date),
    INDEX idx_product_brand (product_brand)
);
```

### xiaohongshu_comments

Stores comments on Xiaohongshu posts.

```sql
CREATE TABLE xiaohongshu_comments (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    comment_id VARCHAR(100) UNIQUE NOT NULL,
    
    -- Foreign key
    post_id VARCHAR(100) NOT NULL,
    parent_comment_id VARCHAR(100),
    
    -- Content
    content TEXT NOT NULL,
    
    -- Author
    author VARCHAR(100) NOT NULL,
    author_id VARCHAR(100),
    
    -- Engagement
    likes_count INTEGER DEFAULT 0,
    
    -- Timestamps
    comment_date TIMESTAMP,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Analysis
    sentiment_score FLOAT,
    sentiment_label VARCHAR(20),
    
    -- Status
    is_deleted BOOLEAN DEFAULT FALSE,
    
    -- Indexes
    INDEX idx_comment_id (comment_id),
    INDEX idx_post_id (post_id),
    INDEX idx_comment_date (comment_date),
    
    FOREIGN KEY (post_id) REFERENCES xiaohongshu_posts(post_id) ON DELETE CASCADE
);
```

### douyin_videos

Stores Douyin (TikTok China) videos.

```sql
CREATE TABLE douyin_videos (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    video_id VARCHAR(100) UNIQUE NOT NULL,
    
    -- Content
    title VARCHAR(500),
    description TEXT,
    
    -- Author
    author VARCHAR(100) NOT NULL,
    author_id VARCHAR(100),
    
    -- Metadata
    keyword VARCHAR(200),
    topic VARCHAR(200),
    platform VARCHAR(50) DEFAULT 'douyin',
    
    -- Engagement
    likes_count INTEGER DEFAULT 0,
    comments_count INTEGER DEFAULT 0,
    shares_count INTEGER DEFAULT 0,
    views_count INTEGER DEFAULT 0,
    
    -- Video specific
    video_url VARCHAR(1000),
    cover_url VARCHAR(1000),
    duration INTEGER,                  -- in seconds
    
    -- Music
    music_title VARCHAR(300),
    music_author VARCHAR(100),
    
    -- Tags
    hashtags TEXT[],
    
    -- Location
    location VARCHAR(200),
    
    -- Timestamps
    post_date TIMESTAMP,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Analysis
    sentiment_score FLOAT,
    sentiment_label VARCHAR(20),
    video_transcript TEXT,             -- AI-generated transcript
    
    -- Status
    is_deleted BOOLEAN DEFAULT FALSE,
    
    -- Indexes
    INDEX idx_video_id (video_id),
    INDEX idx_keyword (keyword),
    INDEX idx_author (author),
    INDEX idx_post_date (post_date)
);
```

### hot_rankings

Stores hot topic rankings from various platforms.

```sql
CREATE TABLE hot_rankings (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    
    -- Platform and ranking
    platform VARCHAR(50) NOT NULL,
    rank INTEGER NOT NULL,
    
    -- Topic information
    topic VARCHAR(300) NOT NULL,
    topic_url VARCHAR(1000),
    
    -- Metrics
    heat_value BIGINT,                 -- Platform-specific heat score
    discussion_count INTEGER,
    read_count BIGINT,
    
    -- Category
    category VARCHAR(100),
    
    -- Timestamps
    ranking_date DATE NOT NULL,
    ranking_time TIME,
    crawled_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_platform_date (platform, ranking_date),
    INDEX idx_topic (topic),
    INDEX idx_rank (rank),
    
    -- Unique constraint
    UNIQUE (platform, ranking_date, ranking_time, rank)
);
```

### topics

Stores extracted and analyzed topics.

```sql
CREATE TABLE topics (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    
    -- Topic information
    name VARCHAR(300) UNIQUE NOT NULL,
    category VARCHAR(100),
    
    -- Statistics
    mention_count INTEGER DEFAULT 0,
    positive_count INTEGER DEFAULT 0,
    negative_count INTEGER DEFAULT 0,
    neutral_count INTEGER DEFAULT 0,
    
    -- Platforms
    platforms TEXT[],                  -- Which platforms this topic appears on
    
    -- Timeline
    first_seen TIMESTAMP,
    last_seen TIMESTAMP,
    peak_date DATE,
    
    -- Analysis
    overall_sentiment FLOAT,
    trend VARCHAR(50),                 -- rising, falling, stable
    
    -- Keywords
    related_keywords TEXT[],
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    
    -- Indexes
    INDEX idx_name (name),
    INDEX idx_category (category),
    INDEX idx_trend (trend),
    INDEX idx_updated (updated_at)
);
```

### keywords

Manages keywords for crawling.

```sql
CREATE TABLE keywords (
    -- Primary identifier
    id SERIAL PRIMARY KEY,
    
    -- Keyword
    keyword VARCHAR(200) UNIQUE NOT NULL,
    
    -- Category
    category VARCHAR(100),
    
    -- Priority
    priority INTEGER DEFAULT 5,        -- 1-10, higher = more important
    
    -- Status
    is_active BOOLEAN DEFAULT TRUE,
    
    -- Statistics
    search_count INTEGER DEFAULT 0,
    result_count INTEGER DEFAULT 0,
    
    -- Timestamps
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    last_crawled TIMESTAMP,
    
    -- Indexes
    INDEX idx_keyword (keyword),
    INDEX idx_category (category),
    INDEX idx_active (is_active)
);
```

### crawl_history

Tracks crawling operations.

```sql
CREATE TABLE crawl_history (
    -- Primary identifier
    id BIGSERIAL PRIMARY KEY,
    
    -- Crawl information
    platform VARCHAR(50) NOT NULL,
    keyword VARCHAR(200),
    
    -- Status
    status VARCHAR(50) NOT NULL,       -- success, failed, partial
    
    -- Statistics
    items_crawled INTEGER DEFAULT 0,
    items_saved INTEGER DEFAULT 0,
    items_failed INTEGER DEFAULT 0,
    
    -- Error information
    error_message TEXT,
    
    -- Timestamps
    started_at TIMESTAMP NOT NULL,
    finished_at TIMESTAMP,
    duration INTEGER,                   -- in seconds
    
    -- Indexes
    INDEX idx_platform (platform),
    INDEX idx_started_at (started_at),
    INDEX idx_status (status)
);
```

## Relationships

### Entity Relationship Diagram

```
┌─────────────────┐
│  weibo_posts    │
│  (1)            │
└────────┬────────┘
         │
         │ 1:N
         │
         ↓
┌─────────────────┐
│weibo_comments   │
│  (N)            │
└─────────────────┘

┌─────────────────────┐
│ xiaohongshu_posts   │
│  (1)                │
└──────────┬──────────┘
           │
           │ 1:N
           │
           ↓
┌──────────────────────┐
│xiaohongshu_comments  │
│  (N)                 │
└──────────────────────┘

┌─────────────────┐
│ douyin_videos   │
│  (1)            │
└────────┬────────┘
         │
         │ 1:N
         │
         ↓
┌─────────────────┐
│douyin_comments  │
│  (N)            │
└─────────────────┘

┌─────────────────┐
│   keywords      │
│  (1)            │
└────────┬────────┘
         │
         │ 1:N
         │
         ↓
┌─────────────────┐
│crawl_history    │
│  (N)            │
└─────────────────┘
```

## Indexes

### Index Strategy

1. **Primary Keys**: All tables have auto-incrementing primary keys
2. **Unique Constraints**: Platform-specific IDs are unique
3. **Foreign Keys**: Comments reference their parent posts
4. **Composite Indexes**: For common query patterns
5. **Full-Text Search**: For content search (PostgreSQL only)

### Important Indexes

```sql
-- Fast keyword searches
CREATE INDEX idx_weibo_keyword ON weibo_posts(keyword, post_date DESC);
CREATE INDEX idx_xhs_keyword ON xiaohongshu_posts(keyword, post_date DESC);

-- Sentiment analysis queries
CREATE INDEX idx_weibo_sentiment ON weibo_posts(sentiment_label, sentiment_score);

-- Date range queries
CREATE INDEX idx_weibo_date_range ON weibo_posts(post_date DESC);

-- Author queries
CREATE INDEX idx_weibo_author ON weibo_posts(author, post_date DESC);

-- Full-text search (PostgreSQL)
CREATE INDEX idx_weibo_content_fts ON weibo_posts 
USING gin(to_tsvector('chinese', content));

CREATE INDEX idx_comment_content_fts ON weibo_comments 
USING gin(to_tsvector('chinese', content));
```

## Query Examples

### Common Queries

#### 1. Get Recent Posts by Keyword

```sql
SELECT 
    post_id,
    content,
    author,
    likes_count,
    comments_count,
    post_date,
    sentiment_label
FROM weibo_posts
WHERE keyword = 'Tesla'
    AND post_date >= NOW() - INTERVAL '7 days'
ORDER BY post_date DESC
LIMIT 100;
```

#### 2. Get Top Posts by Engagement

```sql
SELECT 
    post_id,
    content,
    author,
    likes_count,
    comments_count,
    shares_count,
    (likes_count + comments_count * 2 + shares_count * 3) as engagement_score
FROM weibo_posts
WHERE keyword = 'iPhone'
    AND post_date >= NOW() - INTERVAL '30 days'
ORDER BY engagement_score DESC
LIMIT 50;
```

#### 3. Sentiment Analysis Statistics

```sql
SELECT 
    sentiment_label,
    COUNT(*) as count,
    AVG(sentiment_score) as avg_score,
    AVG(likes_count) as avg_likes
FROM weibo_posts
WHERE keyword = 'Tesla'
    AND post_date >= NOW() - INTERVAL '7 days'
    AND sentiment_label IS NOT NULL
GROUP BY sentiment_label
ORDER BY count DESC;
```

#### 4. Get Comments for a Post

```sql
SELECT 
    c.comment_id,
    c.content,
    c.author,
    c.likes_count,
    c.sentiment_label,
    c.comment_date
FROM weibo_comments c
WHERE c.post_id = '12345'
ORDER BY c.likes_count DESC, c.comment_date DESC
LIMIT 500;
```

#### 5. Cross-Platform Topic Analysis

```sql
-- Combine results from multiple platforms
(
    SELECT 
        'weibo' as platform,
        post_id as content_id,
        content,
        author,
        post_date as publish_date,
        likes_count,
        sentiment_label
    FROM weibo_posts
    WHERE keyword = 'iPhone'
        AND post_date >= '2025-01-01'
)
UNION ALL
(
    SELECT 
        'xiaohongshu' as platform,
        post_id as content_id,
        content,
        author,
        post_date as publish_date,
        likes_count,
        sentiment_label
    FROM xiaohongshu_posts
    WHERE keyword = 'iPhone'
        AND post_date >= '2025-01-01'
)
ORDER BY publish_date DESC
LIMIT 200;
```

#### 6. Trending Topics

```sql
SELECT 
    topic,
    COUNT(*) as mention_count,
    AVG(sentiment_score) as avg_sentiment,
    MAX(post_date) as last_mentioned
FROM weibo_posts
WHERE post_date >= NOW() - INTERVAL '24 hours'
    AND topic IS NOT NULL
GROUP BY topic
HAVING COUNT(*) >= 10
ORDER BY mention_count DESC
LIMIT 20;
```

#### 7. Hot Rankings by Platform

```sql
SELECT 
    rank,
    topic,
    heat_value,
    discussion_count,
    category
FROM hot_rankings
WHERE platform = 'weibo'
    AND ranking_date = CURRENT_DATE
ORDER BY rank ASC
LIMIT 50;
```

#### 8. Full-Text Search (PostgreSQL)

```sql
SELECT 
    post_id,
    content,
    author,
    post_date,
    ts_rank(to_tsvector('chinese', content), query) AS rank
FROM weibo_posts,
     to_tsquery('chinese', 'Tesla & electric & vehicle') query
WHERE to_tsvector('chinese', content) @@ query
ORDER BY rank DESC
LIMIT 100;
```

#### 9. Time Series Analysis

```sql
SELECT 
    DATE(post_date) as date,
    COUNT(*) as post_count,
    AVG(sentiment_score) as avg_sentiment,
    SUM(likes_count) as total_likes
FROM weibo_posts
WHERE keyword = 'Tesla'
    AND post_date >= NOW() - INTERVAL '30 days'
GROUP BY DATE(post_date)
ORDER BY date DESC;
```

#### 10. Author Influence Analysis

```sql
SELECT 
    author,
    COUNT(*) as post_count,
    AVG(likes_count) as avg_likes,
    AVG(comments_count) as avg_comments,
    SUM(likes_count + comments_count + shares_count) as total_engagement
FROM weibo_posts
WHERE post_date >= NOW() - INTERVAL '30 days'
GROUP BY author
HAVING COUNT(*) >= 5
ORDER BY total_engagement DESC
LIMIT 100;
```

## Performance Optimization

### 1. Partitioning (for large tables)

```sql
-- Date-based partitioning
CREATE TABLE weibo_posts_2025_01 PARTITION OF weibo_posts
    FOR VALUES FROM ('2025-01-01') TO ('2025-02-01');

CREATE TABLE weibo_posts_2025_02 PARTITION OF weibo_posts
    FOR VALUES FROM ('2025-02-01') TO ('2025-03-01');
```

### 2. Materialized Views

```sql
-- Pre-compute sentiment statistics
CREATE MATERIALIZED VIEW sentiment_stats AS
SELECT 
    keyword,
    DATE(post_date) as date,
    platform,
    sentiment_label,
    COUNT(*) as count,
    AVG(sentiment_score) as avg_score
FROM (
    SELECT keyword, post_date, 'weibo' as platform, 
           sentiment_label, sentiment_score 
    FROM weibo_posts
    UNION ALL
    SELECT keyword, post_date, 'xiaohongshu' as platform,
           sentiment_label, sentiment_score 
    FROM xiaohongshu_posts
) combined
WHERE post_date >= NOW() - INTERVAL '90 days'
GROUP BY keyword, DATE(post_date), platform, sentiment_label;

-- Refresh periodically
REFRESH MATERIALIZED VIEW sentiment_stats;
```

### 3. Query Optimization Tips

- Use `EXPLAIN ANALYZE` to understand query performance
- Add indexes on frequently queried columns
- Avoid `SELECT *`, specify needed columns
- Use `LIMIT` to restrict result sets
- Consider denormalization for read-heavy workloads
- Use connection pooling
- Cache frequent queries in application layer

## Data Retention

### Retention Policy

```sql
-- Delete old crawl history (keep last 6 months)
DELETE FROM crawl_history 
WHERE started_at < NOW() - INTERVAL '6 months';

-- Archive old posts (keep last 2 years)
INSERT INTO weibo_posts_archive 
SELECT * FROM weibo_posts 
WHERE post_date < NOW() - INTERVAL '2 years';

DELETE FROM weibo_posts 
WHERE post_date < NOW() - INTERVAL '2 years';
```

## Backup & Recovery

### Backup Command

```bash
# PostgreSQL
pg_dump -U bettafish -d bettafish -F c -f backup_$(date +%Y%m%d).dump

# MySQL
mysqldump -u bettafish -p bettafish > backup_$(date +%Y%m%d).sql
```

### Restore Command

```bash
# PostgreSQL
pg_restore -U bettafish -d bettafish backup_20250115.dump

# MySQL
mysql -u bettafish -p bettafish < backup_20250115.sql
```

## Next Steps

- **[API Reference](04-API-REFERENCE.md)**: Learn how to query the database via APIs
- **[Development Guide](07-DEVELOPMENT-GUIDE.md)**: Extend the schema
- **[Crawler Guide](08-CRAWLER-GUIDE.md)**: Populate the database with data
