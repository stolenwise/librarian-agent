# Librarian Agent - Configuration Reference

## Quick Config Changes

### Increase Daily Download Limit
```json
"storage": {
  "max_per_cycle": 50  // Changed from 20
}
```

### Change Library Location
```json
"storage": {
  "library_path": "/mnt/storage/Library"  // Your path
}
```

### Add Custom Category
```json
"storage": {
  "categories": [
    // ... existing categories ...
    "My Custom Category"
  ]
}
```

### Change Daily Run Time
```json
"schedules": {
  "daily_acquisition": "03:00"  // Changed from 02:00
}
```

### Disable Internet Archive (for speed)
```json
"sources": {
  "archive": {
    "enabled": false  // Disable this source
  }
}
```

### Use Different Ollama Model
```json
"processing": {
  "ollama_model": "neural-chat:7b"  // Or any other model you have
}
```

---

## Full Configuration Reference

See `config/librarian.config.json` for complete, documented configuration file.

