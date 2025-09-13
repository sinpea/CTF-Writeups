
# Agent47's flag (Q9, Encriptify 2.0)

## Description


This challenge revolved around Agent47's secret that he had hidden on his site.
At first glance, the site looked ordinary, but contestants quickly realized that simply browsing wasn’t enough.

To recover the flag, players had to think like an infiltrator: spot weaknesses, exploit them, and work through multiple layers of misdirection. Each vulnerability/clue opened a new path, eventually leading to the final hidden flag.

Goal: Exploit the site, follow Agent 47’s clues, and retrieve the flag.

## Solution

The solution to this problem can be divided into three sections:

### 3.1 The Home Page

This is the page we first land on.

https://test-project-2441139.vercel.app

The page gives us a brief overview of what the premise is and how we can proceed to the next stage (or page, if you will). The information that is visible to the user via the frontend is rudimentary in nature other than the narrative it provides.


If we use our in browser dev tools to inspect the page, we find the following code snippet: 

```
</div>
  <div class="intro-para">
    <p>
        It's been a  ... dear ones.
    </p>
  </div>
  <img>
  <div hidden>
    <h2>
      Message from Agent47: Somethings just cannot be inspected
    </h2>
  </div>  
  </img>
```

Here lies our first clue: Somethings just cannot be inspected. Therefore, to better examine the page our next aim will be to examine the raw page source.

By using view source or by just looking up the response of the request:

``` 
GET https://test-project-2441139.vercel.app/
```

Now, we find some code blocks that are not visible in the 
inspect tab, which shows the elements currently part of the DOM.

```
<div hidden id="delete_me">My daughter loved chocolate cakes when she was 10</div>
  <div>
    <h1>Agent47 has disappeared</h1>
  </div>
  <div class="intro-para">
    <p>
    It's been a few years since Agent47 has been
    last seen. He was a top scientist at SUlFU1QgU2hpYnB1cg 
    While he was a controversial figure in
    ...
    </p>
    </div>
    <div hidden>
    <h2>
      Message from Agent47: Somethings just cannot be inspected
    </h2>
    </div>  
    </img>
    <footer style="background:#222; color:#fff;width:100vw  ;text-align:center;position:fixed;bottom: 0px;">
    <p>&copy; Look where no body would</p>
    <div hidden id="delete_me_too">
        If only someone shared my love for Base64 encoding and hiding valuable "hints" about 
        where to go next inside those...
  </div>
</footer>
```

Here we see that the next clue is hiding within a Base64 string. The only base64 elements visible in the source is 
SUlFU1QgU2hpYnB1cg . However, this when decoded only gives us 'IIEST Shibpur'. Which is just 'red-herring'. The only other element on this page that has a possibility of being
encoded in base64 is the background image.

On opening ``` styles.css ``` we find the base64 image we are looking for:

``` 
body {
  background-image:
  linear-gradient(to top,rgba(0, 0, 0, 0.7), 
  rgba(255, 255, 255, 0)), 
  url("data:image/jpeg;base64,/9j/4AAQSkZJRgABAQ...5b6pntGJurD/r03//Z");
}

.intro-para{
  width:400px;
}

```
Within this base64 encoded image we find the following substring:

```
the0hint0is0here00go0to0getage0/use0underscore
```
This marks the end of the solution and now we move on to
the next page as shown in this clue.

``` 
https://test-project-2441139.vercel.app/get_age

```
### 3.2 Getting the Hash

Page: https://test-project-2441139.vercel.app/get_age

The objective of this page is to find the hash. We see an
input field asking for the age at which Agent47's daughter started liking chocolate cake. 
In the previous section we had obtained the following piece of information:
```
<div hidden id="delete_me">My daughter loved chocolate cakes when she was 10</div>
```

So the answer will be any number in the range 1-10.

However, the only response we get after submitting all numbers from 1-10 is:

//insert image Here

