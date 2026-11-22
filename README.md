# Dhyan: CERT & Study Tracker

A task tracker for students. Track assignments, certifications, chores and anything else, with **no account needed to start**. Verify your email later and your tasks follow you.

I built Dhyan as a learning project to get hands-on with **FastAPI**, **PostgreSQL** and JavaScript's **`fetch()`**. It is a preparatory project, a warm-up for bigger things.

## Links

- **Live demo:** [dhyan.onrender.com](https://dhyan.onrender.com/)
- **Source code:** [github.com/Atul-Verma-Dev/dhyan-a-todo-app](https://github.com/Atul-Verma-Dev/dhyan-a-todo-app)

## Screenshots

<!-- Save images in a screenshots/ folder, then replace each placeholder path below. -->

### Home: pick a category and add a task
![Home page](screenshots/home.png)

### Dynamic forms (Academia, Certification, Chore, Other)
![Task form](screenshots/form.png)

### Task list: update and delete
![Task list](screenshots/task-list.png)

### Email sync: OTP verification
![Email OTP](screenshots/otp.png)

### Verified state
![Verified](screenshots/verified.png)

## Features

- **Four task categories:** Academia, Certification, Chore, Other. Each has its own form, built dynamically in the DOM.
- **Full CRUD:** add, view, update and delete tasks.
- **Optimistic delete:** the card disappears instantly and the request follows.
- **Anonymous sessions:** a UUID cookie (`httponly`, `samesite=lax`, 1 year) lets you start with no login.
- **Email OTP sync:** verify your email and all anonymous tasks are moved onto it.
- **Hardened OTP flow:**
  - 5-digit code generated with Python's `secrets` module
  - expires after 5 minutes
  - locks after 5 wrong attempts
  - maximum 3 code sends per session
  - Resend Code and Change Email options

## Tech stack

| Layer | Tools |
|---|---|
| Backend | Python, FastAPI, Pydantic |
| Database | PostgreSQL (Neon), psycopg2 |
| Frontend | HTML, CSS, vanilla JavaScript (`fetch`, async/await), Jinja2 |
| Email | Brevo transactional email API (`requests`) |
| Hosting | Render (free tier) |

## Project structure

```
.
├── main.py              # FastAPI app, routes, OTP logic
├── requirements.txt
├── .env                 # secrets (NOT committed)
├── .gitignore
├── screenshots/         # README images
├── static/              # static files (contains .gitkeep)
└── templates/
    ├── home.html        # task forms + email sync
    ├── task-list.html   # task list, update and delete
    ├── about.html
    └── feedback.html
```

## Getting started

### 1. Clone and install

```bash
git clone https://github.com/Atul-Verma-Dev/<repo-name>.git
cd <repo-name>
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt
```

`requirements.txt`:

```
fastapi
uvicorn
jinja2
python-dotenv
psycopg2-binary
pydantic
requests
```

### 2. Create a `.env` file

```
DATABASE_URL=postgresql://user:password@host/dbname?sslmode=require
BREVO_API_KEY=your_brevo_api_key
SENDER_EMAIL=your_verified_brevo_sender@example.com
```

- `DATABASE_URL`: your Neon (or any PostgreSQL) connection string.
- `BREVO_API_KEY`: an API key from Brevo (**SMTP & API → API Keys**; it starts with `xkeysib-`).
- `SENDER_EMAIL`: a sender address you have verified in Brevo.
- Make sure `.env` is listed in `.gitignore` **before your first commit**.

The `Data` table is created automatically on startup.

### 3. Run

```bash
uvicorn main:app --reload
```

Open `http://127.0.0.1:8000`.

## API endpoints

| Method | Route | Purpose |
|---|---|---|
| GET | `/` | Home page |
| GET | `/tasks` | Task list page |
| POST | `/api/person` | Add a task |
| POST | `/api/update` | Fetch one task for editing |
| POST | `/api/update_db` | Save an edited task |
| POST | `/api/delete` | Delete a task |
| POST | `/api/email_id` | Send an OTP to an email |
| POST | `/api/code` | Verify an OTP |
| GET | `/api/verified` | Check whether the session is verified |

## Database

One table, `Data`:

| Column | Type |
|---|---|
| id | INT, identity, primary key |
| email_or_uuid | VARCHAR(500): session UUID, or email once verified |
| type | VARCHAR(20) |
| title | VARCHAR(500) |
| task_type | VARCHAR(20) |
| subject | VARCHAR(500) |
| description | TEXT |
| deadline | TIMESTAMP |

## How sessions and OTP work

1. A first-time visitor gets a UUID cookie. Tasks are saved under that UUID.
2. The visitor enters an email and the server sends a 5-digit code through the Brevo API (in a background task).
3. On the correct code, `UPDATE Data SET email_or_uuid = <email> WHERE email_or_uuid = <uuid>` moves every anonymous task onto the email.
4. All later reads and writes for that session use the email.

## Security notes

- Every SQL query is **parameterized** (`%s`) to prevent SQL injection.
- Update and delete use `WHERE id = %s AND email_or_uuid = %s`, so users can only change their own tasks.
- OTP codes use `secrets`, expire, and are attempt-limited and send-limited.
- Secrets live in environment variables, never in git.

## Known limitations

- **Sessions are stored in an in-memory dict.** A server restart or free-tier sleep resets verification. Planned fix: store sessions in PostgreSQL.
- **`/api/update` (the read) has no ownership check yet.** Planned fix: add `AND email_or_uuid = %s` like the other routes.
- **Brevo free plan limit:** 300 emails per day, which is plenty for a learning project.
- Free-tier hosting sleeps after inactivity, so the first request after idle can take about a minute.

## Deploying on Render

1. Push the repo to GitHub (confirm `.env` is not tracked).
2. Render: **New → Web Service**, then pick the repo.
3. Build command: `pip install -r requirements.txt`
4. Start command: `uvicorn main:app --host 0.0.0.0 --port $PORT`
5. Environment variables: `DATABASE_URL`, `BREVO_API_KEY`, `SENDER_EMAIL`, and `PYTHON_VERSION=3.12.3` (matches `psycopg2-binary` wheels).
6. Deploy.

## What I learned

- Cookies, sessions and designing a safe OTP flow
- Parameterized queries and authorization checks
- Connecting a frontend to an API with `fetch()` and async/await
- Environment variables, deployment, and serverless vs always-on hosting
- How SMTP differs from HTTPS email APIs, and why I moved OTP delivery to Brevo after Gmail SMTP failed from the hosted server
- Debugging with server logs, the browser console and the Network tab

## Notes on AI assistance

The CSS styling and the OTP expiry/attempt-limit code were written with AI help. The backend structure, session design, ownership checks and fetch logic are my own, and I debugged the issues along the way myself.

## Roadmap

- [ ] Move sessions into PostgreSQL
- [ ] Add ownership check to `/api/update`
- [ ] Tidy imports and add basic tests

## Author

**Atul Verma**: [github.com/Atul-Verma-Dev](https://github.com/Atul-Verma-Dev)
