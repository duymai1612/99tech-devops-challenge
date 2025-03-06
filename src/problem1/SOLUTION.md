Provide your CLI command here:

```bash
grep '"symbol": "TSLA"' transaction-log.txt | jq -r '.order_id' | while read -r order_id; do curl -s "https://example.com/api/$order_id"; done > output.txt
```

Explanation of the command:
1. `grep '"symbol": "TSLA"'` - Finds all lines containing TSLA orders
2. `jq -r '.order_id'` - Extracts just the order_id field from each JSON line
3. `while read -r order_id; do ... done` - Loops through each order_id
4. `curl -s "https://example.com/api/$order_id"` - Makes a GET request for each order_id (-s for silent mode)
5. `> output.txt` - Writes all responses to output.txt