The way out here however, is to trigger an Integer overflow. In this case we need to cause an Int32 overflow.
When we enter the number $ 2^32 + n $, where ` n ` is the number we want to actually put in the input field.

The answer is $ n = 10 $ and we get the following bcrypt hash as our clue for the next stage of this problem:


```
$2a$12$N/6mr3/ruGBdON/MCBJrdeLKBiXe1OqlXN2Fmz4dHvVVHbV8nK9yC
```

If we look at the ` <style></style> ` tags in this page, we can find the clue that will lead us to the next stage of this question.

```
/*Your next destination is the route: (think of a Roman Conqueror, the betrayed one known for encrypting his messages): /orjlq */
```

The cipher that is talked about here is the [Caesar Cipher](https://en.wikipedia.org/wiki/Caesar_cipher). In this specific case, all the characters of the reqiuired route ` /login `  are replaced by characters that 3 letters down the alphabet. So, ` /orjlq ` is decoded to ` /login `.

So now, we move on to the next section :)

### 3.3 The Final Login

Page: https://test-project-2441139.vercel.app/login

This is the last page of our problem. The contestant is expected to identify how the page works and try to
find a way to login and successfuly Capture The Flag!

This page however has a few oddities.
When this page is opened with a normal browser, we can see a failed request has taken place as soon as the page gets loaded. The response to this request (Note: Most techniques discussed in this part of the doc are used for web scraping and is only shown for educational purposes, always respect robots.txt):

#### REQUEST:
``` 
GET /nonce
``` 
#### RESPONSE:
```
{"message":"not supported"}
``` 

On further inspection of the webpage, we find two blocks of code that will be necessary for getting past this challenge.

```
:root{
      --hint : "So you found me? use browser_agent:Mozilla/5.0 (AgentOS; AgentWeb/1.0; rv:120.0) Gecko/20100101 AgentWeb/120.0",
      --psss: "You remember putting in your age? No? Go to the only style.css on this website and look where to go there"
    }
```
```
<!--you will need this 1210nonce -> hash ... i think you can find the algo ;)-->
</body>
</html>
```

Lets look at the first block of code. We get the hint that a new browser agent , ` Mozilla/5.0 (AgentOS; AgentWeb/1.0; rv:120.0) Gecko/20100101 AgentWeb/120.0 ` must be used to get a proper view of the web page.

After changing the browser agent, we can actually start observing how the page behaves.

We observe the following requests and responses as soon as the page loads:

#### GET /nonce
Request:

```
GET /nonce
```
Parameters/Payload:
```
none
```
Response:
```
{"nonce":"bdf4d8549a572d476c0b4305e6ddf673"}
```

Inference:

` nonce ` is a random string that has been generated by the server, to identify the current session. This concept is often used in cookie based authentication methods.

#### POST /get-token

```
POST /get-token
```

Parameters/Payload:
```
{
  "nonce":"bdf4d8549a572d476c0b4305e6ddf673",
  "fingerprint":"b3b79a17226ff237e7a937d9dc89d7278c86978ef1f24d44beae52aa8a9ef819"
}
```
Response:
```
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ2ZXJpZmllZCI6dHJ1ZSwiaWF0IjoxNzU3NzQ2NjAzLCJleHAiOjE3NTc3NDY5MDN9.A2hSQCW-qBXe5Cq7mgcffE8zlWJ5Y9NSfjdEKiC0qvw"
}
```

Now, this is where our second clue from this page comes into play.

```
<!--you will need this 1210nonce -> hash ... i think you can find the algo ;)-->
```

When we examine the obfuscated javascript code on this page, we find this:

```
_0x35024f=await hashStringSHA256(_0x3f8186);
```

Hence, the hashing algorithm used is SHA256. Or is it? This can be easily confirmed by examining the hash like
values that we found in our requests. The parameter for interest in this case is ` fingerprint `.

We use the following python script to obtain the hash value and compare:

