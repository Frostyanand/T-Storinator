# T-Storinator
Uses Telegram Api to upload and manage files on telegram cloud for infinite backup storage .

# 🔧 Core Logic Engine - Telegram Infinite Cloud Storage System

This module handles the **backend brain** of the Telegram Infinite Storage System. It processes user-uploaded files and folders by preparing, encrypting, chunking, and queueing them for upload or download via the Telegram API.

This README documents all design decisions, architecture, and responsibilities of the core logic engine. Use this to stay organized, scale the system, or onboard contributors.

---

## 🚀 Purpose

To manage the **entire file lifecycle**, from user input to upload/download while:
- Maintaining original folder structure
- Supporting AES encryption
- Respecting Telegram API limits (100 calls/min for userbots)
- Avoiding performance bottlenecks by using intelligent scheduling
- Ensuring data integrity and future-safe metadata tracking

---

## 📂 High-Level Workflow

1. **Frontend Input**: Receives file/folder paths + encryption flag from user
2. **Preprocessing**: Recursively walks through folders, generates a flat file list with relative paths and size
3. **Encryption**: Encrypts files using AES-256 (user's key stored in profile)
4. **Chunking**: Splits files > 2GB into 1.9GB chunks
5. **Queue Scheduling**: Mixes small & large files intelligently to avoid hitting Telegram’s API rate limit
6. **Upload/Download Queue**: Files are scheduled for Telegram upload/download
7. **Database Logging**: Tracks metadata, structure, encryption info, and message IDs
8. **Restore Mode**: Rebuilds original folder hierarchy using stored relative paths and downloads files

---

## 🔐 Encryption System

### 📌 What We Use
- **AES-256 CBC mode**
- Implemented using the [`cryptography`](https://cryptography.io/en/latest/) Python package
- One key per user, defined in the user's profile (stored securely in frontend/backend)

### 📁 Encryption Scope
- If `encrypt=True`, the entire file or folder (recursively) will be encrypted
- Only encrypted **at rest**; file is decrypted in-memory before sending on download

### 📦 File Format
- Encrypted files use the same `relative_path`, just with `.enc` extension
- Initialization Vectors (IVs) are generated per file and saved in DB alongside other metadata

---

## 🧠 Intelligent Queue Scheduling

### Problem:
- Telegram allows only **~100 API calls/min** (for upload/download requests via userbot)

### Solution:
- Our scheduler:
  - Prioritizes **mixing large files** (chunked uploads that take minutes) with **small files**
  - Uploads large chunks without rate limit worries
  - Keeps API usage low per minute while bandwidth stays saturated

---

## 🧱 Database Design

We're using **SQLite** for simplicity and portability.

### Tables:
- `files`
  - `file_id`, `relative_path`, `size_bytes`, `encrypted`, `chunked`, `folder_base`, `iv`, etc.
- `chunks`
  - `chunk_id`, `file_id`, `index`, `telegram_msg_id`, `status`
- `folders`
  - (optional) if visual folder tree UI is implemented later
- `users`
  - Encryption key, Telegram info, preferences

---

## 🛠 Folder Structure Retention

### Strategy:
We store **relative paths** from the user's selected root directory.

Example:
User selected path: D:/Backup/Photos Actual file: D:/Backup/Photos/vacation/beach.jpg → Stored as: Photos/vacation/beach.jpg


### On Restore:
- Rebuild folder structure using `os.makedirs(os.path.dirname(relative_path))`
- Files are written to exact original paths

---

## 🧪 Data Integrity Check

Future feature (optional but recommended):
- Store checksums (e.g., SHA256) of each file/chunk
- Verify integrity during download before decrypting

---

## 🔧 Modules To Be Implemented

### ✅ 1. Preprocessing Module
- [ ] Accepts file/folder paths
- [ ] Generates flat file list with:
  - `absolute_path`
  - `relative_path`
  - `size_bytes`
  - `encrypt` flag

### ✅ 2. Encryption Module (AES)
- [ ] Uses user's key from profile
- [ ] AES-256-CBC encryption via `cryptography`
- [ ] IV generation + storage
- [ ] File/stream encryption & decryption

### ✅ 3. Chunking Module
- [ ] Detects files > 2GB
- [ ] Splits into 1.9GB chunks
- [ ] Stores index + metadata

### ✅ 4. Upload/Download Queue
- [ ] Adds jobs to Telegram API engine
- [ ] Avoids API rate limit bottlenecks

### ✅ 5. Intelligent Scheduler
- [ ] Mixes small and large files
- [ ] Keeps upload engine busy but under Telegram limits

### ✅ 6. Database Layer
- [ ] Efficient SQLite interface
- [ ] File, chunk, and folder tracking
- [ ] Easy lookup for restore/download

### ✅ 7. Restore Engine
- [ ] Fetches chunks
- [ ] Reconstructs original files
- [ ] Restores directory hierarchy

---

## 📎 Developer Notes

- Avoid vibe coding; each module should have:
  - Clear input/output
  - Unit tests if possible
  - Defined interface (can be reused later)
- Follow modular architecture: plug pieces together, don’t intertwine them
- Keep logs + error tracking for debugging tricky edge cases (e.g., failed uploads, file locks)

---

## 📌 Tech Stack

- Python 3.10+
- [cryptography](https://pypi.org/project/cryptography/) for AES
- SQLite3 for DB
- Telegram User API (via [Telethon](https://github.com/LonamiWebs/Telethon))
- (Optional) C++ backend wrappers via Cython for perf

---

## 🗓 TODO + Progress Tracker

| Module               | Status  | Notes |
|----------------------|---------|-------|
| Preprocessing        | 🟡 WIP  | Flat file list + rel paths |
| Encryption (AES)     | 🔲 TODO | Use user pass from profile |
| Chunking             | 🔲 TODO | 1.9 GB per chunk |
| Scheduler            | 🔲 TODO | Mix small & large files |
| Upload Engine        | 🔲 TODO | Interface with Telethon |
| Database Layer       | 🟡 WIP  | SQLite schema defined |
| Restore Engine       | 🔲 TODO | Use rel path to recreate tree |

---

## 💡 Ideas for Future

- Cloud sync with versioning
- Download via shared links
- Remote wipe/encrypt/delete option
- Telegram bot-based upload helper

---

## 🙋🏻 Questions or Confusions?

DM the core dev (you 😂) or refer to this README anytime your brain is foggy.

---

