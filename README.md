Mastering curl for CTF Challenges: A Step-by-Step Guide to Finding Flags
As a cybersecurity enthusiast, I’ve been diving into Capture The Flag (CTF) challenges to sharpen my skills. Recently, I tackled a web-based challenge that required using curl to authenticate to a server, manage session cookies, and send JSON POST requests to retrieve a flag. In this post, I’ll walk you through the process I followed to solve the challenge on the server 83.136.251.68:48581, sharing detailed notes on using curl effectively. Whether you’re new to CTFs or looking to brush up on your curl skills, this guide has you covered!

The Challenge Setup
The goal of this CTF challenge was to interact with a web application, authenticate using the credentials admin:admin, and use a search function to find a flag in the format FLAG{...}. The search function required a JSON POST request to /search.php, and I had a valid session cookie: PHPSESSID=hnjcoa6bvghqatlscamgvmh38h. The challenge provided a clue: searching for "london" returned ["London (UK)"], but I needed to find the right search term to get the flag. Let’s break down how I used curl to solve this.

What is curl?
curl (Client URL) is a powerful command-line tool for making HTTP requests. It’s a go-to tool for CTF players, pentesters, and developers because it lets you craft precise requests, manage cookies, and inspect responses. In this challenge, I used curl to:

Authenticate to the server and obtain a session cookie.
Send a JSON POST request to the search endpoint to find the flag.
Below are my detailed notes on each step, with explanations of the curl flags and options I used.

Step 1: Authenticating to the Server
To interact with the web application, I first needed to log in and obtain an authenticated session cookie (PHPSESSID). The login form expected a POST request with the credentials username=admin&password=admin.

Command
bash

Collapse

Wrap

Copy
curl -X POST -d "username=admin&password=admin" -u admin:admin -c cookies.txt http://83.136.251.68:48581/
What Each Flag Does
-X POST:
Tells curl to use the HTTP POST method. The login form requires POST to submit the credentials.
Without this, curl defaults to GET, which would fail.
-d "username=admin&password=admin":
Sends the form data in the request body, mimicking filling out the username and password fields.
The format key=value&key=value is application/x-www-form-urlencoded, the standard for HTML forms.
In Windows Command Prompt, I used quotes ("...") to avoid parsing issues.
-u admin:admin:
Adds HTTP Basic Authentication with the username admin and password admin.
This sends an Authorization: Basic ... header with the Base64-encoded credentials (admin:admin becomes YWRtaW46YWRtaW4=).
The challenge required Basic Auth for some endpoints, though the login step might work without it.
-c cookies.txt:
Saves cookies from the response to cookies.txt.
After a successful login, the server sends a Set-Cookie header (e.g., Set-Cookie: PHPSESSID=hnjcoa6bvghqatlscamgvmh38h; path=/), which this flag stores.
http://83.136.251.68:48581/:
The login endpoint URL. If this fails (e.g., 404 Not Found), try /login or /login.php.
Alternative: View the Cookie Directly
To see the Set-Cookie header without saving to a file:

bash

Collapse

Wrap

Copy
curl -X POST -d "username=admin&password=admin" -u admin:admin -i http://83.136.251.68:48581/
-i:
Includes response headers in the output.
Look for Set-Cookie: PHPSESSID=hnjcoa6bvghqatlscamgvmh38h; path=/ to get the cookie value.
What to Expect
If successful, cookies.txt will contain the PHPSESSID.
The response body should include <em>Type a city name and hit <strong>Enter</strong></em>, confirming you’re logged in.
Troubleshooting
401 Unauthorized: Double-check the credentials or remove -u admin:admin if Basic Auth isn’t needed.
404 Not Found: The login endpoint might be /login or /login.php. Adjust the URL and retry.
No Cookie: Login failed. Check the response for error messages.
Step 2: Sending the Search Request with the Cookie
With the session cookie (PHPSESSID=hnjcoa6bvghqatlscamgvmh38h), I could interact with the search function by sending a JSON POST request to /search.php. The challenge provided a clue: searching for "london" returned ["London (UK)"].

