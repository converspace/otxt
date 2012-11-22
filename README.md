# otxt

__WORK IN PROGRESS__

Distribute peer-to-peer (P2P) Twitter replacement, because Twitter should have been a protocol not a product.

Key ideas:
* Focus on 140 characters
* Hub-less pub/sub. Push fan-out responsiblity to clients.
* Use HTTPS.
    * Where HTTPS is not available, use delegated HTTPS endpoints. (see below)
 

## Alice wants to follow Bob.

### Alice discovers Bob's HTTPS otxt endpoint.

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


### Alice sends follow request to Bob

```http
POST /otxt-endpoint HTTP/1.1
Host: bobs.host
Content-Type: application/x-www-url-form-encoded

from=alice.host
action=follow
secret=
id=
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


### Bob's post is sent to Alice

```http
POST /otxt-endpoint HTTP/1.1
Host: alice.host
Content-Type: application/x-www-url-form-encoded

from=bob.host
action=post
url=<this_post_url>
content=<max 140 chars>
in_reply_to=<post_url>
to=<urls_of_users>
cc=<urls_of_users>
hash=<hash_using_alices_secret>
```

```http
HTTP/1.1 200 OK
```

If Alice cannot validate the hash or had not sent a follow request, an error response is sent instead. Bob MUST assume unfollow (this is a simple workaround to skip verifying follow requests)?


## Delegated HTTPS

* Alice signs up with SecureEndpoint to accept otxt responses on her behalf
* SecureEndpoint validates that Alice is the owner of alice.host
* After verification, Alice is given a HTTPS otxt endpoint URL she can use and allows her to specify a notification callback URL.
* When SecureEndpoint receives otxt requests, it sends a notification to Alice's notification callback URL.
* Alice connects to SecureEndpoint over HTTPS using creds given to her by SecureEndpoint and retrieves her otxt requests. 

## TODO
* unsolicited mentions.
    * these will be missing the hash param. Receivers must check that they are present in either to or cc and simply visit the post url and verify that its contents are the same and that the from url is a substr of the post url?
* profiles (including followers)
    * Should just pick up micoformats from alice.host. There should be a simple notification to all followers on profile change.

