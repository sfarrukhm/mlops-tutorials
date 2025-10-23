# 🧭 Data Version Control (DVC) for Beginners — A Step-by-Step Guide

Data versioning is one of those ideas that makes perfect sense *after* you've seen it in action — but before that, it feels like magic. This tutorial walks you from zero to a clear, practical understanding of how to use **DVC (Data Version Control)** alongside **Git** to version your machine learning data.

We'll use the [**NYC Taxi Trip dataset**](https://www.nyc.gov/site/tlc/about/tlc-trip-record-data.page) as an example — monthly parquet files that grow over time — and see how DVC helps us manage, version, and reproduce our work. 

Note: Pay close attention to the comments in the code blocks. They provide answers to any questions that might arise as you proceed.

---

## 🚀 TL;DR — Quick Start (Five Commands to Get Started)

```bash
# 1. Start fresh with Git and DVC
git init && dvc init          # Initialize both version control systems

# 2. Tell DVC to track your data
dvc add data/                 # DVC creates a pointer file for your data

# 3. Save the pointer to Git (not the actual data!)
git add data.dvc .gitignore   # Only tracking metadata, not raw files

# 4. Set up where to store the actual data
dvc remote add -d store /path # This can be local or cloud storage

# 5. Upload your data to that storage
dvc push                      # Send data to remote, sync with team

```

That's the essence! But read on to understand *why* these work and build a solid mental model. ⬇️

---

## 1. Why Data Versioning Matters

When you're training machine learning models, your data is constantly changing:

- You collect new data every month.
- You clean and preprocess it differently over time.
- You test different feature sets.

Soon, you find yourself asking:

> "Wait, which dataset did I train this model on?"
> 

Git is great for versioning code, but it's terrible for big binary files like `.parquet` or `.csv`. Pushing gigabytes of data to GitHub? Forget it.

That's where **DVC** comes in. DVC is like **Git's sidekick** — it helps you version, share, and reproduce datasets and models, while Git still handles your code and metadata.

---

## 2. Core Concepts (Understand Before You Touch the Terminal)

Let's build a quick mental model before we type anything.

| Concept | What It Is | Why It Matters |
| --- | --- | --- |
| **Git** | Tracks small text/code files | Keeps your project history |
| **DVC** | Tracks large data & model files | Enables reproducibility |
| **.dvc file** | A small text file that points to your actual data | Git stores this file, not the data |
| **DVC cache** | Local storage where DVC keeps data versions (by hash) | Speeds up switching versions |
| **Remote storage** | Cloud or local path for your actual dataset | Lets you share with teammates |
| **Data hash** | Unique fingerprint of data content | Identifies dataset version |

🧠 **Think of Git as your project's table of contents, and DVC as the warehouse that stores the big stuff your table of contents refers to.**

### Quick Reference: When to Use What

| I want to... | Use this |
| --- | --- |
| Track code, configs, small files | `git add` |
| Track datasets, models, large files | `dvc add` |
| Run a data pipeline | `dvc repro` |
| Share data with teammates | `dvc push` |
| Get teammate's data | `dvc pull` |

---

## 3. Setting Up the Project

Let's start from scratch and set up both Git and DVC:

```bash
# Step 1: Create a new project directory
mkdir nyc-taxi-dvc-demo
# This creates a folder for our project

# Step 2: Navigate into the directory
cd nyc-taxi-dvc-demo
# Now we're inside our project folder

# Step 3: Initialize Git (for code versioning)
git init
# This creates a .git/ folder to track our code history
# Output: "Initialized empty Git repository in ..."

# Step 4: Install DVC (if not already installed)
pip install dvc
# This installs the DVC command-line tool
# You only need to do this once per environment

# Step 5: Initialize DVC (for data versioning)
dvc init
# This creates:
#   - .dvc/ directory (DVC's configuration and cache)
#   - .dvcignore file (like .gitignore, but for DVC)
# Output: "Initialized DVC repository"

# Step 6: Commit the DVC setup to Git
git commit -m "Initialize Git and DVC"
# This saves the DVC configuration files to Git
# Now Git knows about DVC, and they can work together!

```

