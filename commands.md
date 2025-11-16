# Commands to activate and deactivate virtual environments

## Activate virtual environment  
```bash
source .venv/bin/activate
````

## Deactivate virtual environment

```bash
deactivate
```

---

# Installing and managing dependencies with uv

`uv` is a fast package and project manager that can replace pip and virtualenv.

### Install uv (if not already installed)

On macOS / Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Or via pip:

```bash
pip install uv
```

### Use uv to install FastMCP

```bash
uv add fastmcp
```

Or using uv’s pip interface:

```bash
uv pip install fastmcp
```

### Verify FastMCP installation

```bash
fastmcp version
```

---

# Running FastMCP servers and related commands

## Start a FastMCP server for development

```bash
uv run fastmcp dev <file_path>
```

This launches the server in development mode with the MCP Inspector UI.

## Run a FastMCP server (production / general run)

```bash
uv run fastmcp run <file_path>
```

This runs the server normally.

## Install a FastMCP server into a client application

```bash
uv run fastmcp install <platform_name> <file_path>
```

Platform may be a client such as “claude”, “chatgpt”, or any other MCP-compatible client.

---

# Example workflow summary

1. Create a Python virtual environment.

   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```
2. Install dependencies using uv.

   ```bash
   uv add fastmcp
   ```
3. Write your FastMCP server script (e.g., `server.py`).

   ```python
   from fastmcp import FastMCP

   mcp = FastMCP("DemoServer")

   @mcp.tool()
   def add(a: int, b: int) -> int:
       return a + b

   if __name__ == "__main__":
       mcp.run(transport="stdio")
   ```
4. Run in development mode to test and inspect.

   ```bash
   uv run fastmcp dev server.py
   ```
5. Install the server into a client.

   ```bash
   uv run fastmcp install claude server.py
   ```
6. Deactivate your environment.

   ```bash
   deactivate
   ```

---

# Notes and Tips

* Ensure that `uv` is available in your system path.
* When connecting FastMCP servers to clients like Claude Desktop, you may need to edit the client configuration JSON to include your server in the `mcpServers` section.
* When using `uv run fastmcp install`, ensure that your server's dependencies are available either in the project directory or specified through uv options.
* Use version pinning (for example, `fastmcp==2.x.x`) for stable production deployments.
