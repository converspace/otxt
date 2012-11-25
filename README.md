# otxt (WORK IN PROGRESS)

A distributed P2P HTTP based SMS replacement.

## Key ideas
* Private, individual & group short text messaging, aka Dark Social.
* Short text messages limited to 256 characters. No new lines allowed.
* Users identified by URLs intead of mobile numbers.
* Links are first class citizens.
    * compose: `This is how you [link](http://example.com)`
    * display: `This is how you <a href="http://example.com">link</a>`
    * characters counted: `This is how you link`
* Not IM
    * No presense
    * But requires adding recipients to ones contact before sending a msg.
* Support in-reply-to for better conversations.
* Security via HTTPS.
    * Where HTTPS is not available, use delegated HTTPS endpoints. (see below)







## Alice wants to add Bob to her otxt contacts

### Alice's otxt host discovers Bob's HTTPS otxt endpoint.

```http
GET / HTTP/1.1
Host: bobs.host
```

```http
HTTP/1.1 200 OK
Link: <https://bobs.host/otxt-endpoint>; rel="http://otxt.org/"

<html>
...
<link href="https://bobs.host/otxt-endpoint" rel="http://otxt.org/" />
...
```


### Alice's otxt host sends add request to Bob's otxt endpoint

```http
POST /otxt-endpoint HTTP/1.1
Host: bobs.host
Content-Type: application/x-www-url-form-encoded

from=alice.host
action=add
secret=
id=<this_request_id>
intro=<256_chars_introduction>
```

```http
HTTP/1.1 202 Accepted
```


### Bob discovers Alice's HTTPS otxt endpoint


```http
GET / HTTP/1.1
Host: alice.host
```

```http
HTTP/1.1 200 OK
Link: <https://alice.host/otxt-endpoint>; rel="http://otxt.org/"

<html>
...
<link href="https://alice.host/otxt-endpoint" rel="http://otxt.org/" />
...
```
### Bob's otxt host verifies the add request and sends Alice's otxt host his secret
...

### Bob sends a message to Alice

```http
POST /otxt-endpoint HTTP/1.1
Host: alice.host
Content-Type: application/x-www-url-form-encoded

action=send
id=<this_txt_id>
from=bob.host
to=<urls_of_users>
txt=<256_chars_msg>
in_reply_to=<some_txt_id>
hash=<hash_of_..._using_alices_secret>
```

```http
HTTP/1.1 200 OK
```


## Delegated HTTPS

* Alice signs up with SecureEndpoint to accept otxt responses on her behalf
* SecureEndpoint validates that Alice is the owner of alice.host
* After verification, Alice is given a HTTPS otxt endpoint URL she can use and allows her to specify a notification callback URL.
* When SecureEndpoint receives otxt requests, it sends a notification to Alice's notification callback URL.
* Alice connects to SecureEndpoint over HTTPS using creds given to her by SecureEndpoint and retrieves her otxt requests. 

