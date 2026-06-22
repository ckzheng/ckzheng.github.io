---
title: "Policy Gradient Methods: A Quick Refresher"
date: 2026-06-22T16:00:00+08:00
draft: false
math: true
tags: ["reinforcement-learning", "policy-gradient", "notes"]
categories: ["RL"]
summary: "A compact walkthrough of the policy gradient theorem, REINFORCE, and why we subtract a baseline — with the math written out."
ShowToc: true
TocOpen: false
---

This is a sample post demonstrating **math rendering**, **code blocks**, **a table of contents**, and **tags**. Delete it once you've confirmed everything works, or keep it as a template.

## The objective

In policy gradient methods we directly optimize a parameterized policy $\pi_\theta(a \mid s)$ to maximize the expected return:

$$
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}\!\left[ \sum_{t=0}^{T} \gamma^t r_t \right].
$$

The key result — the **policy gradient theorem** — tells us we can compute the gradient without differentiating through the environment dynamics:

$$
\nabla_\theta J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}\!\left[ \sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t \mid s_t)\, \Psi_t \right],
$$

where $\Psi_t$ can be the total return, the reward-to-go, or an advantage estimate $A^{\pi}(s_t, a_t)$.

## Why subtract a baseline?

Inline math works too: subtracting a state-dependent baseline $b(s_t)$ leaves the gradient *unbiased* because $\mathbb{E}_{a \sim \pi_\theta}\!\left[\nabla_\theta \log \pi_\theta(a \mid s)\right] = 0$, yet it can dramatically reduce variance. A common choice is the value function $b(s_t) = V^{\pi}(s_t)$, which gives the advantage $A^{\pi}(s_t, a_t) = Q^{\pi}(s_t, a_t) - V^{\pi}(s_t)$.

## REINFORCE in a few lines

```python
import torch

def reinforce_loss(log_probs, returns, baseline=None):
    """Vanilla REINFORCE loss with an optional baseline.

    log_probs: Tensor [T]  — log pi(a_t | s_t)
    returns:   Tensor [T]  — discounted reward-to-go G_t
    baseline:  Tensor [T]  — optional V(s_t) estimate
    """
    advantages = returns if baseline is None else returns - baseline
    # Negative because optimizers minimize.
    return -(log_probs * advantages.detach()).sum()
```

## Takeaways

1. Policy gradients optimize the policy **directly**, sidestepping value-function bootstrapping.
2. The score-function trick $\nabla_\theta \log \pi_\theta$ is what makes the estimator tractable.
3. Baselines reduce variance for free — the foundation for actor-critic and later PPO.

> Next up: writing out the PPO clipped objective and where the trust-region intuition comes from.
