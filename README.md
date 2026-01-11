# AI News Automation System for Telegram

An advanced automation system built with n8n that collects, processes, and publishes AI news from multiple sources to Telegram channels with intelligent AI-powered editorial workflows.

## 🎯 Overview

This project automates the entire news publishing pipeline - from content collection to final publication - with human editorial oversight and AI assistance. The system monitors multiple news sources (websites and Telegram channels), processes content through AI models, and publishes curated news to production channels.

## ✨ Key Features

- **Multi-source Content Collection**: Automated scraping from websites (TechCrunch, AI Magazine, NextWeb, etc.) and Telegram channels
- **Direct Telegram Forwarding**: Forward any post from a Telegram channel to your workspace bot - the system automatically scrapes the last 10 posts from that channel for processing
- **AI-Powered Content Processing**: Uses Google Gemini for news summarization, translation, and formatting
- **Smart Hashtag Generation**: Automatic categorization with predefined hashtag taxonomy
- **Editorial Workflow**: Human-in-the-loop approval system with AI-assisted editing
- **Telegraph Integration**: Automatic creation of detailed article pages for web sources
- **Intelligent Media Handling**: Supports images and videos with automatic processing via S3-compatible storage
- **Duplicate Detection**: Prevents republishing of the same news from multiple sources
- **Iterative Editing**: Natural language editing commands powered by AI

## 🏗️ Architecture

### Data Flow

```
Sources (Websites/TG Channels) 
    ↓
Content Parsing & Storage (NocoDB + S3)
    ↓
AI Processing (Gemini)
    ↓
Editorial Review (Telegram Workspace)
    ↓
Publication (Production TG Channel)
```

### Core Components

1. **Collection Layer** (Individual workflows per source)
   - Website parsers using Browserless/HTTP requests
   - Telegram channel parsers via RapidAPI
   - Direct forwarding feature: forward any Telegram post to workspace → auto-scrapes last 10 posts from source channel
   - Scheduled triggers (8:00 & 17:00 for sites, 7:00 & 16:00 for channels)

2. **Storage Layer**
   - **NocoDB**: Structured data (titles, summaries, metadata)
   - **S3 (Cloudflare R2)**: Media files (images, videos, documents)
   - **Cloudflare Worker**: Custom proxy for S3 access with proper CORS and media handling

3. **Processing Layer**
   - **Primary AI**: Google Gemini 2.5 Flash
   - **Tasks**: Translation, summarization, headline generation, commentary creation
   - **Telegraph API**: Long-form article generation with custom API token

4. **Editorial Layer**
   - Telegram bot interface for content review
   - Interactive buttons for approval/editing
   - Natural language editing commands

5. **Publication Layer**
   - Automated posting to production channels
   - Media attachment handling
   - HTML formatting for Telegram

## 📋 Workflow Types

### Parsing Workflows
- **Sites**: `Scraping Website (example).json`
- **TG Channels**: `Scraping TG-Channel (example).json`
- **Direct Forwarding**: `TG Forwarded (example).json` - handles posts forwarded directly to workspace bot
- Each workflow handles source-specific parsing logic and data normalization

### Processing Workflows
- **`Posting (example).json`**: Main orchestrator for website news
- **`Posting TG (example).json`**: Handler for Telegram channel news
- Both support iterative AI-powered editing

### Scheduling Workflows
- **`Start 8_00 and 17_00 Sites (example).json`**: Triggers website parsers
- **`Start 7_00 and 16_00 TG Channels (example).json`**: Triggers channel parsers

## 🤖 AI Capabilities

### Content Generation
- **Headlines**: Catchy, max 10 words, attention-grabbing
- **Summaries**: Concise 2-3 sentence essence extraction
- **Commentary**: Sharp, thought-provoking insights using 5 creative angles:
  - Hidden Motive
  - Human Dimension
  - Big Picture
  - Ironic Twist
  - Future Forecast

### Content Editing
- Natural language editing commands
- Surgical text replacement
- Component-level regeneration (headline, summary, commentary, hashtags)

### Smart Features
- Automatic hyperlink insertion for company/product mentions
- URL preservation in rewrites
- Language-specific output (customizable - default Ukrainian content, English hashtags)

## 📊 Hashtag System

Predefined taxonomy covering:
- **Companies**: `#openai`, `#google`, `#microsoft`, `#meta`, `#anthropic`, etc.
- **Technologies**: `#llm`, `#genai`, `#computervision`, `#robotics`, etc.
- **Products**: `#chatgpt`, `#gemini`, `#claude`, `#sora`, etc.
- **Hardware**: `#gpu`, `#tpu`, `#blackwell`, `#hopper`, etc.
- **Business**: `#startups`, `#investments`, `#regulation`, etc.

## 🔄 Editorial Workflow Example