### 🔍 What You'll See After Setup

After running these commands, your project structure looks like this:

```
nyc-taxi-dvc-demo/
├── .git/          ← Git's internal files (tracks code)
├── .dvc/          ← DVC's internal files (tracks data config)
│   ├── config     ← DVC settings (remotes, cache location, etc.)
│   └── .gitignore ← Tells Git to ignore DVC's cache
└── .dvcignore     ← Tells DVC what files to ignore

```

At this point, your project is Git + DVC ready! 🎉

---

## 4. Adding and Tracking Your First Dataset

Let's create some sample data files to track:

```bash
# Step 1: Create the directory structure
mkdir -p data/raw
# The -p flag creates parent directories if they don't exist
# Now we have: data/raw/

# Step 2: Simulate adding dataset files
# (In real life, these would be your actual data files)
# For this tutorial, let's say you've placed these files:
#   data/raw/2023-01.parquet  (e.g., 50 MB)
#   data/raw/2023-02.parquet  (e.g., 52 MB)

```

Your data folder now looks like this:

```
data/
  raw/
    2023-01.parquet  ← January taxi data
    2023-02.parquet  ← February taxi data

```

Now let's tell DVC to track this data:

```bash
# Step 3: Track the data folder with DVC
dvc add data/raw
# What happens here:
#   1. DVC calculates an MD5 hash of all files in data/raw/
#   2. DVC copies these files to its cache (.dvc/cache/)
#   3. DVC creates a pointer file: data/raw.dvc
#   4. DVC adds "data/raw/" to .gitignore automatically
# Output: "Adding... 100%|████████| 2/2"

# Step 4: Add the pointer file to Git
git add data/raw.dvc .gitignore
# We're adding:
#   - data/raw.dvc (the pointer file - small, ~100 bytes)
#   - .gitignore (updated to exclude data/raw/)
# Note: We're NOT adding the actual data files!

# Step 5: Commit to Git
git commit -m "Track raw taxi data with DVC"
# This saves the pointer file to Git history
# The actual data is stored separately by DVC

```

### 🧐 What Just Happened? A Detailed Breakdown

**Note:** When you `dvc add` a directory, DVC creates a `.dvc` file with the same name (e.g., `data/raw.dvc`). This small text file is what Git tracks, while the actual data stays in DVC's cache.

Here's what happens behind the scenes:

1. **Hash Calculation** 🔢
    
    ```
    DVC reads each file and calculates its MD5 hash
    2023-01.parquet → md5: a1b2c3d4...
    2023-02.parquet → md5: e5f6g7h8...
    Combined hash for entire directory → md5: 6a4e8db1...
    
    ```
    
2. **Cache Storage** 💾
    
    ```
    Files are copied to .dvc/cache/:
    .dvc/cache/a1/b2c3d4... → copy of 2023-01.parquet
    .dvc/cache/e5/f6g7h8... → copy of 2023-02.parquet
    
    ```
    
3. **Pointer File Creation** 📄
    
    ```
    data/raw.dvc is created (shown below)
    
    ```
    
4. **Git Ignore Update** 🚫
    
    ```
    .gitignore now contains:
    /data/raw
    
    ```
    

**Important:** Your files in `data/raw/` are still there and usable! They haven't moved. DVC just made a backup in its cache and created a pointer file.

### 📄 Inside the Pointer File

Open `data/raw.dvc` and you'll see:

```yaml
outs:
- md5: 6a4e8db12a3c54b7d14d3e2a6f4e8b  # ← Hash of the directory contents 
  size: 25874301                     # ← Total size in bytes of all files 
  nfiles: 2                          # ← Number of files in the directory
  hash: md5                          # ← Hash type used 
  path: data/raw                     # ← Path in the workspace that is tracked by DVC

```

