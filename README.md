# Reading Hall Management System

A server-rendered Flask app for managing one or more reading halls
("libraries"), each with its own seat range, students, and fees.
Protected by a single admin login — no public access, no student accounts.

Stack: Flask + Jinja2 + SQLite + hand-written CSS. No JS frameworks, no REST API.

## Run it

```bash
pip install -r requirements.txt
python app.py
```

Open **http://127.0.0.1:5000**. On first run you'll be sent to `/setup` to
create the one admin account (username + password). From then on every page
requires login — logging out or losing your session bounces you back to the
login page.

The SQLite database (`database.db`) is created automatically on first run.

## What's new in this version

- **Admin login.** Nobody can view or change data without signing in.
  Session cookie is signed with `app.secret_key` in `app.py` — change that
  value to something long and random before using this anywhere but your own
  machine. There's also an in-app "Change Password" screen under the account
  menu.
- **Multiple libraries.** From the "Libraries" screen you can add as many
  reading halls as you want, each with its own name and seat range. Every
  dashboard, seat grid, student list, and fee report is scoped to the
  library you're currently managing.
- **Custom seat ranges.** Instead of a hard-coded seat count, each library
  is created with a **start seat number** and an **end seat number**
  (e.g. 101–150). Seats are generated for that inclusive range. Editing a
  library's range is blocked if it would strand an occupied seat outside
  the new range.
- **Vacant Seats tab.** A dedicated page per library that buckets every
  seat into:
  - **Fully Vacant** — nobody in either half
  - **First Half Vacant** — second half is taken, first half is free
  - **Second Half Vacant** — first half is taken, second half is free

  Fully-occupied seats (Full Time, or both halves taken) never appear here.

## Project layout

```
reading_hall/
├── app.py                 All routes, auth, DB access, and business-rule logic
├── requirements.txt
├── templates/              Jinja2 templates (base.html + one per page)
└── static/style.css        Hand-written CSS (no frameworks)
```

## Core seat business rule (per library)

Each seat can be occupied as **Full Time**, **First Half**, and/or **Second
Half**. A seat can have up to two simultaneous occupants — one per half —
but never a half plus Full Time, and never two people in the same half.
This is enforced in `check_conflict()` in `app.py` on every add and edit.

## Routes

| Method | Path                                     | Purpose                              |
|--------|-------------------------------------------|---------------------------------------|
| GET    | `/`                                        | Redirects to setup / login / libraries |
| GET/POST | `/setup`                                | One-time admin account creation        |
| GET/POST | `/login`                                | Admin login                            |
| GET    | `/logout`                                 | End session                            |
| GET/POST | `/account/password`                     | Change admin password                  |
| GET    | `/libraries`                               | List of libraries                      |
| GET/POST | `/libraries/add`                        | Create a library (name + seat range)   |
| GET/POST | `/libraries/<id>/edit`                  | Rename / resize a library               |
| GET/POST | `/libraries/<id>/delete`                | Delete a library and its entries        |
| GET    | `/library/<id>/dashboard`                 | Summary + seat layout for one library  |
| GET    | `/library/<id>/seats`                     | Full seat grid                         |
| GET    | `/library/<id>/vacant-seats`              | Fully / First Half / Second Half vacant lists |
| GET    | `/library/<id>/seat/<seat_no>`            | Seat detail (both halves)              |
| GET/POST | `/library/<id>/add`                     | Add a new seat entry                   |
| GET/POST | `/library/<id>/edit/<entry_id>`         | Edit an existing entry                 |
| GET/POST | `/library/<id>/delete/<entry_id>`       | Confirm + vacate a seat                |
| POST   | `/library/<id>/renew/<entry_id>`          | Quick fee/due-date renewal             |
| GET    | `/library/<id>/students`                  | Search by name / mobile / seat         |
| GET    | `/library/<id>/fees-due`                  | Overdue and due-soon lists             |

All `/library/...` and `/libraries...` routes require an active admin session.
