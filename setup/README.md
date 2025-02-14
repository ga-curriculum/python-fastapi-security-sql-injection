<h1>
  <span class="headline">FastAPI Security SQL Injection</span>
  <span class="subhead">Setup</span>
</h1>

## Setup

For this Lesson, we’ll start by working with a pre-existing application that utilizes a PostgreSQL database instance, SQLAlchemy models and a database seeded with initial data.

> If you have a complete, working application from the `Python FastAPI Authorization` Lesson, you may choose to use **a copy of** that codebase as starter code instead of the repo provided below.

### 1. Clone the starter code

We’ve provided a starter repository with the base application code. Clone the repository to your local machine and rename the folder for this lesson:

```bash
git clone https://git.generalassemb.ly/modular-curriculum-all-courses/python-fastapi-authorization-solution python-fastapi-security-sql-injection
```

### 2. Install dependencies:

This project uses `pipenv` for managing dependencies. To install everything the project needs, run:

```sh
 pipenv install
```

### 3. Activate the virtual environment:

```sh
 pipenv shell
```

### 4. Set up the database:

Set up your PostgreSQL database:

- Ensure PostgreSQL is installed and running on your machine.
- Create a database named `teas_db` if it does not already exist:

```bash
createdb teas_db
```

Seed the database with data:

- Run the `seed.py` file to reset the database by dropping existing tables and repopulating it with starter data:

```bash
pipenv run python seed.py
```

> You should see output indicating the database was successfully seeded. If there are any errors, check the `db_URI` in the `config/environment.py` file.

### 5. Start the development server:

```bash
pipenv run uvicorn main:app --reload
```

> You should now have the app running. Visit [`http://127.0.0.1:8000`](http://127.0.0.1:8000) in your browser to confirm it’s working.

You can test each endpoint using FastAPI’s built-in documentation.

> Navigate to FastAPI Documentation: Open [`http://localhost:8000/docs`](http://localhost:8000/docs) in your browser.

Open the application in Visual Studio Code:

```bash
code .
```

### Notes on Configuration

- The database connection string and secret are defined in the `config/environment.py` file:

  ```python
  db_URI = "postgresql://<username>@localhost:5432/teas_db"
  secret = "mysecretcode"
  ```

- Ensure your PostgreSQL instance is configured to allow connections with the provided credentials.
- **_Modify your database connection string to use your username as the `<user.name>`._**

### Setting Up a User in PostgreSQL

To connect to a specific PostgreSQL user, use the following command:

```sh
psql -U <username>
```

#### Handling "Role Does Not Exist" Error

If you see this error:

```sh
Error: FATAL: role "<username>" does not exist
```

it means that the specified user does not exist in PostgreSQL.

#### Creating a New PostgreSQL User

To create the user, run the following command inside `psql`:

```sql
CREATE ROLE "<username>" WITH LOGIN PASSWORD 'your_secure_password';
```

> 🔹 **Replace** `<username>` with your desired username and **choose a secure password**.

This will allow you to connect using one of the following database connection strings:

#### Connection Strings:

If **no password is required**:

```python
db_URI = "postgresql://<username>@localhost:5432/teas_db"
```

If **a password is required**:

```python
db_URI = "postgresql://<username>:<your_secure_password>@localhost:5432/teas_db"
```

This ensures that PostgreSQL correctly authenticates and allows access to the `teas_db` database.
