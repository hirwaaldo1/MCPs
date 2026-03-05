# 🔧 Fixing the Artifactory 401 Unauthorized Error with `uv`

## What Is This Error?

When you try to run the project using `uv run main.py`, you might see an error like this:

```
× Failed to download `click==8.1.8`
├─▶ Failed to fetch: `https://artifactory.infra.ant.dev/artifactory/api/pypi/pypi-all/...`
╰─▶ HTTP status client error (401 Unauthorized)
```

### Why Does This Happen?

The project's configuration points to a **private internal package registry** (Artifactory) hosted at `artifactory.infra.ant.dev`. When `uv` tries to download Python packages, it sends the request to this private server instead of the public [PyPI](https://pypi.org) registry.

A **401 Unauthorized** error means the server rejected the request because it requires valid login credentials — which you don't have if you're setting up the project on your own machine.

> **Note:** `artifactory.infra.ant.dev` is a private company server. You cannot create an account for it publicly. Access is managed by the organization's IT/DevOps team.

---

## ✅ The Fix — Use Public PyPI Instead

Since all the packages used in this project (`click`, `mcp`, `typer`, etc.) are available on the public PyPI registry, you can simply redirect `uv` to download from there instead.

### Step 1 — Update `pyproject.toml`

Open your `pyproject.toml` file and look for a section like this:

```toml
[[tool.uv.index]]
url = "https://artifactory.infra.ant.dev/artifactory/api/pypi/pypi-all/simple"
```

**Replace it** with the public PyPI index:

```toml
[[tool.uv.index]]
url = "https://pypi.org/simple"
default = true
```

Or simply **delete** the `[[tool.uv.index]]` block entirely — `uv` defaults to PyPI automatically.

---

### Step 2 — Clear the `uv` Cache

Previous failed download attempts may be cached. Clear them to start fresh:

```bash
uv cache clean
```

---

### Step 3 — Run the Project

```bash
uv run main.py
```

It should now download all packages successfully from the public PyPI registry.

---

## 🚀 Quick One-Liner (No Config Changes Needed)

If you don't want to modify any files, you can override the registry directly in the terminal for a single run:

```bash
uv run --index-url https://pypi.org/simple main.py
```

---

## 🐍 Alternative — Use `pip` Instead of `uv`

If you prefer not to use `uv` at all, you can install dependencies and run the project with standard Python tools:

```bash
# Install dependencies
pip install mcp[cli]

# Run the project
python main.py
```

---

## Summary

| Problem | Solution |
|---|---|
| `uv` points to private Artifactory registry | Redirect to public PyPI |
| 401 Unauthorized on package download | Remove or replace `[[tool.uv.index]]` in `pyproject.toml` |
| Still failing after config change | Run `uv cache clean` then retry |
| Don't want to edit config files | Use `--index-url https://pypi.org/simple` flag |

---

## Need Access to the Private Artifactory Registry?

If your work requires using the private registry (e.g., for internal packages), contact your **IT or DevOps team** and request:
- An Artifactory username and API token
- Access to the correct permission group

Once you have credentials, you can configure them like this:

```bash
# Add to your ~/.netrc file
machine artifactory.infra.ant.dev
login YOUR_USERNAME
password YOUR_API_TOKEN
```
