<h1>
  <span class="headline">FastAPI Security SQL Injection</span>
  <span class="subhead">Preventing SQL Injection in FastAPI Controllers</span>
</h1>

**Learning objective:** By the end of this lesson, students will be able to implement secure coding practices to prevent SQL injection vulnerabilities using SQLAlchemy's ORM methods.

## Preventing SQL injection in FastAPI

In this lesson, we will look at how to write secure database queries in FastAPI using SQLAlchemy to prevent SQL injection.

### The secure way to query a database in FastAPI

Here’s an example of a **more secure** endpoint for looking up teas by _name_:

```py
# /controllers/teas.py

from sqlalchemy import or_

@router.get("/teas/{tea_name}", response_model=List[TeaSchema])
def get_teas_by_name(tea_name: str, db: Session = Depends(get_db)):
    tea = db.query(TeaModel).filter(TeaModel.name.ilike(f"%{tea_name}%")).all()
    if not tea:
        raise HTTPException(status_code=404, detail="Tea not found")
    return tea
```

### Why is this more secure?

| Feature                      | How It Prevents SQL Injection                                                        |
| ---------------------------- | ------------------------------------------------------------------------------------ |
| **Parameterized Queries**    | Prevents user input from being executed as SQL code.                                 |
| **ORM Methods**              | Uses `filter()` instead of raw SQL strings, ensuring queries are safely constructed. |
| **Automatic Query Escaping** | SQLAlchemy handles input sanitization, reducing injection risks.                     |

### Is there an even safer way?

Actually, yes!

In our original tea controller, we built a function to retrieve a tea by its **id**:

```py
@router.get("/teas/{tea_id}", response_model=TeaSchema)
def get_single_tea(tea_id: int, db: Session = Depends(get_db)):
    tea = db.query(TeaModel).filter(TeaModel.id == tea_id).first()
    if not tea:
        raise HTTPException(status_code=404, detail="Tea not found")
    return tea
```

This function is **more secure** compared to a function that allows searching by **name**.

Let’s compare them:

| Query Type                               | Example Route | SQL Injection Risk             | Why?                                                                                                                              |
| ---------------------------------------- | ------------- | ------------------------------ | --------------------------------------------------------------------------------------------------------------------------------- |
| **ID-Based Query (Safe)**                | `/teas/5`     | **Very Low** ✅                | - ID is an integer, SQLAlchemy **escapes** it automatically. <br> - Injection is nearly impossible.                               |
| **Name-Based Query (Potentially Risky)** | `/teas/chai`  | **Higher if using raw SQL** ⚠️ | - User input is a **string**, so it could be manipulated. <br> - If raw SQL concatenation was used, an attacker could inject SQL. |

### Why id-based queries are safer

1. **Type Restriction**: `tea_id` is an **integer**, making it harder for an attacker to inject SQL.
2. **Automatic Query Escaping**: SQLAlchemy **safely binds parameters**, preventing direct string manipulation.
3. **More Predictable Behavior**: Searching by **ID** ensures **only one result** is returned, reducing unexpected behavior.

### Key takeaways

- **Always prefer ID-based lookups** when possible. They are safer and more efficient.
- **Avoid raw SQL queries** unless absolutely necessary.
- **Use ORM filtering methods** like `.filter()` and `.ilike()` to safely query data.
- **Never concatenate user input** directly into SQL statements, as this can lead to SQL injection vulnerabilities.

By using these best practices, you can help keep your FastAPI app safe from SQL injection attacks. Sticking to _ID-based lookups_, using _ORM filtering methods_, and _avoiding raw SQL_ will protect your database and keep your API running securely.
