# 🔬 IDA Pro Binary Analysis Automation Guide

Complete guide to automating IDA Pro binary analysis using MCP (Model Context Protocol) with Claude Code and other AI coding assistants.

---

## 🎯 What You Can Do

Once configured, you can tell Claude things like:

- "Open this malware sample in IDA and give me an overview"
- "Decompile the main function and explain what it does"
- "Find all functions that call CryptEncrypt and trace the data flow"
- "Rename all functions based on their behavior and add comments"
- "Find the decryption routine and extract the algorithm"
- "Trace all xrefs to this suspicious string"
- "Generate a call graph for the networking functions"
- "Export all function signatures to a C header file"

Claude will execute these **automatically** using IDA Pro's full analysis capabilities!

---

## 📋 Step-by-Step Setup

### Prerequisites

1. **IDA Pro** (Professional or Free version)
   - Download: https://hex-rays.com/ida-pro/
   - Install location will be needed (e.g., `C:\Program Files\IDA Pro`)

2. **Python 3.x** (for idalib-mcp)
   - IDA Pro comes with Python, but system Python is also needed

3. **Node.js** (for MCP bridge, if needed)

---

### Step 1: Install IDA Pro MCP

```bash
# Set IDA installation directory (required!)
# Windows:
setx IDADIR "C:\Program Files\IDA Pro"

# Linux/macOS:
export IDADIR="/opt/idapro"
echo 'export IDADIR="/opt/idapro"' >> ~/.bashrc  # Make permanent

# Install ida-pro-mcp from GitHub (NOT from PyPI!)
pip install git+https://github.com/mrexodia/ida-pro-mcp.git

# Install the IDA plugin
ida-pro-mcp --install
# Choose: Streamable HTTP + Global + All clients

# Verify installation
ida-pro-mcp --config
```

> ⚠️ **Important:** PyPI has a package called `ida-mcp` (different author) - that's NOT the one we need! Always install from GitHub: `mrexodia/ida-pro-mcp`

---

### Step 2: Configure Claude Code

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

**Verify the IDA Pro section** in `~/.claude/mcp.json`:

```json
{
  "mcpServers": {
    "idapro": {
      "url": "http://127.0.0.1:13337/mcp",
      "disabled": false
    }
  }
}
```

---

### Step 3: Start IDA Pro MCP Server

**Windows:**
```powershell
cd reverse-skill\skills\ida-reverse
powershell -File scripts\start.ps1
```

**Linux/macOS:**
```bash
cd reverse-skill/skills/ida-reverse
bash scripts/start.sh
```

**Expected output:** `OK:72` (72 tools available)

If you get `ERR:timeout`:
- Check IDADIR is set correctly
- Verify `ida-pro-mcp --config` shows correct paths
- Check `idalib-mcp` is in PATH: `which idalib-mcp` (Linux) or `where idalib-mcp` (Windows)

---

### Step 4: Restart Claude Code

Close and reopen Claude Code to load the new MCP configuration.

---

## ✅ Verification Tests

### Test 1: Check Connection

In Claude Code (from **any directory**):

```
Show me what IDA Pro MCP tools are available
```

**Expected Response:**
Claude lists all 72 tools including:
- `idapro_survey_binary`
- `idapro_decompile`
- `idapro_analyze_function`
- `idapro_xrefs_to`
- etc.

### Test 2: Open a Binary

```
Open calc.exe from System32 in IDA Pro
```

**Expected:** Claude calls the open script and reports success (e.g., `OK:calc.exe:session_id`)

### Test 3: Analyze a Function

```
Give me an overview of the binary that's currently open in IDA
```

**Expected:** Claude calls `idapro_survey_binary` and shows architecture, entry point, imports, strings, etc.

---

## 🚀 Advanced Automation Examples

### Example 1: Full Binary Analysis Workflow

**Prompt:**
```
Open malware.exe in IDA with timeout of 600 seconds.
Give me a complete analysis including:
- Architecture and entry point
- All imported crypto functions
- Suspicious strings (URLs, IPs, registry keys)
- Functions with high xref count (likely important)
- Decompile the top 3 most-referenced functions
```

**What Claude Does:**
1. `scripts/open.ps1 -Path malware.exe -TimeoutSeconds 600`
2. `idapro_survey_binary(detail_level="full")`
3. `idapro_entity_query` to filter crypto imports
4. `idapro_find_regex` for IP/URL patterns
5. `idapro_list_funcs` sorted by xref count
6. `idapro_decompile` on top 3 functions
7. Generates markdown report

### Example 2: String Decryption Analysis

**Prompt:**
```
In the currently open binary:
1. Find all calls to VirtualAlloc
2. For each caller, decompile it and check if it accesses encrypted strings
3. If found, trace the data flow backward to find the decryption key
4. Generate a Python script to decrypt all strings
```

