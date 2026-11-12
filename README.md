# 🔍 ForensicGuard

**Real-Time Digital Forensic Log Analysis and Automated Suspicious Activity Response System**

[![Python](https://img.shields.io/badge/Python-3.11%2B-blue?logo=python)](https://python.org)
[![Platform](https://img.shields.io/badge/Platform-Windows%2010%2F11-blue?logo=windows)](https://microsoft.com/windows)
[![GUI](https://img.shields.io/badge/GUI-PySide6-green)](https://doc.qt.io/qtforpython/)
[![ML](https://img.shields.io/badge/ML-scikit--learn%20Random%20Forest-orange)](https://scikit-learn.org)
[![License](https://img.shields.io/badge/License-Academic%20Project-lightgrey)]()

---

## 📌 Overview

ForensicGuard is a **Windows endpoint forensic monitoring system** that continuously observes, records, and analyses system activity in real time. It detects suspicious behaviour using a **50-rule risk-scoring engine** and an optional **Random Forest machine learning classifier**, stores structured forensic evidence in a local SQLite database, and presents a live dashboard through a modern PySide6 GUI.

The system runs multiple daemon threads simultaneously to monitor:

| Monitor | Description |
|---|---|
| **File Monitor** | Watches filesystem events (create, modify, delete, rename) via `watchdog` |
| **File Transfer Monitor** | Detects clipboard-based exfiltration, removable drive writes, and MTP transfers |
| **USB Monitor** | Tracks USB device plug/unplug events with VID/PID metadata via WMI |
| **Process Monitor** | Tracks process lifecycles (spawn, terminate) via `psutil` |
| **Active Window Monitor** | Records foreground window focus changes using Win32 API |
| **Security Monitor** | Reads Windows Security Event Log for logon/logoff and privilege events |
| **Event Engine Worker** | Async queue consumer that routes events through the detection pipeline |

> **Scope note:** Network monitoring and browser monitoring modules exist as placeholders; full implementation is a planned future enhancement.

---

## ✨ Key Features

- **Real-time multi-threaded monitoring** — 7 daemon threads running in parallel
- **50-rule risk registry** — deterministic scoring across file, process, USB, clipboard, and security events
- **Machine Learning anomaly detection** — Random Forest classifier (scikit-learn) trained on labelled forensic event features
- **Forensic evidence database** — SQLite schema (`forensic_events`, `risk_alerts` tables) with timestamps, severity, and event metadata
- **Live PySide6 dashboard** — event feed, risk alerts, and session statistics
- **Cloud authentication** — Supabase-backed user login with secure token management
- **PDF forensic reports** — session-scoped, paginated PDF generation using ReportLab
- **Windows toast notifications** — Windows 10/11 native alerts for high-severity events via `win10toast`
- **Deduplication & debouncing** — prevents alert fatigue from repeated identical events

---

## 🏗️ Project Architecture

```
Forensic-project/
├── main.py                    # Entry point – starts all 7 daemon threads + GUI
├── requirements.txt           # Python dependencies
├── .env.example               # Template for environment variables (copy to .env)
├── config.py                  # Application configuration
│
├── auth/                      # User authentication
│   └── login_window.py        # PySide6 login dialog with Supabase integration
│
├── alerts/                    # Alert management
│   └── alert_manager.py       # State-machine alert manager + Windows notifications
│
├── database/                  # Data persistence
│   ├── database.py            # SQLite schema initialisation & CRUD operations
│   └── riskalert_db.py        # Risk alert storage and retrieval
│
├── detection/                 # Risk scoring and rule engine
│   ├── risk_engine.py         # Composite risk scorer
│   ├── rule_registry.py       # 50 forensic detection rules (4 FULL, 41 PARTIAL, 5 PENDING)
│   ├── event_risk_handler.py  # Debounced event-to-rule dispatcher
│   └── ml_detector.py         # Random Forest inference wrapper
│
├── event_engine/              # Async event queue
│   ├── engine.py              # Thread-safe event queue
│   └── worker.py              # Queue consumer / event dispatcher
│
├── features/                  # ML feature extraction
│   └── feature_aggregator.py  # Aggregates raw events into ML feature vectors
│
├── gui/                       # PySide6 graphical interface
│   ├── app.py                 # GUI entry point
│   ├── main_window.py         # Main dashboard window
│   ├── controller.py          # MVC controller
│   └── styles.py              # Qt stylesheet
│
├── ml/                        # ML model training utilities
│   ├── build_dataset.py       # Dataset builder from forensic_events DB
│   └── forensicguard_dataset.csv  # Training data snapshot
│
├── models/                    # Trained ML artefacts
│   └── forensicguard_rf.pkl   # Trained Random Forest model (~8.4 MB)
│
├── monitors/                  # System monitoring modules
│   ├── file_monitor.py        # Filesystem events (watchdog)
│   ├── file_transfer_monitor.py   # Clipboard / removable drive / MTP exfiltration
│   ├── usb_monitor.py         # USB device events (WMI)
│   ├── process_monitor.py     # Process lifecycle (psutil)
│   ├── active_window.py       # Foreground window tracking (Win32)
│   ├── security_monitor.py    # Windows Security Event Log
│   ├── browser_monitor.py     # [PLACEHOLDER – not yet implemented]
│   └── network_monitor.py     # [PLACEHOLDER – not yet implemented]
│
├── reports/                   # Forensic PDF report generation
│   ├── forensic_report.py     # ReportLab PDF builder
│   └── session_context.py     # Session-scoped report context
│
├── supabase/                  # Cloud auth client
│   ├── auth.py                # Authentication helpers
│   └── client.py              # Supabase client initialisation
│
├── training/                  # Model training scripts
│   └── ...
│
├── utils/                     # Utilities
│   └── logger.py              # Centralised logging and event queue publisher
│
└── test_*.py                  # Comprehensive unit test suite (58/60 tests pass)
```

---

## 🖥️ System Requirements

| Requirement | Minimum |
|---|---|
| Operating System | Windows 10 (64-bit) or Windows 11 |
| Python | 3.11 or later |
| RAM | 4 GB (8 GB recommended for ML inference) |
| Disk | 500 MB free space |
| Privileges | **Run as Administrator** (required for Security Event Log and USB monitoring) |

> ForensicGuard is a **Windows-only** application. It uses `pywin32`, `WMI`, and Windows-specific APIs that are not available on Linux or macOS.

---

## ⚙️ Installation

### 1. Clone the repository

```bash
git clone https://github.com/sundeep-codes/Major_project_forensic_guard.git
cd Major_project_forensic_guard/Forensic-project
```

### 2. Create and activate a virtual environment

```powershell
python -m venv .venv
.venv\Scripts\Activate.ps1
```

> **PowerShell execution policy:** If you get a script execution error, run:
> ```powershell
> Set-ExecutionPolicy -ExecutionPolicy RemoteSigned -Scope CurrentUser
> ```

### 3. Install dependencies

```powershell
pip install -r requirements.txt
```

> Installation may take a few minutes. PySide6 alone is ~100 MB.

### 4. Configure environment variables

```powershell
Copy-Item .env.example .env
notepad .env
```

Edit `.env` and fill in your Supabase credentials:

```env
SUPABASE_URL=https://YOUR_PROJECT_ID.supabase.co
SUPABASE_PUBLISHABLE_KEY=your-supabase-anon-key-here
```

> **Getting Supabase credentials:**
> 1. Go to [https://supabase.com/dashboard](https://supabase.com/dashboard)
> 2. Open your project → **Settings → API**
> 3. Copy the **Project URL** and the **anon / public** key
> 4. Use **only** the `anon` key — never the `service_role` key

### 5. Set up the Supabase authentication table

In your Supabase project, ensure the `auth.users` table is enabled (it is by default). ForensicGuard uses Supabase Auth for user login; no custom table setup is required.

---

## ▶️ Running the Application

**Always run as Administrator:**

```powershell
# From within the Forensic-project directory, with .venv active:
python main.py
```

On startup you will see:

```
======================================================================
        FORENSICGUARD - REAL TIME MONITOR
======================================================================

All Monitors Started Successfully...
```

The login window will appear. After signing in, the main dashboard opens showing the live event feed.

---

## 🧪 Running the Tests

```powershell
# Activate venv first
.venv\Scripts\Activate.ps1

# Run the core test suite
python -m unittest -v test_rule_registry test_copy_rules test_rules_4_5 test_rules_6_25 test_rules_26_50 test_logger_encoding

# Run all tests
python -m unittest discover -v
```

**Test results (verified):** 58/60 tests pass. Two tests require a running display session (PySide6 QApplication) and will be skipped in headless environments.

---

## 🤖 Machine Learning Model

ForensicGuard includes a pre-trained **Random Forest classifier** (`models/forensicguard_rf.pkl`, ~8.4 MB) built with scikit-learn.

**To retrain the model on fresh forensic event data:**

```powershell
# 1. Collect forensic events by running the application normally
# 2. Build a labelled dataset from the SQLite database
python ml/build_dataset.py

# 3. Retrain (see training/ directory for training scripts)
```

**Model inputs:** Aggregated feature vectors from `features/feature_aggregator.py` (event type counts, timing intervals, process metadata).

**Current status:** The ML layer enhances rule-based detection; it does not replace it.

---

## 🔒 Security Notes

- `.env` is listed in `.gitignore` and **must never be committed** to version control.
- The SQLite databases (`forensic.db`, `riskalert.db`) contain sensitive endpoint activity data and are excluded from this repository.
- ForensicGuard requires elevated privileges for Security Event Log access. Always audit the source code before running any endpoint monitoring tool.

---

## 🗺️ Rule Engine Status

The 50-rule risk registry (`detection/rule_registry.py`) is at the following implementation level:

| Status | Count | Description |
|---|---|---|
| ✅ FULL | 4 | Rules fully implemented and tested |
| ⚠️ PARTIAL | 41 | Rules implemented with known gaps or TODO items |
| 🔴 PENDING | 5 | Rules 1–5: placeholder stubs, not yet implemented |

---

## 🚀 Future Enhancements

- Full implementation of network monitoring (`network_monitor.py`)
- Browser activity monitoring (`browser_monitor.py`)
- Completion of pending detection rules (Rules 1–5)
- Automated response actions (process kill, quarantine, network block)
- Remote log aggregation and SIEM integration
- Cross-platform support (Linux)

---

## 👥 Project Team

| Name | USN | Role |
|---|---|---|
| H B Sundeep Patil | 1KT23AI021 | Backend, ML Pipeline, Database |
| Keerthan G K | 1KT23AI029 | Detection Engine, Rule Registry |
| Darshan C M | 1KT23AI015 | Monitors, Event Engine |
| Praveen G K | 1KT23AI040 | GUI, Authentication, Reports |

**Institution:** Sri Krishna Institute of Technology, Bengaluru  
**Department:** Artificial Intelligence and Machine Learning  
**Degree:** Bachelor of Engineering (B.E.) — 2023–2027  
**University:** Visvesvaraya Technological University (VTU), Belagavi

---

## 📄 Acknowledgements

This project was developed as a final major project submission for the Bachelor of Engineering programme at Sri Krishna Institute of Technology, Bengaluru, affiliated with Visvesvaraya Technological University, Belagavi.

---

*ForensicGuard — Protecting endpoints through intelligent forensic monitoring.*
