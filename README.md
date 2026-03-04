# Archnemix Overnight ShortGen (Public)

**Archnemix Overnight ShortGen** is the public-facing workflow skeleton for **Archnemix ShortGen**, an AI-driven automated system that generates Reddit-style YouTube Shorts from text scripts.  

> ⚠️ **Important:** This repository is **public for workflow execution purposes only**. All proprietary code and core processing logic are contained in a **private repository**. Viewing or forking this repo does **not** grant access to the main pipeline or assets.  

---

## Features

- Provides a **workflow skeleton** (`.github/workflows/overnight_batch.yml`) for automated batch processing.
- Pulls a **private Docker image** containing the core ShortGen pipeline.
- Minimal public code to **trigger the batch pipeline** while keeping proprietary logic secure.
- Can be used to run overnight batch generation using GitHub Actions minutes.

---

## How It Works (Public View)

1. The workflow triggers on `workflow_dispatch` (manual) or scheduled events.
2. It pulls the **private Docker image** from GHCR.io using secrets.
3. The Docker container runs all the proprietary scripts internally — **no code from the container is visible here**.
4. Any commands or arguments required by the workflow are stored as **GitHub Secrets** to keep the logic hidden.

---

## Repository Structure (Public)
