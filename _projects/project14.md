---
layout: page
title: Personalized Fashion Recommender
description: A system that suggests clothing items based on user-selected outfits, leveraging fashion datasets to provide tailored recommendations for personal styling.
importance: 15
category: _under-construction
published: false
related_publications: false
---

## Overview/Problem Statement

Fashion recommendation systems often fail to account for real-time user selections or personal style preferences, leading to generic suggestions that don't match individual tastes. This project enhances personal styling tools by offering contextual recommendations based on chosen items.

This project develops a recommender that uses user-selected clothing sets to suggest complementary pieces. As the sole developer, I am managing dataset integration, algorithm design, and model implementation.

## Process/Research

Investigated datasets, recommendation techniques, and implementation strategies.

1. **Data Sources:** Sourcing comprehensive fashion data.
    * **Research:** Explored public datasets with clothing attributes, images, and user interactions.
    * **Decision:** Utilized H&M Personalized Fashion Recommendations and Polyvore Outfits datasets for transaction histories, item metadata, and outfit compatibility.

2. **Recommendation Algorithm:** Selecting an effective method for item-based suggestions.
    * **Research:** Compared content-based filtering (using item features like color and style) vs. collaborative filtering (based on user patterns).
    * **Decision:** Hybrid approach with content-based filtering for similarity matching on selected items, enhanced by collaborative elements for broader suggestions.

3. **Integration Features:** Incorporating visual and attribute-based matching.
    * **Research:** Techniques for image similarity and attribute embedding.
    * **Decision:** Used DeepFashion dataset elements for visual features, processed via libraries like TensorFlow.

## Solution/Implementation

The system is built in Python, processing datasets and generating recommendations.

* **Data Pipeline:** Load and preprocess datasets (e.g., H&M for transactions, Polyvore for outfits) using pandas and scikit-learn.
* **Model:** Content-based recommender with cosine similarity on item embeddings, filtered by user-selected sets.
* **Interface:** Web or app prototype allowing users to input selections and view suggestions with images.

## Status

This project is currently in progress and approaching completion. I will update this page with results, reflections, and code as I finalize the work and gain new insights to improve myself.

> Code and design files is yet to be uploaded.
