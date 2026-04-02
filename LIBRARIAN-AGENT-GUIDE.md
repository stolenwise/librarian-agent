# LIBRARIAN AGENT - Complete Setup & Operation Guide

**Date:** 2026-04-01  
**Version:** 1.0  
**Status:** Production-Ready

## Table of Contents
1. What is the Librarian Agent?
2. System Architecture
3. Installation & Setup
4. Configuration
5. Operations & Monitoring
6. Troubleshooting
7. Advanced Usage

---

## 1. What is the Librarian Agent?

The **Librarian Agent** is an autonomous system that:

- **Discovers** public domain books from 3 major sources
- **Downloads** books automatically 24/7
- **Catalogs** books with metadata (title, author, category)
- **Organizes** files into categorized directories
- **Maintains** persistent state across restarts
- **Notifies** you on Discord when books are added

### Real-World Example

You wake up and have 47 new books in your library:
- Plato's Republic (Philosophy → Classical Greek)
- Dante's Divine Comedy (Fiction → Medieval)
- Shakespeare's Sonnets (Fiction → Renaissance)
- Newton's Principia Mathematica (Philosophy → Enlightenment Science)

All automatically discovered, downloaded, categorized, and organized while you slept.

### Why?

Building a comprehensive personal library of public domain literature is:
- **Tedious** — Manually searching multiple sources
- **Time-consuming** — Finding, downloading, organizing books
- **Error-prone** — Duplicate downloads, missing metadata
- **Never-ending** — New books constantly being digitized

The Librarian Agent automates this entirely.

---

## 2. System Architecture

### Components

```
┌─────────────────────────────────────────────────┐
│  DISCOVERY LAYER                                │
│  • Project Gutenberg (via Gutendex API)         │
│  • Internet Archive                             │
│  • Open Library                                 │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│  PROCESSING LAYER                               │
│  • Fetch & Validate                             │
│  • Metadata Extraction                          │
│  • Ollama Analysis (categorization)             │
│  • Deduplication (SHA256)                       │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│  STORAGE LAYER                                  │
│  • Organized directories                        │
│  • State persistence                            │
│  • Logging                                      │
│  • Backup capability                            │
└─────────────────────────────────────────────────┘
                        ↓
┌─────────────────────────────────────────────────┐
│  AUTOMATION LAYER                               │
│  • systemd service (24/7 operation)             │
│  • n8n workflows (scheduling)                   │
│  • CLI commands (manual control)                │
│  • Discord notifications                        │
└─────────────────────────────────────────────────┘
```

### Data Flow

```
User (via CLI/n8n)
    ↓
Librarian Agent
    ↓
┌─────────────────────────────────────────────┐
│ Search Phase: Discover books from 3 sources │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ Validate Phase: Check integrity & metadata  │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ Process Phase: Ollama categorization        │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ Dedup Phase: Check SHA256 hash              │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ Store Phase: Save to library                │
└─────────────────────────────────────────────┘
    ↓
┌─────────────────────────────────────────────┐
│ Notify Phase: Send Discord message          │
└─────────────────────────────────────────────┘
    ↓
Library Updated + State Persisted
```

---

## 3. Installation & Setup

### Prerequisites Checklist

- [ ] Python 3.9 or higher
- [ ] 500 GB free disk space
- [ ] Ollama installed and running
- [ ] systemd available (Linux)
- [ ] n8n instance running (optional, for automation)
- [ ] Discord webhook URL (optional, for notifications)

### Step-by-Step Installation

#### Step 1: Extract Portfolio
```bash
unzip librarian-agent-portfolio.zip
cd librarian-agent-portfolio
```

#### Step 2: Verify Files
```bash
# Check directory structure
ls -la
# Should show: scripts/, workflows/, config/, data/, systemd/, logs/, README.md, etc.

# Check all scripts are present
ls scripts/ | wc -l
# Should show: 10 files (.disabled files)
```

#### Step 3: Enable Scripts
Scripts are disabled by default (`.disabled` extension). Enable them:
```bash
cd scripts
for f in *.disabled; do mv "$f" "${f%.disabled}"; done
chmod +x *.py *.sh
cd ..

# Verify
ls scripts/ | grep -v "\.disabled" | wc -l
# Should show: 10 files (now without .disabled extension)
```

#### Step 4: Create Environment File
```bash
# Copy template
cp config/.env.template config/.env

# Edit with your settings
nano config/.env
```

