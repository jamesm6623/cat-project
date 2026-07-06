# cat-project
# Developer Guide: How Cursor AI Agent Fixed an MSYS2 Python Virtual Environment Trapped in Windows PowerShell

This documentation outlines a non-trivial troubleshooting scenario involving **Cursor's AI Agent capabilities**. It highlights how the AI systematically resolved an environment conflict where an MSYS2-based Python runtime broke standard Windows virtual environment (`venv`) behaviors inside the Cursor/VS Code editor.

---

## 🛑 The Core Technical Paradox
When attempting to build a standard Python project workflow on Windows, the environment collapsed into a loop of `ModuleNotFoundError: No module named 'requests'`. 

The root cause was architectural:
1. **The Ghost Runtime:** The active global interpreter was `D:\C\ucrt64\bin\python.exe` (part of the MSYS2 / UCRT64 toolchain).
2. **The Folder Structure Mismatch:** When MSYS2 Python compiles a virtual environment (`python -m venv .venv`), it structures directories using a **Linux/Unix blueprint** (`.venv\bin\python.exe`) rather than the expected **Windows native layout** (`.venv\Scripts\python.exe`).
3. **IDE Blindness:** Cursor and VS Code's automated "Run" utilities natively search for `Scripts\python.exe` on Windows hosts. Failing to locate it, the IDE silently defaulted back to the global MSYS2 system interpreter, which heavily restricted unmanaged `pip` actions due to **PEP 668** (`externally-managed-environment`).

---

## 🛠️ The Cursor Agent AI Intervention & Self-Healing Pipeline

Rather than throwing standard boilerplate errors, the **Cursor AI Agent** sequentially manipulated the local workspace structure, terminal policies, and editor configuration files to orchestrate a seamless fix:

### 1. Re-engineering Workspace Paths (The Symlink Masterstroke)
To satisfy both the Unix-style MSYS2 execution engine and the Windows-centric Cursor IDE interpreter-scanner, the Cursor Agent directly injected a folder symlink via shell commands:
* **The Fix:** It mapped `.venv\Scripts` to target `.venv\bin`.
* **The Result:** Cursor instantly identified it as a compliant standard Windows virtual environment, surfacing it inside the `Select Interpreter` selection matrix.

### 2. Dependency Sourcing & Local Configuration Overrides
The Agent bypassed standard proxy limitations and isolated the required modules locally:
* **Mirror Fast-Track:** It injected the `requests` module (v2.34.2) into the localized environment utilizing the high-speed Tsinghua (TSINGHUA) mirror registry.
* **Declarative Tracking:** Automatically generated a clean `requirements.txt` manifest within the workspace root directory.

### 3. Automated IDE Infrastructure Re-routing
To prevent user error from accidentally running global scripts via hotkeys, the Agent updated local IDE settings blueprints:
* **`.vscode/settings.json`:** Overrode the base settings layer to force set the workspace target path:
  ```json
  "python.defaultInterpreterPath": "${workspaceFolder}/.venv/Scripts/python.exe"