**Claude Workflow:**
1. `idapro_find(type="imports", targets=["VirtualAlloc"])`
2. `idapro_xrefs_to` to get all callers
3. `idapro_decompile` each caller function
4. `idapro_trace_data_flow` backward from string references
5. `idapro_get_bytes` to extract encrypted data
6. Analyze decryption algorithm from pseudocode
7. Generate Python decryption script

### Example 3: Vulnerability Discovery

**Prompt:**
```
Scan this binary for potential buffer overflow vulnerabilities:
1. Find all functions using strcpy, strcat, sprintf
2. Decompile each and check if buffer size is validated
3. Identify potentially exploitable cases
4. Generate a vulnerability report
```

**Claude Workflow:**
1. `idapro_entity_query(kind="imports")` filter dangerous functions
2. `idapro_xrefs_to` for each dangerous import
3. `idapro_analyze_function` with full context
4. Analyze pseudocode for missing bounds checks
5. `idapro_set_comments` to mark vulnerable code
6. Generate CVE-style report

### Example 4: Function Reconstruction

**Prompt:**
```
This binary is heavily obfuscated. Help me clean it up:
1. Survey the binary and identify the real entry point
2. Define all undefined functions
3. Infer types for function parameters
4. Rename functions based on their behavior
5. Add meaningful comments
6. Export cleaned function signatures to a C header
```

**Claude Workflow:**
1. `idapro_survey_binary`
2. `idapro_define_func` on unrecognized code
3. `idapro_infer_types` for all functions
4. `idapro_analyze_function` to understand behavior
5. `idapro_rename` batch operation
6. `idapro_set_comments` on key points
7. `idapro_export_funcs(format="c_header")`

---

## 🛠️ Complete Tool Reference (72 Tools)

### Overview & Survey (4 tools)
- `idapro_survey_binary` - Quick overview (architecture, imports, strings, hot functions)
- `idapro_list_funcs` - List all functions (paginated, filterable)
- `idapro_list_globals` - List global variables
- `idapro_entity_query` - Unified query (functions/globals/imports/strings/names)

### Decompilation & Disassembly (4 tools)
- `idapro_decompile` - Get pseudocode (C-like)
- `idapro_disasm` - Get assembly listing
- `idapro_analyze_function` - Comprehensive analysis (pseudocode + strings + xrefs + callgraph)
- `idapro_func_profile` - Function metrics (size, complexity, xref count)

### Cross-References & Data Flow (5 tools)
- `idapro_xrefs_to` - Who references this address?
- `idapro_xref_query` - Advanced xref filtering
- `idapro_callees` - List called functions
- `idapro_callgraph` - Generate call tree
- `idapro_trace_data_flow` - Trace data propagation (forward/backward)

### Search (4 tools)
- `idapro_find_regex` - Regex string search
- `idapro_search_text` - Text search in disassembly
- `idapro_find_bytes` - Byte pattern search (with ?? wildcards)
- `idapro_find` - Advanced search (immediates/strings/references)

### Memory & Data Reading (6 tools)
- `idapro_get_bytes` - Read raw bytes
- `idapro_get_string` - Read string at address
- `idapro_get_int` - Read integer value
- `idapro_get_global_value` - Read global variable
- `idapro_read_struct` - Read structure fields
- `idapro_search_structs` - Search defined structures

### Modification Operations (8 tools)
- `idapro_set_comments` - Add comments (synced to pseudocode)
- `idapro_append_comments` - Append to existing comments
- `idapro_rename` - Batch rename (functions/globals/locals/stack vars)
- `idapro_patch_asm` - Patch assembly instructions
- `idapro_patch` - Patch raw bytes
- `idapro_define_func` - Define new function
- `idapro_undefine` - Undefine code/data
- `idapro_define_code` - Convert data to code

### Type System (5 tools)
- `idapro_declare_type` - Declare C structures/enums/unions
- `idapro_set_type` - Apply type to function/variable
- `idapro_infer_types` - AI-powered type inference
- `idapro_type_query` - Query declared types
- `idapro_type_inspect` - View type details

### Stack Frame Analysis (3 tools)
- `idapro_stack_frame` - View stack variables
- `idapro_declare_stack` - Declare stack variable
- `idapro_delete_stack` - Remove stack variable

### Signatures (3 tools)
- `idapro_make_signature` - Generate byte signature
- `idapro_make_signature_for_function` - Function signature
- `idapro_find_xref_signatures` - Signature for xref code

### Session Management (7 tools)
- `idapro_idalib_list` - List all open sessions
- `idapro_idalib_current` - Show current session
- `idapro_idalib_switch` - Switch between sessions
- `idapro_idalib_close` - Close session
- `idapro_idalib_save` - Save IDB database
- `idapro_idalib_health` - Check worker health
- `idapro_server_health` - Check server status

