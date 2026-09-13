---
title: "A Network Arena for Benchmarking AI Agents on Network Troubleshooting"
tags: [troubleshooting, academic-paper]
---

**Source:** Wang, Z., Cornacchia, A., Sacco, A., Galante, F., Canini, M., & Jiang, D. (2025). *A Network Arena for Benchmarking AI Agents on Network Troubleshooting.* arXiv. https://arxiv.org/abs/2512.16381

**What it covers:** This paper introduces NIKA, a benchmark for evaluating LLM-driven network incident diagnosis. It packages hundreds of real incident scenarios into a standardized test set for measuring whether an AI agent can actually diagnose a network problem correctly.

**Why it's valuable:** It's an open-source, standardized, and repeatable way to test whether an AI agent can diagnose a real problem, rather than relying on a vendor's curated demo. That matters when you're trying to compare tools objectively instead of taking a sales pitch at face value.

**Who would find it useful:** Engineers evaluating AI-based troubleshooting tools, or anyone building their own agentic troubleshooting workflow who wants a way to validate it against a known set of incidents.

**Limitations:** The paper's own testing shows that even strong models struggle to localize faults accurately, which is a meaningful limitation of the benchmark itself — it's still an open problem, not a solved one.