**Required variables:**
```bash
DISCORD_WEBHOOK_LIBRARIAN=https://discord.com/api/webhooks/xxx/yyy
LIBRARY_PATH=/path/to/your/library
AEGIS_HOME=/home/lewis/aegis
OLLAMA_HOST=http://localhost:11434
```

#### Step 5: Verify Configuration
```bash
# Check main config
cat config/librarian.config.json | python3 -m json.tool

# Should show valid JSON with no errors
```

#### Step 6: Start Ollama
```bash
# Terminal 1: Start Ollama server
ollama serve

# Terminal 2: Pull the model
ollama pull mistral:7b

# Terminal 3: Verify it works
curl http://localhost:11434/api/models
```

#### Step 7: Test Agent Manually
```bash
cd scripts

# Check status
python3 librarian-cli.py --status

# Should output current library status

# Search for a book
python3 librarian-cli.py --search "Plato"

# Should find and display Plato works
```

#### Step 8: Create Library Directory
```bash
mkdir -p ~/Desktop/"Private Library"
chmod 755 ~/Desktop/"Private Library"

# Create category subdirectories (optional, auto-created on first run)
mkdir -p ~/Desktop/"Private Library"/{Fiction,Philosophy,Theology,Poetry,Drama,Essays,History,Reference}
```

#### Step 9: Install systemd Service
```bash
# Copy service file
sudo cp systemd/aegis-librarian.service /etc/systemd/system/

# Reload systemd
sudo systemctl daemon-reload

# Enable service (starts on boot)
sudo systemctl enable aegis-librarian

# Start service
sudo systemctl start aegis-librarian

# Check status
sudo systemctl status aegis-librarian

# View logs
sudo journalctl -u aegis-librarian -f
```

#### Step 10: Import n8n Workflows (Optional)
```bash
# Log into your n8n instance
# Navigate to: Settings → Import

# Import these workflows:
# 1. workflows/librarian-daily-acquisition.json
# 2. workflows/librarian-want-list-monitor.json
# 3. workflows/librarian-canon-queue-hourly.json
# 4. workflows/librarian-manual-search.json

# Each workflow will be available in your n8n dashboard
```

---

## 4. Configuration

### Main Configuration (config/librarian.config.json)

#### Sources Configuration
```json
{
  "sources": {
    "gutenberg": {
      "enabled": true,
      "api_url": "https://gutendex.com/books",
      "rate_limit_ms": 1000,
      "timeout_sec": 30,
      "preferred_format": "pdf"
    },
    "archive": {
      "enabled": true,
      "api_url": "https://archive.org/advancedsearch.php",
      "rate_limit_ms": 2000,
      "timeout_sec": 30
    },
    "openlibrary": {
      "enabled": true,
      "api_url": "https://openlibrary.org/api/search.json",
      "rate_limit_ms": 1500,
      "timeout_sec": 30
    }
  }
}
```

**Options:**
- `enabled` — Include this source in searches (true/false)
- `rate_limit_ms` — Milliseconds to wait between requests (respect their TOS)
- `timeout_sec` — Seconds to wait for response
- `preferred_format` — For Gutenberg: pdf, html, txt

#### Processing Configuration
```json
{
  "processing": {
    "ollama_host": "http://localhost:11434",
    "ollama_model": "mistral:7b",
    "metadata_confidence_threshold": 0.7,
    "max_retries": 3,
    "retry_backoff_seconds": [5, 15, 60],
    "parallel_searches": 3
  }
}
```

**Options:**
- `ollama_host` — Local Ollama server address
- `ollama_model` — Model to use (must be pulled: `ollama pull mistral:7b`)
- `metadata_confidence_threshold` — 0.0-1.0, escalate if below
- `max_retries` — How many times to retry failed downloads
- `retry_backoff_seconds` — Exponential backoff times [1st, 2nd, 3rd]
- `parallel_searches` — Simultaneous searches (1-5 recommended)

#### Storage Configuration
```json
{
  "storage": {
    "library_path": "/home/lewis/Desktop/Private Library",
    "categories": [
      "Fiction", "Philosophy", "Politics", "Theology",
      "Poetry", "Plays", "Media Ecology", "Science",
      "History", "Reference", "Unsorted"
    ],
    "file_format": "pdf",
    "max_per_cycle": 20,
    "storage_limit_gb": 500,
    "storage_alert_threshold": 0.8,
    "file_organization": "{category}/{title}_{author}.pdf"
  }
}
```

