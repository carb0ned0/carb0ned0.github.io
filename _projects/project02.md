---
layout: page
title: BareCodeX
description: An video codec built from scratch in C++ to demystify the fundamental pipeline of video compression.
# img: assets/img/12.jpg
importance: 2
category: personal
related_publications: false
---

## Overview

We consume gigabytes of video daily, streamed seamlessly by complex codecs like H.264 or AV1. These technologies are "black boxes" to most—incredibly efficient, but impenetrably complex. My challenge was: **Could I build a working video codec from first principles?**

The goal of BareCodeX was not to compete with industrial codecs, but to build a "glass box"—a simple, C++ application that demystifies the core concepts. My objective was to implement a complete encode/decode pipeline to understand, hands-on, how modern compression achieves its seemingly magical results. I focused on three core techniques: **color space conversion**, **inter-frame prediction**, and **entropy coding**.

## Research

This project was a deep dive into applied research and implementation. My process wasn't about user interviews, but about deconstructing a complex technical problem. I broke my research down into three foundational questions:

1. **"How can we shrink data *before* prediction?"**
    * This led me to color models and human psychovisual perception. I learned that the human eye is far more sensitive to brightness (luma) than to color (chroma).
    * **Decision:** Implement **YUV420p color space conversion**. This involves **Chroma Subsampling**, where I store the luma (Y) for every pixel but only store the color (U & V) for every 2x2 block. This decision *alone* is a 50% lossy compression step that is virtually unnoticeable to the viewer.

2. **"How can we exploit redundancy *between* frames?"**
    * In a typical video, most of the screen is identical from one frame to the next. Storing every full frame is incredibly wasteful.
    * **Decision:** Implement **Delta Encoding**, a simple form of inter-frame prediction. The first frame (the **Keyframe**) is stored completely. For every subsequent frame, I only store the *difference* (`current_frame - previous_frame`). This "delta frame" is mostly zeros, making it highly compressible.

3. **"How do we pack the final data efficiently?"**
    * After delta encoding, the data stream (one big keyframe and many sparse delta frames) is full of repeating patterns, especially zeros.
    * **Decision:** Use a standard **Entropy Coding** algorithm. I chose not to reinvent the wheel and instead used the industry-standard **DEFLATE** algorithm via the `zlib` library. This losslessly compresses the repeating data, acting as the final compression stage.

## Implementation

The final solution is a C++ program that functions as the core logic, glued together by an `ffmpeg` workflow for practical pre- and post-processing.

#### 1. The Core C++ Codec Engine

The `barecodex` program reads raw RGB24 video from `stdin` and processes it in three stages:

* **Color Conversion (Lossy):** A custom C++ function reads a `width * height * 3` byte RGB frame and computes the new YUV420p frame, which is only `width * height * 1.5` bytes. This required careful pointer arithmetic to handle the Y, U, and V planes correctly.
* **Delta Encoding (Lossless):** The program holds two YUV frames in memory: `previous_frame` and `current_frame`. It performs a byte-by-byte subtraction to generate the `delta_frame`, which is then passed to the compressor. The first frame is flagged as a keyframe and sent as-is.
* **Entropy Coding (Lossless):** The program uses `zlib`'s streaming interface to feed all frames (the keyframe and all deltas) into the DEFLATE algorithm, writing the compressed byte stream to disk.

To validate the entire process, the program immediately reverses it, re-inflating the data, reconstructing the frames by adding the deltas, and converting the YUV back to RGB.

#### 2. The `ffmpeg` Workflow

A codec is useless without a way to get data in and out. I made the pragmatic trade-off to use `ffmpeg` as a helper tool rather than writing my own MP4 container parser.

* **Pre-processing:** `ffmpeg` is used to (1) strip the original audio and save it, and (2) decode the input video and pipe it as a raw RGB24 stream to `barecodex`'s `stdin`.
* **Post-processing:** `ffmpeg` is used again to "mux" (re-combine) the `decoded.rgb24` video file (output by `barecodex`) with the original `original_audio.aac` file, creating the final, playable `.mp4`.

## Results

The project was a complete success, resulting in a fully functional, albeit simple, video codec.

* **Metrics:** The pipeline regularly achieves **80-90% compression** on test videos, successfully demonstrating the power of the three-stage approach.
* **Outcome:** A playable video file (`final_video_with_audio.mp4`) that has been fully encoded and decoded by my C++ program, with the original audio perfectly preserved.
* **Key Lesson:** The biggest takeaway was the power of a layered approach. No single step provides all the compression. Instead, it's a pipeline where each stage prepares the data to be more compressible for the next (YUV -> Delta -> DEFLATE). I also learned the critical importance of memory management, as an off-by-one error in my YUV conversion logic (the "even dimensions" bug) initially caused massive memory corruption.

<div class="row">
    <div class="col-sm mt-3 mt-md-0">
        {% include figure.liquid loading="eager" path="assets/img/barecodex-after-run.png" title="Test Results." class="img-fluid rounded z-depth-1" %}
    </div>
</div>
<div class="caption">
    Test Results.
</div>

**If I had more time,** I would implement true **Motion Compensation** (storing motion vectors for blocks) instead of simple delta encoding, which is the key differentiator in modern codecs like H.264.

## Reflection

* **Core Technologies:**
  * **C++:** Chosen for its performance and low-level memory control. Processing millions of pixels per second requires the raw speed that C++ provides. There is no garbage collector, which is essential for real-time data streaming.
  * **`zlib`:** A practical choice. It's the industry-standard implementation of DEFLATE. Using it allowed me to focus on the *video logic*, not on writing a compression algorithm itself.
  * **`ffmpeg`:** Chosen as the "glue." It's the Swiss Army knife of video. By using it for container parsing and muxing, I narrowed the project's scope to be achievable, focusing *only* on the core "codec" logic.

The greatest challenge was debugging the data pipeline. When the output video was just a block of green static, it was incredibly difficult to know which of the three stages (or the `ffmpeg` pipe) was to blame. This forced me to write a built-in verification decoder *at the same time* as the encoder, allowing me to isolate and test each component. This project was a powerful lesson in data-oriented design and the power of demystifying the "black boxes" we use every day.

> View the BareCodeX source code and documentation on [GitHub](https://github.com/carb0ned0/BareCodeX).
