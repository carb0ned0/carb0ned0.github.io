---
layout: page
title: Reinforcement Learning Game Bot
description: Trained an agent to play games like T-Rex Runner, Flappy Bird, Tic-Tac-Toe, Tetris, and Snake, showcasing reinforcement learning for AI and robotics applications.
# img: assets/img/12.jpg
importance: 15
category: _under-construction

related_publications: false
---

## Overview/Problem Statement

Games provide ideal environments for testing reinforcement learning (RL) algorithms, where agents learn optimal strategies through trial and error. This skill is highly hireable as RL is crucial for robotics and AI, pairing well with warehouse robot projects.

This project involves training an RL agent to master multiple games. As the sole developer, I am designing the environments, selecting algorithms, and tuning hyperparameters to achieve high performance.

## Process/Research

Focused on RL fundamentals and game-specific challenges.

1. **Environment Setup:** Standardized game simulations.
    * **Research:** Libraries for reproducible environments.
    * **Decision:** OpenAI Gym for a variety of games like CartPole (as a proxy for Flappy Bird), custom wrappers for others.

2. **Algorithm Selection:** Stable and efficient RL method.
    * **Research:** Compared DQN, A2C, and PPO for discrete action spaces.
    * **Decision:** PPO for its balance of performance and stability in dynamic environments.

3. **Training Optimization:** Handling reward shaping and exploration.
    * **Research:** Techniques like curriculum learning for complex games like Tetris.

## Solution/Implementation

The implementation uses TensorFlow to build and train the PPO agent.

* **Game Environments:** Integrated with OpenAI Gym, including custom environments for Snake and Tetris.
* **Agent Architecture:** Policy and value networks with shared features, trained via proximal policy optimization.
* **Training Loop:** Episodes with experience collection, policy updates, and logging for progress tracking.

## Status

This project is currently in progress and nearing completion. I will update this page with results, reflections, and code as I finalize the work and gain new insights to improve myself.

> Code and design files is yet to be uploaded.
