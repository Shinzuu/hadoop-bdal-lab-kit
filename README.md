# 🐘 Hadoop Big Data Analytics — Lab Kit (Linux + Windows)

A **self-contained, follow-along kit** that takes you from a blank machine to a working Hadoop
cluster, then through the core Big Data Analytics lab experiments — with **every command, config
file, program, input, and expected output included**, and step-by-step instructions for **both Linux
and Windows**.

> **Who this is for:** a classmate who missed the lab, anyone setting Hadoop up for the first time, or
> future-you who forgot how it worked. You can even hand this whole repo to an LLM (ChatGPT / Claude)
> and have it walk you through every step — see *"Using this with an LLM."*

---

## The story — why this kit exists

The official college lab manual is written for **Windows**, and it has a gap that trips up almost
everyone: the step *"download a bin file and paste it there"* — with no explanation of **what** that
file is or **where** to get it. That missing piece is `winutils.exe` (and `hadoop.dll`), and without
it the **NameNode simply refuses to start**. Students follow the manual line by line, hit a wall, and
have no idea why — while the lecturer's machine "just works" because he quietly dropped those files in
years ago.

Here's the root cause: **Apache Hadoop's download is built for Linux.** It ships without the small
Windows-native helper binaries Hadoop needs to touch the Windows filesystem. On Linux, none of this is
a problem — everything works out of the box. So this kit does two things:

1. **Sets Hadoop up the clean way on Linux** (or on Windows via WSL2, which runs real Linux inside
   Windows and sidesteps `winutils` entirely), and
2. **Still explains the native-Windows `winutils` fix** for anyone whose college requires raw Windows.

Everything here was **actually run and verified end-to-end** on a single-node Hadoop 3.4.1 cluster —
the commands, the outputs, the gotchas. Nothing is copied blindly from a manual.

---

## What you'll build

A single-node ("pseudo-distributed") **Hadoop 3.4.1** cluster running two layers:

```
        ┌──────────────── HADOOP ────────────────┐
   STORAGE (HDFS)                       COMPUTE (MapReduce on YARN)
   ├─ NameNode  = the index / brain     ├─ Map    = process each piece
   └─ DataNode  = the blocks / shelves   └─ Reduce = combine the results
```

- **HDFS** stores files by splitting them into blocks spread across the cluster. The **NameNode**
  holds the index (which file = which blocks, on which machines); the **DataNode** holds the actual
  bytes. Lose the NameNode and the blocks become meaningless numbers — which is exactly why it's the
  most critical daemon, and why its failure on Windows is so fatal.
- **MapReduce** processes that stored data. **Map** runs a function on each block *where it already
  lives*; **Reduce** combines the results. You write only Map and Reduce — Hadoop handles the hard
  distributed-systems parts (splitting, shuffling, sorting, fault tolerance) for you. The same two
  functions scale unchanged from a 5-line file to billions of rows across a thousand machines.

By the end you'll have done a full install, basic HDFS file operations, and a real custom MapReduce
program (matrix multiplication).

---

## The labs — do them in order

| Lab | Folder | The story it tells |
|-----|--------|--------------------|
| **1** | [`Lab-1-Hadoop-Setup/`](Lab-1-Hadoop-Setup/) | **Get Hadoop running.** Install Java 11 + Hadoop 3.4.1, set up passwordless SSH, write the 4 config files, format HDFS, and start all 5 daemons. Covers Linux, WSL2, and native Windows (with the `winutils` fix). **Do this first — the other labs need it.** |
| **2** | [`Lab-2-HDFS-Basic-Commands/`](Lab-2-HDFS-Basic-Commands/) | **Prove it works & learn to move data.** Verify the cluster, then practise the core HDFS commands: make directories, copy files in and out (`put`/`get`), read them, and delete them. The big idea: HDFS is a *separate filesystem* you reach only through `hadoop fs -...`. |
| **3** | [`Lab-3-Matrix-Multiplication/`](Lab-3-Matrix-Multiplication/) | **Write real MapReduce.** Compile and run a custom Java program that multiplies two matrices — the classic example of turning a math problem into Map + Reduce. Includes the program, input, and the verified answer to check against. |

Each lab folder has its **own `README.md`** with numbered steps for **Linux/WSL2** and **native
Windows**, side by side.

---

## Which path is for you?

- **You're on Linux** → follow the "Linux" steps throughout. Easiest, cleanest.
- **You're on Windows** → **use WSL2** (Lab 1, Path B). It runs a real Ubuntu inside Windows, so you
  follow the identical Linux steps and **never touch `winutils`.** Strongly recommended.
- **Your college requires raw Windows** → Lab 1, Path C walks through native Windows + the `winutils`
  fix, including the version-match rule that catches everyone.

## Prerequisites

- A 64-bit machine, ~5 GB free disk, and an internet connection.
- **Java JDK 8 or 11** — *not* 17+, which Hadoop 3.4.x does not support. Lab 1 installs this for you.
- Basic comfort with a terminal (or the willingness to copy-paste carefully and read the notes).

## Quick start (after Lab 1 is done)

```bash
hadoop-start                 # boot the cluster — expect 5 daemons in jps
# ... work through Lab 2 or Lab 3 ...
hadoop-stop                  # shut down when finished
```

Web dashboards: **HDFS / NameNode** → http://localhost:9870 · **YARN jobs** → http://localhost:8088

---

## Using this with an LLM

Because every command and expected output lives in these files, an LLM has full context to guide and
debug you. Paste something like:

> *"I'm setting up Hadoop for a Big Data Analytics lab on **[Linux / Windows]**. Here are my lab kit's
> instructions [paste or attach the relevant `README.md`]. Walk me through it one step at a time, wait
> for me to confirm each step worked before moving on, and help me debug any errors. Start with Lab 1."*

---

## Golden rules (learned the hard way)

1. ⚠️ **Never re-run `hdfs namenode -format`** on a working cluster — it **erases all HDFS data**. You
   format exactly once, during Lab 1 setup.
2. A **MapReduce output directory must not already exist** — delete it first with
   `hdfs dfs -rm -r <dir>`, or the job refuses to run.
3. The `WARN util.NativeCodeLoader ...` message on every command is **cosmetic** — ignore it. It just
   means Hadoop's optional C speed-ups aren't prebuilt for your platform, so it uses Java instead.
4. **"Safe mode is ON"** for ~30 seconds right after starting is **normal** — the NameNode is waiting
   for the DataNode to report its blocks. It clears itself.
5. On Linux, **spaces in local file paths break `hadoop fs` commands** (Hadoop parses them as URIs).
   `cd` into the folder and pass just the filename.

---

## Credits & sources

- The **matrix-multiplication program and input** (Lab 3 — `MatrixMultiplication.java`, `matrix.txt`)
  come from **<https://github.com/hossain-tamim/big_data_analytics_lab>**. Full credit for that program
  to the original author.
- The setup, HDFS-command, and dual-OS (Linux + Windows) guides were written up and verified
  end-to-end while working through the labs on a single-node Hadoop 3.4.1 cluster.

---

*Single-node Hadoop 3.4.1 + Java 11. Verified end-to-end on Linux; Windows paths provided for both
WSL2 and native `winutils` setups. Shared so the next person doesn't have to hit the same wall.*