**What this means:**

- `md5`: A unique fingerprint of your data (changes if data changes)
- `size`: Total size of all files combined
- `path`: Where the data lives in your project

This tiny file (about 100 bytes) represents 25+ MB of data! 🎉

---

## 5. Storing Data Remotely

Now let's set up remote storage so you can share data with your team and back it up.

### Option A: Local Storage (Best for Learning) 🎓

Perfect for understanding DVC without cloud setup costs:

```bash
# Step 1: Create a local directory to act as "remote" storage
mkdir -p /tmp/dvc-storage
# This is our fake "cloud" storage for learning
# In production, you'd use S3, GCS, Azure, etc.

# Step 2: Tell DVC about this remote location
dvc remote add -d myremote /tmp/dvc-storage
# Breaking down the command:
#   - 'remote add' = configure a storage location
#   - '-d' = make this the DEFAULT remote
#   - 'myremote' = name for this remote (you choose this)
#   - '/tmp/dvc-storage' = path to storage location
# Output: "Setting 'myremote' as a default remote."

# Step 3: Save this configuration to Git
git add .dvc/config
# The remote configuration is stored in .dvc/config
# This lets your teammates use the same remote

git commit -m "Configure DVC remote storage"
# Now everyone who clones the repo knows where data lives

```

After this, your `.dvc/config` file contains:

```
[core]
    remote = myremote
['remote "myremote"']
    url = /tmp/dvc-storage

```

### Option B: Cloud Storage (Production Use) ☁️

For real projects, you'll want cloud storage:

```bash
# For AWS S3:
# Prerequisites:
#   1. Install AWS CLI: pip install awscli
#   2. Configure credentials: aws configure

dvc remote add -d myremote s3://my-bucket-name/dvc-storage
# This tells DVC to use an S3 bucket
# Make sure the bucket exists and you have access!

# For Google Cloud Storage:
# Prerequisites:
#   1. Install: pip install dvc[gs]
#   2. Authenticate: gcloud auth login

dvc remote add -d myremote gs://my-bucket-name/dvc-storage

# For Azure Blob Storage:
# Prerequisites:
#   1. Install: pip install dvc[azure]
#   2. Set up connection string or credentials

dvc remote add -d myremote azure://mycontainer/path

# After configuring any cloud remote:
git add .dvc/config
git commit -m "Configure DVC cloud remote storage"

```

### Pushing Data to Remote

Now let's upload our data:

```bash
# Upload all tracked data to the remote
dvc push
# What happens:
#   1. DVC checks which files are in .dvc/cache/
#   2. DVC checks which files are already in remote storage
#   3. DVC uploads only the NEW or CHANGED files
#   4. Progress bar shows upload status
# Output:
#   "Transferring... 100%|████████| 2/2"
#   "2 files pushed"

```

💡 **What's happening behind the scenes:**

```
Your Computer                     Remote Storage
-------------                     --------------
.dvc/cache/
  ├── a1/b2c3d4... (50 MB)   →   /tmp/dvc-storage/a1/b2c3d4...
  └── e5/f6g7h8... (52 MB)   →   /tmp/dvc-storage/e5/f6g7h8...

Git Repository (GitHub)
------------------
  ├── data/raw.dvc (100 bytes) ✓
  ├── train.py                 ✓
  └── .dvc/config              ✓

```

**Key insight:**

- **GitHub** stores: code + tiny pointer files (`.dvc` files)
- **Remote storage** stores: actual large data files
- This keeps your Git repo small and fast! 🚀

---

## 6. Versioning Data Changes

Time passes, and you get new data. Let's see how DVC handles updates:

```bash
# Simulate receiving March data
# In real life, you'd download or generate this file
# Let's say you now have:
#   data/raw/2023-01.parquet  (unchanged)
#   data/raw/2023-02.parquet  (unchanged)
#   data/raw/2023-03.parquet  (NEW! - 53 MB)

```

