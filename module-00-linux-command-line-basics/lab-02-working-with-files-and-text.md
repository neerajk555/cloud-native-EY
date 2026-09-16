# Lab 0.2: Working with Files & Text

**Module:** 0 — Linux Command Line Basics (pre-course warm-up)
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free

## Objective
Practice creating, viewing, editing, copying, and searching inside text files from the command line — you'll do this constantly when editing Dockerfiles, YAML manifests, and buildspecs later in the course.

## Prerequisites
- Completed Lab 0.1

## Steps

1. **Create a file with content directly from the command line** (no text editor needed):
   ```bash
   cd ~/linux-practice
   cat > notes.txt << 'EOF_TEXT'
   Cloud Native Training Notes
   Module 1: Introduction
   Module 2: Microservices
   Module 3: Docker and ECR
   TODO: review Kubernetes basics
   EOF_TEXT
   ```
2. **View the file's contents:**
   ```bash
   cat notes.txt          # print the whole file
   head -n 2 notes.txt    # print just the first 2 lines
   tail -n 2 notes.txt    # print just the last 2 lines
   ```
3. **Copy and rename files:**
   ```bash
   cp notes.txt notes-backup.txt
   mv notes-backup.txt archive-notes.txt   # mv renames (or moves) a file
   ls
   ```
4. **Search inside a file with `grep`:**
   ```bash
   grep "Module" notes.txt          # find every line containing "Module"
   grep -i "todo" notes.txt         # -i = case-insensitive search
   grep -c "Module" notes.txt       # -c = count matching lines
   ```
5. **Edit the file directly in the terminal** using `nano` (a beginner-friendly editor):
   ```bash
   nano notes.txt
   ```
   Add a new line: `Module 4: Kubernetes on EKS`
   Save and exit: `Ctrl+O` (write out), `Enter` to confirm filename, then `Ctrl+X` (exit).
6. **Append to a file without opening an editor:**
   ```bash
   echo "Module 5: Design Patterns" >> notes.txt
   cat notes.txt
   ```
   Note: `>>` appends; a single `>` would **overwrite** the entire file — an important distinction to remember.
7. **Count words, lines, and characters:**
   ```bash
   wc notes.txt
   wc -l notes.txt    # just the line count
   ```

## Expected Result / Validation
```bash
grep "Module" notes.txt
```
Should return 5 lines (Modules 1 through 5), confirming both your `nano` edit and your `echo >>` append succeeded correctly.

## Cleanup
No AWS resources were touched.
```bash
rm notes.txt archive-notes.txt
```

## Troubleshooting
- **Accidentally overwrote a file with `>` instead of `>>`** → There's no undo for this; always double-check you meant `>>` (append) rather than `>` (overwrite) before running the command.
- **Stuck inside `nano` and don't know how to exit** → Press `Ctrl+X`; if you made changes, it will ask whether to save (`Y`/`N`) before exiting.
