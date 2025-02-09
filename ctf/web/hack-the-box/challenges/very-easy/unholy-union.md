# Unholy Union

So we start off with a clear SQLI, we can input some text and see how the database reacts:This writeup documents the process of exploiting a SQL Injection vulnerability in a web application to retrieve a hidden flag. The challenge involved exploring how unsafe input handling could expose sensitive database information.

<figure><img src="../../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

During the initial interaction with the web application, I identified an input field that appeared to interface directly with the backend database. By submitting various payloads, it became clear that the input was vulnerable to SQL Injection.

*   Observed Error:

    > "The used SELECT statements have a different number of columns"

This message suggested that the application attempted to execute a SQL query with mismatched columns, giving a clue about the vulnerability.

I began by injecting a simple SQL payload (`'`) to confirm the vulnerability. As expected, the database responded with an error, indicating improper sanitization of user input.

<figure><img src="../../../../../.gitbook/assets/image (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

To exploit the vulnerability further, I enumerated the number of columns in the query using `UNION SELECT`. The goal was to match the query's structure so the database would not throw an error.\
When the number of columns in the `UNION SELECT` statement matched the original query, the error stopped. This revealed the exact number of columns.

<figure><img src="../../../../../.gitbook/assets/image (2) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>no error with 5 elements</p></figcaption></figure>

After matching the query structure, I leveraged the `information_schema.tables` view to enumerate all table names in the database.

```
'UNION SELECT 1, table_name, 3, 4, 5 FROM information_schema.tables-- -
```

<figure><img src="../../../../../.gitbook/assets/image (3) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Among the tables, I identified one named **`flag`**, which seemed to hold the sensitive information.

<figure><img src="../../../../../.gitbook/assets/image (4) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

Next, I listed the columns of the `flag` table using`information_schema.columns`.

```
'UNION SELECT 1, column_name, 3, 4, 5 FROM information_schema.columns WHERE table_name='flag'-- -
```

<figure><img src="../../../../../.gitbook/assets/image (5) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption></figcaption></figure>

The `flag` table contained a column named `flag`.

With the `flag` column identified, I crafted a final payload to retrieve its content.

<figure><img src="../../../../../.gitbook/assets/image (6) (1) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p>'UNION SELECT 1, flag, 3, 4, 5 FROM flag-- -</p></figcaption></figure>
