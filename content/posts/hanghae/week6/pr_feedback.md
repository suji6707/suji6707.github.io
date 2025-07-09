+++
title = 'week6 PR 피드백'
date = 2025-07-07T00:20:22+09:00
draft = true
+++

🟢🟢🟢WEEK6 피드백

- Exponential Backoff: the wait time increases exponentially (baseDelay * 2^attempt). This prevents thundering herd problems where all failed clients retry at exactly the same time.
- Jitter: Adding randomness to the backoff time (0-50% variation)