### Utilities (5 tools)
- `idapro_int_convert` - Base conversion (hex/dec/oct/bin) **Always use this!**
- `idapro_export_funcs` - Export to JSON/C header/prototypes
- `idapro_py_eval` - Execute Python in IDA context
- `idapro_server_warmup` - Preload subsystems
- `idapro_open_file` - Open file in IDA GUI (requires dbg extension)

---

## 🎬 Demo Workflow: Malware Analysis

### Complete Malware Teardown

**Step 1: Initial Reconnaissance**
```
Open suspicious.exe in IDA with 600 second timeout.
Show me architecture, imports, and any obfuscation indicators.
```

**Step 2: String Analysis**
```
Find all strings containing:
- IP addresses
- URLs
- Registry keys
- File paths
- Common C2 indicators (http://, ftp://, cmd.exe)
```

**Step 3: Crypto Detection**
```
Find all imports related to cryptography:
- CryptEncrypt/CryptDecrypt
- BCrypt* functions
- OpenSSL functions
Show me all functions that call these APIs.
```

**Step 4: Network Behavior**
```
Find all network-related imports (socket, connect, send, recv, URLDownloadToFile).
For each caller:
- Decompile the function
- Find what data is being sent
- Check for hardcoded C2 addresses
```

**Step 5: Persistence Mechanisms**
```
Find all registry and file operations:
- RegSetValue/RegCreateKey
- CreateFile/WriteFile
- Service creation APIs
Identify persistence mechanisms.
```

**Step 6: Data Exfiltration**
```
Find functions that:
- Read files (GetFileAttributes, ReadFile)
- Encode/encrypt data
- Send data over network
Trace the complete exfiltration pipeline.
```

**Step 7: Generate IOCs**
```
Extract all indicators of compromise:
- C2 domains/IPs
- File paths created
- Registry keys modified
- Mutex names
- User-agent strings
Generate YARA rule and Suricata signatures.
```

**Step 8: Report Generation**
```
Create a comprehensive malware analysis report with:
- Executive summary
- Technical analysis
- IOC list
- MITRE ATT&CK mapping
- Remediation steps
```

Claude will execute **all 8 steps automatically** using IDA Pro MCP tools!

---

## 🐛 Troubleshooting

### "ERR:timeout" when starting server

**Cause:** `idalib-mcp` not found or IDADIR not set.

**Fix:**
1. Verify IDADIR: `echo $IDADIR` (Linux) or `echo %IDADIR%` (Windows)
2. Check installation: `ida-pro-mcp --config`
3. Reinstall if needed: `pip install --force-reinstall git+https://github.com/mrexodia/ida-pro-mcp.git`
4. Run plugin install again: `ida-pro-mcp --install`

### "No database bound" error

**Cause:** No binary file is currently open.

**Fix:**
1. Use the open script: `powershell -File scripts\open.ps1 -Path "file.exe"`
2. Or tell Claude: "Open sample.exe in IDA"

### "ERR:open_timeout_600s"

**Cause:** Binary is very large or complex, auto-analysis taking too long.

**Fix:**
1. Increase timeout: `-TimeoutSeconds 1200` (20 minutes)
2. Or skip auto-analysis: `-NoAutoAnalysis` (faster open, manual analysis later)
3. For huge binaries (>100MB), consider radare2 first for quick reconnaissance

### "Failed to open database - file locked"

**Cause:** Old IDA worker process holding the .idb file.

**Fix:**
The `open.ps1` script auto-detects this and copies to temp directory with GUID prefix.
Output will show: `OK:abc123-sample.exe:session_id (temp copy)`

### Tools showing "schema validation error"

**Cause:** MCP client bug with `idalib_open` tool.

**Fix:**
Use the bundled `open.ps1` script instead of calling `idalib_open` directly. The script bypasses MCP validation by calling the HTTP API directly.

---

## 📊 Performance Tips

### For Large Binaries
- Use `-NoAutoAnalysis` for initial open
- Call `idapro_server_warmup` after open
- Query specific functions instead of listing all
- Use `detail_level="minimal"` in survey

### For Stripped Binaries
- Start with `idapro_survey_binary` to understand structure
- Use `idapro_define_func` on unrecognized code
- Apply `idapro_infer_types` extensively
- Cross-reference with known library signatures

### For Obfuscated Code
- Use `idapro_trace_data_flow` to defeat control flow flattening
- Look for string decryption loops with `idapro_find_regex`
- Use `idapro_py_eval` for custom deobfuscation scripts
- Save progress frequently with `idapro_idalib_save`

### Memory Optimization
- Close unused sessions with `idapro_idalib_close`
- Work on one binary at a time
- Use `idapro_server_health` to monitor worker status
- Restart server if workers become unhealthy

---

