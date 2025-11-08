---
layout: page
title: Fine-Tuning LLM
description: Fine-tuned Llama 2 using LoRA and QLoRA for custom tasks, leveraging Hugging Face tools for efficient adaptation.
# img: assets/img/12.jpg
importance: 11
category: _under-construction
related_publications: false
---

## Overview/Problem Statement

Fine-tuning LLMs adapts them to specific domains efficiently, avoiding full retraining.

Based on the tutorial, this project fine-tunes Llama 2 (7B) on custom datasets. As the sole developer, I handled quantization, LoRA setup, and training.

## Process/Research

Efficient fine-tuning methods.

1. **Dataset Prep:** Formatted for instructions.
    * **Research:** OpenAssistant dataset.
    * **Decision:** 1K samples.

2. **Quantization:** 4-bit for memory.
    * **Research:** BitsAndBytes.

3. **PEFT:** LoRA for parameter efficiency.
    * **Research:** Rank, target modules.

## Solution/Implementation

Used Hugging Face ecosystem.

* **Config:** 4-bit quant, LoRA rank 64.
* **Trainer:** SFTTrainer for supervised fine-tuning.
* **Inference:** Text generation pipeline.

## Status

This project is currently in progress and nearing completion. I will update this page with results, reflections, and code as I finalize the work and gain new insights to improve myself.

> Code and design files is yet to be uploaded.
