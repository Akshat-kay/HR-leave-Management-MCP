```markdown
<!-- Badges row -->
<p align="center">
  <img src="https://img.shields.io/badge/Model%20Context%20Protocol-MCP%20Server-4B8BF5?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Python-3.11%2B-3776AB?style=for-the-badge&logo=python" />
  <img src="https://img.shields.io/badge/uv-Project%20Manager-000000?style=for-the-badge" />
  <img src="https://img.shields.io/badge/License-MIT-green?style=for-the-badge" />
</p>

<h1 align="center">🧭 HR Leave Management MCP 🧑‍💼</h1>

<p align="center">
Model Context Protocol (MCP) server for HR teams to <b>check leave balances</b>, <b>request/approve leave</b>, and stay <b>holiday-aware</b>.
Built on <b>FastMCP</b>. Works with MCP hosts like Claude Desktop or CLI-based agents.
</p>

---

## 🌈 Table of Contents

- [✨ Highlights](#-highlights)
- [🏗️ Architecture](#️-architecture)
- [🧩 Auto-Generated Features Table](#-auto-generated-features-table)
- [🔧 Requirements](#-requirements)
- [🚀 Quick Start](#-quick-start)
- [🧪 Discover & Call (CLI)](#-discover--call-cli)
- [🧠 Data Model](#-data-model)
- [🌎 Holiday Awareness (NagerDate)](#-holiday-awareness-nagerdate)
- [📦 Project Layout](#-project-layout)
- [🛡️ Troubleshooting](#️-troubleshooting)
- [🗺️ Roadmap](#️-roadmap)
- [🤝 Contributing](#-contributing)
- [📜 License](#-license)

---

## ✨ Highlights

- **MCP-native** endpoints (tools/resources/prompts) discoverable by any MCP host.
- **Demo-ready** in-memory store; easy migration to DB/HRIS.
- **Holiday-aware** design via Nager.Date integration.
- **Reproducible** workflow using `uv` and the `mcp` CLI.
- **Interview-friendly** structure with descriptive docstrings.

---

## 🏗️ Architecture

```

MCP Host (Claude Desktop / CLI / Agent)
│  tools() / resources() / prompts()
▼
HR Leave Management MCP (FastMCP)
├─ Tools: e.g., get_leave_balance, request_leave, approve_leave
├─ Resources: e.g., latest_request, policy docs
├─ Prompts: e.g., leave_summary
└─ Data: in-memory store (swap to DB/HRIS later)

````

---

## 🧩 Auto-Generated Features Table

```bash
uv run --active mcp describe main.py --json \
| python - <<'PY'
import json,sys
d=json.load(sys.stdin)
def rows(kind, items):
    if not items: return ""
    out=[f"\n### {kind}\n\n| Name | Parameters |\n|---|---|"]
    for it in items:
        name=it.get("name","")
        params=[]
        schema=it.get("inputSchema") or it.get("parameters") or {}
        props=(schema.get("properties") or {})
        for k,v in props.items():
            t=v.get("type","any")
            params.append(f"{k}: {t}")
        out.append(f"| `{name}` | {', '.join(params) if params else '—'} |")
    return "\n".join(out)

print("<!-- BEGIN: FEATURES TABLE (auto) -->")
print("## Detected Endpoints")
print(rows("🛠️ Tools", d.get("tools")))
print(rows("📚 Resources", d.get("resources")))
print(rows("💬 Prompts", d.get("prompts")))
print("\n*Generated via* `uv run --active mcp describe main.py --json`.")
print("<!-- END: FEATURES TABLE (auto) -->")
PY
````

```markdown
<!-- BEGIN: FEATURES TABLE (auto) -->

## Detected Endpoints

<!-- Paste the generated table here -->

<!-- END: FEATURES TABLE (auto) -->
```

---

## 🔧 Requirements

* Python **3.11+**
* [`uv`](https://github.com/astral-sh/uv) (project/env manager)
* `mcp` CLI

Install:

```bash
pipx install uv || pip install uv
pipx install mcp || pip install mcp
```

---

## 🚀 Quick Start

```bash
# 1) Clone
git clone https://github.com/Akshat-kay/HR-leave-Management-MCP.git
cd HR-leave-Management-MCP

# 2) Sync environment & dependencies
uv sync

# 3) (Optional) Target active venv to avoid warnings
uv run --active python -V

# 4) Register the MCP server with your host registry
uv run --active mcp install main.py
# Alternative:
# python -m uv run mcp install main.py

# 5) Develop locally with hot reload
uv run --active mcp dev main.py
```

Using Claude Desktop: add this server under **Settings → Tools (MCP)** and point to `mcp dev main.py` (or use the config produced by `mcp install`).

---

## 🧪 Discover & Call (CLI)

List endpoints:

```bash
uv run --active mcp describe main.py
```

Call a tool:

```bash
uv run --active mcp call main.py get_leave_balance --json '{"employee_id":"E001"}'
```

Fetch a resource:

```bash
uv run --active mcp resource main.py latest_request
```

Run a prompt:

```bash
uv run --active mcp prompt main.py leave_summary --json '{"quarter":"Q4-2025"}'
```

---

## 🧠 Data Model

Example in-memory structure:

```python
employee_leaves = {
  "E001": {"balance": 18, "history": ["2024-12-25", "2025-01-01"]},
  "E002": {"balance": 20, "history": []}
}
# balance → integer days remaining
# history → ISO-8601 approved leave dates
```

Migrate to SQLite/Postgres + SQLModel/SQLAlchemy, or connect to an HRIS when needed.

---

## 🌎 Holiday Awareness (NagerDate)

Example sketch:

```python
import datetime as dt
import json, urllib.request

def is_public_holiday(country="CA", date=None):
    date = date or dt.date.today()
    url = f"https://date.nager.at/api/v3/PublicHolidays/{date.year}/{country}"
    with urllib.request.urlopen(url) as r:
        holidays = json.load(r)
    return any(h["date"] == date.isoformat() for h in holidays)
```

Cache responses and consider an offline fallback map for demos.

---

## 📦 Project Layout

```
HR-leave-Management-MCP/
├─ main.py           # FastMCP server: tools/resources/prompts
├─ pyproject.toml    # uv/PEP 621 metadata
├─ uv.lock           # reproducible lockfile
└─ README.md         # this file
```

---

## 🛡️ Troubleshooting

* Decorators must be **called**: `@mcp.tool()`, `@mcp.resource()`, `@mcp.prompt()`.
* Use `__file__` (two underscores) for file paths; avoid `___file__`.
* If `VIRTUAL_ENV does not match`, add `--active` to `uv run ...`.
* Prefer `os.path.join` + `os.path.dirname(__file__)` for Windows-safe paths.

---

## 🗺️ Roadmap

* Role-based authorization for tool access
* Database backend (SQLite/Postgres)
* HRIS adapters (BambooHR, Bob, Workday)
* Metrics & tracing (success rate, p95 latency, audit logs)
* Admin prompts (policy checks, pre-approval rules)

---

## 🤝 Contributing

* Keep endpoints small and focused.
* Use JSON-serializable I/O and clear docstrings.
* Add tests for tool behaviors and error paths.

---

## 📜 License

MIT

```
::contentReference[oaicite:0]{index=0}
```
