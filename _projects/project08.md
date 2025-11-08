---
layout: page
title: HTTPS-Server
description: A high-performance, event-driven HTTPS server and WSGI host built from scratch in Python using only standard libraries.
# img: assets/img/12.jpg
importance: 8
category: personal
related_publications: false
---

## Overview

Imagine deploying a web app, only for it to crumble under traffic because the underlying server can't keep up. That's the reality hidden behind frameworks like Flask or Django. To truly understand—and master—web serving, I built **HTTPS-Server**: a high-performance, non-blocking HTTPS/WSGI server using only Python's standard library. No threads, no external deps—just raw sockets and clever I/O management.

As the solo developer, I tackled the full stack: from socket binding to SSL encryption and WSGI compliance. The result? A server that securely serves static files, streams videos, caches responses, and runs Python apps, all while handling hundreds of concurrent connections without breaking a sweat.

## Research: Diving into the Networking Abyss

Building a server isn't about slapping together code—it's about solving the "how does the internet even work?" puzzle. I started with the basics:

- **TCP/IP and Sockets**: I pored over Python's `socket` docs and Beej's Guide to Network Programming to grasp binding, listening, and data transmission.
- **HTTP/1.1 Specs**: RFC 2616/7230 became my bedtime reading. I learned to parse raw requests (e.g., GET /static/index.html HTTP/1.1) and craft responses with headers like Content-Range for streaming.
- **WSGI (PEP 3333)**: This spec ensures portability; I studied how to bridge raw HTTP to Python's `environ` dict and `start_response` callback.
- **Concurrency Challenges**: The infamous C10k problem (handling 10,000 clients) led me to non-blocking I/O. Threads are inefficient for I/O-bound tasks, so I researched event-driven models. Python's `select` evolved into `selectors`—perfect for monitoring multiple sockets in one thread, à la Nginx or Node.js.

I iterated through prototypes: a blocking server first (easy but slow), then non-blocking. Debugging involved Wireshark for packet inspection and Stack Overflow for edge cases like partial reads.

## Implementation: From Sockets to Streaming

The codebase is modular, with `webServer.py` as the core. Here's how it comes together:

- **Event-Driven Core**:
  The `WSGIServer` class binds a socket on port 8443 and uses `selectors.DefaultSelector()` for non-blocking magic. The `serve_forever()` loop calls `select()`, dispatching events to `accept()` (new clients) or `read()` (data handling).

  ```python
  def serve_forever(self):
      while True:
          events = self.selector.select(timeout=None)
          for key, mask in events:
              callback = key.data
              callback(key.fileobj)
  ```

  This keeps the server responsive, even under load.

- **HTTPS Integration**:
  Using `ssl.SSLContext`, I wrap sockets for TLS. Self-signed certs are auto-generated via `run.sh` (OpenSSL command). Handshake failures? Gracefully closed with logging.

- **WSGI Magic**:
  For dynamic content, `get_environ()` parses requests into a spec-compliant dict. The app (e.g., `wsgiapp.py`) gets called, and `start_response()` sets headers. Static paths bypass this for speed.

- **Static Serving Smarts**:
  - **Security**: Path normalization prevents directory traversal.
  - **Streaming**: Parse `Range: bytes=0-20` for partial responses (HTTP 206)—crucial for `sample.mp4`.
  - **Caching**: Compute `ETag` (MD5 hash) and `Last-Modified`; return 304 if unchanged.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/httpserver-running server.png" title="httpserver-running server" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Launching the server with `./run.sh`—certs generated, ready for HTTPS traffic</div>

## Results: Proof in the Performance

The server shines in action. Visiting `https://localhost:8443/static/index.html` loads a styled page with embedded video, JS, and CSS—all over secure HTTPS.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/httpserver-Web Browser output.png" title="httpserver-Web Browser output" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">The server in action: Serving index.html with video streaming and styling</div>

Testing via `test.sh` confirms features:

- Static files served correctly.
- Range requests stream MP4 chunks.
- HEAD requests return metadata without bodies.
- Caching skips unchanged files.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/httpserver-test script output.png" title="httpserver-test script output" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Automated tests validating static serving, ranges, and headers</div>

For concurrency, `client.py` simulates loads (e.g., 5 clients, 100 connections each). The server handled it flawlessly, logging accepts without delays—proving the non-blocking design scales.

<div class="row justify-content-sm-center">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid path="assets/img/httpserver-load testing output.png" title="httpserver-load testing output" class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">Stress-testing with concurrent clients—server stays responsive</div>

Key wins: Zero external libs, full WSGI support, and educational depth. Lessons? Sockets are finicky (unregister them promptly!), and non-blocking shifts your mindset from sequential to event-based.

## Reflection: Why Build It, and What's Next?

This project stemmed from curiosity: frameworks abstract too much. By rebuilding from scratch, I demystified the "black box" of web servers, gaining skills transferable to real-world ops (e.g., debugging production issues).

**Tech Stack**:

- `socket`: Core networking.
- `ssl`: TLS encryption.
- `selectors`: Event multiplexing.
- `hashlib` & `mimetypes`: Caching and content types.
- No threads—pure async I/O for efficiency.

Challenges? Connection management was a nightmare—leaky descriptors haunted my dreams (fixed with try-finally closes). But the payoff? A server that's lightweight, educational, and fun to tinker with.

Future: Add Keep-Alive for persistent connections and a thread pool for CPU-heavy apps. Maybe integrate with Docker for easy deployment.

> Explore the source on [GitHub](https://github.com/carb0ned0/HTTP-Server). Got questions or collab ideas? Hit me up!