**Options:**
- `library_path` — Where books are stored
- `categories` — Customize your categories
- `file_format` — Expected format (pdf, html, etc.)
- `max_per_cycle` — Max books per day (1-100)
- `storage_limit_gb` — Total space allocated
- `storage_alert_threshold` — 0.8 = alert at 80% full
- `file_organization` — Directory structure and naming

#### Deduplication Configuration
```json
{
  "deduplication": {
    "method": "sha256_hash",
    "also_fuzzy_match": true,
    "fuzzy_match_threshold": 0.85
  }
}
```

**Options:**
- `method` — "sha256_hash" (default)
- `also_fuzzy_match` — Check for similar books (slow but catches variations)
- `fuzzy_match_threshold` — 0.0-1.0, similarity required to mark as duplicate

#### Scheduling Configuration
```json
{
  "schedules": {
    "daily_acquisition": "02:00",
    "want_list_check": "on_create",
    "manual_search": "on_demand"
  }
}
```

**Options:**
- `daily_acquisition` — Time for main daily run (24-hour format)
- `want_list_check` — "on_create", "every_6h", "every_12h"
- `manual_search` — Triggered by CLI/API

---

## 5. Operations & Monitoring

### Command-Line Interface

#### Check Status
```bash
python3 scripts/librarian-cli.py --status
```

Output:
```
LIBRARIAN STATUS
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Agent Status: Running
Last Cycle: 2026-04-01 02:15:32
Books Downloaded Today: 12
Books This Week: 47
Total Books: 3,421

Library Stats:
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Total Storage Used: 247 GB / 500 GB (49%)
Fiction: 1,200 books (98 GB)
Philosophy: 450 books (35 GB)
Theology: 380 books (28 GB)
Poetry: 280 books (15 GB)
Reference: 711 books (71 GB)
Unsorted: 0 books (0 GB)
```

#### Search for a Book
```bash
# Simple search
python3 scripts/librarian-cli.py --search "Plato"

# Search in specific category
python3 scripts/librarian-cli.py --search "Aquinas" --category Theology

# Advanced search
python3 scripts/librarian-cli.py --search "Dante" --author "Dante Alighieri"
```

#### Add to Want-List
```bash
python3 scripts/librarian-cli.py --add-want "Cervantes Don Quixote"

# Agent will prioritize searching for this book
```

#### Validate Library
```bash
# Check for issues
python3 scripts/librarian-validate.py

# Output shows:
# - Corruption detected
# - Duplicate files
# - Missing metadata
# - File integrity issues

# Fix issues
python3 scripts/librarian-validate.py --fix-corrupted
python3 scripts/librarian-validate.py --remove-duplicates
```

#### View Statistics
```bash
python3 scripts/librarian-status.py

# Shows:
# - Books per category
# - Storage usage breakdown
# - Download statistics
# - Failure rates
# - Last 10 downloads
```

#### Run Backup
```bash
python3 scripts/librarian-backup.py

# Creates:
# - Backup of entire library
# - State file backup
# - Config backup
# - Timestamped archive
```

### Systemd Service Management

#### Check Service Status
```bash
sudo systemctl status aegis-librarian
```

#### View Live Logs
```bash
sudo journalctl -u aegis-librarian -f

# Follow logs in real-time
```

#### View Last N Hours of Logs
```bash
sudo journalctl -u aegis-librarian --since "2 hours ago"
```

#### Stop Service (for maintenance)
```bash
sudo systemctl stop aegis-librarian
# Do maintenance
sudo systemctl start aegis-librarian
```

#### Restart Service
```bash
sudo systemctl restart aegis-librarian
```

#### Check if Running on Boot
```bash
sudo systemctl is-enabled aegis-librarian
# Output: enabled (if starts on boot)
```

### Monitoring via Discord

The agent sends Discord notifications for:

- **Book added:** "{Title} by {Author} → {Category}"
- **Duplicate prevented:** "Skipped duplicate of {Title}"
- **Failure alert:** "3 consecutive failures detected"
- **Storage warning:** "Library at 80% capacity (400/500 GB)"
- **Agent error:** "Escalation to Claude recommended"

Set `DISCORD_WEBHOOK_LIBRARIAN` in `.env` to enable notifications.

---

## 6. Troubleshooting

### Agent Won't Start
```bash
# Check service status
sudo systemctl status aegis-librarian

# View error logs
sudo journalctl -u aegis-librarian -n 50

# Try manual start for debugging
cd scripts
python3 librarian-agent.py
# Look for error messages
```