### Method 1: Scheduled Scraping
1. **Collection**: Parser finds new TechCrunch article
2. **Processing**: Gemini generates summary with headline, commentary, hashtags
3. **Review**: Editor receives formatted post in workspace channel
4. **Actions**:
   - ✅ **Approve**: Post immediately to production
   - 🔄 **Edit**: Type command like "переробити коментар" (rewrite commentary)
   - ❌ **Reject**: Discard post
5. **Publication**: Automated posting with Telegraph link (for website sources)

### Method 2: Direct Forwarding
1. **Forward**: Send any Telegram post to your workspace bot
2. **Auto-scraping**: System automatically fetches last 10 posts from source channel
3. **Processing**: All 10 posts go through AI processing pipeline
4. **Review**: Posts appear in workspace for approval
5. **Publication**: Approve individually to post to production channel

## 🛠️ Technical Stack

- **Automation**: n8n
- **AI Models**: Google Gemini 2.5 Flash
- **Database**: NocoDB (PostgreSQL-based)
- **Storage**: S3-compatible (Cloudflare R2)
- **Media Proxy**: Cloudflare Workers (custom deployment required)
- **APIs**: 
  - Telegram Bot API
  - Telegraph API (custom token required)
  - RapidAPI (Telegram parser)
  - Browserless (web scraping)
- **Scheduling**: n8n cron triggers

## 📁 Project Structure

```
workflows/
├── parsing/
│   ├── sites/              # Website parsers
│   ├── telegram/           # TG channel parsers
│   └── TG Forwarded.json   # Direct forward handler
├── processing/
│   ├── Posting.json        # Website news processor
│   └── Posting TG.json     # TG news processor
└── scheduling/
    ├── Start 8_00...json   # Site parsers scheduler
    └── Start 7_00...json   # TG parsers scheduler
```

## 🚀 Setup Guide

### Prerequisites
- n8n instance (self-hosted or cloud)
- Google Cloud account (Gemini API access)
- Telegram Bot Token
- NocoDB instance
- S3-compatible storage (Cloudflare R2 recommended)
- Cloudflare account (for Workers deployment)
- RapidAPI subscription (for TG parsing)

### Configuration Steps

#### 1. Import Workflows
- Import all JSON files into n8n
- Configure credentials for each service

#### 2. Database Setup
- Create NocoDB tables (Main), (Connector)
- Table Main. Required fields: Title, Published, Summary, URL, Site, media, TelegraphURL, STATUS, DocumentID
- Table Connector. Required fields: Title, Body, Image URL

#### 3. Storage & Media Proxy Configuration

**a) Set up S3 Storage:**
- Create S3 bucket (Cloudflare R2 or compatible)
- Configure bucket for private access
- Note your bucket name, access key, and secret key

**b) Deploy Cloudflare Worker for S3 Access:**

The system requires a Cloudflare Worker to proxy S3 requests with proper CORS headers and media handling. This is necessary because:
- S3 buckets are private for security
- Direct S3 URLs don't work well with Telegram/web players
- Worker handles video/image streaming with range requests
- Worker provides proper content-type headers

**To deploy your worker:**

1. Go to Cloudflare Dashboard → Workers & Pages
2. Create a new Worker
3. Write your own implementation based on these requirements:
   - Accept requests in format: `https://your-worker.workers.dev/<bucket>/<path/to/file>`
   - Implement AWS Signature Version 4 (SigV4) authentication
   - Support HTTP Range requests for video streaming
   - Set CORS headers: `Access-Control-Allow-Origin: *`
   - Handle content types: video/mp4, video/webm, image/jpeg, etc.
   - Set `Content-Disposition: inline` for browser viewing
   - Implement edge caching for performance

4. Configure environment variables:
   - `WASABI_KEY`: Your S3 access key
   - `WASABI_SECRET`: Your S3 secret key
   - `REGION_DEFAULT`: Your S3 region (e.g., `eu-central-2`)
   - `MAP_BUCKETS_TO_REGION`: JSON mapping of bucket names to regions

5. Deploy and note your worker URL (e.g., `https://your-worker.workers.dev`)

6. Update all workflow nodes that reference media URLs to use your worker:
   - Replace `https://example.workers.dev/` with your actual worker URL
   - Format: `https://your-worker.workers.dev/bucket-name/path/to/file`

**Reference implementation requirements:**
- S3 path-style requests with proper URL encoding
- RFC3986-compliant encoding for special characters
- HMAC-SHA256 signing for AWS authentication
- Support for video range requests (crucial for Telegram video playback)
- Cache-Control headers for optimal performance
- Accept-Ranges: bytes support

#### 4. Telegraph API Setup

**Create your own Telegraph API token:**

1. Visit: https://api.telegra.ph/createAccount
2. Required parameters:
   ```
   short_name: YourChannelName
   author_name: Your Name
   author_url: https://t.me/yourchannel (optional)
   ```

3. Make request (can use browser, Postman, or curl):
   ```bash
   curl -X POST https://api.telegra.ph/createAccount \
     -d 'short_name=MyNewsBot' \
     -d 'author_name=News Team' \
     -d 'author_url=https://t.me/mynewschannel'
   ```

