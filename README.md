# otxt (WORK IN PROGRESS)

A distributed P2P HTTP-based SMS replacement.

## Key ideas
* Private, individual & group short text messaging, aka Dark Social.
    * group messaging is like email/irc and not sms, i.e., everyone knows who the members of the group are (from the `to` param)
* Short text messages limited to 256 characters. No new lines allowed.
* Users identified by URLs intead of mobile numbers.
* Links are first class citizens.
    * compose: `This is how you [link](http://example.com)`
    * display: `This is how you <a href="http://example.com">link</a>`
    * characters counted: `This is how you link`
* Not IM
    * No presense
    * But requires adding recipients to ones contact before sending a msg (how does tihs affect group messaging?)
* Support in-reply-to for better conversations.
* Security via HTTPS.



## Alice wants to add Bob to her otxt contacts

### Alice's otxt host discovers Bob's HTTPS otxt endpoint.

```http
HEAD / HTTP/1.1
Host: bob.host
```

```http
HTTP/1.1 200 OK
Link: <https://bob.host/otxt-endpoint>; rel="http://otxt.org/"
```


### Alice's otxt host sends add request to Bob's otxt endpoint

```http
POST /otxt-endpoint HTTP/1.1
Host: bob.host
Content-Type: application/x-www-url-form-encoded

from=alice.host
to=bob.host
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
HEAD / HTTP/1.1
Host: alice.host
```

```http
HTTP/1.1 200 OK
Link: <https://alice.host/otxt-endpoint>; rel="http://otxt.org/"
```
### Bob's otxt host verifies Alice's add request

```http
POST /otxt-endpoint HTTP/1.1
Host: alice.host
Content-Type: application/x-www-url-form-encoded

from=bob.host
to=alice.host
action=verify
id=<this_request_id>
hash=<hash_of_from+id_using_alices_secret>
```

```http
HTTP/1.1 200 OK
```


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
datetime=<datetime>
hash=<hash_of_..._using_alices_secret>
```

```http
HTTP/1.1 200 OK
```

### TODO
* Should otxt support unsolicited messages?
