# Chatty Cathy: 

**Challenge:** Chatty Cathy  
**Event:** DEADFACE CTF  
**Category:** OSINT / Web  
**Points:** 15  
**Author:** @RP-01?

## Challenge Description

There is a DEADFACE member who writes a travel blog under the pseudonym **"Alice"**. The objective is to investigate GhostTown and locate the travel blog, which may contain information useful for identifying the member.

**Flag Format:** `deadface{flag_text}`

## Solution

### 1. Investigating GhostTown

We started by exploring the GhostTown platform:

https://ghosttown.deadface.io/

While investigating the travel-related discussions, we discovered a conversation involving a user named `lamia415`.

The discussion contained a reference to a travel blog, along with a warning that the blog revealed the user's first name.

This provided a lead to investigate the external website.

### 2. Accessing the Travel Blog

The blog was hosted on the following address:

```text
https://170.187.156.131:8443/
```

Since the challenge explicitly mentioned that port `8443` was relevant, we accessed the website using curl:

```bash
curl -k -i https://170.187.156.131:8443/
```

The request returned HTTP 200 OK.

The homepage title was:

```text
Wander with Alice
```

The website contained travel-related posts and an introductory section.

One particularly interesting sentence was:

```text
I will keep posting these on both profiles in case this stops working
```

This suggested that Alice might maintain multiple online profiles.

### 3. Inspecting the HTML Source

We downloaded the homepage HTML for further investigation:

```bash
curl -k https://170.187.156.131:8443/ -o index.html
```

Next, we extracted the referenced resources:

```bash
grep -Eoi '(src|href)="[^"]+"' index.html
```

The output revealed an image reference:

```html
<img src="static/other1.png">
```

However, attempting to access this image returned HTTP 404 NOT FOUND.

### 4. Directory Enumeration

To discover potentially hidden directories and files, we used FFUF with the common directory wordlist:

```bash
ffuf -k -u https://170.187.156.131:8443/FUZZ \
-w /usr/share/wordlists/dirb/common.txt \
-e .txt,.html,.png,.jpg,.bak
```

Among the discovered resources, `robots.txt` returned HTTP 200 OK.

### 5. Discovering the Flag

We inspected the robots.txt file:

```bash
curl -k https://170.187.156.131:8443/robots.txt
```

The server returned:

```text
User-agent: *
Disallow: /secret_members
# deadface{y0u_m3ddling_k!ds}
```

The flag was directly exposed in an HTML-style comment within the robots.txt file.

## Flag

```text
deadface{y0u_m3ddling_k!ds}
```

## Key Takeaways

- Investigate forum discussions for links to external websites.
- Inspect HTML source code for hidden clues and referenced resources.
- Use directory enumeration to discover overlooked files.
- Always inspect `robots.txt` during web reconnaissance.
- Comments in publicly accessible files can accidentally expose sensitive information.

**Lesson:** Files like `robots.txt` are intended to provide crawling instructions to search engines, not to protect sensitive information. Never store secrets or flags in publicly accessible resources.
