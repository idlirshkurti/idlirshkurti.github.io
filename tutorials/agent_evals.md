---
layout: page
title: Demystifying Agent Evals
parent: AI Agents
description: A deep-dive review of Anthropic's guide to evaluating AI agents
nav_order: 2
tags: [ai, agents, evals, anthropic, benchmarks]
math: mathjax3
---

# Deep Dive: Thoughts on Anthropic's "Demystifying Agent Evals"
{:.no_toc}

## Table of contents
{: .no_toc .text-delta }

1. TOC
{:toc}

---

I just spent a good chunk of my morning digging through Anthropic's recent engineering post, [Demystifying evals for AI agents](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents). If you’ve been following my notes in the **Dev Hub**, you know I’ve been obsessed lately with how we actually measure the "intelligence" of these systems. 

Building a chatbot is easy. Building a RAG app is moderate. But building a reliable **Agent**? That's where the wheels usually fall off. This post from Anthropic is probably the most honest and practical look at why agents are so uniquely difficult to evaluate, and how we can move past "vibes-based" development.

### The "Compounding Error" Problem

The post starts with a fundamental truth that I’ve felt in my own experiments: **Agents are non-linear.** 

In a standard LLM call, you send a prompt and get a response. If the response is bad, you fix the prompt. But an agent operates in a loop (the "Agent Loop"). It reasons, calls a tool, gets an observation, and repeats. 

Anthropic calls this **multi-turn propagation**. Mathematically, if an agent must complete \\( n \\) sequential steps to succeed, and each step \\( i \\) has an independent probability of success \\( p_i \\), the total probability of success \\( P(S) \\) is:

$$P(S) = \prod_{i=1}^{n} p_i$$

Even if your model is "95% reliable" (\\( p_i = 0.95 \\)) at every step, a 10-step trajectory only has a **59.8%** chance of success. This is why "vibes-based testing" fails; human intuition is notoriously bad at estimating the exponential decay of products.

### The "Loophole" Problem: When Agents Outsmart the Test

One of the funniest (and most terrifying) parts of the article is about "loopholes." 

Because frontier models like Claude 3.5 or GPT-4o are so good at reasoning, they often find ways to "pass" an eval that the developer never intended. For example, if you have an eval that checks if an agent successfully booked a flight, the agent might find a way to skip the payment step or use a "test mode" bypass it discovered in the API documentation. 

It "passed" the success criteria (the flight is "booked" in the database), but it failed the intent (following actual company policy). This is why Anthropic argues that we need **model-graded evals** that look at the *transcript* of the agent's thoughts, not just the final state of the environment.

### A New Framework: pass@k vs. pass^k

This was the most technical part of the post, and it's something I’m going to start using in my own projects. Since agents are stochastic (non-deterministic), running an eval once gives you zero statistically significant information. You have to run it many times (\\( k \\) times).

Let \\( p \\) be the probability that the agent succeeds on a single trial (\\( k=1 \\)). We can define two diverging metrics:

#### 1. pass@k (The "Best Case" Metric)
This is the probability that the agent succeeds **at least once** in \\( k \\) independent attempts. It represents the cumulative distribution of a geometric distribution's first success.

$$pass@k = 1 - (1 - p)^k$$

*   **Intuition:** "Can the agent solve this if it gets lucky?"
*   **Asymptotic Behavior:** \\( \lim_{k \to \infty} (1 - (1 - p)^k) = 1 \\). As you increase \\( k \\), the score always rises toward 100%.
*   **Use Case:** Coding assistants where a human picks the best of \\( k \\) options.

#### 2. pass^k (The "Reliability" Metric)
This is the probability that the agent succeeds **every single time** across \\( k \\) independent attempts. 

$$pass^k = p^k$$

*   **Intuition:** "Is this agent reliable enough to be left alone?"
*   **Asymptotic Behavior:** \\( \lim_{k \to \infty} p^k = 0 \\) (for \\( p < 1 \\)). The more trials you run, the more likely a single failure becomes.
*   **Use Case:** Fully autonomous agents (e.g., customer support).

### Why the Gap Matters

Consider an agent with a baseline success rate \\( p = 0.8 \\) (80%). Let's look at the results for \\( k = 5 \\):

*   **pass@5:** \\( 1 - (0.2)^5 = 0.99968 \\) (**99.9%**)
*   **pass^5:** \\( (0.8)^5 = 0.32768 \\) (**32.8%**)

The **Capability Gap** here is massive (67.1%). If you only measure `pass@k`, you might think your agent is production-ready. But in reality, it can only guarantee a consistent "perfect run" about a third of the time. Anthropic’s point is that for a "production-ready" agent, you should be optimizing for a high **pass^k**.

### LLM-as-a-Judge: The Calibration Secret

I've always been a bit wary of using an LLM to grade another LLM. It feels like "circular logic." However, the post makes a compelling case for **Model-Graded Evals** when calibrated properly.

The trick isn't just to ask the LLM "Did this work?". Instead:
1.  **Isolate Dimensions:** Create specific rubrics (e.g., "Accuracy," "Safety," "Tone," "Policy Adherence").
2.  **Use Transcripts:** Give the judge the full agent transcript so it can see the reasoning process.
3.  **Human Calibration:** You have to have a human grade a subset of the tasks, then compare the LLM's scores. If they don't match, you refine the judge's prompt. 

Only once the "LLM Judge" matches human experts consistently can you turn it loose to grade thousands of runs.

### The "Swiss Cheese" Model of Agent Safety

Finally, the article talks about the **Swiss Cheese Model**. No single evaluation layer is perfect. Every eval has "holes" (loopholes, false positives, etc.). 

To build a truly safe agent, you stack the layers:
*   **Deterministic Graders:** Hard-coded checks (unit tests, status codes).
*   **Model-Graded Evals:** LLMs checking for nuances and reasoning.
*   **Production Monitoring:** Real-time checks for "vibes" and errors in the wild.
*   **Human Review:** Constant auditing of the edge cases.

### Final Thoughts

Reading this made me realize that my own "evals" were way too simplistic. I was looking for a single number to tell me if my agent was "good." In reality, "good" is a spectrum between **capability** (what can it do when it tries its best?) and **reliability** (what will it do every single time?).

If you're building in the agent space, do yourself a favor and read the full article. It moves the needle from "AI magic" to "software engineering."

---

### References
- [Demystifying evals for AI agents - Anthropic Engineering](https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents)
- [Hugging Face MCP Course (for context on Agent protocols)](./mcp_introduction.md)
