# Lab 0.3: Permissions & Ownership

**Module:** 0 — Linux Command Line Basics (pre-course warm-up)
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free

## Objective
Understand Linux file permissions (read/write/execute) and ownership, and practice changing them with `chmod` and `chown` — essential when a script "won't run" or a container complains about permission errors later in the course.

## Prerequisites
- Completed Lab 0.1

## Steps

1. **Create a simple shell script:**
   ```bash
   cd ~/linux-practice
   cat > greet.sh << 'EOF_TEXT'
   #!/bin/bash
   echo "Hello from a shell script!"
   EOF_TEXT
   ```
2. **Try to run it directly (this should fail):**
   ```bash
   ./greet.sh
   ```
   You should see a `Permission denied` error — the file isn't marked as executable yet.
3. **Inspect current permissions:**
   ```bash
   ls -l greet.sh
   ```
   The first column (e.g., `-rw-r--r--`) shows: file type, owner permissions, group permissions, other permissions (read/write/execute, in that order, for each).
4. **Make it executable for yourself (the owner):**
   ```bash
   chmod u+x greet.sh
   ls -l greet.sh
   ```
   Notice the owner section now includes an `x`.
5. **Run it again (should succeed this time):**
   ```bash
   ./greet.sh
   ```
6. **Practice numeric (octal) permission notation** — a common alternative to `u+x` style:
   ```bash
   chmod 755 greet.sh   # owner: read+write+execute (7), group: read+execute (5), others: read+execute (5)
   ls -l greet.sh
   ```
7. **Check file ownership:**
   ```bash
   ls -l greet.sh
   whoami
   ```
   The username shown by `whoami` should match the owner column in the `ls -l` output.
8. **(Optional, if you have sudo access) Practice changing ownership** — this is illustrative only, since you already own the file:
   ```bash
   sudo chown $(whoami):$(whoami) greet.sh
   ```

## Expected Result / Validation
```bash
./greet.sh
```
Should print `Hello from a shell script!` without any permission errors, and:
```bash
ls -l greet.sh
```
Should show `-rwxr-xr-x` (matching the `755` set in step 6).

## Cleanup
No AWS resources were touched.
```bash
rm greet.sh
```

## Troubleshooting
- **`bash: ./greet.sh: /bin/bash^M: bad interpreter`** → This means the file has Windows-style line endings; fix with `dos2unix greet.sh` (install via `sudo apt install dos2unix` if needed) or recreate the file using the `cat > ... << 'EOF_TEXT'` method above, which avoids the issue.
- **`chmod: changing permissions ... Operation not permitted`** → You likely don't own the file; check with `ls -l` and use `sudo` only if you understand why the permission change is needed.
