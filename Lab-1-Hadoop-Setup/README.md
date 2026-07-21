# Lab 1 — Install & Configure Hadoop + HDFS (single node)

**Goal:** get a working single-node ("pseudo-distributed") Hadoop 3.4.1 cluster with HDFS + YARN,
so Labs 2 and 3 have something to run on. When you're done, `jps` shows **5 daemons** and the web
UI at `http://localhost:9870` loads.

This guide covers **three paths** — pick ONE:
- **Path A — Linux** (native: Ubuntu/Debian/Arch/etc.)
- **Path B — Windows via WSL2** ✅ *recommended for Windows users* — runs real Linux, no winutils pain
- **Path C — native Windows** (winutils.exe) — only if your college forces raw Windows

> **Versions used (all stable):** Hadoop **3.4.1** (GA), Java **11** (LTS).
> ⚠️ Hadoop 3.4.x supports **only Java 8 or 11** — Java 17+ breaks it. Install JDK 11.

---

## Path A — Linux (native)

### 1. Install Java 11 + SSH
```bash
# Debian/Ubuntu:
sudo apt update && sudo apt install -y openjdk-11-jdk ssh
# Arch/EndeavourOS:
sudo pacman -S --needed jdk11-openjdk openssh
```
Find your `JAVA_HOME` (the folder containing `bin/java`):
```bash
# Ubuntu:  /usr/lib/jvm/java-11-openjdk-amd64
# Arch:    /usr/lib/jvm/java-11-openjdk
dirname "$(dirname "$(readlink -f "$(which java)")")"
```

### 2. Download + extract Hadoop 3.4.1
```bash
cd ~ && mkdir -p bigdata && cd bigdata
wget https://dlcdn.apache.org/hadoop/common/hadoop-3.4.1/hadoop-3.4.1.tar.gz
tar -xzf hadoop-3.4.1.tar.gz
```

### 3. Passwordless SSH to localhost (Hadoop starts daemons over ssh)
```bash
ssh-keygen -t ed25519 -N "" -f ~/.ssh/id_hadoop
cat ~/.ssh/id_hadoop.pub >> ~/.ssh/authorized_keys
chmod 600 ~/.ssh/authorized_keys
# test — must print without asking for a password:
ssh -i ~/.ssh/id_hadoop -o StrictHostKeyChecking=no localhost echo SSH_OK
```

### 4. Environment variables — create `~/bigdata/hadoop-env.sh`
```bash
cat > ~/bigdata/hadoop-env.sh <<'EOF'
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk        # <-- adjust to YOUR path (step 1)
export HADOOP_HOME=$HOME/bigdata/hadoop-3.4.1
export HADOOP_CONF_DIR=$HADOOP_HOME/etc/hadoop
export PATH=$PATH:$HADOOP_HOME/bin:$HADOOP_HOME/sbin
# convenience aliases:
alias hadoop-start='start-dfs.sh && start-yarn.sh && jps'
alias hadoop-stop='stop-yarn.sh && stop-dfs.sh'
alias hadoop-status='jps'
EOF
# load it automatically in every new shell:
echo '[ -f "$HOME/bigdata/hadoop-env.sh" ] && source "$HOME/bigdata/hadoop-env.sh"' >> ~/.bashrc
#   (zsh users: append the same line to ~/.zshrc instead)
source ~/bigdata/hadoop-env.sh
```
Also point Hadoop's own env file at Java — edit `$HADOOP_HOME/etc/hadoop/hadoop-env.sh` and set:
```bash
export JAVA_HOME=/usr/lib/jvm/java-11-openjdk        # same path as above
export HADOOP_SSH_OPTS="-i $HOME/.ssh/id_hadoop -o StrictHostKeyChecking=no -o BatchMode=yes"
```

### 5. Data dirs + the 4 config files
```bash
mkdir -p ~/bigdata/hdfs/{namenode,datanode,tmp}
```
Copy the four XML files from this kit's **`config/`** folder into
`~/bigdata/hadoop-3.4.1/etc/hadoop/` (overwrite the stubs). They set:
`fs.defaultFS=hdfs://localhost:9000`, `dfs.replication=1`, the namenode/datanode dirs,
MapReduce-on-YARN, and the shuffle service.

