---
layout: page
title: Large Language Model from Scratch
description: Implemented a GPT-like language model from scratch using PyTorch, trained on the tiny Shakespeare dataset to understand transformer architecture.
# img: assets/img/12.jpg
importance: 10
category: _under-construction
related_publications: false
---

## Overview/Problem Statement

Large language models like GPT are powerful but opaque. Building one from scratch demystifies the process and teaches core concepts.

Based on Andrej Karpathy's tutorial, this project implements a decoder-only transformer for text generation. As the sole developer, I coded the model, tokenizer, and training loop.

## Process/Research

Deep dive into transformers.

1. **Data Preparation:** Used tiny Shakespeare (1MB).
    * **Research:** Character-level tokenization.
    * **Decision:** Vocabulary of 65 chars.

2. **Model Architecture:** Self-attention, multi-head.
    * **Research:** "Attention is All You Need" paper.
    * **Decision:** Scaled dot-product attention, positional encodings.

3. **Training:** Autoregressive on GPU.
    * **Research:** Adam optimizer, cross-entropy loss.

## Solution/Implementation

PyTorch-based model with embeddings, attention blocks, feedforward layers.

* **Tokenizer:** Char to int mapping.
* **Attention:** Masked for causality.
* **Training Loop:** Batches, validation split.

## Status

This project is currently in progress and nearing completion. I will update this page with results, reflections, and code as I finalize the work and gain new insights to improve myself.

> Code and design files is yet to be uploaded.
