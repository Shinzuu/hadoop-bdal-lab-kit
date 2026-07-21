# Lab 2 — Basic HDFS Commands (Installation Verification)

**Goal:** verify the cluster works and practise the core HDFS commands — make dirs, copy files
in/out, read, and delete. (This is "Experiment 01 / Basic Hadoop HDFS Commands" in the VTU sheet.)

> **The key idea:** HDFS is a **separate filesystem** from your normal disk. Files in your home
> folder are NOT in HDFS. You reach HDFS only through `hadoop fs -...` (or `hdfs dfs -...`, same thing).

## Two constants across every OS
- The `hadoop fs -...` commands are **identical** on Linux, WSL2, and Windows.
- The web UI **http://localhost:9870 → Utilities → Browse the File System** is identical too.
- Only two things differ per-OS: **how you start the cluster**, and **local file paths**.

| | Linux / WSL2 | native Windows |
|---|---|---|
| start cluster | `hadoop-start` | `start-all.cmd` |
| a local path | `~/kit/Lab-2-HDFS-Basic-Commands/command3.txt` | `C:\BDA\command3.txt` |
| everything else | `hadoop fs -...` | `hadoop fs -...` (same) |

The sample files **`command3.txt`** and **`exampleput.txt`** are included in this folder.

---

## The steps (run top to bottom)

**0. Start the cluster** (if not already running)
```bash
hadoop-start          # Windows native: start-all.cmd
jps                   # expect 5 daemons
```

**1. List the HDFS root**
```bash
hadoop fs -ls /
```

**2. Make a directory in HDFS**
```bash
hadoop fs -mkdir /dir1
hadoop fs -ls /                      # /dir1 now appears
```

**3. Make a subdirectory**
```bash
hadoop fs -mkdir /dir1/newdir
hadoop fs -ls /dir1
```

**4. Copy a LOCAL file INTO HDFS** (`copyFromLocal`, aka `put`)
```bash
# Linux/WSL2 — cd into the folder that holds the file first (avoids space-in-path errors):
cd ~/path/to/Lab-2-HDFS-Basic-Commands
hadoop fs -copyFromLocal command3.txt /dir1
# Windows native equivalent:
#   hadoop fs -copyFromLocal C:/BDA/command3.txt /dir1
hadoop fs -ls /dir1                  # confirm command3.txt is there
```
> ⚠️ **Space-in-path gotcha (Linux):** Hadoop reads local paths as URIs and a **space** breaks them.
> If your folder has a space (e.g. `Lab 2`), `cd` into it and pass just the filename.

**5. Copy a file OUT of HDFS to local** (`copyToLocal`, aka `get`)
```bash
mkdir -p downloaded
hadoop fs -copyToLocal /dir1/command3.txt downloaded/    # Windows: ... C:/BB/
```

**6. Upload with `put`, download with `get`** (same as copyFromLocal/copyToLocal)
```bash
hadoop fs -put exampleput.txt /dir1/newdir
hadoop fs -get /dir1/newdir/exampleput.txt downloaded/
```

**7. Read a file's contents**
```bash
hadoop fs -cat /dir1/command3.txt
```

**8. Delete a single file**
```bash
hadoop fs -rm /dir1/command3.txt
```

**9. Delete an EMPTY directory** (`-rmdir` only works when empty)
```bash
hadoop fs -rm /dir1/newdir/exampleput.txt
hadoop fs -rmdir /dir1/newdir
```

**10. Delete a NON-EMPTY directory** (`-r` = recursive; no need to empty first)
```bash
hadoop fs -rm -r /dir1
```

**11. Verify final state + browse in the web UI**
```bash
hadoop fs -ls /                      # back to just /tmp and /user
```
Then open **http://localhost:9870 → Utilities → Browse the File System**.

---

## Command reference (memorize these)
| Task | Command |
|------|---------|
| list | `hadoop fs -ls <path>` |
| make dir | `hadoop fs -mkdir <path>` |
| local → HDFS | `hadoop fs -copyFromLocal <local> <hdfs>` (or `-put`) |
| HDFS → local | `hadoop fs -copyToLocal <hdfs> <local>` (or `-get`) |
| show contents | `hadoop fs -cat <path>` |
| delete file | `hadoop fs -rm <path>` |
| delete empty dir | `hadoop fs -rmdir <path>` |
| delete dir + contents | `hadoop fs -rm -r <path>` |

**Note:** the `WARN util.NativeCodeLoader ...` line on every command is harmless — ignore it.
A path with **no leading `/`** (e.g. `hadoop fs -ls`) is relative to your HDFS home `/user/<name>`;
a leading `/` is from the HDFS root.
