# Root Cause Analysis

Fill this in as you go. Bullet points are fine — be specific, not long.

---

## Defect 1: Proxy image fails to build

### Symptom / error output
The proxy Docker image fails to build, crashing with exit code: 100, during the apt-get install step.
### How I diagnosed it
Diagnostic command: docker compose build proxy (I scrolled up through the logs to examine the operations immediately preceding the crash to identify the trigger and located the interactive prompt error).
### Root cause
The COPY nginx.conf instruction was executed before apt-get install nginx. The package manager detected an existing configuration file at /etc/nginx/nginx.conf, triggered an interactive prompt asking whether to overwrite it (*** nginx.conf (Y/I/N/O/D/Z) ?), and immediately crashed because standard input (stdin) is unavailable in a non-interactive Docker build
### Fix
Reordered proxy/Dockerfile. Moved the COPY instruction to execute after the RUN apt-get install instruction, allowing the installation to finish normally before overwriting the default configuration file.

---

## Defect 2: Proxy cannot reach the backend

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix, and why I chose it over the alternatives

---

## Defect 4: CI pipeline fails

### Symptom / error output

### How I diagnosed it

### Root cause

### Fix

---

## Bonus questions

Optional. Answer only the ones you got to — number them, skip the rest.

>

---

## Anything you would do differently with more time
