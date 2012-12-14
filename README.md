# otxt (WORK IN PROGRESS)

HTTP-based distributed P2P short message communication protocol (to replace SMS).

## Key ideas
* Focus on Dark Social (SMS, Email, Whatsapp, [Pair](http://trypair.com/))
* Private one-to-one short text messaging like SMS.
* Messages limited to 256 characters. No new lines allowed.
* Users identified by URLs intead of mobile numbers.
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

<html>
...
<link href="https://bobs.host/otxt-endpoint" rel="http://otxt.org/" />
...
```


### Alice's otxt host sends pairing request to Bob's otxt endpoint

```http
POST /otxt-endpoint HTTP/1.1
Host: bob.host
Content-Type: application/x-www-url-form-encoded

from=alice.host
to=bob.host
action=pair
secret=
id=tag:alice@example.com,2012-12-02:alice.host:pair:2
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

<html>
...
<link href="https://alice.host/otxt-endpoint" rel="http://otxt.org/" />
...
```

### Bob's otxt host verifies Alice's add request

```http
POST /otxt-endpoint HTTP/1.1
Host: alice.host
Content-Type: application/x-www-url-form-encoded

from=bob.host
to=alice.host
action=verify
id=tag:alice@example.com,2012-12-02:alice.host:pair:2
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
id=tag:alice@example.com,2012-12-02:alice.host:txt:2
from=alice.host
to=bob.host
txt=<256_chars_msg>
in_reply_to=tag:bob@example.net,2012-12-01:bob.host:txt:22
datetime=<datetime>
hash=<hash_of_..._using_shared_secret>
```

```http
HTTP/1.1 200 OK
```

## TODO
* Discovery over http is not secure. Should that be made over HTTPS as well?
* group messaging?
   * This is currently not possible because you need to add a contact before you can talk to them and not everyone in the group (comma separated `to` param) would have added everyone.
   * One way to do this is to treat a group just like any other person.
      * When a user creates a group, the group pairs with all members of the group.
      * When someone sends a msg to the group, the group forwards the msg to the members.
   * Alternatively, a new action `relay` can be introduced so that msgs from people that haven't paired can be sent via paired ones.
* fetch profile?
   * GET /otxt-endpoint?profile=(all|email|name|avatar)&of=<>&from=<>&hash=<>
   * Allowed only after pairing since this will require the shared secret.
* Multi-Media (using additional params)
   * `img=<image_url>`
   * `vid=<video_url>`
   * `aud=<audio_url>`
* Links are first class citizens (Might have to skip this to enforce the 256 char limit?).
    * compose: `This is how you [link](http://example.com)`
    * display: `This is how you <a href="http://example.com">link</a>`
    * characters counted: `This is how you link`

## See Also
* TagURI (http://taguri.org/)
   * http://web.archive.org/web/20110514113830/http://diveintomark.org/archives/2004/05/28/howto-atom-id
   * http://www.rfc-editor.org/rfc/rfc4151.txt
   * http://www.goer.org/Journal/2004/06/atom_ids_whats_wrong_with_domain_timestamp.html
