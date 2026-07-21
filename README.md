# 🐘 Hadoop Big Data Analytics — Lab Kit (Linux + Windows)

A **self-contained, follow-along kit** for the Big Data Analytics Lab (BDAL / VTU scheme). It takes
you from **zero** to a working Hadoop cluster, then through the core lab experiments. Everything you
need is in this folder — programs, input data, expected outputs, and step-by-step guides for **both
Linux and Windows**.

> Made for a classmate who missed the lab. You can also **hand this whole folder to an LLM**
> (ChatGPT / Claude) and ask it to walk you through it — see "Using this with an LLM" below.

---

## What you'll build
A single-node ("pseudo-distributed") **Hadoop 3.4.1** cluster with **HDFS** (storage) and **YARN**
(compute), then run real MapReduce jobs on it. By the end you'll have done installation, basic HDFS
file operations, and a custom matrix-multiplication MapReduce program.

## The labs (do them in order)
| Lab | Folder | What you do |
|-----|--------|-------------|
| **1** | [`Lab-1-Hadoop-Setup/`](Lab-1-Hadoop-Setup/) | Install + configure Hadoop & HDFS. **Do this first.** |
| **2** | [`Lab-2-HDFS-Basic-Commands/`](Lab-2-HDFS-Basic-Commands/) | Verify the cluster; core HDFS commands (mkdir, put, get, cat, rm). |
| **3** | [`Lab-3-Matrix-Multiplication/`](Lab-3-Matrix-Multiplication/) | Write, compile & run a MapReduce matrix-multiplication program. |

Each lab has its **own `README.md`** with numbered steps for **Linux/WSL2** and **native Windows**.

---

## Which path is for you?
- **You're on Linux** → follow the "Linux" steps everywhere. Easiest.
- **You're on Windows** → **use WSL2** (Path B in Lab 1). It runs real Linux inside Windows and
  avoids the infamous `winutils.exe` problem. Strongly recommended.
- **Your college forces raw Windows** → Lab 1 Path C covers native Windows + winutils.

> **Why Windows is finicky:** the Apache Hadoop download is built for Linux and is missing two
> Windows-only helper files (`winutils.exe`, `hadoop.dll`). Their absence is the #1 cause of
> "the NameNode won't start" on Windows. Lab 1 explains the fix; WSL2 sidesteps it entirely.

## Prerequisites
- A 64-bit machine with ~5 GB free disk and internet.
- **Java JDK 8 or 11** (NOT 17+ — Hadoop 3.4.x doesn't support it). Lab 1 installs this.
- Basic comfort with a terminal (or willingness to copy-paste carefully).

---

## Quick start (Linux/WSL2, once Lab 1 is done)
```bash
hadoop-start                          # boot the cluster (5 daemons)
# ... do Lab 2 or Lab 3 ...
hadoop-stop                           # shut down when finished
```
Web dashboards: **HDFS** → http://localhost:9870 · **YARN jobs** → http://localhost:8088

## Core mental model (read this once)
```
        ┌──────────────── HADOOP ────────────────┐
   STORAGE (HDFS)                       COMPUTE (MapReduce on YARN)
   ├─ NameNode  = the index/brain       ├─ Map    = process each piece
   └─ DataNode  = the blocks/shelves    └─ Reduce = combine the results
```
- **HDFS** stores files split into blocks across the cluster. **NameNode** holds the index
  (which file = which blocks); **DataNode** holds the actual bytes.
- **MapReduce** processes that data: **Map** runs on each piece, **Reduce** combines. You only
  write Map and Reduce — Hadoop handles splitting, shuffling, and fault tolerance.

---

## Using this with an LLM
Paste something like this to ChatGPT/Claude, then attach or paste the relevant lab's `README.md`:

> *"I'm setting up Hadoop for a Big Data Analytics lab on **[Linux / Windows]**. Here is my lab kit's
> instructions. Walk me through it one step at a time, wait for me to confirm each step worked before
> the next, and help me debug any errors. Start with Lab 1."*

Because every command and expected output is in these files, the LLM has full context to guide and
troubleshoot you.

## Golden rules (don't skip)
1. ⚠️ **Never re-run `hdfs namenode -format`** on a working cluster — it **erases all HDFS data**.
   You format exactly once, during Lab 1 setup.
2. A **MapReduce output directory must not already exist** — delete it first (`hdfs dfs -rm -r <dir>`).
3. The `WARN util.NativeCodeLoader ...` message is **cosmetic** — ignore it.
4. "Safe mode is ON" for ~30s right after starting is **normal** — it clears itself.

---

## Credits & sources
- The **matrix-multiplication program and input data** (Lab 3 — `MatrixMultiplication.java`,
  `matrix.txt`) originate from the lab materials at
  **<https://github.com/hossain-tamim/big_data_analytics_lab>**. All credit for that program to the
  original author.
- The setup, HDFS-command, and Windows/Linux dual-OS guides were written up and verified end-to-end
  on a single-node Hadoop 3.4.1 cluster while working through the labs.

---

*Single-node Hadoop 3.4.1 + Java 11. Verified end-to-end on Linux; Windows paths provided for WSL2
and native winutils setups.*