```
import hashlib

#put the nonce here
nonce = ""
hash_input = f"1210{nonce}".encode()
hash = hashlib.sha256(fingerprint_input).hexdigest()
print(hash)
```

We find that ` hash ` matches the ` fingerprint ` parameter on our POST request to ` /get-token `. 

Now, we want to find out how exactly the process for login works.
Lets try and enter a randomish value as our username and password.

//insert image here

We get two more requests when we try to login:
  - ` POST /gentok `
  - ` POST /login `

Lets examine these requests in more detail.

#### POST /gentok

Request:
```
POST /gentok
```

Payload:
```
{
  "hash":"$2b$12$qjjDivWAqAKzqVTSobDEZOgHKDbYfw545oQ5J2o1tag5ocIVdiVnS",
  "verification":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ2ZXJpZmllZCI6dHJ1ZSwiaWF0IjoxNzU3NzQ2NjAzLCJleHAiOjE3NTc3NDY5MDN9.A2hSQCW-qBXe5Cq7mgcffE8zlWJ5Y9NSfjdEKiC0qvw"
}
```

Response:
```
{
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJoYXNoIjoiJDJiJDEyJHFqakRpdldBcUFLenFWVFNvYkRFWk9nSEtEYllmdzU0NW9RNUoybzF0YWc1b2NJVmRpVm5TIiwidmVyaWZpY2F0aW9uIjoiZXlKaGJHY2lPaUpJVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5LmV5SjJaWEpwWm1sbFpDSTZkSEoxWlN3aWFXRjBJam94TnpVM056UTJOakF6TENKbGVIQWlPakUzTlRjM05EWTVNRE45LkEyaFNRQ1ctcUJYZTVDcTdtZ2NmZkU4emxXSjVZOU5TZmpkRUtpQzBxdnciLCJpYXQiOjE3NTc3NDY2MTJ9.xbi-BSK4yU7yznpE0h-RvRxG-5OPhjpIma5W6w6wuro"
}
```

Inference:

We see that the ` verification ` parameter carries the same parameter as the response ` token ` from ` POST /get-token `
and that the ` hash ` parameter is the bcrypt hash of the password we entered in the page. 

(Note: Usually, passwords and such credentials are not hashed client-side but use HTTPS/TLS to encrypt and transmit the data).

#### POST /login

This is the big bad request that holds the key to our success.

Request:
```
POST /login
```

Payload:
```
{
  "token":"eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJoYXNoIjoiJDJiJDEyJHFqakRpdldBcUFLenFWVFNvYkRFWk9nSEtEYllmdzU0NW9RNUoybzF0YWc1b2NJVmRpVm5TIiwidmVyaWZpY2F0aW9uIjoiZXlKaGJHY2lPaUpJVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5LmV5SjJaWEpwWm1sbFpDSTZkSEoxWlN3aWFXRjBJam94TnpVM056UTJOakF6TENKbGVIQWlPakUzTlRjM05EWTVNRE45LkEyaFNRQ1ctcUJYZTVDcTdtZ2NmZkU4emxXSjVZOU5TZmpkRUtpQzBxdnciLCJpYXQiOjE3NTc3NDY2MTJ9.xbi-BSK4yU7yznpE0h-RvRxG-5OPhjpIma5W6w6wuro",
  "nonce":"bdf4d8549a572d476c0b4305e6ddf673"
}
```

Response:
```
{
  "error":"error"
}
```
Well, unless you are lucky enough to input the right password. (No, the right password will not be shared), you will get the above response. If you are lucky enough, you will get this response:
```
{
  "flag":"ZmxhZ3thZ2VudDQ3X3dhc19oZXJlfQ"
}
```

Inference:

We see that the ` nonce  ` parameter is the value we received as a response to ` GET /nonce ` and the ` token ` parameter is the response received from the ` POST /gentok ` request. Since, the page actually sends the bcrypt hashed password and not the actual password, we can inject the hash that we have (this is the reason why hashes are not usually computed client-side).

