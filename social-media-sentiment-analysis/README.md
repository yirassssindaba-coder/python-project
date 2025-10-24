# Social Media Sentiment Analysis - Setup Guide

This guide provides complete PowerShell instructions for setting up the virtual environment, installing dependencies, fixing SSL_CERT_FILE issues, and running JupyterLab for the Social Media Sentiment Analysis project.

---

## ⚡ Quick Start

**If you've already completed the full setup below, use this quick command to start JupyterLab:**

```powershell
# Navigate to project directory
Set-Location "path\to\social-media-sentiment-analysis"

# Activate virtual environment
. .\venv\Scripts\Activate.ps1

# Start JupyterLab
python -m jupyter lab
```

After running these commands, JupyterLab will open in your browser at `http://localhost:8888/lab`

---

## 📋 Prerequisites

- Windows operating system with PowerShell
- Python 3.11, 3.12, or 3.14 installed
- Internet connection for downloading packages

---

## 🚀 Complete Setup Instructions

Run all PowerShell commands from the project root directory. Execute each block sequentially and verify success before proceeding to the next step.

### General Notes

- Open PowerShell (Administrator privileges only required when specified)
- Do not paste all commands at once — run block by block (1 → 2 → 3, etc.)
- Commands assume venv is in the project root (`.\venv`). Adjust paths if your structure differs.
- Package installations in venv are permanent for that venv (persist until venv is deleted)
- Setting SSL_CERT_FILE with `[Environment]::SetEnvironmentVariable(...,'User')` makes it permanent for your user account

---

## Step 1: Verify Project Location

Ensure you are in the correct project directory and verify the presence of important files.

```powershell
# Check current location
Get-Location

# List files and folders in current directory
Get-ChildItem -Name

# Verify important files/folders exist
Test-Path .\venv
Test-Path .\src\main.py
Test-Path .\README.md
```

**Expected Output:**
- Current directory path should end with `social-media-sentiment-analysis`
- `Test-Path` commands should return `True` (or `False` for venv if not yet created)

---

## Step 2: Check for Nested Folders

Verify you're not in a nested duplicate folder structure.

```powershell
# List all directories
Get-ChildItem -Directory | Select-Object Name

# If you see nested project folder, navigate to correct parent:
# Set-Location "C:\path\to\social-media-sentiment-analysis"
```

**Expected Output:**
- Should see folders like `src`, `venv` (if created)
- Should NOT see another `social-media-sentiment-analysis` folder inside

---

## Step 3: Fix Session PATH (If Commands Not Found)

If `where.exe` or `python` commands are not found, restore the session PATH.

```powershell
# Check if PATH exists
Write-Host "Session PATH exists?"
if ($env:PATH) { "Yes" } else { "No or empty" }

# Verify common commands are accessible
Get-Command where.exe -ErrorAction SilentlyContinue
Get-Command python -ErrorAction SilentlyContinue

# If commands not found, restore PATH (session-only)
$machine = [Environment]::GetEnvironmentVariable('Path','Machine')
$user = [Environment]::GetEnvironmentVariable('Path','User')
if ($user) {
    $env:PATH = $machine + ';' + $user
} else {
    $env:PATH = $machine
}
if (-not ($env:PATH -match 'Windows\\System32')) {
    $env:PATH += ';C:\Windows\System32'
}

# Verify commands again
Get-Command where.exe -ErrorAction SilentlyContinue
Get-Command python -ErrorAction SilentlyContinue
```

**Expected Output:**
- Commands should now be found and display their paths

---

## Step 4: Locate Python Interpreter

Find the system Python interpreter and set the `$PY` variable.

```powershell
# Clear any existing PY variable
Remove-Variable PY -ErrorAction SilentlyContinue

# Try to find python in PATH
$cmd = Get-Command python -ErrorAction SilentlyContinue
if ($cmd) {
    $PY = $cmd.Source
} else {
    # Search common Python installation locations
    $candidates = @(
        "$env:LOCALAPPDATA\Programs\Python\Python314\python.exe",
        "$env:LOCALAPPDATA\Programs\Python\Python312\python.exe",
        "$env:LOCALAPPDATA\Programs\Python\Python311\python.exe",
        "C:\Python314\python.exe",
        "C:\Python312\python.exe",
        "C:\Python311\python.exe"
    )
    foreach ($p in $candidates) {
        if (-not $PY -and (Test-Path $p)) {
            $PY = $p
        }
    }
}

# Verify Python was found
if (-not $PY) {
    Write-Error "Python not found. Please install Python or provide the full path to python.exe"
    throw
}

Write-Host "Using Python: $PY"
& $PY --version
```

