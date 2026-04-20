# LIBRARIAN AGENT - Autonomous Library Builder

**Version:** 1.0  
**Status:** Production-Ready  
**Created:** 2026-04-01

## Builder's Notes & Retrospective

**What is this**
I built an autonomous agent that works through the St. John's College Great Books canon — 251 works — one title at a time, queries Project Gutenberg, Internet Archive, and Open Library for a downloadable epub or pdf, deduplicates against a SHA-256 state file, uses a local mistral:7b model via Ollama to categorize the book into one of 11 folders, and posts per-book status to a Discord channel I can monitor from my phone or desktop. It runs on a Linux VM in my homelab as a systemd service and is one module inside a larger AI Agent OS I'm building. The motivation is literature preservation — I wanted a system that would quietly build an organized library of public domain books while I was asleep or at work, with the end state being files I can move to a flash drive.

**Why this approach**
I chose local Ollama + mistral:7b over calling a cloud model like Claude because the agent runs on my homelab and privacy and cost both matter — but I kept an escalation path to Claude for low-confidence cases, so the cheap local model handles volume and the expensive cloud model only gets invoked when it earns its keep. The orchestration stack (Python, SQLite, n8n, systemd, Discord webhooks) came out of building this through Claude Code, which is a different skill than how I hand-built my earlier Second Brain agent. The biggest taste decision was scope: I deliberately chose a bounded, measurable list — the canon — over "download every public domain book that exists," because a small target is the only way to actually evaluate whether the agent is doing its job. I originally capped downloads at 15/hour as a safety measure and pulled the limit once I trusted the notification layer.

**What would break**
Discord notifications have historically been the flakiest part, and n8n is the reason — it's unforgiving about misconfiguration, and a silent notifier defeats the whole point of hands-off operation. There's an orphaned cron job from the earlier rate-limited design that will keep restarting the agent hourly unless explicitly disabled — I learned this the hard way. API schema drift across Gutenberg, Internet Archive, or Open Library is a latent risk I've chosen not to defend against; if a response shape changes, the agent would need to be re-engineered. The pipeline includes format validation and in practice every downloaded book I've spot-checked has been correct, but I haven't stress-tested what happens when an API returns a malformed response, so the robustness of that validation layer isn't fully characterized. Manual spot-checking works at single-operator scale — which is what I built for — but wouldn't hold up if the system ran truly unattended at much higher volume.

**What I learned**
I learned I can actually build agents that do sustained, meaningful work unattended — that was a bet I wasn't sure would pay off before this project. Orchestrating a stack through Claude Code is a genuinely different skill than hand-assembling one node at a time the way I did for my Second Brain agent. The most concrete lesson came from a cron job I couldn't kill: I told Claude Code to stop the agent three or four times, it worked each time for an hour, and then the thing would restart. I eventually learned that "stop the running process" and "stop the thing that keeps starting the process" are two different commands, and the AI assistant was doing exactly what I asked — I was asking the wrong question. When an AI does what you asked but the problem persists, the fix is asking a different question, not repeating the same one harder. That reframe generalizes way beyond this project.


## What is the Librarian Agent?

The Librarian Agent is an **autonomous system** that continuously discovers, downloads, catalogs, and organizes **public domain books** from the internet. It operates 24/7 as a systemd service, building a comprehensive digital library with zero human intervention.

### Core Purpose

Build an organized, categorized personal library of public domain literature from multiple authoritative sources automatically.

### What It Does

✅ **Discovers books** from 3 major sources:
   - Project Gutenberg (via Gutendex API)
   - Internet Archive
   - Open Library

✅ **Downloads books automatically**
   - Respects rate limits
   - Handles retries with exponential backoff
   - Validates downloads

✅ **Catalogs books** with metadata:
   - Title, author, publication date
   - Category classification
   - Content analysis via Ollama

✅ **Organizes files** systematically:
   - Categorized directories (Fiction, Philosophy, Theology, Poetry, etc.)
   - Standardized naming: `{title}_{author}.pdf`
   - Automatic deduplication (SHA256 hashing)

✅ **Maintains state** persistently:
   - JSON state file tracks downloaded books
   - Prevents duplicates across runs
   - Handles interruptions gracefully

✅ **Runs autonomously**:
   - Systemd service for 24/7 operation
   - Scheduled tasks (daily acquisition, want-list monitoring)
   - Discord notifications on successful downloads
   - Can be triggered on-demand via CLI

---

## Key Features