### 6. Format HDFS (ONE TIME ONLY) and start
```bash
hdfs namenode -format      # ⚠️ ONLY the first time — this WIPES HDFS if re-run
hadoop-start               # starts everything, prints jps
```
✅ You should see: **NameNode, DataNode, SecondaryNameNode, ResourceManager, NodeManager**.

---

## Path B — Windows via WSL2 ✅ (recommended for Windows)

WSL2 runs a real Ubuntu inside Windows, so you avoid winutils entirely and follow the Linux steps.

1. Open **PowerShell as Administrator**:
   ```powershell
   wsl --install -d Ubuntu
   ```
   Reboot when asked; on first launch of the **Ubuntu** app, set a username + password.
2. Inside Ubuntu, **follow Path A above, steps 1–6, exactly.** Everything is identical.
   - On WSL2/Ubuntu, `JAVA_HOME` is usually `/usr/lib/jvm/java-11-openjdk-amd64`.
   - You edit `~/.bashrc` (WSL Ubuntu uses bash by default).
3. Access the web UI from Windows' browser at the same `http://localhost:9870`.

---

## Path C — native Windows (winutils.exe)  ⚠️ the tricky one

The Apache download is **built for Linux** and is missing two Windows-native helpers Hadoop needs:
**`winutils.exe`** and **`hadoop.dll`**. Without them the NameNode crashes with
`UnsatisfiedLinkError ... NativeIO$Windows.access0` or
`Could not locate ... winutils.exe`. **This is the #1 reason "the NameNode won't run" on Windows.**

1. **Install JDK 8 or 11** to a path **without spaces** (e.g. `C:\Java\jdk-11`). Spaces in
   `C:\Program Files\...` break Hadoop scripts.
2. **Use Hadoop 3.3.6 on Windows** (not 3.4.1) — reliable prebuilt winutils exist for 3.3.6.
   Extract to `C:\hadoop`.
3. **Get winutils — THE MISSING STEP.** Download the folder matching your Hadoop version from
   <https://github.com/cdarlint/winutils> (it has `hadoop-3.3.6/bin`). Copy **all** files from that
   `bin` folder into `C:\hadoop\bin`. Also copy `hadoop.dll` into `C:\Windows\System32`.
   **Rule: winutils version must match Hadoop major.minor** (3.3.x winutils ↔ 3.3.x Hadoop).
4. **Environment variables** (System → Environment Variables): `JAVA_HOME=C:\Java\jdk-11`,
   `HADOOP_HOME=C:\hadoop`, and add `%HADOOP_HOME%\bin` + `%HADOOP_HOME%\sbin` to `Path`.
5. **Config XMLs** in `C:\hadoop\etc\hadoop\` — use this kit's `config/` files but change the dir
   values to Windows paths, e.g. `<value>file:///C:/hadoop/data/namenode</value>`. In
   `hadoop-env.cmd` set `set JAVA_HOME=C:\Java\jdk-11`.
6. **Format + start** in Command Prompt (Run as Administrator):
   ```cmd
   hdfs namenode -format
   start-all.cmd
   jps
   ```
   Each daemon opens its own window; `jps` should list NameNode, DataNode, ResourceManager,
   NodeManager. If DataNode is missing after a re-format, it's a cluster-ID mismatch — delete the
   contents of the datanode dir and restart.

### Windows checklist if NameNode still won't start
- [ ] `winutils.exe` AND `hadoop.dll` both in `%HADOOP_HOME%\bin`?
- [ ] winutils version == Hadoop version?
- [ ] `JAVA_HOME` set, no spaces, Java 8/11 (not 17+)?
- [ ] Ran `hdfs namenode -format` once before first start?

---

## Daily use (all paths)
```bash
hadoop-start     # start HDFS + YARN         (Windows native: start-all.cmd)
hadoop-status    # jps — expect 5 daemons
hadoop-stop      # stop everything           (Windows native: stop-all.cmd)
```
- HDFS / NameNode UI: **http://localhost:9870**
- YARN ResourceManager UI: **http://localhost:8088**

## Golden rules
- ⚠️ **Never re-run `hdfs namenode -format`** on a working cluster — it erases all HDFS data.
- The `WARN util.NativeCodeLoader` message is **cosmetic** — ignore it.
- Right after `hadoop-start`, "Safe mode is ON" for ~30s is **normal** (NameNode waiting for the
  DataNode to report blocks); it clears automatically.