This question can now be solved in multiple ways. For this writeup I will be writing a python script to do my HTTP requests but alternate approaches are also just as feasible. However, we need to fool the server into thinking that we are actually a browser sending the request. Thus we will need to mimick every single request sent out by the browser and finally inject the hash where it is required. 

CODE:

```
import requests
import hashlib

BASE_URL = "https://test-project-2441139.vercel.app"  # change to your server
USER_AGENT = "Mozilla/5.0 (AgentOS; AgentWeb/1.0; rv:120.0) Gecko/20100101 AgentWeb/120.0"

headers = {
    "User-Agent": USER_AGENT,
    "Content-Type": "application/json"
}

# 1. GET /nonce
resp = requests.get(f"{BASE_URL}/nonce", headers={"User-Agent": USER_AGENT})
resp.raise_for_status()
nonce = resp.json().get("nonce")
print(f"[1] Nonce: {nonce}")

# 2. POST /get-token
fingerprint_input = f"1210{nonce}".encode()
fingerprint = hashlib.sha256(fingerprint_input).hexdigest()

resp = requests.post(
    f"{BASE_URL}/get-token",
    headers=headers,
    json={"nonce": nonce, "fingerprint": fingerprint}
)
resp.raise_for_status()
verification = resp.json().get("token")
print(f"[2] Verification Token: {verification}")

# 3. POST /gentok
hash_value = "$2a$12$7LXMGodW8.kQjvamEji7AeBvp.2dFTQjZU62A7H1V.J6iaYkpYHfm"  # replace with your actual hash
resp = requests.post(
    f"{BASE_URL}/gentok",
    headers=headers,
    json={"hash": hash_value, "verification": verification}
)
resp.raise_for_status()
gentok_token = resp.json().get("token")
print(f"[3] GenTok Token: {gentok_token}")

# 4. POST /login
resp = requests.post(
    f"{BASE_URL}/login",
    headers=headers,
    json={"token": gentok_token, "nonce": nonce}
)
resp.raise_for_status()
login_result = resp.json()
print(f"[4] Login Response: {login_result}")
```
Output:
```
[1] Nonce: 4efbc145851d0379e72949b84592b382
[2] Verification Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ2ZXJpZmllZCI6dHJ1ZSwiaWF0IjoxNzU3NzQzNzExLCJleHAiOjE3NTc3NDQwMTF9.AP0_8AFwqxa--7e0TM8X8ATemMcYjZEwWVwWrhEnPv0
[3] GenTok Token: eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJoYXNoIjoiJDJhJDEyJDdMWE1Hb2RXOC5rUWp2YW1Famk3QWVCdnAuMmRGVFFqWlU2MkE3SDFWLko2aWFZa3BZSGZtIiwidmVyaWZpY2F0aW9uIjoiZXlKaGJHY2lPaUpJVXpJMU5pSXNJblI1Y0NJNklrcFhWQ0o5LmV5SjJaWEpwWm1sbFpDSTZkSEoxWlN3aWFXRjBJam94TnpVM056UXpOekV4TENKbGVIQWlPakUzTlRjM05EUXdNVEY5LkFQMF84QUZ3cXhhLS03ZTBUTThYOEFUZW1NY1lqWkV3V1Z3V3JoRW5QdjAiLCJpYXQiOjE3NTc3NDM3MTJ9.Siz68lJCRfi6oXjPmtbxA6QNqtO_eBpa1ZJFBB-1e2Q
[4] Login Response: {'flag': 'ZmxhZ3thZ2VudDQ3X3dhc19oZXJlfQ'}
```

With this, we come to the end of our challenge. The response obtained is the base64 encoded version of the flag (without padding token). 

Lets decode it.

```
$ echo "ZmxhZ3thZ2VudDQ3X3dhc19oZXJlfQ==" | base64 -d
flag{agent47_was_here}

```

## Answer
flag{agent47_was_here}