Your data folder now looks like:

```
data/
  raw/
    2023-01.parquet  ← January (already tracked)
    2023-02.parquet  ← February (already tracked)
    2023-03.parquet  ← March (NEW!)

```

Now let's version this change:

```bash
# Step 1: Tell DVC about the updated directory
dvc add data/raw
# What DVC does:
#   1. Scans data/raw/ and finds 3 files now
#   2. Recognizes 2023-01 and 2023-02 from cache (skips them)
#   3. Calculates hash for NEW file (2023-03.parquet)
#   4. Adds 2023-03.parquet to cache
#   5. Updates data/raw.dvc with NEW combined hash
# Output: "Adding... 100%|████████| 1/1 (only 1 new file!)"

# Step 2: Check what changed
git diff data/raw.dvc
# Output shows:
#   - md5: 6a4e8db1...  (old hash)
#   + md5: 9f8e7d6c...  (new hash)
#   - nfiles: 2         (old number of files)
#   - nfiles: 3         (new number of files)
#   - size: 25874301    (old size: ~25 MB)
#   + size: 78912450    (new size: ~78 MB)
# The hash changed = data version changed!

# Step 3: Commit the new data version
git add data/raw.dvc
git commit -m "Add March 2023 taxi data"
# This creates a NEW Git commit that points to the NEW data version

# Step 4: Upload ONLY the new file to remote
dvc push
# DVC is smart! It only uploads what's new:
#   - 2023-01.parquet → already in remote, skip ✓
#   - 2023-02.parquet → already in remote, skip ✓
#   - 2023-03.parquet → NEW, upload this! 📤
# Output: "1 file pushed"

```

### ✨ Deduplication Magic

This is called **deduplication** — DVC only stores what changed. Even with datasets that grow from 25 MB to 78 MB, you only upload the 53 MB difference!

### ⏳ Time Travel: Going Back to Previous Data

Want to go back to the February version?

```bash
# Step 1: Find the commit where you had only Jan + Feb data
git log --oneline
# Output:
#   9f8e7d6 (HEAD) Add March 2023 taxi data  ← current
#   3a2b1c0 Track raw taxi data with DVC     ← old version
#   1a2b3c4 Initialize Git and DVC

# Step 2: Go back to that commit
git checkout 3a2b1c0
# Git now rolls back:
#   - train.py to old version
#   - data/raw.dvc to old hash (6a4e8db1...)
# But your data/raw/ still has 3 files!

# Step 3: Sync data to match the .dvc file
dvc checkout
# DVC reads data/raw.dvc and sees: "hash should be 6a4e8db1..."
# DVC checks cache and restores ONLY Jan + Feb files
# March file disappears! ✨
# Output: "A  data/raw/2023-01.parquet"
#         "A  data/raw/2023-02.parquet"
#         "D  data/raw/2023-03.parquet"  ← Deleted

# Alternative: If files aren't in local cache
dvc pull
# This downloads the old data version from remote storage

```

**What just happened?**

1. `git checkout` moved your **code and .dvc pointers** back in time
2. `dvc checkout` moved your **actual data** back in time
3. Your workspace now perfectly matches the past state! 🎉

### Going Back to Present

```bash
# Return to the latest version
git checkout main  # or master, depending on your default branch

# Sync data to latest version
dvc checkout
# Now you have all 3 months again!

```

---

## 7. Linking Code and Data Versions

This is where DVC truly shines: keeping code and data in sync.

### Visual Timeline 📅

```
Commit A (Jan data)          Commit B (Feb data)          Commit C (March data)
├── train.py v1              ├── train.py v2              ├── train.py v3
├── data/raw.dvc             ├── data/raw.dvc             ├── data/raw.dvc
│   (hash: abc123)           │   (hash: def456)           │   (hash: ghi789)
│   Points to: Jan           │   Points to: Jan + Feb     │   Points to: Jan + Feb + Mar

```

