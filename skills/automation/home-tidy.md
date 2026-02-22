# Home Tidy - Directory Organization Command

Organize loose files in the home directory based on learned patterns and rules.

## Instructions

### 1. Load Organization Patterns
First retrieve learned patterns from memory bridge:
```bash
curl -s "http://localhost:8115/retrieve?namespace=home_organization&key=patterns" | jq -r '
  if .data.value then
    "=== ORGANIZATION PATTERNS ===",
    (.data.value | to_entries[] | "  \(.key): \(.value)")
  else
    "No custom patterns found (using defaults)"
  end
'
```

### 2. Assess Current State
Count loose files in home directory:
```bash
echo "=== HOME DIRECTORY STATE ==="
echo "Loose files (excluding hidden):"
ls -p ~ 2>/dev/null | grep -v "/" | grep -v "CLAUDE.md" | wc -l

echo ""
echo "By type:"
echo "  Images: $(find ~ -maxdepth 1 -type f \( -name '*.jpg' -o -name '*.png' -o -name '*.webp' -o -name '*.gif' \) 2>/dev/null | wc -l)"
echo "  Documents: $(find ~ -maxdepth 1 -type f \( -name '*.pdf' -o -name '*.doc*' -o -name '*.txt' \) 2>/dev/null | wc -l)"
echo "  Logs: $(find ~ -maxdepth 1 -type f -name '*.log' 2>/dev/null | wc -l)"
echo "  Archives: $(find ~ -maxdepth 1 -type f \( -name '*.zip' -o -name '*.tar*' -o -name '*.gz' \) 2>/dev/null | wc -l)"
echo "  Downloads: $(find ~ -maxdepth 1 -type f -name '*.crdownload' 2>/dev/null | wc -l)"
echo "  Other: $(ls -p ~ | grep -v "/" | grep -v "CLAUDE.md" | grep -v -E '\.(jpg|png|webp|gif|pdf|doc|txt|log|zip|tar|gz|crdownload)$' | wc -l)"
```

### 3. Run Standard Cleanup
Execute the existing home-tidy script:
```bash
~/bin/home-tidy
```

### 4. Report Summary
Show what was done and remaining items:
```bash
echo ""
echo "=== AFTER CLEANUP ==="
echo "Remaining loose files:"
ls -p ~ | grep -v "/" | grep -v "CLAUDE.md"
echo ""
echo "Total: $(ls -p ~ | grep -v "/" | grep -v "CLAUDE.md" | wc -l) files"
```

### 5. Store Session Record
Log this cleanup session to memory bridge:
```bash
curl -s -X POST http://localhost:8115/store -H "Content-Type: application/json" -d "{
  \"namespace\": \"home_organization\",
  \"key\": \"session_$(date +%Y%m%d_%H%M%S)\",
  \"value\": {
    \"timestamp\": \"$(date -Iseconds)\",
    \"type\": \"manual_tidy\",
    \"remaining_files\": $(ls -p ~ | grep -v "/" | grep -v "CLAUDE.md" | wc -l)
  }
}" >/dev/null && echo "Session logged to memory bridge"
```

## Pattern Learning

If you notice files that should be moved differently:

1. **Suggest a new pattern** to the user
2. **If approved**, store it:
```bash
# Example: Store pattern for .deb files going to ~/Downloads/installers/
curl -s -X POST http://localhost:8115/store -H "Content-Type: application/json" -d '{
  "namespace": "home_organization",
  "key": "patterns",
  "value": {
    "*.deb": "~/Downloads/installers/",
    "*.AppImage": "~/Applications/",
    "Screenshot*.png": "~/Pictures/Screenshots/"
  }
}'
```

## Manual Organization

For files requiring judgment or elevated permissions:
- Use `pkexec-move --dry-run SOURCE DEST` to preview
- Use `pkexec-move SOURCE DEST` to execute with PolicyKit elevation if needed

## Expected Outcome
- Images older than 7 days moved to ~/Pictures/
- Logs older than 7 days moved to ~/Backups/
- Failed downloads (empty .crdownload) deleted
- Session recorded in memory bridge for pattern learning
