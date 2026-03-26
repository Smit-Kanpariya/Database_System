# MUC Project — Setup Guide (What You Still Need To Do)

Everything in the code is already done. This guide covers only the manual steps left on your side.
Cross-checked against the official CSCI 3461 project spec.

---

## Deadlines at a Glance

| # | What to submit | Where | Due |
|---|---------------|--------|-----|
| 1 | Group member names | Brightspace | Mar 11 ✅ |
| 2 | `design.pdf` (DB schema) | Brightspace | Mar 20 ✅ |
| 3 | `scripts.zip` on Linux server + submit the file path | Brightspace | **Apr 06** ⚠️ |
| 4 | `report.pdf` (web app description) | Brightspace | **Apr 06** ⚠️ |
| 5 | In-class presentation | In class | **Apr 08** ⚠️ |
| 6 | Peer evaluation (your own file) | Brightspace | **Apr 09** ⚠️ |

---

## Step 1 — Put your real credentials in 3 files

Open each file below and replace `uXX` and `password` with your actual SMU username and password.

### File 1 of 3: `scripts/j2sql_parts.sh` — look at lines 7–9
```bash
USER="u42"       # ← change this to your username, e.g. u42
PASS="yourpass"  # ← change this to your password
DB="u42"         # ← same as your username
```

### File 2 of 3: `scripts/j2sql_supp+order.sh` — look at lines 7–9
```bash
USER="u42"
PASS="yourpass"
DB="u42"
```

### File 3 of 3: `public_html/app.py` — look at lines 17–21
```python
DB_CONFIG = {
    'user':     'u42',        # ← your username
    'password': 'yourpass',   # ← your password
    'host':     'dbcourse.cs.smu.ca',
    'database': 'u42',        # ← same as username
    'connection_timeout': 5
}
```

---

## Step 2 — Upload the project to the SMU Linux server

Open a terminal on your Mac and run:

```bash
scp -r /Users/smitcanbe/Desktop/WINTER-26/DBcourse/Project/ u42@dbcourse.cs.smu.ca:~/MUC-Database-System/
```

> ⚠️ The folder **must** be called `MUC-Database-System` exactly — the Python scripts look for files using that folder name. Replace `u42` with your username.

Then log into the server:

```bash
ssh u42@dbcourse.cs.smu.ca
```

Once logged in, check where exactly you are:

```bash
pwd
# This tells you your home directory path, e.g. /home/course/u42
```

Write this down — you'll need it for the `scripts.zip` submission path.

---

## Step 3 — Create the database tables and load all data

On the server, run the scripts **in this exact order** (parts must go first):

```bash
cd ~/MUC-Database-System/scripts

bash j2sql_parts.sh
```

You should see:
```
Parts data import complete!
```

Then immediately run:

```bash
bash "j2sql_supp+order.sh"
```

You should see:
```
Suppliers and Orders data import complete!
```

### If you get "JSON file not found" error:
The Python scripts look for data files at `/home/course/u42/MUC-Database-System/data/`.
If your home folder is different, update the `BASE_DIR` line near the top of these 3 files:
- `scripts/parse_parts.py`
- `scripts/parse_suppliers.py`
- `scripts/parse_orders.py`

Change the line to match your actual path, for example:
```python
BASE_DIR = Path("/home/u42/MUC-Database-System")
```

### If you need to re-run (tables already exist):
The scripts use `CREATE TABLE IF NOT EXISTS` so re-running won't crash. But data may duplicate.
To start fresh, log into MySQL and drop the tables first:
```bash
mysql -u u42 -p u42
```
```sql
DROP TABLE IF EXISTS OrderPart;
DROP TABLE IF EXISTS Orders;
DROP TABLE IF EXISTS SupplierPhone;
DROP TABLE IF EXISTS supplier;
DROP TABLE IF EXISTS parts;
EXIT;
```
Then re-run the scripts from the top.

---

## Step 4 — Start the web app on the server

```bash
cd ~/MUC-Database-System/public_html
pip3 install -r requirements.txt
python3 app.py
```

App will run on port 5010. Open in your browser:

```
http://dbcourse.cs.smu.ca:5010
```

> Keep this terminal window open while the app is running. If you close it, the app stops.
> For the presentation, start the app right before class.

---

## Step 5 — Create `scripts.zip` on the server and submit to Brightspace

The prof requires you to zip it **on the Linux server** (not your laptop). While logged into the server:

