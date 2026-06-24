# 🔥 Burp Suite Automation Guide

Complete guide to automating Burp Suite using MCP (Model Context Protocol) with Claude Code and other AI coding assistants.

---

## 🎯 What You Can Do

Once configured, you can tell Claude things like:

- "Find all POST requests in Burp history that contain 'login'"
- "Brute force the verification code from 000000 to 999999"
- "Scan proxy history for SQL injection patterns"
- "Send this request to Repeater and test with different payloads"
- "Generate a CSRF PoC for the last captured request"
- "Start a Collaborator session and poll for DNS callbacks"

Claude will execute these **automatically** without manual clicking in Burp!

---

## 📋 Step-by-Step Setup

### Step 1: Build the Burp Extension

```bash
# Clone the repository
git clone https://github.com/Darkhearted007/reverse-skill.git
cd reverse-skill/burp-mcp-full
```

**Windows:**
```cmd
# Edit build.bat and set your JDK path
notepad build.bat
# Change line 4: set JAVA_HOME=C:\Program Files\Java\jdk-17

# Build
build.bat
```

**Linux/macOS:**
```bash
# Build
chmod +x build.sh
./build.sh
```

**Output:** `build/libs/burp-mcp-full.jar` ✅

---

### Step 2: Load Extension in Burp Suite

1. **Open Burp Suite**
2. Go to **Extensions** tab
3. Click **Add**
4. **Extension Type:** Java
5. Click **Select file...** → Choose `build/libs/burp-mcp-full.jar`
6. Click **Next**

**Verify Success:**
- In the **Extensions** tab → **Output** subtab
- You should see: `[MCP] Server started on http://127.0.0.1:9876`

---

### Step 3: Configure Claude Code

Pull the latest config from your repo:

```bash
cd ~/reverse-skill
git pull origin main
```

Copy MCP config to Claude's global directory:

```bash
# macOS/Linux
mkdir -p ~/.claude
cp .claude/mcp.json ~/.claude/mcp.json

# Windows
mkdir %USERPROFILE%\.claude 2>nul
copy .claude\mcp.json %USERPROFILE%\.claude\mcp.json
```

**Edit the path** in `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "burpsuite": {
      "command": "node",
      "args": ["/absolute/path/to/reverse-skill/burp-mcp-full/mcp-bridge.js"],
      "env": {},
      "disabled": false
    }
  }
}
```

Replace `/absolute/path/to/reverse-skill` with your actual directory!

---

### Step 4: Restart Claude Code

Close and reopen Claude Code to load the new MCP configuration.

---

## ✅ Verification Tests

### Test 1: Check Connection

In Claude Code (from **any directory**):

```
Show me what Burp MCP tools are available
```

**Expected Response:**
Claude lists all 63 tools including:
- `burp_proxy_history`
- `burp_intruder_attack`
- `burp_send_request`
- `burp_scan_active`
- etc.

### Test 2: View Proxy History

First, browse some websites with Burp proxy enabled, then ask:

```
Show me the last 10 requests from Burp proxy history
```

**Expected:** Claude calls `burp_proxy_history` and displays the captured requests.

### Test 3: Send a Test Request

```
Use Burp to send a GET request to https://httpbin.org/get
```

**Expected:** Claude calls `burp_send_request` and shows the response.

---

## 🚀 Advanced Automation Examples

### Example 1: Automated Brute Force Attack

**Prompt:**
```
Use Burp Intruder to brute force verification codes on:
https://target.com/api/verify?code=@@

Range: 000000 to 999999
Pad with leading zeros to 6 digits
Filter results where response length != 176 bytes
```

**What Claude Does:**
1. Calls `burp_intruder_attack` with the parameters
2. Tests all 1 million combinations
3. Returns only successful responses

### Example 2: SQL Injection Scanner

**Prompt:**
```
Analyze Burp proxy history and find all POST requests with user input.
For each request, send it to Burp Scanner and check for SQL injection.
Generate a report of vulnerable endpoints.
```

**Claude Workflow:**
1. `burp_proxy_history` → filter POST requests
2. `burp_scan_active` → scan each request
3. `burp_scan_results` → collect vulnerabilities
4. Generate markdown report

### Example 3: Collaborator-based SSRF Detection

