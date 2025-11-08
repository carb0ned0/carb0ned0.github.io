---
layout: page
title: DBLite
description: A lightweight, in-memory, Redis-like database server and client built from scratch in Python with support for rich data types and high-concurrency.
# img: assets/img/12.jpg
importance: 7
category: personal
related_publications: false
---

## Overview

In-memory databases like Redis are the high-performance backbone of modern web applications, but to many, they're a "black box" *(I know I am abusing this analogy but I really like this word :)*. We use `SET` and `LPUSH`, but how does a server actually handle these requests? How does it juggle connections from hundreds of clients without dropping the ball? How does it store a list, a set, and a string in the same database without everything turning into a tangled mess?

My challenge was to crack open this black box by building one from scratch. The goal: a database server from first principles that speaks a **Redis-like protocol**, wrangles multiple data types, and handles a barrage of concurrent clients. And to be able to do that I wore all the hats—from network architect to data sheriff.

## Research

This project's journey was more about architectural plot twists than a straight UX highway. I zeroed in on four key riddles to shape the system.

1. **"How should the client and server chit-chat?"**

      * **Research:** I could've whipped up my own JSON mishmash, or borrowed from the pros.
      * **Decision:** Went with implementing a **Redis-like protocol** from the ground up. This locked in a solid spec for requests and responses (e.g., `b'*'` for arrays, `b'$'` for strings), forcing me to craft a genuine `ProtocolHandler` that chews on byte-streams like a pro, not just slurps JSON.

2. **"How to handle thousands of clients without a meltdown?"**

      * **Research:** Python's `socketserver` does multi-threading like a reliable old truck, but it chugs under heavy load. For real speed, async event-driven I/O is the rocket fuel.
      * **Decision:** **Built it both ways, because why not?** The server sniffs for `gevent`—if it's there, it flips to a zippy `StreamServer` for concurrency chaos. No gevent? It defaults to the standard lib's `ThreadingMixIn` for that no-fuss, zero-dependency life. Flexibility for the win!

3. **"How to store a list, a set, and a string at the same key?"**

      * **The Problem:** Spoiler: You can't without rules, or it's anarchy.
      * **Decision:** Wrapped everything in a `Value = namedtuple('Value', ('data_type', 'value'))`. The main `kv` store holds this duo, tagging the data (like a `deque` for lists or a `set` for sets) with a type flag (`QUEUE`, `SET`). It's like labeling your fridge shelves to avoid milk in the veggie drawer.

4. **"How to enforce type-safety without if-statement spaghetti?"**

      * **Research:** Littering every command with checks would've been messy, like crumbs in your keyboard.
      * **Decision:** Enter the Python **decorator** hero! My `@enforce_datatype` interceptor swoops in, verifies the type from the `Value` tuple, and yells "WRONGTYPE" if it's off—keeping the actual command code as clean as a whistle.

## Implementation

The endgame: a sleek single-file Python app that hums as a persistent server, plus a client sidekick for poking it.

* **`ProtocolHandler` Class:** The bouncer at the door, sifting raw bytes via a `self.handlers` dict to parse that Redis-inspired lingo—handling `b'+'` (simple strings), `b':'` (integers), `b'$'` (bulk strings), and `b'*'` (arrays) like a champ.
* **`DBLiteServer` Class:** The beating heart. Its `run()` method pulls off the concurrency double-act:

  ```python
  if self.use_gevent:
      self.pool = Pool(max_clients)
      self.server = StreamServer((host, port), ...)
  else:
      class ThreadedTCPServer(socketserver.ThreadingMixIn, ...):
          ...
      self.server = ThreadedTCPServer((host, port), ...)
  ```

  And voila, your server springs to life in the terminal, ready to field requests!

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/dblite-server.png" title="DBLite Running Server" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">DBLite Server running at port 6379.</div>

* **Data Model & Type Safety:** That `@enforce_datatype` decorator is the MVP—making commands read like poetry:

  ```python
  @enforce_datatype(QUEUE) # <-- The gatekeeper that keeps types in check
  def lpush(self, key, *values):
      self.kv[key].value.extendleft(values) # Short, sweet, and drama-free
      return len(self.kv[key].value)
  ```

* **Expiry Management:** For `EXPIRE` shenanigans, I leaned on `heapq` (a min-heap) to queue up `(expiry_timestamp, key)` pairs. The `clean_expired()` method just peeks at the front-runners, popping expired keys without sweating the whole list—efficient, like skipping the line at the DMV.
* **Persistence:** `SAVE` and `RESTORE` hitch a ride on Python's `pickle`, dumping and reloading the whole shebang (types included) to disk. Quick and dirty, but it works.

## Results

Boom—the project birthed a rock-solid, Redis-inspired database server that's more than just talk.

* **Metrics:** The full arsenal (strings, lists, hashes, sets, expiry, persistence) gets a thumbs-up from the `test_dblite.py` suite, which runs like clockwork and flashes that sweet "OK" to confirm no gremlins lurking.
<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/dblite-client.png" title="Testing DBLite Server" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Testing DBLite Server with an automated script.</div>
* **Outcome:** A lightweight powerhouse for caching, queues, or whatever floats your boat in real apps.
* **Key Lesson:** Protocols are basically byte-sized handshakes. Building one demystified the whole client-server tango—like peeking behind the wizard's curtain.
* **Future Work:** Pickle's handy but in a bit of a... pickle (heh) for security. Next up: Swap it for a custom append-only file (AOF) like Redis's, logging commands for that bulletproof vibe.

## Reflection

* **Core Stack:** **Python 3**
* **Key Libraries:**
  * **`socketserver`:** The trusty base for multi-threaded TCP fun.
  * **`gevent` (Optional):** The async ninja for when you want to crank concurrency to 11.
  * **`heapq`:** The unsung hero behind expiry's smooth operation.
  * **`pickle`:** Quick persistence, with a side of caution.
  * **`unittest`:** The safety net that catches bugs before they bite.

**Why this stack?** Python's the Swiss Army knife—handling fancy decorators and gritty sockets without breaking a sweat. That hybrid concurrency (gevent or bust) screams "practical wizardry," blending no-deps simplicity with performance punch.

> Peek at the DBLite code and docs on [GitHub](https://github.com/carb0ned0/DBLite). Who knows, it might just SET your next project on fire!

There you go—your article, leveled up with a touch of whimsy and those images for extra flair. If it still feels off, hit me with specifics, and we'll tweak it like fine-tuning a finicky expiry timer. What's next on your portfolio adventure?
