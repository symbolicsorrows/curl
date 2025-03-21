# curl

curl (Client URL) is a command-line tool for transferring data with URLs. It’s widely used in CTF challenges, web application testing, and API interactions because it allows precise control over HTTP requests. In this challenge, we’re using curl to:

Authenticate to the server to obtain a session cookie (PHPSESSID).
Send a JSON POST request to /search.php to search for the flag, using the authenticated cookie.
The current server is 83.136.251.68:48581, and we have a valid session cookie: PHPSESSID=hnjcoa6bvghqatlscamgvmh38h. The goal is to find a flag in the format FLAG{...} by searching with the correct term.

1. Authenticating to the Server
The first step is to log in to the web application to obtain an authenticated session cookie. The instructions indicate the login form expects a POST request with username=admin&password=admin.

curl -X POST -d "username=admin&password=admin" -u admin:admin -c cookies.txt http://83.136.251.68:48581/