### Multi-Source Discovery
- **Project Gutenberg (via Gutendex)** — 70,000+ public domain books
- **Internet Archive** — Full-text search and metadata
- **Open Library** — Comprehensive book catalog and metadata

### Intelligent Organization
- **11 built-in categories:**
  - St. John's Great Books Canon (Ancient Greek & Roman, Medieval, Renaissance, Enlightenment, 19th & 20th century)
  - Theology, Poetry, Philosophy, History, Politics
  - Reference, Unsorted

- **Automatic categorization** using pattern matching and Ollama analysis
- **Customizable organization** — Configure categories and file naming

### Deduplication & Validation
- **SHA256 hashing** to prevent duplicate downloads
- **Fuzzy matching** (85% threshold) for similar books
- **Format validation** before storing
- **Storage tracking** (500 GB limit with 80% alert threshold)

### Scheduling & Automation
- **Daily acquisition** (2 AM by default)
- **Want-list monitoring** — Searches for specific books you want
- **Manual search** — On-demand CLI commands
- **Hourly canon queue** — Dedicated workflow for important works

### Error Handling & Escalation
- **3-retry mechanism** with exponential backoff (5s, 15s, 60s)
- **Consecutive failure alerts** (alert after 3 failures)
- **Low-confidence escalation** to Claude for difficult cases
- **Format error notifications** via Discord

### 24/7 Operation
- **Systemd service** for reliable background execution
- **Persistent state** across restarts
- **Graceful shutdown** handling
- **Comprehensive logging** (file and console)

---

## Directory Structure

```
librarian-agent-portfolio/
├── README.md                          ← You are here
├── LIBRARIAN-AGENT-GUIDE.md           ← Complete setup & operation guide
├── CONFIGURATION.md                   ← Detailed configuration reference
│
├── scripts/                           ← All Python and shell scripts
│   ├── librarian-agent.py.disabled    ← Main agent (core logic)
│   ├── librarian-cli.py.disabled      ← Command-line interface
│   ├── librarian-status.py.disabled   ← Status monitoring
│   ├── librarian-validate.py.disabled ← Book validation
│   ├── librarian-canon-scheduler.py.disabled ← Canon queue management
│   ├── librarian-download-hourly.py.disabled ← Hourly download cycle
│   ├── librarian-backup.py.disabled   ← Library backup
│   ├── librarian-costs.py.disabled    ← Cost analysis
│   ├── librarian-with-discord.sh.disabled ← Discord integration wrapper
│   └── start-librarian.sh.disabled    ← Service startup script
│
├── workflows/                         ← n8n automation workflows
│   ├── librarian-daily-acquisition.json ← Main daily acquisition
│   ├── librarian-daily-acquisition-fixed.json ← Fixed version
│   ├── librarian-want-list-monitor.json ← Want-list searching
│   ├── librarian-canon-queue-hourly.json ← Hourly canon download
│   └── librarian-manual-search.json   ← On-demand search
│
├── config/                            ← Configuration files
│   ├── librarian.config.json          ← Main configuration
│   ├── librarian-canon-schedule.json  ← Canon queue schedule
│   └── .env.template                  ← Environment variables
│
├── data/                              ← State & workflow data
│   ├── librarian-state.json           ← Persistent state
│   ├── librarian-daily-acquisition-fixed.json
│   └── librarian-canon-queue-hourly.json
│
├── systemd/                           ← Service definitions
│   └── aegis-librarian.service        ← Systemd service file
│
└── logs/                              ← Log files
    ├── librarian.log                  ← Agent logs
    └── librarian-continuous.log       ← Continuous operation log
```

---

## How It Works

### 1. Discovery Phase
The agent searches all 3 sources simultaneously (respecting rate limits):
- Queries by category keywords
- Looks for books matching want-list
- Checks for new releases in canon

### 2. Validation Phase
Downloaded books are validated:
- File integrity check
- Format validation (PDF expected)
- Metadata extraction
- Duplicate detection (SHA256 hash)

### 3. Catalog Phase
Books are analyzed and cataloged:
- Title and author extraction
- Category classification via Ollama
- Metadata enrichment
- Organization in file system

### 4. Storage Phase
Books are organized:
- Directory structure: `{category}/{title}_{author}.pdf`
- State file updated with SHA256 hash
- Storage limits enforced
- Notifications sent (Discord)

### 5. Monitoring Phase
Continuous operation monitoring:
- Tracks consecutive failures
- Escalates issues appropriately
- Maintains state persistence
- Logs all operations

---

## Configuration

### Main Config (librarian.config.json)