### Ollama Connection Error
```bash
# Check if Ollama is running
curl http://localhost:11434/api/models

# If fails, restart Ollama
killall ollama
ollama serve &

# Pull required model
ollama pull mistral:7b
```

### Books Not Downloading
```bash
# Check source API availability
curl https://gutendex.com/books | head

# Check rate limits aren't being hit
grep "rate_limit" config/librarian.config.json

# Try manual search
python3 scripts/librarian-cli.py --search "test author"

# Check logs for specific errors
grep ERROR logs/librarian.log | tail -20
```

### Storage Full Error
```bash
# Check usage
python3 scripts/librarian-status.py

# Increase limit in config
nano config/librarian.config.json
# Change: "storage_limit_gb": 500 → higher value

# Or clean up categories
du -sh ~/Desktop/"Private Library"/* | sort -rh
# Delete least valuable category
```

### Duplicate Detection Not Working
```bash
# Check deduplication config
grep -A5 "deduplication" config/librarian.config.json

# Enable fuzzy matching
nano config/librarian.config.json
# Set: "also_fuzzy_match": true

# Run validation to clean existing duplicates
python3 scripts/librarian-validate.py --remove-duplicates
```

---

## 7. Advanced Usage

### Custom Categories

Edit `config/librarian.config.json`:

```json
{
  "storage": {
    "categories": [
      "Fiction",
      "Philosophy",
      "My Custom Category",
      "Another Category"
    ]
  }
}
```

Directories will be created automatically on first book in that category.

### Custom File Organization

Change naming pattern:

```json
{
  "storage": {
    "file_organization": "{author}/{title}.pdf"
    // Now: Author Name/Book Title.pdf
  }
}
```

Or:

```json
{
  "storage": {
    "file_organization": "{category}/{author}/{title}.pdf"
    // Now: Philosophy/Plato/Republic.pdf
  }
}
```

### Disable Specific Sources

If a source is causing issues:

```json
{
  "sources": {
    "archive": {
      "enabled": false  // Skip this source
    }
  }
}
```

### Adjust Retry Strategy

```json
{
  "processing": {
    "max_retries": 5,  // More retries
    "retry_backoff_seconds": [1, 2, 5, 10, 30]
  }
}
```

### Run Manual Cycle

Force a download cycle outside schedule:

```bash
python3 scripts/librarian-agent.py --run-now
```

### Export Library Metadata

```bash
python3 scripts/librarian-status.py --export-csv > library.csv
# CSV with all books, categories, file sizes
```

### Migrate Library to New Location

```bash
# 1. Copy entire library
cp -r ~/Desktop/"Private Library" /new/location/"Private Library"

# 2. Update config
nano config/librarian.config.json
# Change: "library_path": "/new/location/Private Library"

# 3. Restart service
sudo systemctl restart aegis-librarian
```

---

## Performance Tuning

### For Faster Downloads
```json
{
  "processing": {
    "parallel_searches": 5,      // More parallel (if your internet is fast)
    "max_retries": 2              // Fewer retries
  },
  "sources": {
    "gutenberg": {"rate_limit_ms": 500},  // Faster (respect TOS)
    "archive": {"rate_limit_ms": 1000},
    "openlibrary": {"rate_limit_ms": 750}
  }
}
```

### For Lower CPU Usage
```json
{
  "processing": {
    "parallel_searches": 1,       // Sequential searches
    "ollama_model": "phi:2.2b"   // Lighter model (if available)
  },
  "storage": {
    "max_per_cycle": 5            // Fewer books per cycle
  }
}
```

### For Reliability
```json
{
  "processing": {
    "max_retries": 5,             // More retries
    "retry_backoff_seconds": [10, 30, 60, 120, 300]
  },
  "sources": {
    "gutenberg": {"rate_limit_ms": 2000},  // Slower (safer)
    "archive": {"rate_limit_ms": 3000}
  }
}
```

---

## Next Steps

1. ✅ Extract portfolio
2. ✅ Enable scripts
3. ✅ Configure environment
4. ✅ Set up Ollama
5. ✅ Test CLI commands
6. ✅ Install systemd service
7. ✅ Import n8n workflows
8. ✅ Monitor first cycle
9. → Start building your library!

The Librarian Agent is now running 24/7, autonomously building your library.

---

**Questions?** See README.md for quick reference.  
**Issues?** Check "Troubleshooting" section above.  
**Advanced setup?** See "Advanced Usage" section.

---

**Librarian Agent v1.0**  
**Built for:** Autonomous library building  
**Maintained:** Actively  
**Status:** Production-Ready
