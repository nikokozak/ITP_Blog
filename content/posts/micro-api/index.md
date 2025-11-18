+++
date = '2025-11-10T13:48:44-05:00'
draft = false
title = 'Micro Api'
tags = ['Understanding Networks']
+++

**API Address**: https://api.nkozak.com/
**Routes**: Start with just `GET`ing the root, it provides instructions.

At the moment the welcome API route responds with:

```
Welcome! This is my little API.

It is written in Racket, and will continue to grow until the end of class.


=== MESSAGE BOARD ===

  GET /message

    Retrieve the current message from the "wall"


  POST /message

    Post a message to the wall

    Body: {"name": "Your name", "message": "Something you feel like saying"}


=== DYNAMIC COLOR ===

  GET /color

    Returns current RGB color values that evolve smoothly over time

    Response: {"r": 0-255, "g": 0-255, "b": 0-255, "rate": number}


  POST /color

    Influence the color evolution

    Body (all optional):

      {"rate": 0-10}        - Speed of color changes (0=frozen, 10=fastest)

      {"r": 0-255}          - Target red value

      {"g": 0-255}          - Target green value

      {"b": 0-255}          - Target blue value

    Example: {"rate": 2.5, "r": 255, "g": 0, "b": 128}


=== SYSTEM ===

  GET /health

    Health check endpoint
```

I spent the afternoon putting together the rudiments of a little servlet. At the moment it fulfills the use-case of the class exercise: an easy-to-use API that responds with some interesting data, that someone else in the class can pull. My idea is to eventually create a small game that teaches users how to best use `curl`. The game would progress from simple `GET` requests to sending multi-part forms and handling cookies, user-agents, etc. I have the basic "curriculum" laid out. The rules for endpoints would go something like:

```
Level 6: DELETE with headers
DELETE /level/6/{id} with header Confirm: yes
{
  "success": true,
  "lesson": "Headers carry metadata; DELETE removes resources",
  "instruction": "Pretend to be a bot: GET /level/7 with custom User-Agent",
  "required_header": "User-Agent: CurlBot/1.0",
  "hint": "curl -H 'User-Agent: CurlBot/1.0' ..."
}
```

Where every reply gives you info on the next endpoint.

But, for now, I've just installed a basic server that serves as a message wall - you POST a message, and can GET the last posted message. Nothing fancy.

The server is written in `Racket`, which I'm trying to work with a bit more - other than for creating DSL's, I never really dug too deap into its use as a general-purpose language. The nice things about lisps for dispatching (i.e. perfect for routing in a server) is how sane the language is:

```racket
; Dispatch-values
(define-values (servlet-dispatch servlet-url)
  (dispatch-rules
  [("message") #:method "get" get-message-endpoint]
  [("message") #:method "post" post-message-endpoint]
  [("health") #:method "get" health-endpoint]
  [("") #:method "get" welcome-endpoint]
  [else welcome-endpoint]))
```

Other affordances, like inline evaluation of JSON structs, is also nice:

```racket
(define (health-endpoint req)
  (response/jsexpr
    (hasheq 'status "healthy"
            'timestamp (exact->inexact (current-seconds)))))
```

I deployed the server to my DigitalOcean VPS, provided it with a `/health` endpoint for monitoring, and set it up as a service using `systemctl`. This allows me to restart the server automatically if anything goes wrong. I then set up a reverse-proxy with Caddy, routing `api.nkozak.com` requests to port `4321`.