**Prompt:**
```
Generate 5 Burp Collaborator payloads.
Inject them into the 'url' parameter of the last captured request.
Poll for DNS/HTTP callbacks every 5 seconds for 1 minute.
Report any successful out-of-band interactions.
```

**Claude Workflow:**
1. `burp_collaborator_generate` → get payloads
2. `burp_send_request` → inject into target
3. `burp_collaborator_poll` → check for callbacks
4. Report SSRF vulnerability if detected

### Example 4: Automated CSRF PoC Generation

**Prompt:**
```
Take request #5 from proxy history and generate a CSRF proof-of-concept HTML page.
Save it to csrf-poc.html.
```

**Claude Workflow:**
1. `burp_proxy_detail` → get request #5
2. `burp_generate_csrf_poc` → create HTML
3. Write to file system

---

## 🛠️ Complete Tool Reference (63 Tools)

### Core HTTP Operations (10 tools)
- `burp_proxy_history` - View captured traffic
- `burp_send_request` - Send arbitrary requests
- `burp_send_to_repeater` - Manual testing in Repeater
- `burp_repeater_send` - Automated request replay
- `burp_repeater_modify_send` - Modify & send requests
- `burp_sitemap` - Browse discovered endpoints
- `burp_search_history` - Regex search in traffic
- `burp_compare` - Diff two responses
- `burp_export_request` - Generate curl/Python code
- `burp_convert_request` - Change HTTP method

### Intruder Attacks (8 tools)
- `burp_intruder_attack` - Numeric range brute force
- `burp_intruder_attack_async` - Multi-threaded attack
- `burp_intruder_attack_wordlist` - Dictionary attack
- `burp_intruder_pitchfork` - Parallel multi-param
- `burp_intruder_cluster_bomb` - Cartesian product
- `burp_intruder_battering_ram` - Same payload everywhere
- `burp_intruder_with_options` - Advanced options (throttle, encoding)
- `burp_send_to_intruder` - Prepare for manual Intruder

### Scanner & Crawler (6 tools)
- `burp_scan` - Passive vulnerability scan
- `burp_scan_active` - Active scanning
- `burp_scan_results` - Get discovered issues
- `burp_scan_issue_detail` - Vulnerability details
- `burp_crawl` - Start spider/crawler
- `burp_add_issue` - Manually log vulnerability

### Scope Management (3 tools)
- `burp_get_scope` - Check if URL in scope
- `burp_add_to_scope` - Add target to scope
- `burp_remove_from_scope` - Remove from scope

### Collaborator (SSRF/XXE/OOB) (2 tools)
- `burp_collaborator_generate` - Get Collaborator payloads
- `burp_collaborator_poll` - Check for callbacks

### Encoding & Utilities (8 tools)
- `burp_encode` - Base64/URL/Hex encode
- `burp_decode` - Base64/URL decode
- `burp_payload_process` - Hash/transform payloads
- `burp_extract_from_response` - Regex extraction
- `burp_token_analysis` - Analyze randomness
- `burp_sequencer` - Token entropy analysis
- `burp_cookie_jar` - View cookies
- `burp_generate_csrf_poc` - Create CSRF PoC

### Proxy Control (5 tools)
- `burp_intercept_toggle` - Enable/disable intercept
- `burp_highlight` - Color-code requests
- `burp_annotate` - Add notes
- `burp_proxy_listeners` - View proxy config
- `burp_proxy_match_replace` - Auto-modify rules

### Configuration (7 tools)
- `burp_export_config` - Export settings
- `burp_import_config` - Import settings
- `burp_set_upstream_proxy` - SOCKS/HTTP proxy
- `burp_set_dns_override` - Custom DNS
- `burp_set_http2` - HTTP/2 toggle
- `burp_export_cert` - CA certificate
- `burp_save_project` - Save state

### Extensions & Logging (4 tools)
- `burp_extensions_list` - List loaded extensions
- `burp_log` - Write to extension output
- `burp_register_http_handler` - Auto-modify HTTP
- `burp_remove_http_handler` - Clear handlers

