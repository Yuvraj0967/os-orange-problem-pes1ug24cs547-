# PES-VCS — A Version Control System from Scratch

> A Git-inspired, local version control system built in C that tracks file changes, stores snapshots efficiently, and supports commit history. Every component maps directly to operating system and filesystem concepts.

---

## Table of Contents

- [Overview](#overview)
- [Prerequisites](#prerequisites)
- [Installation](#installation)
- [Author Configuration](#author-configuration)
- [Commands](#commands)
- [Project Structure](#project-structure)
- [How It Works](#how-it-works)
- [Phases](#phases)
- [Analysis Questions](#analysis-questions)
- [Submission Requirements](#submission-requirements)

---

## Overview

PES-VCS is a simplified implementation of Git's core internals. It uses:

- **Content-addressable storage** — every file is identified by the SHA-256 hash of its contents
- **Tree objects** — directory snapshots stored as linked structures
- **Commit objects** — timestamped snapshots with parent pointers forming a history chain
- **A staging index** — a text-based file tracking what's ready to commit

---

## Prerequisites

```bash
sudo apt update && sudo apt install -y gcc build-essential libssl-dev
```

**Platform:** Ubuntu 22.04

---

## Installation

```bash
# Clone the repository
git clone https://github.com/<your-username>/<SRN>-pes-vcs.git
cd <SRN>-pes-vcs

# Build the pes binary
make

# Build pes + test binaries
make all

# Remove build artifacts
make clean
```

---

## Author Configuration

PES-VCS reads the author name from the `PES_AUTHOR` environment variable:

```bash
export PES_AUTHOR="Your Name <PESXUG24CS042>"
```

If unset, it defaults to `"PES User <pes@localhost>"`.

---

## Commands

### `pes init`
Initializes a new PES-VCS repository in the current directory.

```bash
./pes init
```

Creates the `.pes/` directory with the following structure:
```
.pes/
├── objects/        # Content-addressable storage
├── refs/
│   └── heads/
│       └── main    # Branch pointer
├── index           # Staging area
└── HEAD            # Current branch reference
```

---

### `pes add <file>...`
Stages one or more files for the next commit.

```bash
./pes add file.txt
./pes add src/main.c README.md
```

- Computes the SHA-256 blob hash of each file
- Stores the blob in `.pes/objects/`
- Updates the `.pes/index` staging area

---

### `pes status`
Shows the current state of the working directory and staging area.

```bash
./pes status
```

**Example output:**
```
Staged changes:
  staged:     hello.txt
  staged:     src/main.c

Unstaged changes:
  modified:   README.md
  deleted:    old_file.txt

Untracked files:
  untracked:  notes.txt
```

---

### `pes commit -m "<message>"`
Creates a commit from all staged files.

```bash
./pes commit -m "Initial commit"
./pes commit -m "Add new feature"
```

- Builds a tree object from the current index
- Creates a commit object pointing to the tree and the parent commit
- Updates `.pes/refs/heads/main` to point to the new commit

---

### `pes log`
Walks and displays the full commit history from HEAD.

```bash
./pes log
```

**Example output:**
```
commit a1b2c3d4e5f6...
Author: Alice <PESXUG24CS042>
Date:   1699900000

    Add farewell message

commit 9f8e7d6c5b4a...
Author: Alice <PESXUG24CS042>
Date:   1699890000

    Initial commit
```

---

## Project Structure

```
.
├── pes.h           # Core data structures and constants (do not modify)
├── object.c        # Content-addressable object store
├── tree.h          # Tree object interface (do not modify)
├── tree.c          # Tree serialization and construction
├── index.h         # Staging area interface (do not modify)
├── index.c         # Staging area (text-based index file)
├── commit.h        # Commit object interface (do not modify)
├── commit.c        # Commit creation and history
├── pes.c           # CLI entry point and command dispatch (do not modify)
├── test_objects.c  # Phase 1 test program
├── test_tree.c     # Phase 2 test program
├── test_sequence.sh# End-to-end integration test
└── Makefile        # Build system (do not modify)
```

---

## How It Works

### The Three Object Types

**Blob** — raw file contents (no filename, no permissions):
```
blob 16\0Hello, World!\n
```

**Tree** — a directory snapshot listing blobs and sub-trees:
```
100644 blob a1b2c3d4... README.md
100755 blob e5f6a7b8... build.sh
040000 tree 9c0d1e2f... src
```

**Commit** — ties a tree snapshot to metadata and parent history:
```
tree 9c0d1e2f3a4b...
parent a1b2c3d4e5f6...
author Alice <alice@example.com> 1699900000

Add new feature
```

### Content-Addressable Storage

Objects are stored by the SHA-256 hash of their content, sharded by the first two hex characters:

```
.pes/objects/
├── 2f/
│   └── 8a3b5c7d9e...
├── a1/
│   └── 9c4e6f8a0b...
```

This gives deduplication (identical files stored once), integrity verification, and immutability.

### Reference Chain

```
HEAD  →  refs/heads/main  →  <commit hash>
```

Each commit points to its parent, forming a linked list of history.

---

## Phases

### Phase 1 — Object Storage
Implements `object_write` and `object_read` in `object.c`.

```bash
make test_objects
./test_objects
```

### Phase 2 — Tree Objects
Implements `tree_from_index` in `tree.c`.

```bash
make test_tree
./test_tree
```

### Phase 3 — Staging Area (Index)
Implements `index_load`, `index_save`, and `index_add` in `index.c`.

```bash
./pes init
echo "hello" > file1.txt
./pes add file1.txt
./pes status
cat .pes/index
```

### Phase 4 — Commits and History
Implements `commit_create` in `commit.c`.

```bash
./pes init
echo "Hello" > hello.txt
./pes add hello.txt
./pes commit -m "Initial commit"
./pes log

# Full integration test
make test-integration
```

---

## Analysis Questions

### Branching and Checkout

**Q5.1** — How would `pes checkout <branch>` work? What files change in `.pes/`? What makes it complex?

**Q5.2** — How would you detect a "dirty working directory" conflict when switching branches, using only the index and object store?

**Q5.3** — What is "Detached HEAD"? What happens to commits made in that state, and how can a user recover them?

### Garbage Collection

**Q6.1** — Describe an algorithm to find and delete unreachable objects. What data structure tracks reachable hashes? Estimate objects visited for 100,000 commits across 50 branches.

**Q6.2** — Why is running GC concurrently with a commit dangerous? Describe a race condition and how Git's real GC avoids it.

---

## Submission Requirements

### Screenshots

| Phase | ID  | What to Capture |
|-------|-----|----------------|
| 1     | 1A  | `./test_objects` — all tests passing |
| 1     | 1B  | `find .pes/objects -type f` — sharded structure |
| 2     | 2A  | `./test_tree` — all tests passing |
| 2     | 2B  | `xxd .pes/objects/XX/YYY... \| head -20` — raw binary tree |
| 3     | 3A  | `pes init → pes add → pes status` sequence |
| 3     | 3B  | `cat .pes/index` — text-format index |
| 4     | 4A  | `pes log` — three commits with hashes and messages |
| 4     | 4B  | `find .pes -type f \| sort` — object store growth |
| 4     | 4C  | `cat .pes/refs/heads/main` and `cat .pes/HEAD` |
| Final | --  | `make test-integration` — full integration test |

### Commit History

- Minimum **5 commits per phase** with clear, descriptive messages
- More granular commits are preferred — they demonstrate step-by-step understanding

### Repository

- Repository must be **public** on GitHub
- Named as: `<SRN>-pes-vcs` (e.g., `PESXUG24CS042-pes-vcs`)
- Lab report (`README.md` or `report.pdf`) must be at the **root** of the repository

---

## License

This project is a PES University lab assignment. All skeleton code and specifications are provided by the course instructors.