**Expected Output:**
- Display path to Python executable
- Display Python version (e.g., `Python 3.12.0`)

---

## Step 5: Create or Verify Virtual Environment

Check if venv exists and is valid. Recreate if necessary.

```powershell
# Check if venv Python executable exists
Test-Path .\venv\Scripts\python.exe

# List venv Scripts directory contents
Get-ChildItem .\venv\Scripts\* -ErrorAction SilentlyContinue | Select-Object Name

# If venv doesn't exist or is corrupted, recreate it
if (-not (Test-Path .\venv\Scripts\python.exe)) {
    if (Test-Path .\venv) {
        Remove-Item -Recurse -Force .\venv
    }
    & $PY -m venv .\venv
}

# Verify venv was created successfully
Test-Path .\venv\Scripts\python.exe
```

**Expected Output:**
- Final `Test-Path` should return `True`
- venv folder should be created with Scripts subdirectory

---

## Step 6: Set Execution Policy for PowerShell Scripts

Allow PowerShell scripts to run so you can activate the virtual environment.

```powershell
# Check current execution policies
Get-ExecutionPolicy -List

# Set execution policy for current user
Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser -Force
```

**Note:** You may be prompted to confirm. Answer `Y` (Yes).

**Expected Output:**
- CurrentUser execution policy should be set to `RemoteSigned`

---

## Step 7: Activate Virtual Environment

Activate the venv and verify the Python interpreter is from the venv.

```powershell
# Activate virtual environment using dot-sourcing
. .\venv\Scripts\Activate.ps1

# Verify activation - should show venv Python, not system Python
python --version
python -c "import sys; print('sys.executable =', sys.executable)"
python -m pip --version
```

**Expected Output:**
- Prompt should show `(venv)` prefix
- `sys.executable` should point to `.\venv\Scripts\python.exe`
- pip version should be displayed

**Troubleshooting:**
- If `Activate.ps1` not found, use: `.\venv\Scripts\python.exe --version`

---

## Step 8: Install All Required Packages

Install all necessary packages for data science, Jupyter, sentiment analysis, and web scraping.

```powershell
# Upgrade pip, setuptools, and wheel first
python -m pip install --upgrade pip setuptools wheel

# Install all required packages
python -m pip install --upgrade certifi `
    pandas numpy scipy scikit-learn matplotlib seaborn `
    nltk `
    ipywidgets notebook qtconsole widgetsnbextension `
    jupyter jupyterlab ipykernel `
    openpyxl xlrd xlsxwriter `
    plotly `
    requests beautifulsoup4 lxml `
    joblib tqdm

# Enable widgets extension for classic notebook
jupyter nbextension enable --py widgetsnbextension --sys-prefix

# Download NLTK 'vader_lexicon' for sentiment analysis
python -c "import nltk; nltk.download('vader_lexicon', quiet=True)"

# Save installed packages to requirements.txt
python -m pip freeze > requirements.txt
```

**Note about Python 3.14:**
- If using Python 3.14, **do not install pyarrow** (wheel not yet available)
- If pyarrow is needed, use Python 3.11 or 3.12 for your venv

**Optional - Install pyarrow (Python 3.11/3.12 only):**
```powershell
# python -m pip install --upgrade pyarrow
```

**Expected Output:**
- All packages should install successfully
- `requirements.txt` should be created/updated
- Jupyter widgets extension should be enabled

---

## Step 9: Fix SSL Certificate Issues (Permanent)

Configure SSL_CERT_FILE environment variable to use certifi's certificate bundle.