### WebSocket & Advanced (10 tools)
- `burp_proxy_websocket` - WebSocket message history
- `burp_websocket_send` - Send WebSocket messages
- `burp_proxy_clear` - Clear proxy history
- `burp_proxy_history_filtered` - Filter by color/notes
- `burp_proxy_detail` - Get full request/response
- `burp_intercept_modify` - Intercept guidance
- `burp_target_info` - Target information
- `burp_burp_version` - Burp Suite version
- `burp_register_proxy_rule` - Auto-intercept rules
- `burp_remove_proxy_rule` - Clear intercept rules

---

## 🎬 Demo Workflow: End-to-End Automation

### Full SQL Injection Discovery & Exploitation

**Step 1: Capture Traffic**
```
Show me all POST requests from proxy history that go to /api/
```

**Step 2: Identify Injection Points**
```
For each request, identify parameters that accept user input
```

**Step 3: Test for SQLi**
```
Send each request to Burp Scanner with focus on SQL injection checks
```

**Step 4: Brute Force Exploitation**
```
If vulnerable parameter found, use Intruder to extract database name character by character using:
?id=1' AND SUBSTRING(database(),{pos},1)='{char}'--
```

**Step 5: Generate Report**
```
Create a markdown report with:
- Vulnerable endpoint
- Parameter name
- Payload that worked
- Extracted data
- Remediation advice
```

**Step 6: Export Evidence**
```
Save the successful request as curl command for reproduction
```

Claude will execute **all 6 steps automatically** using Burp MCP tools!

---

## 🐛 Troubleshooting

### "Cannot connect to Burp at 127.0.0.1:9876"

**Cause:** Burp extension not loaded or not started.

**Fix:**
1. Verify Burp extension is loaded: **Extensions** → Look for "BurpMCP"
2. Check extension output: Should show `[MCP] Server started on http://127.0.0.1:9876`
3. Test manually: `curl http://127.0.0.1:9876/tools`
4. If fails, reload extension or restart Burp

### "burp_* tools not showing in Claude"

**Cause:** MCP configuration not loaded or path incorrect.

**Fix:**
1. Verify `mcp.json` exists: `cat ~/.claude/mcp.json`
2. Check absolute path to `mcp-bridge.js` (no `~` or relative paths)
3. Test bridge manually: `node /path/to/mcp-bridge.js`
4. Restart Claude Code
5. Check Claude logs for MCP errors

### "Tool call failed" errors

**Cause:** Burp frozen, proxy not running, or request timeout.

**Fix:**
1. Check Burp is responsive (try clicking in Burp UI)
2. Verify proxy is running: **Proxy** → **Intercept** tab
3. Check extension output for Java errors
4. If timeout issues, increase timeout in mcp-bridge.js
5. Try a simpler command first: `burp_burp_version`

### "Parse error" or "Invalid JSON"

**Cause:** Burp returned non-JSON response or truncated output.

**Fix:**
1. Check Burp extension output for errors
2. Verify Burp has enough memory (increase heap size)
3. Try with smaller dataset (use `limit` parameter)
4. Check for special characters in responses that break JSON

---

## 📊 Performance Tips

### For Large Attacks
- Use `burp_intruder_attack_async` (multi-threaded) instead of synchronous
- Set `throttle_ms` to avoid rate limiting
- Use `success_length_not` to filter results server-side
- Break large ranges into batches

### For Deep Scanning
- Start with passive scan (`burp_scan`) first
- Use active scan only on interesting endpoints
- Poll `burp_scan_results` periodically instead of waiting
- Set scan speed to "fast" for automated workflows

### For API Testing
- Use `burp_repeater_modify_send` to iterate quickly
- Store successful auth tokens in variables
- Chain tools: history → extract → inject → scan
- Use `burp_search_history` with regex for efficient filtering

### Memory Optimization
- Regularly call `burp_proxy_clear` to free memory
- Limit history size with `limit` parameter
- Use filters instead of fetching all data
- Close unused Repeater/Intruder tabs in Burp

---

## 🔒 Security Best Practices

### Authorization
- **ALWAYS** ensure you have written authorization before testing
- Document authorization in `skills/field-journal/precedent-auth.md`
- Never test production systems without explicit permission
- Respect scope boundaries defined in authorization

