## 目次

- [HPC\_Note2](#hpc_note2)
  - [SSH](#ssh)
  - [OpenMPのメモ](#openmpのメモ)
  - [Message Passing Interface](#message-passing-interface)
    - [MPIの主な関数](#mpiの主な関数)
    - [SPMD(Single Program Multipule Data)](#spmdsingle-program-multipule-data)
  - [Pre-PACS version10](#pre-pacs-version10)
    - [MPIプログラムの実行手順](#mpiプログラムの実行手順)
    - [手順例](#手順例)
    - [ppxの構成](#ppxの構成)


# HPC_Note2

僕が並列処理プログラミングで困ったことを残しておくためのメモ


## SSH

- sshをシェルスクリプトで実行する方法

```bash
#remote Pre-PACS versionX
shh ppxsvc_via_ppxge
```

- ppxに多段sshする方法

  - ~/.ssh/config (sshコマンドがデフォルトで読み込む設定ファイル) に以下を追記する。
  その後上記のシェルスクリプトを実行

```
Host ppxsvc_via_ppxgw
	HostName ppxsvc
	User <ユーザー名>
	IdentityFile <ppxのssh接続に使う秘密鍵のパス>
	ProxyCommand ssh -W %h:%p ppxgw

Host ppxgw
	HostName ppxsvc.ccs.tsukuba.ac.jp
	User <ユーザー名>
	IdentityFile <ppxのssh接続に使う秘密鍵のパス>
	ForwardAgent yes
```

- hostnameの確認

```bash
scutil --get HostName
scutil --get LocalHostName
scutil --get ComputerName
```

- ファイルに実行権限を付与

```
chmod +x filename
```

## OpenMP

- 



## Message Passing Interface

- macbookの論理コアと物理コア数の確認、どちらもM1pro chipでは8
```
sysctl -n hw.physicalcpu
sysctl -n hw.logicalcpu
```

- ppxでの確認方法

```
lscpu
```

### MPIの主な関数

- 初期設定に関するもの
  - MPIの初期化を行い、引数のうちMPI設定に関係するものを処理する
    ```c
    int MPI_Init(int *argc, char **argv[]);
    ```
  - communicator中のプロセスを求める。
    ```c
    int MPI_Comm_size(MPI_Comm comm, int *size);
    ```
  - communicator中の自分のrankを調べる
    ```c
    int MPI_Comm_rank(MPI_Comm comm, int *rank);
    ```
  - プロセスが実行されているホスト名を得る。
    ```c
    int MPI_Get_processor_name(char name[], int *length);
    ```
  - そのプロセスのMPIとしての実行を終了する。
    ```c
    int MPI_Finalize();
    ```

- point-to-point通信に関するもの
  - データを送信する
    ```c
    int MPI_Send(void* buf, int count, MPI_Datatype datatype, int dest, int tag, MPI_Comm comm);
    ```
  - データを受信する
    ```c
    int MPI_Recv(void* buf, int count, MPI_Datatype datatype, int source, int tag, MPI_Comm comm. MPI_Status *status);
    ```

- collective通信に関するもの
  - rootの持つデータを全プロセスに放送する。
    ```c
    int MPI_Bcast(void* buffer, int count, MPI_Datatype datatype, int root,     MPI_Comm comm);
    ```
  - 全プロセスに分散しているデータをまとめ、rootプロセスに一つの配列として渡す。
    ```c
    int MPI_Gather(void* sendbuf, int sendcount, MPI_Datatype sendtype, void* recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm);
    ```
  - rootプロセス中の配列データを全プロセスに分配する。
    ```c
    int MPI_Scatter(void* sendbuf, int sendcount, MPI_Datatype sendtype, void* recvbuf, int recvcount, MPI_Datatype recvtype, int root, MPI_Comm comm);
    ```
  - 全プロセスが持つ部分データを集め、その全体を全プロセスに渡す。
    ```c
    int MPI_Allgather(void* sendbuf, int sendcount, MPI_Datatype sendtype, void* recvbuf, int recvcount, MPI_Datatype recvtype, MPI_Comm comm);
    ```
  - 全プロセスが持つ部分データを，それぞれ一部ずつが他のプロセスに渡るように分配する（全員に
同じデータが行かない）．
    ```c
    int MPI_Alltoall(void* sendbuf, int sendcount, MPI_Datatype sendtype,
    void* recvbuf, int recvcount, MPI_Datatype recvtype,
    MPI_Comm comm);
    ``` 
  - リダクション(reduction) 通信を実行し，結果をroot に返す．
    ```c
    int MPI_Reduce(void* sendbuf, void* recvbuf, int count, MPI_Datatype datatype,
    MPI_Op op, int root, MPI_Comm comm);
    ```
  - リダクション通信に似ているが，各プロセスに返されるのは，自分のrank より低いrank のプロセ
スだけからなる集合でのリダクション結果である．
    ```c
    int MPI_Scan(void* sendbuf, void* recvbuf, int count, MPI_Datatype datatype,
    MPI_Op op, MPI_Comm comm )
    ```

- その他
  - 全てのプロセスがこの命令を実行し終わるのを待つ(同期)
    ```c
    int MPI_Barrier(MPI_Comm comm );
    ```
  - wall clockの値を求める。
    ```c
    double MPI_Wtime();
    ```

- データの型
MPI_CHAR, MPI_SHORT, MPI_INT, MPI_LONG, MPI_UNSIGNED_CHAR, MPI_UNSIGNED_SHORT,
MPI_UNSIGNED, MPI_UNSIGNED_LONG, MPI_FLOAT, MPI_DOUBLE, MPI_LONG_DOUBLE
- リダクション演算
MPI_MAX, MPI_MIN, MPI_SUM, MPI_PROD, MPI_LAND, MPI_BAND, MPI_LOR, MPI_BOR, MPI_LXOR,
MPI_BXOR, MPI_MAXLOC, MPI_MINLOC

### SPMD(Single Program Multipule Data)

| 特徴   | SIMD | SPMD  |
|--------|------|--------|
|最適化   | 同じ命令を複数のデータに同時に適用 | 同じプログラムに異なるデータで並列実行 |
| データ   | 同じデータに対して同じ演算を一度に行う | 異なるデータを各スレッドまたはプロセスで処理|
| 並列性の粒度 | 1つのスレッド内で複数のデータを同時に処理 | 複数のスレッドまたはプロセスで異なるデータを並列処理 |
| 適用される計算 | 同一の演算を多数のデータに適用 | 異なるデータに対して同じプログラムを実行 |
| 適用   | 行列計算、ベクトル演算、画像処理など   | 分散計算、数値シュミレーション、データ処理 |
| ハードウェア依存性 | 主にSSE,AVX,GPUなどの命令セットに依存 | マルチコアCPUや分散システムで利用 |




## Pre-PACS version10

### MPIプログラムの実行手順

1.MPI環境の読み込み

- 環境切り替えソフトウェアModuleがPPXにインストールされている．
- module load/unload <環境名>で所望の環境に合わせて環境変数などを設定できる．コマンドmodule availにより利用可能な環境を確認できる．コマンドmodule listで現在読み込んでいる環境を確認できる．
- 必ず，他のコンパイラ環境を設定した**後に**，module load openmpi/<version>をする必要がある

2.MPIプログラムの記述

3.コンパイル&リンク

- コマンドmpiccを用いる．module loadをしていないと利用できない．
- [MPIライブラリ毎のコンパイラ指定方法](http://hpcmemo.hatenablog.com/entry/2014/01/24/144121)

4.MPIプログラムの実行

- コマンドsallocを使用して実行ノードを確保した後に，コマンドmpirunで実行する．
  - コマンドsrunで直接実行できなかった．できる人がいたら教えてほしい．
- 利用可能な実行ノードはコマンドsinfoで調べることができる．syn...というパーティション名を除いて，どのパーティションでも動かせることができる．STATEがallocなら他の人が使っているので待機する必要がある．idleならすぐに利用できる．
- 組み合わせ例：(2019-07-15現在：構成に変更があれば使えなくなります)
  - salloc -p comq -N 1 -w ppx00
  - salloc -p comq -N 1 -w ppx01
  - salloc -p volta16 -N 1 -w ppx2-01
  - salloc -p comq -N 2 -w ppx00 -w ppx01
- コマンドmpirunにオプションを与えることで，並列数やプロセスを実行するマシンの指定などができる．

### 手順例

```bash
* 1. MPI環境の読み込み
module load gcc/4.8.5
module load openmpi/3.1.0
* 2. MPIプログラムの記述
wget https://www.hpcs.cs.tsukuba.ac.jp/~kashino/pictures/log/2019-07-15/mpi_hello_world.c
* 3. コンパイル&リンク
mpicc -o hello-world mpi_hello_world.c 
* 4. MPIプログラムを実行
PART=comq
NODE=ppx00
NP=4
salloc -p ${PART} -N 1\
            -w ${NODE}\
            -- mpirun -np ${NP} ./hello-world
```
### ppxの構成

<img src="./Images/PPX.png" alt="ISA" width="600">

詳しくはこちら：
[アーキテクチャチームがオープンラボで使ったPPXの仕様が書かれた資料](https://www.hpcs.cs.tsukuba.ac.jp/~taisuke/OpenLab2018-Arch.pdf)

## GitHub

- 今あるローカルのファイルをリモートリポジトリに登録する手順

1. 対象フォルダに移動
```bash
cd /path/to/your/project
```

2. Gitリポジトリを初期化
- Gitリポジトリを新規作成するコマンド、git initを実行すると隠しディレクトリ.git/が作成される。
```bash
git init
```

3.リモートリポジトリを登録
```bash
git remote add origin https:://github.com/Username/project.git
````

