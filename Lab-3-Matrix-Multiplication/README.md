# Lab 3 — Matrix Multiplication with MapReduce

**Goal:** write, compile, and run a **custom Java MapReduce** program that multiplies two matrices
`C = A × B`, on your Hadoop cluster. Unlike word count, Hadoop has **no built-in jar** for this — you
compile the program yourself. (This is "Lab Experiment No. 2 / Matrix Multiplication" in the VTU sheet.)

Files in this folder:
- **`MatrixMultiplication.java`** — the program (Mapper + Reducer + driver)
- **`matrix.txt`** — the input matrices
- **`expected-output.txt`** — the correct answer to check against

> **Source:** `MatrixMultiplication.java` and `matrix.txt` come from
> <https://github.com/hossain-tamim/big_data_analytics_lab> — credit to the original author.
> This folder adds the dual-OS (Linux + Windows) run instructions and verified expected output.

---

## The math
The input `matrix.txt` encodes:
```
A (2×2)          B (2×2)          C = A × B (2×2)
[1 2]            [5 6]            [19 22]
[3 4]            [7 8]            [43 50]
```
Each result cell is a dot product: `C[i][k] = Σⱼ A[i][j]·B[j][k]`
e.g. `C[0][0] = 1·5 + 2·7 = 19`. **Compute all four by hand, then confirm the job matches.**

## How it works in MapReduce
> **The reducer key is the output cell `(i,k)`.** The **Map** phase *scatters* each input element to
> every output cell it contributes to; the **Reduce** phase *gathers* the A and B values for one cell,
> pairs them by the shared index `j`, multiplies, and sums.

## Input format (`matrix.txt`) — one element per line
`MatrixName,row,col,value`
```
A,0,0,1   A,0,1,2   A,1,0,3   A,1,1,4
B,0,0,5   B,0,1,6   B,1,0,7   B,1,1,8
```
> ⚠️ The program hardcodes `commonDim = 2` (in `main`). That value must equal the shared dimension
> of your matrices. This sample is 2×2 × 2×2, so `commonDim = 2` is correct. If you change the
> matrix sizes, edit that line and recompile.

---

## Run it — Linux / WSL2

```bash
# 0. make sure the cluster is up
hadoop-start

# 1. put the source + input in one folder (call it 'mat'), then cd in
mkdir -p ~/mat && cp MatrixMultiplication.java matrix.txt ~/mat/ && cd ~/mat

# 2. put the input into HDFS
hdfs dfs -mkdir -p /bda2
hdfs dfs -copyFromLocal -f matrix.txt /bda2
hdfs dfs -ls /bda2

# 3. compile against Hadoop's classpath, then package a jar
javac -classpath "$(hadoop classpath)" -d . MatrixMultiplication.java
jar -cf MatrixMultiplication.jar *.class

# 4. delete any old output (MapReduce refuses to overwrite), then run
hdfs dfs -rm -r -skipTrash /matrixoutput 2>/dev/null
hadoop jar MatrixMultiplication.jar MatrixMultiplication /bda2/matrix.txt /matrixoutput

# 5. read the result
hdfs dfs -cat /matrixoutput/part-r-00000
```

## Run it — native Windows

```cmd
:: 0. start the cluster
start-all.cmd

:: 1. put MatrixMultiplication.java + matrix.txt in a folder, e.g. C:\Users\You\Desktop\mat
cd C:\Users\You\Desktop\mat

:: 2. put the input into HDFS
hdfs dfs -mkdir /bda2
hdfs dfs -copyFromLocal C:/Users/You/Desktop/mat/matrix.txt /bda2

:: 3. compile against the Hadoop classpath (semicolons separate paths on Windows)
javac -classpath "%HADOOP_HOME%\share\hadoop\common\*;%HADOOP_HOME%\share\hadoop\common\lib\*;%HADOOP_HOME%\share\hadoop\hdfs\*;%HADOOP_HOME%\share\hadoop\mapreduce\*" -d . MatrixMultiplication.java
jar -cvf MatrixMultiplication.jar -C . .

:: 4. run (delete old output first if it exists: hdfs dfs -rm -r /matrixoutput)
hadoop jar MatrixMultiplication.jar MatrixMultiplication /bda2/matrix.txt /matrixoutput

:: 5. read the result
hdfs dfs -cat /matrixoutput/part-r-00000
```

---

## Expected output (verified)
```
0,0	19          C = A × B =  [ 19  22 ]
0,1	22                       [ 43  50 ]
1,0	43
1,1	50
```
Reads as `row,col <TAB> value`. Matches `expected-output.txt`. ✅

## Gotchas
- **Output dir must NOT exist** before running — delete it first with `hdfs dfs -rm -r <output>`.
- Watch the job live at **http://localhost:8088** (YARN UI) → it should reach
  `map 100% reduce 100%` → `completed successfully`.
- The `WARN util.NativeCodeLoader` line is harmless.
- To multiply **different** matrices: edit `matrix.txt`, set `commonDim` in the `.java` to the shared
  dimension, recompile, re-run.