```powershell
# Get certifi CA bundle path
$cert = & python -c "import certifi; print(certifi.where())"

# Set for current session
$env:SSL_CERT_FILE = $cert
Write-Host "Using SSL_CERT_FILE (session) = $env:SSL_CERT_FILE"

# Set permanently for user account (persists across new shells/Jupyter servers)
[Environment]::SetEnvironmentVariable('SSL_CERT_FILE', $cert, 'User')
Write-Host "SSL_CERT_FILE set permanently for User to: $cert"

# Verify SSL-related environment variables
Get-ChildItem Env: | Where-Object Name -match 'CERT|SSL|REQUESTS|CURL' | Format-Table Name,Value -AutoSize
```

**Note:**
- If you don't want permanent setting, skip the `SetEnvironmentVariable` line
- Permanent setting requires new PowerShell sessions to pick up the change

**Expected Output:**
- Display certifi certificate path
- Show SSL_CERT_FILE environment variable is set

---

## Step 10: Reinstall Jupyter Executables (If Needed)

Reinstall Jupyter if you encountered "Unable to create process" errors.

```powershell
# Check existing Jupyter executables
Get-ChildItem .\venv\Scripts\*jupyter* -Force | Select-Object Name,FullName

# Remove potentially corrupted executables
Remove-Item .\venv\Scripts\jupyter.exe -Force -ErrorAction SilentlyContinue
Remove-Item .\venv\Scripts\jupyter-lab.exe -Force -ErrorAction SilentlyContinue
Remove-Item .\venv\Scripts\jupyter-notebook.exe -Force -ErrorAction SilentlyContinue

# Reinstall Jupyter to regenerate executables
python -m pip install --upgrade --force-reinstall jupyter jupyterlab ipykernel
```

**Expected Output:**
- Jupyter executables should be recreated in `.\venv\Scripts\`

---

## Step 11: Register IPython Kernel

Register the venv as a Jupyter kernel for use in Jupyter notebooks.

```powershell
# Install kernel for current user
python -m ipykernel install --user --name "social_media_sentiment" --display-name "Python (social-media-sentiment)"

# List all registered kernels
jupyter kernelspec list
```

**Expected Output:**
- Kernel `social_media_sentiment` should appear in the list
- This kernel persists until you explicitly uninstall it

---

## Step 12: Launch JupyterLab

Start JupyterLab server and verify it runs without errors.

```powershell
# Start JupyterLab
python -m jupyter lab

# For verbose debug output (if needed):
# python -m jupyter lab --debug
```

**Expected Output:**
- JupyterLab server starts without FileNotFoundError related to SSL_CERT_FILE
- No stacktrace for pypi extension manager
- URL displayed in terminal (e.g., `http://localhost:8888/lab?token=...`)
- Browser opens automatically to JupyterLab interface

**To Stop JupyterLab:**
- Press `Ctrl+C` in the PowerShell terminal

---

## 🔍 Quick Troubleshooting

If you encounter errors, run these diagnostic commands:

```powershell
# Check SSL-related environment variables
Get-ChildItem Env: | Where-Object Name -match 'CERT|SSL|REQUESTS|CURL' | Format-Table Name,Value -AutoSize

# Check Jupyter version
jupyter --version
python -m jupyter lab --version

# Check Jupyter executable location
(Get-Command jupyter -ErrorAction SilentlyContinue).Source

# Run JupyterLab in debug mode
python -m jupyter lab --debug
```

---

## ✅ Final Verification

After completing the setup, perform these checks to ensure everything is working correctly.

### Check Versions and Registered Kernels

```powershell
# Check Python version
python --version

# Check Jupyter version
jupyter --version

# List registered kernels
jupyter kernelspec list
```

**Expected Output:**
- Python and Jupyter versions displayed
- `social_media_sentiment` kernel listed

### Verify Core Jupyter Packages

```powershell
# Check that important packages are installed
python -c "import ipywidgets, notebook, qtconsole; print('ipywidgets', getattr(ipywidgets,'__version__','n/a')); print('notebook', getattr(notebook,'__version__','n/a')); print('qtconsole', getattr(qtconsole,'__version__','n/a'))"
```

**Expected Output:**
- Version numbers for ipywidgets, notebook, and qtconsole

### Test Sentiment Analysis in Notebook

Open JupyterLab and create a new notebook. Run this code in a cell:

