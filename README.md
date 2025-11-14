# 🗺️ Google Maps Scraper API

A **high-performance FastAPI service** for scraping Google Maps data with **parallel processing** and **comprehensive data extraction**. Ideal for n8n users and automation workflows.

[![Docker](https://img.shields.io/badge/docker-%230db7ed.svg?style=flat-square&logo=docker&logoColor=white)](https://www.docker.com/)
[![Python](https://img.shields.io/badge/python-3.10-blue.svg?style=flat-square&logo=python&logoColor=white)](https://www.python.org/)
[![FastAPI](https://img.shields.io/badge/FastAPI-005571?style=flat-square&logo=fastapi)](https://fastapi.tiangolo.com/)

---

## ✨ Key Features

- ⚡ **3x faster** with parallel processing (3 workers)
- 📊 **Two extraction modes**: Quick list or complete details
- 📞 **Full contact info**: Phone, website, address, opening hours
- 🛡️ **Anti-detection**: Random delays, realistic user-agent, WebDriver masking
- 🐳 **Docker optimized**: shm_size, single-process mode for stability
- 🌍 **Multi-language support**: en, fr, es, de, and more
- 🔄 **n8n compatible**: Ready-to-use with automation workflows

---

## 🚀 Quick Start

### Prerequisites

- Docker & Docker Compose
- 2 GB RAM minimum
- Port 8001 available

### Installation

```bash
# Clone the repository
git clone https://github.com/conor-is-my-name/google-maps-scraper.git
cd google-maps-scraper

# Start with Docker
docker compose up -d

# Test the API
curl "http://localhost:8001/health"
```

Expected response:
```json
{
  "status": "healthy",
  "service": "google-maps-scraper"
}
```

---

## 📖 API Usage

### Endpoints

- **GET `/scrape-get`** - Main endpoint for scraping (recommended for n8n)
- **POST `/scrape`** - Alternative POST endpoint with JSON body
- **GET `/health`** - Health check endpoint
- **GET `/`** - Service information

### Parameters

| Parameter | Type | Default | Required | Description |
|-----------|------|---------|----------|-------------|
| `query` | string | - | ✅ | Search query (e.g., "hotels in 98392") |
| `max_places` | int | 10 | ❌ | Maximum results (1-100) |
| `lang` | string | "en" | ❌ | Language code (en, fr, es, de, etc.) |
| `headless` | bool | true | ❌ | Run browser in headless mode |
| **`details`** | **bool** | **false** | ❌ | **Extract full details (phone, website, etc.)** |

---

## 🎯 Extraction Modes

### Quick Mode (`details=false`)

Fast extraction of basic information - **~2 seconds per place**

**Returns:**
- ✅ Name
- ✅ URL
- ✅ Rating
- ✅ Review count
- ✅ Category

```bash
curl "http://localhost:8001/scrape-get?query=restaurant%20paris&max_places=20&details=false"
```

**Use cases:** Rankings, comparisons, quick lists

---

### Full Details Mode (`details=true`)

Complete extraction including contact information - **~2-3 seconds per place**

**Returns:**
- ✅ Name, URL, Rating, Review count, Category
- ✅ **Phone number**
- ✅ **Website**
- ✅ **Full address**
- ✅ **Opening hours**

```bash
curl "http://localhost:8001/scrape-get?query=restaurant%20paris&max_places=10&details=true"
```

**Use cases:** Directory creation, CRM import, contact lists

---

## 📊 Performance

| Places | Quick Mode | Full Details | Sequential (old) |
|--------|------------|--------------|------------------|
| 5 | ~10s | ~15s | ~45s |
| 10 | ~20s | ~30s | ~90s |
| 20 | ~40s | ~60s | ~180s |

**Speedup:** 2-3x faster than sequential scraping thanks to 3 parallel workers.

---

## 💡 Example Requests

### Quick List (Basic Info)

```bash
# Get 20 restaurants with ratings
curl "http://localhost:8001/scrape-get?query=restaurant%20paris&max_places=20&details=false&lang=en"
```

**Response:**
```json
{
  "success": true,
  "query": "restaurant paris",
  "total_results": 20,
  "results": [
    {
      "name": "Le Meurice",
      "url": "https://www.google.com/maps/place/Le+Meurice/...",
      "rating": "4.8",
      "reviews_count": "1234",
      "category": "French restaurant"
    }
  ]
}
```

---

### Full Details (Complete Info)

```bash
# Get 10 restaurants with contact details
curl "http://localhost:8001/scrape-get?query=restaurant%20paris&max_places=10&details=true&lang=fr"
```

**Response:**
```json
{
  "success": true,
  "query": "restaurant paris",
  "total_results": 10,
  "results": [
    {
      "name": "Le Meurice",
      "url": "https://www.google.com/maps/place/Le+Meurice/...",
      "rating": "4.8",
      "reviews_count": "1234",
      "category": "French restaurant",
      "phone": "+33 1 44 58 10 10",
      "website": "https://www.dorchestercollection.com/paris/le-meurice",
      "address": "228 Rue de Rivoli, 75001 Paris",
      "hours": "Open · Closes 10:30 pm"
    }
  ]
}
```

---

## 🔗 n8n Integration

### HTTP Request Node Configuration

```
Method: GET
URL: http://gmaps_scraper_api_service:8001/scrape-get

Query Parameters:
  - query: {{ $json.search_query }}
  - max_places: 20
  - details: true
  - lang: en
  - headless: true
```

### Workflow Example

1. **Trigger** → Schedule or Webhook
2. **HTTP Request** → Google Maps Scraper
3. **Code** → Parse and filter results
4. **Database** → Store in PostgreSQL/MySQL
5. **Notification** → Send to Slack/Email

**Designed for:** [n8n-autoscaling](https://github.com/conor-is-my-name/n8n-autoscaling)

---

## 🐋 Docker Commands

```bash
# Start service
docker compose up -d

# View logs
docker compose logs -f

# Stop service
docker compose down

# Rebuild after changes
docker compose down
docker compose build --no-cache
docker compose up -d

# Quick restart
docker compose restart
```

---

## ⚙️ Configuration

### Docker Compose Settings

**docker-compose.yml** includes optimizations for stability:

```yaml
services:
  gmaps_scraper_api_service:
    shm_size: '2gb'          # Prevents "Page crashed" errors
    environment:
      - DISPLAY=:99          # Xvfb display
      - PYTHONUNBUFFERED=1
    volumes:
      - /dev/shm:/dev/shm    # Shared memory for Chromium
    deploy:
      resources:
        limits:
          cpus: '2'
          memory: 2G
```

### Modify Worker Count

To adjust parallel processing (default: 3 workers):

Edit `gmaps_scraper_server/scraper.py` line ~217:

```python
max_workers=3  # Increase to 5 max (higher = risk of detection)
```

⚠️ **Warning:** More than 5 workers may trigger Google's bot detection.

---

## 🛡️ Anti-Detection Features

The scraper includes multiple protections:

- ✅ **Realistic User-Agent** (Chrome 120)
- ✅ **WebDriver masking** (`navigator.webdriver = undefined`)
- ✅ **Random delays** between actions (1-3 seconds)
- ✅ **Chromium stealth arguments** (`--disable-blink-features=AutomationControlled`)
- ✅ **Limited concurrency** (3 workers max for normal behavior)

**Recommended limits:**
- Max 500 places/day per IP
- Min 60 seconds between requests
- Max 3-5 parallel workers

---

## 🔧 Local Development

### Without Docker

```bash
# Install dependencies
pip install -r requirements.txt

# Install Playwright browsers
playwright install chromium

# Run the API
uvicorn gmaps_scraper_server.main_api:app --reload --host 0.0.0.0 --port 8001
```

The API will be available at `http://localhost:8001`

### With Docker (recommended)

```bash
docker compose up --build
```

---

## 📁 Project Structure

```
google-maps-scraper/
├── docker-compose.yml          # Docker configuration
├── Dockerfile                  # Docker image build
├── requirements.txt            # Python dependencies
├── start.sh                    # Container startup script
├── gmaps_scraper_server/
│   ├── __init__.py
│   ├── main_api.py            # FastAPI endpoints
│   └── scraper.py             # Scraping logic (parallel)
└── debug_screenshots/          # Debug screenshots
```

---

## 🐛 Troubleshooting

### "Page crashed" Error

**Cause:** Insufficient shared memory

**Solution:**
```yaml
# In docker-compose.yml
shm_size: '2gb'  # Increase to 4gb if needed
```

### No Results Returned

**Possible causes:**
1. Google structure changed → Update CSS selectors
2. CAPTCHA detected → Reduce workers, wait 24h
3. Invalid query → Check URL encoding

### View Logs

```bash
docker compose logs -f
```

---

## ⚠️ Important Notes

- ⚠️ **Rate limiting:** Respect Google's terms. Max 500 places/day recommended.
- ⚠️ **Legal compliance:** Check local laws regarding web scraping.
- ⚠️ **Responsible use:** This tool is for educational and legitimate business purposes.
- ⚠️ **No guarantees:** Google may change their structure at any time.

---

## 📈 Changelog

### v2.0.0 (Latest)
- ✨ Added parallel processing (3 workers) - 3x faster
- ✨ Added full details mode (phone, website, address, hours)
- ✨ Added random delays for anti-detection
- 🔧 Fixed "Page crashed" with `--single-process`
- 🔧 Improved data extraction reliability
- 📚 Complete documentation and examples

### v1.0.0
- 🎉 Initial release with basic scraping

---

## 🤝 Contributing

Contributions are welcome! Please:

1. Fork the repository
2. Create a feature branch (`git checkout -b feature/amazing-feature`)
3. Commit your changes (`git commit -m 'Add amazing feature'`)
4. Push to the branch (`git push origin feature/amazing-feature`)
5. Open a Pull Request

---

## 📄 License

MIT License - Free to use, modify, and distribute.

---

## 🙏 Credits

Original project structure by [@conor-is-my-name](https://github.com/conor-is-my-name)

Enhancements:
- Parallel processing implementation
- Full details extraction mode
- Anti-detection improvements
- Docker optimizations
- Comprehensive documentation

---

## 📧 Support

- 🐛 **Issues:** [GitHub Issues](https://github.com/conor-is-my-name/google-maps-scraper/issues)
- 💬 **Discussions:** [GitHub Discussions](https://github.com/conor-is-my-name/google-maps-scraper/discussions)

---

**Built with ❤️ using FastAPI and Playwright**

