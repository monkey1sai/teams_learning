# teams_learning

## Python 環境（使用 uv）

此專案使用本地虛擬環境 `.venv/`，且已在 `.gitignore` 排除，不會加入版控。

### 1. 建立虛擬環境

```bash
uv venv .venv
```

### 2. 啟用虛擬環境

macOS / Linux:

```bash
source .venv/bin/activate
```

Windows PowerShell:

```powershell
.venv\Scripts\Activate.ps1
```

### 3. 安裝套件（若有需求）

若專案提供 `requirements.txt`：

```bash
uv pip install -r requirements.txt
```
