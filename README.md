# News Application

A Django news application where journalists write articles for publishers
(which are approved and posted by th publishers' editors) or publish
independent articles of their own. Readers subscribe to publishers and
journalists to get new articles by email. A RESTful API with token
authentication serves the articles to third-party clients.

GitHub repository: https://github.com/valkyrie-dotcom/news-app

## Features

* **Roles:** Reader, Journalist, Editor and Publisher, each in a group with
  its own permissions.
* **Readers:** read approved articles and newsletters, subscribe to publishers
  and journalists, and receive approved articles by email.
* **Publishers:** register with the name of their publication, then choose
  the editors and journalists who work for it on the **My publication** page.
* **Journalists:** write, edit and delete their own articles and newsletters.
  Articles written for a publisher wait for one of its editors to approve
  them. Independent articles (no publisher) are published after saving.
* **Editors:** review and approve articles of the publishers they  belong
  to, and edit or delete those articles and the newsletters of those
  publishers' journalists. Editors only see their own publishers' articles
  and newsletters. To read everything else, register a separate Reader
  account.
* **Publishing:** when an article is approved, or an independent article is
  saved, it is emailed to subscribers and posted to the app's own
  `/api/approved/` endpoint.
* **Registration:** users confirm their password, and each email address can
  only be used for one account.
* **API:** token authentication and role-based access.

## Requirements

Install these before you start:

* Python 3.12 or later: https://www.python.org/downloads/
* Git: https://git-scm.com/downloads
* MariaDB: https://mariadb.org/download/ (remember the root password you
  choose during installation)

## Installation

The commands below are typed into a terminal:
* On Windows, use **Git Bash** (installed with Git).
* On Mac, use **Terminal**.

### 1. Clone the repository

```bash
git clone https://github.com/valkyrie-dotcom/news-app.git
```

### 2. Go into the project folder

```bash
cd news-app
```

All the commands from here on are run inside this folder.

### 3. Create a virtual environment inside the project folder

```bash
python -m venv venv
```

On Mac, use `python3` instead of `python` if `python` is not found.

### 4. Activate the virtual environment

* Windows (Git Bash):

  ```bash
  source venv/Scripts/activate
  ```

* Windows (Command Prompt):

  ```bat
  venv\Scripts\activate
  ```

* Mac or Linux:

  ```bash
  source venv/bin/activate
  ```

You will see `(venv)` at the start of the line once it is active. Activate
it again each time you open a new terminal.

### 5. Install the packages

```bash
pip install -r requirements.txt
```

### 6. Create the database in MariaDB

Open the MariaDB SQL terminal:
* On Windows, open **MySQL Client (MariaDB)** from the Start menu, or open
  **Command Prompt (MariaDB)** from the Start menu and type the command below.
* On Mac, type the command below in Terminal.

```bash
mysql -u root -p
```

Enter the root password you chose when installing MariaDB. The prompt changes
to `MariaDB [(none)]>`. Type these two lines, pressing Enter after each:

```sql
CREATE DATABASE news_db CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
exit;
```

### 7. Create the `.env` file

Copy the example file:

* Git Bash or Mac:

  ```bash
  cp .env.example .env
  ```

* Command Prompt:

  ```bat
  copy .env.example .env
  ```

Open `.env` in a text editor and set `DB_PASSWORD` to your MariaDB root
password. Leave the email lines commented out to have emails printed in the
terminal instead of sent.

### 8. Create the tables

```bash
python manage.py migrate
```

This also creates the **Readers**, **Editors**, **Journalists** and
**Publishers** groups with their permissions.

## Running the project

```bash
python manage.py runserver
```

Open http://127.0.0.1:8000/ in your browser. Press `Ctrl + C` in the
terminal to stop the server.

If no email details are set in `.env`, emails (approved articles sent to
subscribers) are printed in the terminal.

## Using the app

1. Click **Register** and create an **Editor** account and a **Journalist**
   account.
2. Register a **Publisher** account and fill in the publication name.
3. On **My publication**, tick the editor and the journalist, and click
   **Save**.
4. Log in as the journalist and write an article for the publisher.
5. Log in as the editor, open **Review articles** and approve it.
6. Log in as the journalist again and write an article with no publisher.
   It is published immediately.

Journalists can only write articles for publishers they belong to. An
editor who has not been added to a publisher sees a message saying a
publisher needs to add them.

## Running the tests

```bash
python manage.py test news
```

## Using the API

Get a token with your username and password:

```bash
curl -X POST -d "username=reader1&password=YOUR_PASSWORD" http://127.0.0.1:8000/api/token/
```

Send it with every request:

```bash
curl -H "Authorization: Token YOUR_TOKEN" http://127.0.0.1:8000/api/articles/subscribed/
```

| Method | Endpoint | Who |
|--------|----------|-----|
| POST | `/api/token/` | Anyone with a valid login |
| GET | `/api/articles/` | Logged-in users (editors get their publishers' articles only) |
| GET | `/api/articles/subscribed/` | Readers |
| GET | `/api/articles/<id>/` | Logged-in users (editors their publishers' articles only) |
| POST | `/api/articles/` | Journalists |
| PUT | `/api/articles/<id>/` | The publisher's editors, or the article's journalist. Only editors can send `"approved": true`. |
| DELETE | `/api/articles/<id>/` | The publisher's editors, or the article's journalist |
| POST | `/api/approved/` | Editors and journalists (used by the app when an article is published) |
| GET | `/api/newsletters/`, `/api/newsletters/<id>/` | Logged-in users (editors their publishers' journalists' newsletters only) |

Example body for `POST /api/articles/` (leave out `publisher` for an
independent article, which is published immediately):

```json
{"title": "City council approves new fuel levy", "content": "The council voted...", "publisher": 1}
```

## Project structure

```
news-app/
├── news_site/          # Project settings and URLs
├── news/               # The news app
│   ├── models.py       # User, Publisher, Article, Newsletter
│   ├── views.py        # Website pages
│   ├── api_views.py    # API endpoints
│   ├── serializers.py  # API serializers
│   ├── access.py       # Who can see and manage which articles and newsletters
│   ├── utils.py        # Approval: email subscribers, post to /api/approved/
│   ├── forms.py
│   ├── tests.py        # Automated unit tests
│   ├── static/         # Stylesheet
│   └── templates/
├── Planning/           # Planning document and diagrams
├── manage.py
└── requirements.txt
```