```python
import pandas as pd
import nltk
from nltk.sentiment.vader import SentimentIntensityAnalyzer

# Print pandas version
print('pandas', pd.__version__)

# Download NLTK data (if not already done)
nltk.download('vader_lexicon', quiet=True)

# Test sentiment analyzer
sid = SentimentIntensityAnalyzer()
print(sid.polarity_scores("I love this product"))
```

**Expected Output:**
- Pandas version displayed
- Sentiment scores dictionary (e.g., `{'neg': 0.0, 'neu': 0.192, 'pos': 0.808, 'compound': 0.6369}`)
- No errors

### Verify All Packages Are Installed

```powershell
# Install/verify all required packages
python -m pip install --upgrade pandas numpy scipy scikit-learn matplotlib seaborn nltk ipywidgets notebook qtconsole widgetsnbextension jupyter jupyterlab ipykernel openpyxl xlrd xlsxwriter plotly requests beautifulsoup4 lxml joblib tqdm

# Enable widgets extension
jupyter nbextension enable --py widgetsnbextension --sys-prefix

# Update requirements.txt
python -m pip freeze > requirements.txt
```

**Note:**
- Do not install pyarrow if using Python 3.14 (wheel not available)
- If pyarrow is required, switch to Python 3.11 or 3.12

### Verify SSL Certificate Configuration

```powershell
# Check session SSL_CERT_FILE
$env:SSL_CERT_FILE

# Check permanent user environment variable
[Environment]::GetEnvironmentVariable('SSL_CERT_FILE','User')

# Verify certificate file exists
$certPath = & python -c "import certifi; print(certifi.where())"
Test-Path $certPath
```

**Expected Output:**
- SSL_CERT_FILE points to certifi certificate bundle
- Certificate file exists (Test-Path returns `True`)

---

## 🎉 Setup Complete!

If all verification steps pass, your setup is complete and permanent for this project:

- ✅ All packages installed in venv (persist until venv is deleted)
- ✅ Kernel registered for user (persists until explicitly uninstalled)
- ✅ SSL certificate configured (permanent for user account)
- ✅ JupyterLab ready to use

You can now use the **Quick Start** section at the top of this document to quickly launch JupyterLab in future sessions.

---

## 📝 Additional Notes

### Updating Packages

To update all packages to their latest versions:

```powershell
# Activate venv first
. .\venv\Scripts\Activate.ps1

# Update all packages
python -m pip install --upgrade pip setuptools wheel
python -m pip install --upgrade pandas numpy scipy scikit-learn matplotlib seaborn nltk ipywidgets notebook qtconsole widgetsnbextension jupyter jupyterlab ipykernel openpyxl xlrd xlsxwriter plotly requests beautifulsoup4 lxml joblib tqdm

# Update requirements.txt
python -m pip freeze > requirements.txt
```

### Removing SSL_CERT_FILE (If Needed)

To remove the permanent SSL_CERT_FILE setting:

```powershell
# Remove user environment variable
[Environment]::SetEnvironmentVariable('SSL_CERT_FILE', $null, 'User')

# Verify removal
[Environment]::GetEnvironmentVariable('SSL_CERT_FILE','User')
```

### Uninstalling the Kernel

To remove the registered Jupyter kernel:

```powershell
# List kernels
jupyter kernelspec list

# Remove specific kernel
jupyter kernelspec uninstall social_media_sentiment
```

### Recreating Virtual Environment

If you need to start fresh:

```powershell
# Deactivate venv if active
deactivate

# Remove venv folder
if (Test-Path .\venv) {
    Remove-Item -Recurse -Force .\venv
}

# Recreate venv
python -m venv .\venv

# Then follow Steps 7-12 to reinstall everything
```

---

## 🆘 Getting Help

If you encounter issues:

1. Run the **Quick Troubleshooting** section commands
2. Check error messages carefully
3. Ensure Python version is compatible (3.11, 3.12, or 3.14)
4. Verify execution policy allows scripts to run
5. Make sure you're in the correct project directory

For SSL-related errors:
- Verify SSL_CERT_FILE points to an existing file
- Ensure certifi package is installed
- Try removing and recreating the SSL_CERT_FILE environment variable
