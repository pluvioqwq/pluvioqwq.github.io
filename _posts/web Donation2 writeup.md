
# 0x01

#### Http parameter pollution

HTTP Parameter Pollution (HPP) is a technique where attackers manipulate HTTP parameters to change the behavior of a web application in unintended ways.

In this challenge, we need to duplicate the `to` parameter

# 0x02

Firstly, as same as challenge `Donation`, we first created a account with initial amount of 1000 and try to donate negative amount to gain money. However, this time it turns out that it does not work. 

Then we tried to implement Http parameter pollution approach, where we duplicate the `to`  parameter, the payload will be like this:

```url
to=lisanalgaib&to=<user>&currency=1000
```

By doing such, we actually bypass the restriction ( we cannot donate to other users initially ) and donate money to other users. 

![[Pasted image 20240503185700.png]]

As such, what we need to do is to create mutiple account and donate money to account with username `12345`( which we created initially). By repeating this a few times, we are able to get the flag by using `12345` account