### Scope Management
- Use `burp_add_to_scope` to define authorized targets
- Verify URLs with `burp_get_scope` before attacking
- Configure Burp scope to prevent out-of-scope requests
- Review scope before each session

### Rate Limiting
- Use `throttle_ms` parameter to avoid DoS
- Monitor target server response times
- Back off if errors increase
- Consider time-of-day restrictions

### Data Handling
- Never log credentials or sensitive data
- Anonymize PII in reports and field-journal
- Clear Burp history after sensitive testing
- Use secure storage for evidence

---

## 🎓 Learning Path

### Beginner
1. Start with `burp_proxy_history` to view traffic
2. Use `burp_send_request` to understand request/response
3. Try `burp_encode`/`burp_decode` for basic operations
4. Experiment with `burp_search_history` for filtering

### Intermediate
1. Use `burp_send_to_repeater` for manual testing
2. Try `burp_intruder_attack_wordlist` with small lists
3. Use `burp_scan` for passive vulnerability detection
4. Learn `burp_extract_from_response` for data extraction

### Advanced
1. Chain multiple tools for complex workflows
2. Use `burp_intruder_attack_async` for parallel attacks
3. Implement `burp_collaborator_*` for OOB detection
4. Build custom automation scripts with all 63 tools

---

## 📚 Additional Resources

### MCP Architecture
- HTTP API: `http://127.0.0.1:9876` (Burp extension)
- Stdio Bridge: `mcp-bridge.js` (Node.js)
- Protocol: JSON-RPC 2.0 over stdio
- Client: Claude Code (or any MCP-compatible client)

### File Locations
- Extension source: `burp-mcp-full/src/main/java/com/burpmcp/`
- Bridge source: `burp-mcp-full/mcp-bridge.js`
- Build script: `burp-mcp-full/build.bat` or `build.sh`
- Output JAR: `burp-mcp-full/build/libs/burp-mcp-full.jar`

### Dependencies
- Burp Suite (Professional or Community)
- Java JDK 11+ (for building extension)
- Node.js 14+ (for MCP bridge)
- Claude Code or compatible MCP client

### Port Configuration
- Default Burp HTTP API: `127.0.0.1:9876`
- To change: Edit `McpHttpServer.java` line with `new McpHttpServer(9876)`
- Environment variables for bridge:
  - `BURP_MCP_HOST` (default: 127.0.0.1)
  - `BURP_MCP_PORT` (default: 9876)

---

## 🎉 Quick Start Checklist

- [ ] Build Burp extension with `build.bat` or `build.sh`
- [ ] Load `burp-mcp-full.jar` in Burp Extensions
- [ ] Verify "[MCP] Server started" in extension output
- [ ] Copy `.claude/mcp.json` to `~/.claude/mcp.json`
- [ ] Update absolute path to `mcp-bridge.js`
- [ ] Restart Claude Code
- [ ] Test with: "Show me Burp proxy history"
- [ ] Browse some websites with Burp proxy enabled
- [ ] Try automation: "Find POST requests with passwords"

---

## 💡 Pro Tips

1. **Aliases:** Create custom prompts for common workflows
2. **Templates:** Save complex Intruder configs as prompt templates
3. **Chaining:** Combine multiple tools in a single prompt
4. **Filtering:** Use regex in `burp_search_history` for precision
5. **Reporting:** Always end with report generation for documentation
6. **Evidence:** Export requests as curl for reproducibility
7. **Collaboration:** Share successful automation prompts with your team

---

## 🚀 Next Steps

After mastering Burp automation:

1. **Explore other MCP servers:**
   - `idapro` - Binary reverse engineering
   - `jshook` - JavaScript hooking
   - `anything-analyzer` - Browser automation

2. **Create custom workflows:**
   - Build reusable automation templates
   - Document in `skills/field-journal/`
   - Contribute to the skill pack

3. **Advanced integration:**
   - Chain Burp with other security tools
   - Build multi-stage attack workflows
   - Automate reporting pipelines

---

## 📞 Support

- **Issues:** [GitHub Issues](https://github.com/Darkhearted007/reverse-skill/issues)
- **Community:** [LINUX DO](https://linux.do)
- **Documentation:** See `burp-mcp-full/README.md` for API details

---

**Happy Automating! 🔥**

Last updated: 2026-06-24