```bash
cd ~/MUC-Database-System

zip scripts.zip \
  scripts/make_tables.sql \
  scripts/j2sql_supp+order.sh \
  scripts/parse_suppliers.py \
  scripts/parse_orders.py \
  scripts/parse_parts.py
```

Then find the full path of the zip file:

```bash
echo "$(pwd)/scripts.zip"
# example output: /home/course/u42/MUC-Database-System/scripts.zip
```

Go to **Brightspace → Assignment 3** and submit that path as text. Due **Apr 06**.

---

## Step 6 — Write `report.pdf`

The report is a short document describing how your web app works. Cover these points:

### Section 1 — Project Overview (2–3 sentences)
Say that the app is built with Python Flask, connects to a MySQL database on `dbcourse.cs.smu.ca`, and lets users perform 4 operations on the MUC parts/supplier/orders data.

### Section 2 — File Structure
List the key files and what each one does:

| File | What it does |
|------|-------------|
| `app.py` | The main Flask app. Defines all 4 routes and the SQL queries for each operation |
| `static/style.css` | All the visual styling for the entire app |
| `templates/base.html` | The shared layout — sidebar, navigation, flash messages |
| `templates/index.html` | The home/dashboard page |
| `templates/show_table.html` | Show Table page |
| `templates/add_supplier.html` | Add Supplier form |
| `templates/annual_expenses.html` | Annual Expenses results |
| `templates/budget_projection.html` | Budget Projection results |
| `templates/supplier_details.html` | Shown after a supplier is added |

### Section 3 — How They Work Together
When the user clicks a link in the browser, it sends a request to `app.py`. Flask runs the right function, queries MySQL, and passes the result to a template. The template fills in the data and sends back an HTML page.

### Section 4 — The 4 Operations
Describe each operation in 2–3 sentences:

1. **Show Table** — User picks a table name from a dropdown. Flask runs `SELECT * FROM <table>` and passes the rows to the template which renders a table.

2. **Add Supplier** — User fills in ID, name, email, and phone numbers. Flask runs `INSERT INTO supplier` and `INSERT INTO SupplierPhone` inside a transaction. If the ID already exists, it rolls back and shows an error.

3. **Annual Expenses** — User enters a start and end year. Flask runs a `JOIN` query across `Orders`, `OrderPart`, and `parts` to calculate total spending per year in that range.

4. **Budget Projection** — User enters N years and an inflation rate. Flask fetches the total expense for 2022 (the most recent full year), then calculates compounded projections for each future year.

Save as `report.pdf` and submit to Brightspace by **Apr 06**.

---

## Step 7 — In-class presentation checklist (Apr 08)

Before you walk into class:

- [ ] App is running on the server (`python3 app.py`)
- [ ] Database has data in it (Step 3 was completed)
- [ ] You can open `http://dbcourse.cs.smu.ca:5010` in a browser

### Script for the demo:

| What to say | What to do |
|-------------|-----------|
| "Show Table" | Choose `parts` from dropdown → click Show Table → 100 rows appear |
| "Show Table again" | Choose `supplier` → rows appear |
| "Add Supplier" | Enter ID: `999`, Name: `Test Co`, Email: `test@test.com`, Phone: `555-0000` → click Add |
| "Annual Expenses" | Enter Start: `2018`, End: `2022` → click Show Expenses → table + totals appear |
| "Budget Projection" | Enter Years: `5`, Rate: `3` → click Project → shows 2023–2027 projections |

---

## Step 8 — Peer evaluation (individual, Apr 09)

Write a short evaluation of **each of your group members** (not yourself). Comment on:
- How much they contributed
- What specific parts they worked on
- Their communication and teamwork

Submit your own file in Brightspace. This is confidential — do not share with the group.

---

## Everything That's Already Done (no action needed)

| File | Status |
|------|--------|
| `scripts/parts_table.sql` | ✅ Ready |
| `scripts/j2sql_parts.sh` | ✅ Ready (uses variables) |
| `scripts/make_tables.sql` | ✅ Ready |
| `scripts/j2sql_supp+order.sh` | ✅ Ready (uses variables) |
| `scripts/parse_parts.py` | ✅ Ready |
| `scripts/parse_suppliers.py` | ✅ Ready |
| `scripts/parse_orders.py` | ✅ Ready |
| `public_html/app.py` | ✅ All 4 operations implemented |
| `public_html/templates/` | ✅ All 7 templates done |
| `public_html/static/style.css` | ✅ Done |
| `data/parts_100.json` | ✅ Present |
| `data/suppliers_100.json` | ✅ Present |
| `data/orders_4000.json` | ✅ Present |