**Sources:**
```json
{
  "sources": {
    "gutenberg": { "enabled": true, "rate_limit_ms": 1000 },
    "archive": { "enabled": true, "rate_limit_ms": 2000 },
    "openlibrary": { "enabled": true, "rate_limit_ms": 1500 }
  }
}
```

**Storage:**
```json
{
  "storage": {
    "library_path": "/home/lewis/Desktop/Private Library",
    "max_per_cycle": 20,
    "storage_limit_gb": 500,
    "categories": ["Fiction", "Philosophy", "Theology", "Poetry", ...]
  }
}
```

**Processing:**
```json
{
  "processing": {
    "ollama_host": "http://localhost:11434",
    "ollama_model": "mistral:7b",
    "max_retries": 3,
    "parallel_searches": 3
  }
}
```

**Schedules:**
```json
{
  "schedules": {
    "daily_acquisition": "02:00",
    "want_list_check": "on_create",
    "manual_search": "on_demand"
  }
}
```

See `CONFIGURATION.md` for complete reference.

---

## Getting Started

### Prerequisites
- Python 3.9+
- Ollama running locally (for metadata processing)
- systemd (for service management)
- n8n instance (for workflow automation)
- Discord webhook (for notifications)
- 500 GB storage minimum

### Step 1: Extract the Portfolio
```bash
unzip librarian-agent-portfolio.zip
cd librarian-agent-portfolio
```

### Step 2: Rename Scripts (enable them)
```bash
cd scripts
for f in *.disabled; do mv "$f" "${f%.disabled}"; done
chmod +x *.py *.sh
cd ..
```

### Step 3: Configure Environment
```bash
cp config/.env.template config/.env
# Edit config/.env with your credentials:
# - DISCORD_WEBHOOK_LIBRARIAN=your_webhook_url
# - LIBRARY_PATH=your_library_location
# - AEGIS_HOME=your_aegis_home
```

### Step 4: Review Configuration
```bash
# Edit main config
nano config/librarian.config.json

# Key settings to verify:
# - library_path (where books are stored)
# - storage_limit_gb (maximum storage)
# - categories (adjust to your preferences)
# - Ollama settings (host, model)
```

### Step 5: Start Ollama
```bash
# If not already running
ollama serve
# In another terminal: ollama run mistral:7b
```

### Step 6: Test the Agent
```bash
cd scripts
python3 librarian-cli.py --status
python3 librarian-cli.py --search "Plato"
python3 librarian-validate.py
```

### Step 7: Install Systemd Service
```bash
sudo cp systemd/aegis-librarian.service /etc/systemd/system/
sudo systemctl daemon-reload
sudo systemctl enable aegis-librarian
sudo systemctl start aegis-librarian
sudo systemctl status aegis-librarian
```

### Step 8: Import n8n Workflows
```bash
# Log into your n8n instance
# Import workflows from workflows/ directory:
# - librarian-daily-acquisition.json
# - librarian-want-list-monitor.json
# - librarian-canon-queue-hourly.json
# - librarian-manual-search.json
```

---

## CLI Commands

### Agent Status
```bash
python3 scripts/librarian-cli.py --status
```
Shows current operation status, books downloaded today, storage used.

### Search for a Book
```bash
python3 scripts/librarian-cli.py --search "Dante Divine Comedy"
python3 scripts/librarian-cli.py --search "Plato" --category Philosophy
```

### Validate Library
```bash
python3 scripts/librarian-validate.py
```
Scans library for corruption, duplicates, missing metadata.

### Add to Want-List
```bash
python3 scripts/librarian-cli.py --add-want "Cervantes Don Quixote"
```

### View Library Stats
```bash
python3 scripts/librarian-status.py
```
Storage used, books by category, total count.

### Run Backup
```bash
python3 scripts/librarian-backup.py
```
Creates backup of library and state files.

---

## API Sources

### Project Gutenberg (via Gutendex)
- **URL:** https://gutendex.com/books
- **Access:** Free, no authentication
- **Rate Limit:** 1000ms between requests
- **Books:** 70,000+ public domain
- **Preferred Format:** PDF

### Internet Archive
- **URL:** https://archive.org/advancedsearch.php
- **Access:** Free, no authentication
- **Rate Limit:** 2000ms between requests
- **Features:** Full-text search, metadata rich

### Open Library
- **URL:** https://openlibrary.org/api/search.json
- **Access:** Free, no authentication
- **Rate Limit:** 1500ms between requests
- **Features:** Comprehensive catalog, multiple formats