Each Git commit "freezes" both your **code version** AND your **data version** (via the `.dvc` file hash).

### Real-World Example: Training a Model

```bash
# Scenario: You're experimenting with different model versions

# Experiment 1: Train on January data with simple model
git checkout 3a2b1c0  # Go to commit with Jan data
dvc checkout          # Restore Jan data

# Edit your model
echo "# Simple linear model" >> train.py
python train.py
# Output: Accuracy: 78.5%

# Save this experiment
git add train.py
git commit -m "Experiment 1: Linear model on Jan data"
git tag experiment-v1  # Tag for easy reference

# Experiment 2: Same model, but with Jan + Feb data
git checkout 4b3c2d1  # Go to commit with Jan + Feb
dvc checkout          # Restore Jan + Feb data

python train.py
# Output: Accuracy: 81.2%

git tag experiment-v2

# Experiment 3: New model architecture with all data
git checkout main     # Latest commit (Jan + Feb + Mar)
dvc checkout          # Restore all data

# Implement neural network
echo "# Deep learning model" >> train.py
python train.py
# Output: Accuracy: 85.7%

git add train.py
git commit -m "Experiment 3: Neural network on all data"
git tag experiment-v3

```

### Reproducing Any Experiment 🔁

Months later, your colleague asks: "How did you get 81.2% accuracy?"

```bash
# Simply checkout that exact experiment
git checkout experiment-v2
# This restores:
#   ✓ The code (train.py v2)
#   ✓ The .dvc pointer (pointing to Jan + Feb data)

dvc pull
# This downloads the exact data version
#   ✓ Jan data
#   ✓ Feb data
#   ✗ No March data (didn't exist in v2)

python train.py
# Output: Accuracy: 81.2%  ← Exact same result!

```

**This is reproducibility!** 🎉

Every experiment can be recreated exactly because:

- Git tracks the code version
- DVC tracks the data version
- Both are linked through commits

---

## 8. Managing and Sharing Data

### Check What's Out of Sync 🔍

```bash
# Check the status of your data
dvc status
# Possible outputs:

# Case 1: Everything is in sync
# Output: "Data and pipelines are up to date."

# Case 2: You modified data locally but haven't pushed
# Output:
#   "data/raw.dvc:
#     changed: data/raw"
# Meaning: Your local cache has changes not in remote

# Case 3: Remote has newer data
# Output:
#   "data/raw.dvc:
#     not in cache: data/raw"
# Meaning: You need to pull from remote

```

### Compare Local vs Remote 📊

```bash
# See what files would be pushed/pulled
dvc status --cloud
# Output:
#   "new:      data/raw/2023-03.parquet"
#   "deleted:  data/raw/old_file.parquet"
# This shows differences between local cache and remote

```

### Clean Up Old Data 🧹

Over time, your cache can grow large with old data versions:

```bash
# See cache size
du -sh .dvc/cache
# Output: "2.3G  .dvc/cache"

# Remove data not used in current workspace
dvc gc --workspace
# Keeps only data referenced by current .dvc files
# Deletes everything else from cache

# Remove data not in ANY Git commit
dvc gc --all-commits
# Dangerous! Only keeps data from committed versions
# Removes everything else (even if in remote)

# Recommended: Clean cache but keep remote data
dvc gc --workspace --cloud
# Removes unused files from local cache
# Keeps everything in remote storage
# You can always re-download if needed

```

### Ignore Temporary Files 🚫

```bash
# Create .dvcignore file
cat > .dvcignore << EOF
# Temporary data files
data/tmp/
data/cache/
*.log
*.tmp

# System files
.DS_Store
Thumbs.db
EOF

# Now DVC will skip these patterns
dvc add data/
# Won't track anything matching .dvcignore patterns

```

