## HR LEAVE MANAGEMENT
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