4. Response will contain your `access_token`:
   ```json
   {
     "ok": true,
     "result": {
       "short_name": "MyNewsBot",
       "author_name": "News Team",
       "author_url": "https://t.me/mynewschannel",
       "access_token": "your_token_here",
       "auth_url": "..."
     }
   }
   ```

5. Update all Telegraph API nodes in workflows with your token:
   - Search for `Telegraph` nodes in `Posting.json` and other workflows
   - Replace the existing `access_token` parameter with your token

#### 5. Telegram Setup
- Create workspace channel (private, for editors)
- Create production channel (public, for readers)
- Configure bot with proper permissions
- Add bot to both channels as administrator
- For direct forwarding: ensure bot can receive forwarded messages

#### 6. API Keys
- Google Gemini API key
- Telegraph API token (see step 4)
- RapidAPI key for Telegram92
- Browserless token (optional, for advanced scraping)
- S3 credentials (access key, secret key)

#### 7. Language Customization

The system is currently configured for Ukrainian news. To change language:

**a) Update AI Model Prompts:**

In `Posting.json` and `Posting TG.json`, find all "Message a model" nodes and modify system prompts:

```
Change: "must be in Ukrainian" / "українською мовою"
To: "must be in [YOUR_LANGUAGE]"
```

**b) Keep hashtags in English** for international consistency

**c) Update hashtag taxonomy** if needed for your language/region:
- Modify the hashtag list in system prompts
- Keep format: `#lowercase` without spaces

**d) Adjust date/time formatting:**
- Update locale settings in Code nodes
- Example: `toLocaleString('en-US', ...)` for English dates

**e) Telegraph article language:**
- Update Telegraph API calls to use your language
- Modify article templates in workflow prompts

#### 8. Activate Schedulers
- Enable `Start 8_00 and 17_00 Sites`
- Enable `Start 7_00 and 16_00 TG Channels`
- Test direct forwarding by sending a Telegram post to your workspace bot

## 🌍 Language & Localization

### Supported Languages
The system can work with any language by modifying AI prompts. Current default: Ukrainian

### Language Configuration Checklist
- [ ] AI model system prompts (all "Message a model" nodes)
- [ ] Output language instructions
- [ ] Date/time locale formatting
- [ ] Telegraph article language
- [ ] Bot response messages
- [ ] Hashtag preferences (keep English or customize)

### Multi-language Support
To support multiple languages simultaneously:
1. Create separate workflows per language
2. Use different Telegram channels per language
3. Modify prompts to specify target language
4. Maintain separate hashtag taxonomies if needed


## 🔐 Security Notes

- All credentials stored in n8n credential system
- S3 buckets should be private - access only via signed Worker URLs
- Telegram bot tokens should have minimal required permissions
- NocoDB access tokens should be workspace-specific
- Telegraph tokens are account-specific - keep them secure
- Cloudflare Worker environment variables are encrypted

## ⚙️ Advanced Configuration

### Custom Worker Domain
- Use Cloudflare custom domains for cleaner URLs
- Configure route: `media.yourdomain.com/*`

### Rate Limiting
- RapidAPI has request limits - monitor usage
- Implement caching for frequently accessed channels
- Use n8n execution limits to prevent quota exhaustion

### Error Handling
- Workflows include error outputs - monitor workspace channel
- Failed posts send error notifications
- Retry logic built into critical nodes

## 🤝 Contributing

Contributions welcome! Areas for improvement:
- Additional source parsers
- Advanced AI model integration
- Enhanced duplicate detection
- Multi-language support improvements
- Analytics dashboard
- Custom Worker features

## 💡 Use Cases

- News aggregation services
- Content curation platforms
- Social media management
- Research monitoring
- Competitive intelligence
- Industry newsletters
- Multi-language news distribution
- Automated content translation

## 🆘 Troubleshooting

### Media not displaying
- Check Worker deployment and URL
- Verify S3 credentials in Worker environment variables
- Ensure bucket permissions allow Worker access
- Test Worker URL directly in browser

### Telegraph pages not creating
- Verify Telegraph API token is valid
- Check token hasn't been revoked
- Create new token if needed
- Ensure Telegraph API calls include correct token

### Posts in wrong language
- Review all AI model system prompts
- Check locale settings in Code nodes
- Verify language instructions in prompts
- Test with single post before batch processing

### Direct forwarding not working
- Ensure bot receives forwarded messages permission
- Check RapidAPI quota and credentials
- Verify channel username extraction logic
- Test with public channels first

---

**Note**: This system requires custom deployment of Cloudflare Worker and Telegraph API token. Sample code is provided as reference only - you must implement your own secure version. The system was designed for Ukrainian AI news curation but can be adapted for any language, topic, or content type by modifying prompts, Workers configuration, and source parsers.
