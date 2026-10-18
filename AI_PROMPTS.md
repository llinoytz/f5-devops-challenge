# Generative AI Disclosure

Using AI tools for this challenge is allowed. Be honest here — an accurate
"I used AI heavily" is worth more to us than an inaccurate "I did not".

**Did you use AI tools?** (yes / no):

**Which ones?** (e.g. ChatGPT, Claude, Copilot, Cursor):

> If you answered **no**, write "N/A" in sections 1 and 2 and skip to
> section 3. That is a perfectly good answer.

---

## 1. Key prompts

List the main prompts you used. Paraphrasing is fine.

1. Error Decoding & Syntax: Used to parse verbose Docker BuildKit logs and quickly decode specific CI errors (like exit code 126) to find the exact Git command (`git update-index`) needed for file permissions.
2. Docker Compose Best Practices: Consulted for the correct syntax to implement native `healthcheck` dependencies, and to verify how Docker's internal DNS resolves service names for the Nginx `upstream` block.
3. Security Research: Used as a quick reference to verify the necessary `Dockerfile` commands for migrating to `nginx:alpine` and running the proxy securely as a non-root user.

---

## 2. Where the AI got it wrong

Describe **one concrete case** where the AI's answer was wrong, outdated, or
sub-optimal. Cover: what it suggested, how you noticed it was wrong, and what
you did instead. 3–5 sentences.

**The Error:**
 While troubleshooting the connection between the proxy and the backend (Task 2), the AI initially suggested modifying the Python code in "backend/app.py" to change its host binding address and make it universally accessible (0.0.0.0). 

**How I caught it:**
 I immediately rejected this suggestion because I had read the project's ground rules, which explicitly stated: "Treat backend/app.py as a third-party service you cannot modify." Instead of blindly applying the AI's generated code, I relied on my own understanding of architecture constraints and fixed the issue strictly at the infrastructure level by correctly configuring the "nginx.conf" upstream block and the "docker-compose.yml" network mappings.
> ...

---

## 3. Anything in your submission you could not fully explain?

We would rather know. Write "nothing" if that is the case.

> ...