Command
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"london\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
What Each Flag Does
-X POST:
Specifies a POST request, as required by the search function.
-d "{\"search\":\"london\"}":
Sends the JSON payload {"search":"london"} in the request body.
In Windows Command Prompt, I escaped the inner quotes (\") to ensure the JSON is parsed correctly.
-b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h":
Includes the session cookie in the Cookie header.
This authenticates the request, allowing access to the search function without re-entering credentials.
-u admin:admin:
Adds Basic Auth. The challenge required this for some endpoints, but if the cookie alone works, you can omit it.
-H "Content-Type: application/json":
Sets the Content-Type header to application/json, indicating the payload is JSON.
The challenge confirmed this header is necessary.
http://83.136.251.68:48581/search.php:
The search endpoint URL.
Alternative: Use a Cookie File
If you saved the cookie to cookies.txt:

bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"london\"}" -b cookies.txt -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
-b cookies.txt:
Reads cookies from cookies.txt and includes them in the request.
What to Expect
The response should be:

text

Collapse

Wrap

Copy
["London (UK)"]
This isn’t the flag (we’re looking for FLAG{...}), so we need to try different search terms.

Troubleshooting
A valid authentication cookie is required!:
The cookie has expired. Re-authenticate to get a new one (see Step 1).
401 Unauthorized:
Keep -u admin:admin if required, or remove it if the cookie alone is sufficient.
Invalid JSON:
The server might return text/html instead of application/json. Use -v to inspect:
bash

Collapse

Wrap

Copy
curl -v -X POST -d "{\"search\":\"london\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
Step 3: Finding the Flag by Adjusting the Search Term
The response ["London (UK)"] was a clue, not the flag. I needed to guess the correct search term to reveal the flag. Here’s how I iterated through different terms.

Command Template
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"<term>\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
Replace <term> with different search terms.
Terms I Tried
"London (UK)":
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"London (UK)\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
"London Bridge":
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"London Bridge\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
"flag":
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"flag\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
"London, UK":
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"London, UK\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
"Big Ben":
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"Big Ben\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
"Thames":
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"Thames\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
What to Look For
The flag should be in the format:

text

Collapse

Wrap

Copy
["FLAG{london_something}"]
or

text

Collapse

Wrap

Copy
{"result": "FLAG{london_something}"}
Troubleshooting
No Flag:
Try other London-related terms (e.g., "Buckingham Palace", "Tower of London") or CTF-related terms (e.g., "secret", "key").
Content-Type Mismatch:
If the response isn’t JSON, use -v to inspect the raw output.
Bonus curl Tips
Follow Redirects:
Some login forms redirect after authentication. Use -L to follow redirects:
bash

Collapse

Wrap

Copy
curl -X POST -d "username=admin&password=admin" -u admin:admin -c cookies.txt -L http://83.136.251.68:48581/
Verbose Output:
Use -v to debug:
bash

Collapse

Wrap

Copy
curl -v -X POST -d "{\"search\":\"london\"}" -b "PHPSESSID=hnjcoa6bvghqatlscamgvmh38h" -u admin:admin -H "Content-Type: application/json" http://83.136.251.68:48581/search.php
Test Without Headers:
The challenge suggested trying without the cookie or Content-Type:
bash

Collapse

Wrap

Copy
curl -X POST -d "{\"search\":\"london\"}" http://83.136.251.68:48581/search.php
This should return A valid authentication cookie is required!.
Windows Command Prompt:
Escape quotes in JSON payloads (\") to avoid parsing errors.
If commands fail, wrap them in quotes or use a script file.
Wrapping Up
Using curl to solve this CTF challenge was a great learning experience! I started by authenticating to the server to get a session cookie, then used that cookie to send JSON POST requests to the search endpoint. The clue ["London (UK)"] led me to try different search terms until I found the flag. If you’re working on a similar challenge, I hope these notes help you navigate the process. Let me know in the comments if you have other curl tips or CTF strategies to share!

Happy hacking! 🚀