### Share with Your Team 👥

Your teammate wants to work on the project:

```bash
# Teammate's computer:

# Step 1: Clone the Git repository
git clone https://github.com/you/nyc-taxi-dvc-demo.git
cd nyc-taxi-dvc-demo
# Now they have:
#   ✓ All code
#   ✓ All .dvc pointer files
#   ✗ No actual data yet

# Step 2: Download the data
dvc pull
# DVC reads .dvc/config to find remote location
# DVC reads all .dvc files to know what to download
# DVC downloads data from remote to cache
# DVC places files in correct locations
# Output:
#   "Fetching... 100%|████████| 3/3"
#   "Checkout... 100%|████████| 3/3"

# Step 3: Start working!
python train.py
# They now have the EXACT same environment as you! 🎉

```

---

## 9. Common Pitfalls (and How to Avoid Them)

### Mistake 1: Forgetting to Push 😅

```bash
# You add new data and commit
dvc add data/new_file.csv
git add data/new_file.csv.dvc
git commit -m "Add new data"
git push  # Push to GitHub

# Teammate clones and tries to pull
dvc pull
# Error: "ERROR: failed to pull data from the cloud -
#         data/new_file.csv is missing"

# Why? You never ran dvc push!
# The pointer file is in Git, but actual data isn't in remote

# Fix: Always push after commits
dvc push  # NOW the data is uploaded

```

**Prevention:** Create a Git hook:

```bash
# .git/hooks/pre-push
#!/bin/bash
dvc push
# This automatically runs dvc push before git push

```

### Mistake 2: Forgetting to Pull 🤦

```bash
# You switch branches
git checkout feature-branch
# Your .dvc files change, but data doesn't update

ls data/raw/
# Still shows old files!

# Fix: Run dvc checkout after branch switches
dvc checkout
# Now data matches the branch

```

**Prevention:** Create a Git hook:

```bash
# .git/hooks/post-checkout
#!/bin/bash
dvc checkout
# This automatically syncs data after checking out

```

### Mistake 3: Committing Data to Git 🐘

```bash
# Bad: Adding data directly to Git
git add data/raw/2023-01.parquet  # DON'T DO THIS!
git commit -m "Add data"

# Result: Git repo becomes huge
# .git/ folder: 5 MB → 500 MB

# How to check if you made this mistake
git ls-files | grep -E "\.(csv|parquet|pkl)$"
# If this shows files, they're in Git (bad!)

# Fix: Remove from Git, add to DVC
git rm --cached data/raw/*.parquet
echo "data/raw/*.parquet" >> .gitignore
dvc add data/raw/
git add data/raw.dvc .gitignore
git commit -m "Move data to DVC"

```

### Mistake 4: Tracking Same File in Git and DVC 🤝

```bash
# File is already in Git
git ls-files | grep data/model.pkl
# Output: data/model.pkl  ← Bad! It's in Git

# You run dvc add (thinking it will fix it)
dvc add data/model.pkl
# Now it's in BOTH Git and DVC (worst of both worlds!)

# Correct approach:
# 1. Untrack from Git first
git rm --cached data/model.pkl
echo "data/model.pkl" >> .gitignore

# 2. NOW let DVC take over
dvc add data/model.pkl

# 3. Commit the change
git add data/model.pkl.dvc .gitignore
git commit -m "Move model.pkl to DVC control"

```

---

## 🎉 That’s a Wrap!

And there you have it — a full walkthrough of DVC, from tracking your first dataset to rolling back versions and linking code with data. Hopefully, it makes data versioning feel way less mysterious!

If this helped you, **please hit that ⭐ star on the repo** — it really motivates me to create more tutorials like this.

## 📫 Connect With Me

📧 [smfarrukhm@gmail.com](mailto:smfarrukhm@gmail.com) •
💼 [LinkedIn](https://linkedin.com/in/sfarrukhm)
