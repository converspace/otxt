# otxt (WORK IN PROGRESS)

A distributed P2P HTTP-based SMS replacement.

## Key ideas
* Focus on Dark Social (SMS, Email, Whatsapp, [Pair](http://trypair.com/))
* Private one-to-one short text messaging like SMS.
* Messages limited to 256 characters. No new lines allowed.
* Users identified by URLs intead of mobile numbers.
* Links are first class citizens.
    * compose: `This is how you [link](http://example.com)`
    * display: `This is how you <a href="http://example.com">link</a>`
    * characters counted: `This is how you link`
* Not IM
    * No presense
    * But requires pairing before sending a msg
      * makes the protocol less chatty because of the shared secret instead of verifying authenticity of individual msgs 
      * requiring manual approval helps with SPAM
* Support in-reply-to for better conversations.
* Security via HTTPS.


## Alice pairs with Bob

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
action=pair
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

Now Bob and Alice can send each other messages using the shared secret that Alice orginally sent to Bob.

## Alice sends a message to Bob

```http
POST /otxt-endpoint HTTP/1.1
Host: bob.host
Content-Type: application/x-www-url-form-encoded

action=send
id=<this_txt_id>
from=alice.host
to=bob.host
txt=<256_chars_msg>
in_reply_to=<some_txt_id>
datetime=<datetime>
hash=<hash_of_..._using_shared_secret>
```

```http
HTTP/1.1 200 OK
```

## TODO
* group messaging?
   * This is currently not possible because you need to add a contact before you can talk to them and not everyone in the group (comma separated `to` param) would have added everyone.
   * One way to do this is to treat a group just like any other person.
      * When a user creates a group, the group pairs with all members of the group.
      * When someone sends a msg to the group, the group forwards the msg to the members.
   * Alternatively, a new action `relay` can be introduced so that msgs from people that haven't paired can be sent via paired ones.
* Should otxt support unsolicited messages?