## 🔒 Security Best Practices

### Malware Analysis Safety
- **Always** analyze malware in an isolated VM or sandbox
- Never run malware samples on your host system
- Use snapshots before opening suspicious binaries
- Disable network access in analysis VM

### File Handling
- Verify file hashes before analysis
- Keep malware samples in password-protected archives
- Use dedicated analysis user account (not admin)
- Clear temp files after analysis: `%TEMP%\reverse-skill\`

### Data Protection
- Never analyze corporate binaries without authorization
- Redact sensitive data before sharing reports
- Encrypt IDB files containing proprietary analysis
- Follow responsible disclosure for vulnerabilities

---

## 🎓 Learning Path

### Beginner
1. Start with `idapro_survey_binary` to understand structure
2. Use `idapro_decompile` on simple functions
3. Try `idapro_xrefs_to` to trace string usage
4. Practice with `idapro_set_comments` and `idapro_rename`

### Intermediate
1. Use `idapro_analyze_function` for comprehensive analysis
2. Trace data flow with `idapro_trace_data_flow`
3. Apply type information with `idapro_set_type`
4. Build call graphs with `idapro_callgraph`

### Advanced
1. Write custom analysis scripts with `idapro_py_eval`
2. Create signatures for code reuse with `idapro_make_signature`
3. Batch process multiple binaries using session switching
4. Automate complete analysis pipelines

---

## 📚 Additional Resources

### MCP Architecture
- HTTP API: `http://127.0.0.1:13337/mcp` (ida-pro-mcp server)
- Protocol: JSON-RPC 2.0 over HTTP
- Client: Claude Code (or any MCP-compatible client)

### File Locations
- Start script: `skills/ida-reverse/scripts/start.ps1` (Windows) or `start.sh` (Linux)
- Open script: `skills/ida-reverse/scripts/open.ps1` (Windows) or `open.sh` (Linux)
- Main skill: `skills/ida-reverse/SKILL.md`
- Tool reference: `skills/ida-reverse/references/ida-mcp-cheatsheet.md`

### Dependencies
- IDA Pro (Professional or Free)
- Python 3.x
- ida-pro-mcp from GitHub
- Claude Code or compatible MCP client

### Port Configuration
- Default: `127.0.0.1:13337`
- To change: Set `IDA_MCP_PORT` environment variable or edit `start.ps1`

---

## 🎉 Quick Start Checklist

- [ ] Install IDA Pro
- [ ] Set IDADIR environment variable
- [ ] Install ida-pro-mcp: `pip install git+https://github.com/mrexodia/ida-pro-mcp.git`
- [ ] Run plugin installer: `ida-pro-mcp --install`
- [ ] Verify config: `ida-pro-mcp --config`
- [ ] Copy `.claude/mcp.json` to `~/.claude/mcp.json`
- [ ] Start server: `powershell -File scripts\start.ps1`
- [ ] Verify OK:72 output
- [ ] Restart Claude Code
- [ ] Test: "Show me IDA Pro tools"
- [ ] Open a binary: "Open calc.exe in IDA"
- [ ] Try automation: "Give me an overview of this binary"

---

## 💡 Pro Tips

1. **Always use timeout:** Complex binaries need `powershell -File open.ps1 -Path file.exe -TimeoutSeconds 600`
2. **Progress tracking:** Script outputs `INFO:opening:X/600s` every 10 seconds so you know it's working
3. **Base conversion:** Never manually convert hex/dec - use `idapro_int_convert`
4. **Session management:** Use `idapro_idalib_list` to see all open binaries
5. **Batch operations:** `idapro_rename` supports batch renaming with JSON
6. **Save progress:** Use `idapro_idalib_save` before complex operations
7. **Type inference:** `idapro_infer_types` is very powerful for stripped binaries

---

## 🚀 Next Steps

After mastering IDA Pro automation:

1. **Explore other MCP servers:**
   - `burpsuite` - Web application security testing
   - `jshook` - JavaScript hooking and analysis
   - `radare2` - Free IDA alternative

2. **Combine with other skills:**
   - Use `binary-diff` for patch analysis
   - Chain with `pwn-chain` for exploit development
   - Integrate with `malware-analysis` workflows

3. **Build custom workflows:**
   - Create reusable analysis templates
   - Document in `skills/field-journal/`
   - Share with the community

---

## 📞 Support

- **Issues:** [GitHub Issues](https://github.com/Darkhearted007/reverse-skill/issues)
- **IDA MCP Project:** [mrexodia/ida-pro-mcp](https://github.com/mrexodia/ida-pro-mcp)
- **Community:** [LINUX DO](https://linux.do)
- **Documentation:** See `skills/ida-reverse/SKILL.md` for detailed workflow

---

**Happy Reversing! 🔬**

Last updated: 2026-06-24
