🔍 ICMR + HITEK Search API

A high‑performance search API for billions of Indian records – featuring ICMR + HITEK (phone, Aadhaar, name) and Telegram (username / user_id / phone) lookup. Built with FastAPI + Gradio + DuckDB, with remote Parquet indexes hosted on HuggingFace.

---

🚀 Installation

**Option 1 – Source install (from GitHub)**

```bash
git clone https://github.com/cyber-punk-h/Icmr-and-hitek.git
cd Icmr-and-hitek
python -m venv venv
# Windows
venv\Scripts\activate
# Linux/macOS
source venv/bin/activate
pip install -r requirements.txt
```

**Option 2 – Direct pip install of dependencies**

```bash
pip install fastapi gradio duckdb httpx uvicorn pydantic
```

Or, if a `requirements.txt` is already present, run:

```bash
pip install -r requirements.txt
```

Then start the app:

```bash
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
```

---

https://img.shields.io/badge/version-1.0-blue
https://img.shields.io/badge/python-3.9%2B-blue
https://img.shields.io/badge/Render-ready-success

---

📌 Features

· 🔎 Instant search – phone, Aadhaar, name, father’s name, address, district, state, pincode, town, source.
· 💬 Telegram lookup – search by username or user_id (partial/contains allowed for username) → returns first_name, last_name, phone, email, status, linked accounts.
· ⚡ Billions of records – 2.5B ICMR/HITEK + 1.9B Telegram users, queryable via DuckDB over HTTP/Parquet.
· 🎯 Exact & contains modes – flexible searching.
· 📱 Gradio UI – user‑friendly web interface for non‑technical users.
· 🧩 RESTful API – FastAPI with Swagger docs (/docs).
· 🚀 Dedup & connection – deduplicates results and shows connected numbers (phone, other, Aadhaar).
· 🕒 Auto‑pinger – built‑in background task to keep the app awake (though external uptime monitor is recommended for free hosting tiers).
· 👨‍💻 Developer credit – prominently displayed.

---

🧰 Tech Stack

· Python 3.9+
· FastAPI – REST API framework
· Gradio – UI interface
· DuckDB – in‑process analytical database with Parquet and HTTPFS support
· HuggingFace – remote storage for partitioned Parquet indexes
· Uvicorn – ASGI server
· httpx – async HTTP client for pinger

---

🌍 Deploy on Render (Free/Paid)

1. Push code to GitHub – ensure main.py, requirements.txt, and this README are in the root.
2. On Render.com, create a New Web Service and connect your repo.
3. Fill in:
   · Name: icmr-search (or your choice)
   · Environment: Python 3
   · Build Command: pip install -r requirements.txt
   · Start Command: uvicorn main:app --host 0.0.0.0 --port $PORT
4. (Optional) Set environment variables under Advanced:
   · ICMR_INDEX_SOURCE – remote (default)
   · ICMR_PARALLEL – 2 (threads for DuckDB)
   · ICMR_THREADS_PER_CONN – 2
   · PORT – Render sets this automatically, don’t override.
5. Click Create Web Service.

Your app will be live at https://yourapp.onrender.com within a few minutes.

---

📡 API Endpoints

Endpoint Method Description
/ GET Root – app metadata, record count, indexes, developer credit.
/health GET Health check – returns status: ok and index readiness.
/search GET Primary search endpoint.
/search/parallel POST Batch search – up to 50 queries in one request.
/docs GET Swagger UI interactive documentation.

🔍 Search Examples

1. Unified search (auto‑detects phone / Aadhaar)

```http
GET /search?q=9876543210&limit=10
```

2. Search by specific field

```http
GET /search?field=phoneNumber&q=9876543210
GET /search?field=aadharNumber&q=123456789012
GET /search?field=name&q=Rahul&mode=contains
```

3. Telegram search (username / user_id / phone)

```http
GET /search?field=username&q=umar&mode=contains
GET /search?field=user_id&q=1899021651
GET /search?field=phone&q=79899152324
GET /search?field=first_name&q=Umar&mode=contains
```

4. Use mobile alias (same as q)

```http
GET /search?mobile=9876543210
```

5. Batch search (POST)

```json
POST /search/parallel
{
  "queries": [
    {"field": "phoneNumber", "value": "9876543210", "limit": 5},
    {"field": "aadharNumber", "value": "123456789012"}
  ],
  "limit": 10
}
```

Response format

All search endpoints return JSON with:

· query / number – searched term
· count – number of results
· results – array of records, each with:
  · All original fields (name, fathersName, phoneNumber, aadharNumber, otherNumber, address, district, pincode, state, town, source)
  · connected_numbers – list of phone/other/Aadhaar numbers linked to this record.

---

🧠 How It Works

· The app does not store the full raw database locally.
· It uses remote Parquet indexes hosted on HuggingFace, partitioned into 7 files for phone and 7 for Aadhaar; Telegram uses 4 Parquet parts (TGDATA BY DEADLOX P1–P4).
· DuckDB reads these indexes directly over HTTP (via httpfs extension) and executes queries efficiently.
· ICMR/HITEK search is exact match on indexed fields; Telegram supports exact & contains (partial) search on username, user_id, phone, first_name, etc.
· Duplicate records are collapsed to 1 per person (DUPLICATE_CAP) to avoid repeated identical results.

---

⏰ Keeping the App Alive (Free Tier)

Render’s free tier sleeps after 15 minutes of inactivity.
While the app has an internal pinger that pings /health every 2 minutes, this is internal and does not prevent sleep because sleep is based on external traffic.

➡️ To keep your instance awake, set up an external uptime monitor (like UptimeRobot or cron-job.org) that hits your public /health endpoint every 5 minutes.

Example: https://yourapp.onrender.com/health

---

🛠 Environment Variables

Variable Default Description
ICMR_HF_INDEX_BASE https://huggingface.co/datasets/Kzr0xx/icrm-hitek-full-db-mixed/resolve/main Base URL for ICMR/HITEK remote Parquet indexes.
TG_HF_INDEX_BASE https://huggingface.co/datasets/Kzr0xx/telegram/resolve/main Base URL for Telegram remote Parquet files.
ICMR_INDEX_SOURCE remote (Future use) – currently only remote is supported.
ICMR_PARALLEL 2 Number of parallel workers for search requests.
ICMR_THREADS_PER_CONN 2 DuckDB threads per connection.
PORT 7860 (Gradio default) Port the server listens on – set by Render automatically.

---

🔧 Dependencies

· fastapi
· gradio
· duckdb
· httpx
· uvicorn
· pydantic

All listed in requirements.txt.

---

👨‍💻 Credits

· Developer: @kzr0x
· Channel: @api_wallah
· Dataset: ICMR + HITEK Full DB Mixed – hosted on HuggingFace.

---

📝 Limitations

· Remote-only mode – the app does not support local database; all indexes are downloaded via HTTP on first query.
· Name search – only available in contains mode, and it may be slower (uses ILIKE on the phone index view).
· OtherNumber – not indexed, so exact searches on it will return no results.
· Rate limiting – not implemented in this version.
· Data accuracy – the data is sourced from public datasets; the app only provides search capabilities and does not verify authenticity.

---

📄 License

This project is for educational and research purposes only.
The data is publicly available on HuggingFace; use it responsibly and in compliance with applicable laws.

---

🤝 Contributing

Pull requests are welcome! For major changes, please open an issue first to discuss what you would like to change.

---

📧 Contact

For queries, reach out via Telegram.

---

Happy Searching! 🔍
