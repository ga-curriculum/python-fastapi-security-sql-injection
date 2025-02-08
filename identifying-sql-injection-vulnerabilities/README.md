<h1>
  <span class="headline">SQL Injection Vulnerabilities</span>
  <span class="subhead">Identifying SQL Injection Vulnerabilities</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to identify SQL injection vulnerabilities in SQLAlchemy controller methods and understand the potential risks and consequences of such vulnerabilities.

## SQL injection example in FastAPI

In this lesson, we will examine an example of a vulnerable API endpoint to understand how SQL injection works. This example is purely for learning purposes and should never be used in a production codebase.

### What is sql injection?

SQL injection happens when an attacker manipulates the SQL code your application uses to interact with the database. This can allow the attacker to:

- Access data they are not authorized to view.
- Modify or delete important data.
- Compromise the security and integrity of your application.

Let’s explore an example to understand how this can occur.

### A vulnerable endpoint example

Below is an example of a FastAPI endpoint vulnerable to SQL injection. The endpoint allows users to search for teas in a database by name.

> Feel free to copy and paste this code into your tea controller to try it out. This is just for demonstration purposes and will not damage your application.

```py
# /controllers/teas.py

from sqlalchemy import text # add import for text

@router.get("/lookup-teas-vulnerability/{tea_name}", response_model=List[dict])
def get_single_tea(tea_name: str, db: Session = Depends(get_db)):
    results = db.execute(text(f"SELECT * FROM teas WHERE name LIKE '%{tea_name}%'"))
    keys = list(results.keys())
    values = results.fetchall()

    if not values:
        raise HTTPException(status_code=404, detail="No teas found")

    list_of_dicts = [
        {key: value_set[idx] for idx, key in enumerate(keys)} for value_set in values
    ]

    return list_of_dicts
```

### What's happening in this code?

1. The endpoint receives a `tea_name` parameter from the user via url

```sh
http://localhost:8000/lookup-teas-vulnerability/chai
```

2. It directly injects this parameter into a raw SQL query using `text` from SQLAlchemy.

3. The query looks for teas in the database whose names match the input provided by the user.

### How an attacker exploits this vulnerability

Imagine a legitimate query where the user provides `chai` as the `tea_name`. The SQL query would look like this:

```sql
SELECT * FROM teas WHERE name LIKE '%chai%';
```

This query retrieves all teas containing "chai" in their name. However, if an attacker provides the following malicious input:

```text
chai%'; SELECT * FROM users;--'
```

Using the url:

```text
http://localhost:8000/lookup-teas-vulnerability/chai%25';%20SELECT%20*%20FROM%20users;--
```

The SQL query becomes:

```sql
SELECT * FROM teas WHERE name LIKE '%chai%'; SELECT * FROM users;
```

#### Why this is dangerous:

- The `;` ends the first query, and the second query (`SELECT * FROM users`) retrieves all user data from the database.
- The `--` comments out any additional SQL, ensuring the injected code runs without errors.
- The result is unintended access to sensitive data in the `users` table.

### Preventing sql injection

To secure your application against SQL injection, always follow these best practices:

1. **Use parameterized queries**: Avoid directly injecting user input into SQL queries. Instead, use placeholders to separate SQL logic from user input.

   Example:

   ```py
   results = db.execute(text("SELECT * FROM teas WHERE name LIKE :tea_name"), {"tea_name": f"%{tea_name}%"})
   ```

2. **Sanitize user input**: Clean and validate all user inputs before processing them.

3. **Use an ORM**: Rely on frameworks like SQLAlchemy to abstract away raw SQL and reduce vulnerabilities.
