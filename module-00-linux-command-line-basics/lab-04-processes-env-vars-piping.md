# Lab 0.4: Processes, Environment Variables & Piping

**Module:** 0 — Linux Command Line Basics (pre-course warm-up)
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free

## Objective
Practice viewing/killing processes, setting environment variables (which you'll use constantly with AWS CLI profiles and credentials), and chaining commands together with pipes — a core Linux skill for filtering and transforming command output.

## Prerequisites
- Completed Lab 0.1

## Steps

1. **Start a long-running background process** to practice with:
   ```bash
   sleep 300 &
   ```
   The `&` runs it in the background so your terminal stays free. Note the job number and process ID (PID) printed.
2. **List running processes:**
   ```bash
   ps aux | grep sleep
   ```
   This introduces your first **pipe** (`|`): the output of `ps aux` is fed as input into `grep sleep`, filtering thousands of lines down to just the ones you care about.
3. **Stop the background process:**
   ```bash
   kill %1
   ```
   (`%1` refers to background job #1; alternatively use `kill <PID>` with the process ID from step 1.)
4. **Confirm it's gone:**
   ```bash
   ps aux | grep sleep
   ```
   You should now only see the `grep sleep` command itself matching (grep always matches its own search pattern) — the actual `sleep 300` process should be gone.
5. **Set a temporary environment variable** (exists only in this terminal session):
   ```bash
   export MY_ENV=training
   echo $MY_ENV
   ```
6. **See all environment variables**, filtered to just AWS-related ones (useful for debugging AWS CLI issues later in the course):
   ```bash
   env | grep -i aws
   ```
7. **Chain multiple commands together** with pipes to count something meaningful — e.g., count how many `.md` files exist in your practice folder:
   ```bash
   find ~/linux-practice -name "*.md" | wc -l
   ```
8. **Sort and get unique values** — a classic pipe combo:
   ```bash
   echo -e "banana\napple\nbanana\ncherry\napple" | sort | uniq
   ```
9. **Redirect command output into a file, then view it:**
   ```bash
   ps aux > running-processes.txt
   wc -l running-processes.txt
   ```

## Expected Result / Validation
Step 8 should output exactly three unique fruit names alphabetically sorted:
```
apple
banana
cherry
```
This confirms you understand piping (`|`) — feeding one command's output as the next command's input — which you'll rely on heavily when filtering `kubectl get pods`, `aws` CLI JSON output, and log files in later modules.

## Cleanup
No AWS resources were touched.
```bash
rm -f ~/linux-practice/running-processes.txt
unset MY_ENV
```

## Troubleshooting
- **`kill %1` says "no such job"** → The job may have already finished (5 minutes hadn't elapsed, but if you took a break, check with `jobs`); if so, there's nothing to clean up.
- **`env | grep -i aws` shows nothing** → That's expected if you haven't run `aws configure` yet (Module 1, Lab 1.1 sets this up) — the AWS CLI stores credentials in `~/.aws/` files, not always as environment variables, unless you've explicitly exported them.