---

## Files Included

### Scripts (9 Python/Shell files)
- `librarian-agent.py` — Core agent logic
- `librarian-cli.py` — Command-line interface
- `librarian-status.py` — Status monitoring
- `librarian-validate.py` — Library validation
- `librarian-canon-scheduler.py` — Canon queue
- `librarian-download-hourly.py` — Hourly cycles
- `librarian-backup.py` — Backup utility
- `librarian-costs.py` — Cost analysis
- `start-librarian.sh` & `librarian-with-discord.sh` — Startup wrappers

### Workflows (5 n8n JSON files)
- `librarian-daily-acquisition.json` — Main daily download (2 AM)
- `librarian-want-list-monitor.json` — Periodic want-list checking
- `librarian-canon-queue-hourly.json` — Hourly priority queue
- `librarian-manual-search.json` — On-demand search
- Fixed version of daily acquisition workflow

### Configuration (3 files)
- `librarian.config.json` — Main configuration
- `librarian-canon-schedule.json` — Canon priority schedule
- `.env.template` — Environment variables template

### System Integration
- `aegis-librarian.service` — Systemd service file for 24/7 operation
- Complete logging infrastructure
- State persistence files

---

## Key Capabilities

### Autonomous Operation
- Runs continuously 24/7 as systemd service
- No human intervention required
- Graceful handling of interruptions
- Persistent state across restarts

### Intelligent Discovery
- Multi-source searching (3 major APIs)
- Parallel search capability (3 simultaneous)
- Smart category matching
- Want-list awareness

### Quality Control
- Deduplication (SHA256 hashing)
- Fuzzy matching (85% threshold)
- Format validation
- Metadata confidence checking (70% minimum)

### Scaling & Limits
- 500 GB storage limit
- 80% storage alert threshold
- 20 books max per cycle
- 3 retry mechanism
- Exponential backoff (5s, 15s, 60s)

### Monitoring & Alerts
- Comprehensive logging
- Discord notifications
- Escalation to Claude on low confidence
- Alert on 3 consecutive failures
- Status monitoring CLI

---

## Best Practices

1. **Configure categories** to match your reading interests
2. **Set storage limit** appropriate for your disk space
3. **Monitor logs** regularly (`logs/librarian.log`)
4. **Backup your library** weekly using `librarian-backup.py`
5. **Update want-list** with books you're seeking
6. **Review validation reports** monthly
7. **Monitor storage usage** (80% alert threshold)

---

## Troubleshooting

### Agent Not Running
```bash
# Check systemd status
sudo systemctl status aegis-librarian

# View logs
sudo journalctl -u aegis-librarian -f

# Manual start for debugging
cd scripts
python3 librarian-agent.py
```

### Ollama Connection Issues
```bash
# Verify Ollama is running
curl http://localhost:11434/api/models

# Restart Ollama if needed
killall ollama
ollama serve
```

### Storage Full
```bash
# Check storage usage
python3 scripts/librarian-status.py

# View size by category
du -sh config/storage/*

# Increase storage_limit_gb in config
```

### Duplicate Books
```bash
# Validate and clean duplicates
python3 scripts/librarian-validate.py --fix-duplicates
```

---

## Performance

- **Discovery:** 1-2 books per request (multiple parallel searches)
- **Download:** 5-10 MB average per book
- **Catalog:** ~2-5 seconds per book (Ollama analysis)
- **Cycle time:** 1-2 hours for 20 books (respects rate limits)
- **CPU usage:** Low (Ollama is main consumer)
- **Network:** Minimal (respects rate limits)

---

## License & Attribution

Built as part of the AEGIS Agentic Software Factory initiative.

**Agent:** Librarian  
**Version:** 1.0  
**Date:** 2026-04-01  
**Creator:** Lewis Stone

---

## What's Next

1. **Extract portfolio** — `unzip librarian-agent-portfolio.zip`
2. **Read LIBRARIAN-AGENT-GUIDE.md** — Complete setup instructions
3. **Enable scripts** — Rename `.disabled` files
4. **Configure environment** — Set up `.env` file
5. **Start Ollama** — For metadata processing
6. **Test CLI** — Verify agent is working
7. **Install service** — For 24/7 operation
8. **Import workflows** — Set up n8n automation
9. **Monitor operation** — Check logs and Discord
10. **Build your library** — Watch it grow autonomously!

---

**Portfolio Version:** 1.0  
**Status:** Production-Ready  
**Ready to Deploy:** Yes
