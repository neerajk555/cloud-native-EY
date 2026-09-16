# Lab 0.1: Navigating the Filesystem

**Module:** 0 — Linux Command Line Basics (pre-course warm-up)
**Difficulty:** Beginner
**Duration:** ~15 minutes
**Cost:** ✅ Free (runs entirely on your local machine)

## Objective
Get comfortable moving around a Linux filesystem, creating/removing directories, and understanding absolute vs. relative paths — skills you'll use constantly in every later module.

## Prerequisites
- Access to a terminal on Ubuntu (or any Linux/macOS/WSL2 terminal — commands here are standard across all of them)

## Steps

1. **Find out where you are:**
   ```bash
   pwd
   ```
2. **See what's in your current directory:**
   ```bash
   ls
   ls -la    # -l = long format (details), -a = show hidden files (dotfiles)
   ```
3. **Create a practice folder and move into it:**
   ```bash
   mkdir -p ~/linux-practice/projectA/subfolder
   cd ~/linux-practice
   ```
   Note: `~` is a shortcut for your home directory; `-p` tells `mkdir` to create parent folders as needed.
4. **Practice relative vs. absolute paths:**
   ```bash
   cd projectA          # relative path
   cd subfolder         # relative path, one level deeper
   cd ~/linux-practice   # absolute-style path back to the top
   pwd
   ```
5. **Move up one directory level, then back down using `..`:**
   ```bash
   cd projectA/subfolder
   cd ..                 # moves up one level (back to projectA)
   cd ../..               # moves up two levels (back to linux-practice)
   ```
6. **Create a few more folders in one command using brace expansion:**
   ```bash
   mkdir -p projectA/{logs,config,data}
   ls projectA
   ```
7. **Remove a directory you no longer need:**
   ```bash
   rmdir projectA/data          # only works if the folder is EMPTY
   rm -r projectA/config        # -r removes a folder and everything inside it
   ```

## Expected Result / Validation
```bash
ls ~/linux-practice/projectA
```
Should show `logs` and `subfolder`, but **not** `config` or `data` (both removed in step 7). This confirms you can navigate, create nested structures, and clean them up.

## Cleanup
No AWS resources were touched. If you want to remove your entire practice area:
```bash
rm -rf ~/linux-practice
```
**Caution:** `rm -rf` permanently deletes without a trash bin — always double-check the path before running it.

## Troubleshooting
- **`rmdir: failed to remove ... Directory not empty`** → `rmdir` only deletes empty folders by design (a safety feature); use `rm -r` for non-empty folders.
- **`cd: no such file or directory`** → Run `pwd` and `ls` to confirm exactly where you are and what's actually there before trying to `cd` again.
