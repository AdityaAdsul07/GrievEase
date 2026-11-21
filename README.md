# GrievEase

An AI-assisted grievance management system for universities, built with FastAPI and vanilla JavaScript. Students submit complaints online, a trained ML model auto-categorizes them, staff work on assigned complaints, and admins track everything through an analytics dashboard.

## About the Project

Most university complaint systems still run on paper or email threads, which makes them slow and hard to track. GrievEase replaces that with a simple web portal: students log in and submit a complaint, the system predicts its category and priority, auto-assigns it to the right department, and tracks it through to resolution with a full timeline and SLA deadline. Staff and admins get their own dashboards to manage and monitor complaints.

This was built as a solo project to practice building a complete backend (auth, database design, ML integration, and REST API design) alongside a real, working frontend.

## Features

**Student Portal**
- Login/auto-register with name, email, and admission number
- Submit complaints (with optional file attachment)
- Automatic category and priority prediction
- Duplicate complaint detection before submission
- Track complaint status and full timeline
- Rate resolved complaints (1–5 stars)
- Comment on complaints

**Staff Portal**
- Login with credentials created by an admin
- View only complaints assigned to them
- Update status, add remarks and internal notes

**Admin Panel**
- View and filter all complaints
- Analytics dashboard (category, status, priority, monthly trend, department and staff performance)
- Create, update, and deactivate staff accounts
- Bulk status updates
- Manual and automatic escalation of overdue complaints
- CSV export and JSON database backup
- Audit log of all actions
- Department management

**AI / ML**
- Category prediction: TF-IDF + Logistic Regression trained on ~100 sample complaints across 10 categories (Academic, Hostel, IT Support, Fees, etc.)
- Priority prediction: keyword-based rules that classify a complaint as Low / Medium / High / Critical
- Duplicate detection: keyword-overlap check against a student's other open complaints
- A simple assistant on the landing page that predicts category/priority as you describe an issue, before you log in

**System**
- JWT authentication with role-based access (Student / Staff / Admin / Super Admin)
- SLA deadlines per category, with an escalation job that raises complaints through Reminder → Department Head → Admin → Principal based on age
- In-app notifications for every complaint lifecycle event, refreshed automatically; a WebSocket endpoint is also available for live connections
- Optional email notifications on status changes (SMTP)
- File upload support for complaint attachments

## Tech Stack

- **Backend:** Python, FastAPI, SQLAlchemy
- **Database:** SQLite (local) / PostgreSQL (production)
- **AI/ML:** scikit-learn (TF-IDF + Logistic Regression), joblib
- **Auth:** JWT (python-jose), bcrypt (passlib)
- **Frontend:** HTML, CSS, vanilla JavaScript, Chart.js
- **Email:** SMTP via smtplib
- **Deployment:** Render (backend), Vercel (frontend), Neon (PostgreSQL)

## Project Structure

```
GrievEase/
├── frontend/               # Static HTML/CSS/JS
│   ├── index.html          # Landing page (with AI assistant widget)
│   ├── *-login.html        # Student / staff / admin login
│   ├── *-dashboard.html    # Student / staff / admin dashboards
│   ├── style.css
│   └── script.js
│
├── backend/
│   ├── app/
│   │   ├── main.py              # FastAPI app + WebSocket endpoint
│   │   ├── config.py             # Settings from .env
│   │   ├── database.py           # SQLAlchemy engine setup
│   │   ├── models.py             # ORM models
│   │   ├── schemas.py            # Pydantic request/response schemas
│   │   ├── crud.py               # Database operations, SLA & escalation logic
│   │   ├── auth.py                # JWT + password hashing + role guards
│   │   ├── nlp.py                 # ML category prediction (train/load/predict)
│   │   ├── websocket_manager.py
│   │   ├── routes/                # users, complaints, staff, notifications, admin
│   │   ├── services/               # notifications, email, escalation
│   │   └── ml/                     # trained model + vectorizer (.pkl)
│   ├── requirements.txt
│   └── Procfile
│
├── database/
│   └── sample_data.csv     # Sample complaint data for demo/testing
│
└── deployment/
    ├── render.yaml
    └── vercel.json
```

## Installation and Setup

**Prerequisites:** Python 3.11+

```bash
git clone https://github.com/yourusername/grievease.git
cd grievease/backend

python -m venv venv
source venv/bin/activate      # Windows: venv\Scripts\activate

pip install -r requirements.txt

cp .env.example .env          # then edit .env with your values

uvicorn app.main:app --reload
```

The backend runs at `http://127.0.0.1:8000` (interactive API docs at `/docs`).

Open `frontend/index.html` in a browser, or serve it with any static server (e.g. VS Code Live Server) so it can reach the backend at `http://127.0.0.1:8000`.

**Default credentials**

| Role | Login | Notes |
|---|---|---|
| Admin | `admin` / `admin123` (from `.env`) | Created automatically on first run |
| Student | any email + admission number | Auto-registers on first login |
| Staff | created by an admin | No self-registration |

## Environment Variables

Set these in `backend/.env` (see `.env.example`):

| Variable | Description | Default |
|---|---|---|
| `DATABASE_URL` | SQLite or PostgreSQL connection string | `sqlite:///./grievease.db` |
| `SECRET_KEY` | JWT signing key — change this for any real deployment | — |
| `ADMIN_USERNAME` / `ADMIN_PASSWORD` | Default admin account, created on startup | `admin` / `admin123` |
| `FRONTEND_URL` | Deployed frontend URL, for CORS | — |
| `SMTP_USER` / `SMTP_PASS` | Gmail (or other SMTP) credentials for status-change emails | — (email disabled if unset) |

## API Overview

Full interactive documentation is available at `/docs` once the server is running. Key endpoints:

| Method | Endpoint | Description |
|---|---|---|
| POST | `/api/users/student/login` | Student login / auto-register |
| POST | `/api/users/admin/login` | Admin login |
| POST | `/api/users/staff/login` | Staff login |
| GET | `/api/complaints/stats` | Public complaint counters |
| POST | `/api/complaints/json` | Submit a complaint |
| GET | `/api/complaints/` | List complaints (scoped by role) |
| PUT | `/api/complaints/{id}` | Update status (staff/admin) |
| GET | `/api/complaints/predict-category` | Predict category/priority for given text |
| GET | `/api/complaints/analytics` | Dashboard analytics (admin) |
| POST | `/api/admin/run-escalation` | Manually trigger the escalation check |
| GET | `/api/admin/reports/complaints/csv` | Export complaints as CSV |

## Deployment

The project is set up to deploy as two separate services:

- **Backend → Render:** root directory `backend`, build command `pip install -r requirements.txt`, start command `uvicorn app.main:app --host 0.0.0.0 --port $PORT`. See `deployment/render.yaml`.
- **Frontend → Vercel:** serves the `frontend/` folder as a static site. See `deployment/vercel.json`.
- **Database → Neon (PostgreSQL)** for production, SQLite for local development.

After deploying, set `window.__API_BASE__` in the frontend HTML files to your backend URL, and set `FRONTEND_URL` in the backend's environment variables so CORS allows it.

## Known Limitations

- The category model is trained on a small, hand-written sample set (~100 examples), so it works well for common complaint phrasing but isn't a production-grade classifier.
- Priority prediction and duplicate detection use simple keyword-based rules rather than ML.
- The WebSocket connection is used for connection status; in-app notifications are currently delivered by polling the notifications API every 30 seconds rather than being pushed instantly.

## License

[MIT](LICENSE)
