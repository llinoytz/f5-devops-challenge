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
The test-runner crashes with an "HTTP Error 502: Bad Gateway" when trying to reach "/api/info".
### How I diagnosed it
 Diagnostic command:docker compose up (observing the 502 error in the logs, then inspecting "proxy/nginx.conf").
### Root cause
The Nginx "upstream" block was incorrectly configured to route traffic to "127.0.0.1:8080". In Docker's isolated network environment, 127.0.0.1 points to the proxy's own container, not the backend container.
### Fix
Updated proxy/nginx.conf to use Docker's internal DNS. Changed the upstream server address from 127.0.0.1:8080 to backend:8080, allowing the proxy to correctly resolve and route traffic to the backend service.
---

## Defect 3: Tests still fail after fixing defect 2

### Symptom / error output
When running "docker compose up" from a fresh state (cold start), integration tests fail intermittently with "HTTP Error 502: Bad Gateway" or "Connection refused".
### How I diagnosed it
"docker compose down" followed by "docker compose up" (observing that the test runner and proxy fire requests before the backend finishes initializing and binding to port 8080).
### Root cause
Docker Compose starts all containers concurrently by default. The proxy and test-runner attempted to reach the backend before its server loop was fully up and listening.
### Fix, and why I chose it over the alternatives
I implemented a native Docker Compose healthcheck on the backend service (polling "/health "via Python) and configured the proxy to wait using "depends_on" with "condition: service_healthy".
Why Alternatives Were Rejected:
  1. Modifying backend source code or adding hardcoded delays ("time.sleep"). 
     Reason: Forbidden by challenge constraints. Furthermore, hardcoded sleep durations are brittle—they either introduce unnecessary wait times or fail under heavy load if startup takes longer.
  2. Writing custom shell scripts/retry loops (using nc or curl) inside container entrypoints:
     Reason: It clutters the build files (Dockerfiles) and creates unnecessary dependencies. It is much cleaner to let Docker handle the checks natively.
---

## Defect 4: CI pipeline fails

### Symptom / error output
The GitHub Actions CI pipeline fails during the "Execute test suite" step, returning "exit code 126" (Permission denied) when attempting to run "scripts/run-tests.sh".
### How I diagnosed it
Reviewed the GitHub Actions logs online and ran "git ls-files --stage scripts/run-tests.sh" locally, which confirmed the file mode was 100644(non-executable) instead of 100755.
### Root cause
The test script was initially committed to the Git repository without the executable bit set. When the CI runner checks out the code on a fresh Ubuntu environment, it respects this missing permission and blocks the execution of the script.
### Fix
Updated the file permissions directly within the Git index by running "git update-index --chmod=+x scripts/run-tests.sh", followed by a commit and push. This ensures the script natively retains its executable status for the CI pipeline and anyone who clones the repository in the future.
---

## Bonus 1

### Purpose
Optimize the Proxy Docker image to save storage space, accelerate CI/CD pipelines, and minimize the security attack surface by removing unnecessary packages and tools.

### Changes Made
* Replaced the heavy base image ubuntu:24.04 with the lightweight and lean nginx:alpine.
* Removed unnecessary tools and packages that are not required in production (such as vim, htop, git, curl, and apt-get installations).

### Measurement Results
Image Size Before Optimization: 124MB (Content Size) / 452MB (Disk Usage).
Image Size After Optimization: 28.8MB (Content Size) / 102MB (Disk Usage).
Improvement: A reduction of approximately 77% in image size!
>

---

## Anything you would do differently with more time
