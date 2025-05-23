## 目次

- [HPC\_Note1](#hpc_note1)
  - [1. 演算性能](#1-演算性能)
  - [2. 並行処理と並行処理](#2-並行処理と並行処理)
    - [並行処理](#並行処理)
    - [並列処理](#並列処理)
  - [3.Flynnの分類](#3flynnの分類)
    - [SISD (Single Instrution stream Single Datastream)](#sisd-single-instrution-stream-single-datastream)
    - [SIMD(Single Instruction stream Multiple Datastream)](#simdsingle-instruction-stream-multiple-datastream)
    - [MISD(Multiple Instruction stream Single Datastream)](#misdmultiple-instruction-stream-single-datastream)
    - [MIMD(Multiple Instruction stream Multiple Datastream)](#mimdmultiple-instruction-stream-multiple-datastream)
    - [欠点](#欠点)
  - [4.並列処理アーキテクチャ](#4並列処理アーキテクチャ)
  - [5.分散メモリ型並列計算機](#5分散メモリ型並列計算機)
    - [分散メモリ型並列計算機の仕組み](#分散メモリ型並列計算機の仕組み)
  - [6.共有メモリ型並列計算機](#6共有メモリ型並列計算機)
    - [並列メモリ型並列計算機の仕組み](#並列メモリ型並列計算機の仕組み)
  - [7.共有メモリ計算機アーキテクチャ](#7共有メモリ計算機アーキテクチャ)
    - [SMP (Symmetric Multi Processor)](#smp-symmetric-multi-processor)
    - [NUMA (Non-Uniformed Memory Access)](#numa-non-uniformed-memory-access)
    - [UMA (Uniform Memory Access model)](#uma-uniform-memory-access-model)
    - [分散共有ハイブリッド](#分散共有ハイブリッド)
  - [8.命令セットアーキテクチャ](#8命令セットアーキテクチャ)
    - [CISC (Complex Instruction Set Computer)](#cisc-complex-instruction-set-computer)
    - [RISC (Reduced Instruction Set Computer)](#risc-reduced-instruction-set-computer)
    - [GPU向けアーキテクチャ](#gpu向けアーキテクチャ)
    - [命令セットの具体例](#命令セットの具体例)
  - [9.ネットワークの構成要素](#9ネットワークの構成要素)
    - [並列計算に用いられるネットワーク](#並列計算に用いられるネットワーク)
    - [トポロジー](#トポロジー)
    - [InifiniBand](#inifiniband)
    - [通信の帯域とレイテンシ](#通信の帯域とレイテンシ)
    - [DMA(Direct Memory Access)](#dmadirect-memory-access)
    - [非同期通信](#非同期通信)
  - [10.物理コアと論理コア](#10物理コアと論理コア)
  - [11.実システムの紹介](#11実システムの紹介)
    - [クラスタとMPP](#クラスタとmpp)
    - [Top500](#top500)
    - [Green500](#green500)
  - [12.計算機アーキテクチャ](#12計算機アーキテクチャ)
    - [PC(Program Counter)](#pcprogram-counter)
  - [13.会社の歴史](#13会社の歴史)
    - [Intel](#intel)
    - [AMD](#amd)
    - [ARM](#arm)
  - [FPGA](#fpga)


# HPC_Note1

 > 僕がHigh Performance Computingに関するメモを残し後から体系的に学ぶための場所

## 1. 演算性能

- 演算性能：FLOPS
  - FLOP : Floating point Operatyions は浮動小数点演算数
- 通信性能：B/s
  - 必ずしも1B=8bitではないことに注意

## 2. 並行処理と並行処理

### 並行処理
  - 並行処理(Concurrent Processing)とは、複数の処理が時間的に重なりながら実行されることをさす。具体的には、物理的に同時に実行されるわけではなく、複数の処理が交互に進行することによってユーザには「同時に時移行されているように見える」という特徴がある。
  - > パイプライン処理とか？
  
  - 特徴:
    - 物理的な同時実行：並行処理では、複数のタスクが「時間的に重なる」形で実行されるが、必ずしも物理的に同時に動作するわけではない。シングルコアのプロセッサでも並行処理は可能で、タスクの切り替えが行われる（Context Switch）。
    - 高速化が目的：並列処理の主な目的は、大規模な計算や処理を高速に終えることである。計算処理やデータ処理を小さな単位に分割して、複数のプロセッサで一斉に実行することで、全体の処理時間を大幅に短縮できる。
  
  - 例：Webサーバ: 同時に複数のクライアントからのリクエストを処理する場合、リクエストが物理的に同時に処理されるわけではなく、スレッドやプロセスが切り替わりながら、ユーザーには同時にリクエストが処理されているように見える。
  
### 並列処理

  - [並列処理(Parallel Processing)](#4並列処理アーキテクチャ)とは、複数の処理が物理的に同時に実行されることを指す。これは、複数のプロセッサやCPUコアを利用して、タスクを本当に同時進行で処理する仕組みである。処理が「同時に走っている」ことが実際にハードウェア的に実現される。
  
  - 特徴：
    - 物理的な同時実行：並列処理では、複数の処理が同じ時間に実行されている。これは、マルチコアCPUや複数のプロセッサがある環境で可能。すべてのタスクが同時に処理されるため、処理時間の短縮が期待できる。
    - 高速化が目的：並列処理では、大規模な計算や処理を高速に終えることである。計算処理やデータ処理を小さな単位に分割して、複数のプロセッサーで一斉に実行することで、全体の処理時間を大幅に短縮できる。
    - ハードウェア依存：並列処理を効果的に行うには、ハードウェアの支援が不可欠。たとえば、マルチコアCPUやGPU、スーパーコンピュータなどが必要になる。

  -  例：科学技術計算・シミュレーション: スーパーコンピュータ「富岳」では、気象予測や分子構造解析などの複雑な計算を、何百万ものコアで分散実行することで短時間で処理する。

## 3.Flynnの分類

- 命令流(Instruction Stream)の数とデータ流(Data Stream)の数に着目し、コンピュータを以下の4つの方式に分類する。
- すべての論文で1番引用されてる、69年に描かれて今はMIMDがほぼすべてなので死語
- 命令が一個のプログラムなんてないんだからstreamをつけよう。

### SISD (Single Instrution stream Single Datastream)

- 命令流もデータ流も1つ。基本的なコンピュータの方式で、ユニプロセッサと呼ぶこともある。
- 我々並列処理を考える人にはいらない
 
### SIMD(Single Instruction stream Multiple Datastream)

- 1つの命令流で多数のデータを処理する方式。コントローラは命令を命令メモリから取ってきて、多数のプロセッサがそれぞれのデータに対して同じ処理を行うように指示する。データレベル並列性を簡単に利用することができるためGPUなどの計算加速装置(アクセラレータ)で用いられる。一般的なIntelのコンピュータにもこれと同じ考え方の命令(マルチメディア命令)が装備されている。
> 今は違う意味で使われているね、GPUとかに使われている。

<img src="./Images/SIMD.png" alt="SIMD" width="300">

### MISD(Multiple Instruction stream Single Datastream)

- 命令流が一つなのにデータ流が単数というのは考えにくいため通常はこの形のコンピュータは存在しないと考えられている。アナログコンピュータやパイプライン処理がこれにあたる。

### MIMD(Multiple Instruction stream Multiple Datastream)

- 複数命令流、複数データ流の方式。それぞれのプロセッサが独自に命令を取ってきて独立にデータを処理する。協調作業を行うには動機を取ってデータを交換する必要がある。現在、ノートPC、スマートフォン、サーバ、データセンターで動作している並列コンピュータのうちほとんどはここに分類される。
> 今はほぼ全部これ、並列処理っていたら分類せずとも全部これになるよね

### 欠点
- Flynnの分類はその簡単さにより現在でも使われているが、MIMDの枠が広すぎて並列コンピュータの分類としては機能しない。そこで1970年代のオペレーションシステム研究者を中心に[共有メモリ](#7共有メモリ計算機アーキテクチャ)に注目した分類が提唱された。

## 4.並列処理アーキテクチャ

  - [分散メモリアーキテクチャ(Distributed memory system)](#5分散メモリ型並列計算機)；各プロセスは明示的にデータを交換(送受信)しあう。各プロセッサは独自のメモリ(他プロセッサからはアクセス不可能)をも相互結合もうを用いたメッセージパッシングによってデータ交換を行う。
    - Massage passing(send, receive,...)
  - [共有メモリアーキテクチャ(Shared memory syatem)](#6共有メモリ型並列計算機)；各プロセスは共有するメモリのデータを読み書きで通信しあう。並列プロセッサ間で物理的に共有されるメモリを持ち各プロセッサがload/store命令を発行してデータの読み書きを行う。アーキテクチャのタイプとして[SMP](#smp-symmetric-multi-processor)と[NUMA](#numa-non-uniformed-memory-access)がある。
    - Shared mamory access(write, read,...)
  - hybrid型システム(constellation型)：共有メモリシステムを分散メモリシステムに結合している。

## 5.分散メモリ型並列計算機

### 分散メモリ型並列計算機の仕組み

  - CPUとメモリという一つの計算機システム(ノード)がネットワークで結合されているシステム
  - それぞれの計算機で実行されているプログラムはネットワークを通じてデータ(メッセージ)を交換し、動作する。
  - 比較的簡単に構築可能・拡張性(Scalability)が高い
    - 超並列計算機(MPP：Massively Parallel Processing)：1980年代後半から多数登場したが一部を除き消滅しつつある。(富岳はMPPシステム)
    - クラスタ型計算機；
  - 基本的にCPU+memory(+I/0)という逐次計算機構成を何らかのネットワーク(専用or汎用)で結合しているためハードウェア的にシンプル
  - プログラム上の明示的なMassage passingで通信を行うためユーザプログラミングは面倒
    - MPI(Massage Passing Interface)のような標準的なツールが提供されている。
    - domain decomopositionのような単純なデータ配列やmaster/worker型の処理は比較的容易に記述可能
  - 1980年代からMPPの典型的な実装として登場。現在はPCクラスタの基本的なアーキテクチャになっている。

<img src="./Images/distribute.png" alt="distribute" width="300">

## 6.共有メモリ型並列計算機

### 並列メモリ型並列計算機の仕組み

- それぞれのCPUで実行されているプログラム(スレッド)はメモリ上のデータにお互いにアクセスすることでデータを交換し動作する。
- アーキテクチャはさらにSMPとNUMAに分かれる。
- ハードウェアによる共有メモリの提供によりユーザにとってアプリケーションが非常に書きやすい。
  - multithreadプログラミング環境(POSIX thread)
  - OpenMP (共有メモリを前提とした簡易並列記述システム)
  - Automatic Parallelization(copiler does it for you)
- メモリという極めてPrivateな構成要素を共通化しているため性能を上げるには多くのハードウェア、アーキテクチャ的な工夫が必要
- 多数のプロセッサが一つのメモリ要素をアクセスする状況が簡易に記述できて極端なボトルネックを生じやすい。
  - システムのScalabilityの確保が困難

<img src="./Images/share.png" alt="share" width="300">

## 7.共有メモリ計算機アーキテクチャ

### SMP (Symmetric Multi Processor)

- 各プロセッサから見てどのmemory module への距離も等しい
- 構成としては複数のプロセッサが共通のバスを経由して等しくmemory module に接続されている。
- 大規模システムとしてはHPC2500シリーズ、日立SR16000シリーズ等が該当する
- Coherent chacheとの併用が一般的
- トラフィックが集中した時に性能低下が防げない

<img src="./Images/SMP.png" alt="SMP" width="300">

### NUMA (Non-Uniformed Memory Access)

- CPUに対して固有のmemory moduleがあり共有バスまたはスイッチを介して他のCPUのmoduleも直接アクセス可能
- 遠距離のmemory moduleへのアクセスには時間がかかる。(non-symmetric)
- コモディティスカラプロセッサとしてはAMD(Opteron)が最初に方式を取り入れた。
  - 今では標準になっている。
- 大規模システムではもう存在しない。(NUMAはCPUボード上のみ)
  
<img src="./Images/NUMA.png" alt="NUMA" width="300">

### UMA (Uniform Memory Access model)

- どのプロセッサから見ても同じ時間で読み書きできる共有メモリを持つ方式。典型的なUMAは共有バスやスイッチで複数のプロセッサが単一のメモリを共有する方式で、集中メモリ型と呼ばれる。メモリへのアクセスが集中するため、プロセッサ数を増やすことができないが実現のためのコストが小さいため、ノートPC、デスクトップPC、スマートフォンなどに使われている多くの小規模並列コンピュータがこの分類に入る。

- UMAとSMPの違い

| 観点   | UMA | SMP  |
|--------|------|--------|
| 対象   | メモリアーキテクチャ   | CPUの動作方式 |
| アクセス特性  | メモリへのアクセスが均一   | CPUは全て対等に動作 |
| メモリの種類   | 共有メモリ   | 共有メモリ(通常UMAを使う)   |
|　よく使われる場面   | UMA構成のSMPシステム   | OSやサーバなどのマルチコア対応環境    |

### 分散共有ハイブリッド

- 共通メモリと分散メモリの組み合わせ
- 分散メモリ型システムの各ノードがそれ自身共有メモリアーキテクチャになっている。(SMP or NUMA)
- マイクロプロセッサ自体が1チップで共有メモリ構成(マルチコア)となっていることが大きな要因、近年のマルチプロセッサ普及により急激に主流となった。

<img src="./Images/hybrid.png" alt="hybrid" width="600">

## 8.命令セットアーキテクチャ

- 命令セットアーキテクチャ(ISA : Instruction Set Architecture)はコンピュータのCPUやGPUが理解し、実行する命令のセットを定義するプロトコルのこと。ISAはソフトウェア(Operating SystemやApplication program)とハードウェア(CPU, GPU)間のインターフェースを提供する。ISAがどのような命令をサポートしているかによってCPUの性能、ソフトウェアの効率、開発の容易さなどが大きく影響を受ける。

### CISC (Complex Instruction Set Computer)
- 特徴：複雑な命令を一つの命令で実行できるように設計されたアーキテクチャ。一つの命令で複数のオペレーションを実行することができ、コードの密度が高くなる場合がある。
- 具体例；x86, x86-64(intel), MIPS(初期バージョン)
  
### RISC (Reduced Instruction Set Computer)
- 特徴：シンプルで高速な命令セットを持つアーキテクチャ。命令が少なく単純で1クロックで複数実行できる。RISCアーキテクチャでは複雑な命令を単純な命令に分解することが多いです。
- 具体例：ARM, MIPS(後期バージョン), SPARC, PowerPC

### GPU向けアーキテクチャ
- CUDA(NVIDIA)やOpenCLなど並列計算に特化した命令セットを提供する。

### 命令セットの具体例
- x86とARMはISAの一部で具体例であり、CISCやRISCは設計哲学。x86はCISCで作られたISAの具体例でARMはRISCで作られたISAの具体例。
- これまで、ISAは「x86系」が圧倒的なソフトウェア資産を背景に市場シェアを伸ばしてきたが、スマートフォンの普及に伴ってモバイル分野を制した「ARM v-」は全体の出荷量で「x86系」を大幅に超える規模までに成長した。近年はオープンソースのISA「RISC-V」が注目されている。次なる大きな市場はIoT分野だと考えられていてライセンスフリーの「RISC-V」が猛威を振るうのではないかという予想も立てられている。

参考文献：https://ipsj-catalog.jp/glossary/parlance/18.html

<img src="./Images/ISA.png" alt="ISA" width="600">

## 9.ネットワークの構成要素

### 並列計算に用いられるネットワーク

- 異なるmemory moduleは自由に触れない
- 並列計算ではネットワークの性能が非常に重要になる。
- 通信は計算の本質的な要素ではない。したがって可能な限り通信にかかる時間を短くしたい。

<img src="./Images/network.png" alt="network" width="300">

### トポロジー

<img src="./Images/topology.png" alt="network" width="600">

### InifiniBand

- 高性能な汎用通信網の一つ

<img src="./Images/InfiniBand.png" alt="network" width="600">

### 通信の帯域とレイテンシ

- 帯域(Byte/s)：1秒あたりの通信できる通信量
- レイテンシ(s)：送信側が送信して受信側に届くまでの時間、通信遅延

<img src="./Images/latency.png" alt="latency" width="600">

- 通信はできるだけまとめて行い、回数を減らした方が良い。

<img src="./Images/all.png" alt="all" width="600">

### DMA(Direct Memory Access)

- 高性能なネットワークでは通信アダプタがCPUメモリに直接アクセスできるものがある。このように外部デバイスがCPUメモリに直接アクセスすることをDMAと呼ぶ。
- データ転送中にCPUの介入が不要なため高性能、CPUが自由になるための他の計算を行える。

<img src="./Images/DMA.png" alt="DMA" width="400">

### 非同期通信

- CPUにとって通信はかなりの時間がかかる作業である。通信が終わるまでCPUが何もしないのでは無駄が大きい。
- 計算を行っている間に通信を行う。→非同期通信
- 効果は大きいがプログラミングは複雑になる。

<img src="./Images/com.png" alt="com" width="600">

## 10.物理コアと論理コア

| 種類   | 説明 |
|--------|------|
| 物理コア   | 実際にCPUチップ内に存在する「処理の心臓部」そのもの（物理的なユニット）   |
| 論理コア  | OSから見える処理単位。物理コアが仮想的に「2つに見える」こともある（＝ハイパースレッディング）  |

-  Intel Corei7などのCPUには
   - 物理コア：4つ
   - 論理コア：8つという構成が多い。これは各物理コアがHyper-Threadingにより二つの論理コアに見えるから。
-  M1pro-chipでは
  - 物理コア：8つ
   - 論理コア：8つ

- CCSのPre-PACS versionXの環境

```
Architecture:          x86_64
CPU op-mode(s):        32-bit, 64-bit
Byte Order:            Little Endian
CPU(s):                56
On-line CPU(s) list:   0-55
Thread(s) per core:    2
Core(s) per socket:    14
Socket(s):             2
NUMA node(s):          2
Vendor ID:             GenuineIntel
CPU family:            6
Model:                 79
Model name:            Intel(R) Xeon(R) CPU E5-2660 v4 @ 2.00GHz
Stepping:              1
CPU MHz:               2400.634
CPU max MHz:           3200.0000
CPU min MHz:           1200.0000
BogoMIPS:              4000.27
Virtualization:        VT-x
L1d cache:             32K
L1i cache:             32K
L2 cache:              256K
L3 cache:              35840K
NUMA node0 CPU(s):     0-13,28-41
NUMA node1 CPU(s):     14-27,42-55
```
<img src="./Images/hard-ppx.png" alt="com" width="600">

- ソケット2つの28coreCPUで2threadのクラスタ
  
<img src="./Images/ppx-detail.png" alt="com" width="600">

## 11.実システムの紹介

- 富岳
  - 電気代だけで年間50億円、総工費1100億円、理化学研究所と富士通と文部科学省HPCIとの連携

### クラスタとMPP

|特徴   | クラスタ(Cluster) | MPP(Massively Parallel Processing)  |
|--------|------|--------|
| ノード構成   | 商用コンピュータやサーバがネットワく接続   | 専用の高性能ノードで構成されたシステム |
| 通信方式  | 通常は標準的なネットワーク(Ethernet)   | 高速インターコネクト(InfiniBandなど) |
| スケーラビリティ   | 高いが、ネットワークの帯域が制約になることも   | 非常に高いスケーラビリティ   |
| 並列処理の種類   |分散メモリモデル、タスク並列処理   | データ並列処理、大規模な並列処理が可能   |
| 用途   | 一般的な並列処理、サーバ用途   | 科学技術計算、大規模趣味レーション   |


### Top500

- The 64th edition of the TOP500 reveals that El Capitan has achieved the top spot and is officially the third system to reach exascale computing after Frontier and Aurora. Both systems have since moved down to No. 2 and No. 3 spots, respectively. Additionally, new systems have found their way onto the Top 10.

  The new El Capitan system at the Lawrence Livermore National Laboratory in California, U.S.A., has debuted as the most powerful system on the list with an HPL score of 1.742 EFlop/s. It has 11,039,616 combined CPU and GPU cores and is based on AMD 4th generation EPYC processors with 24 cores at 1.8GHz and AMD Instinct MI300A accelerators. El Capitan relies on a Cray Slingshot 11 network for data transfer and achieves an energy efficiency of 58.89 Gigaflops/watt. This power efficiency rating helped El Capitan achieve No. 18 on the GREEN500 list as well.

  The Frontier system at Oak Ridge National Laboratory in Tennessee, U.S.A, has moved down to the No. 2 spot. It has increased its HPL score from 1.206 Eflop/s on the last list to 1.353 Eflop/s on this list. Frontier has also increased its total core count, from 8,699,904 cores on the last list to 9,066,176 cores on this list. It relies on Cray’s Slingshot 11 network for data transfer.

  The Aurora system at Argonne Leadership Computing Facility in Illinois, U.S.A, has claimed the No. 3 spot on this TOP500 list. The machine kept its HPL benchmark score from the last list, achieving 1.012 Exaflop/s. Aurora is built by Intel based on the HPE Cray EX – Intel Exascale Compute blade which uses Intel Xeon CPU Max Series Processors and Intel Data Center GPU Max Series accelerators that communicate through Cray’s Slingshot-11 network interconnect.

  The Eagle system installed on the Microsoft Azure Cloud in the U.S.A. claimed the No. 4 spot and remains the highest-ranked cloud-based system on the TOP500. It has an HPL score of 561.2 PFlop/s

  The only other new system in the TOP 5 is the HPC6 system at No. 5. This machine is installed at Eni S.p.A center in Ferrera Erbognone, Italy and has the same architecture as the No. 2 system Frontier. The HPC6 system at Eni achieved an HPL benchmark of 477.90 PFlop/s and is now the fastest system in Europe.

### Green500

- In the Green500 the systems of the TOP500 are ranked by how much computational performance they deliver on the HPL benchmark per Watt of electrical power consumed. This electrical power efficiency is measured in Gigaflops/Watt. This ranking is not driven by the size of a system but by its technology and the ranking order looks therefor very different from the TOP500. The computational efficiency of a system tends to slightly decrease with system size, which among technologically identical system gives smaller system the advantage. 

  > 30th / TSUBAME4.0 - HPE Cray XD665, AMD EPYC 9654 96C 3.55GHz, Nvidia H100 SXM5 94Gb, Infiniband NDR200, Redhat Linux, HPE

  > 33th / Miyabi-G - Supermicro ARS 111GL DNHR LCC, Grace Hopper Superchip 72C 3GHz, Infiniband NDR200, Rocky Linux, Fujitsu
Joint Center for Advanced High Performance Computing

  > 48th / Pegasus - NEC LX 102Bk-6, Xeon Platinum 8468 48C 2.1GHz, NVIDIA H100 PCIe 80GB, Infiniband NDR200, Ubuntu 22.04, NEC
Center for Computational Sciences, University of Tsukuba

  > 86th / Supercomputer Fugaku - Supercomputer Fugaku, A64FX 48C 2.2GHz, Tofu interconnect D, Fujitsu
RIKEN Center for Computational Science

## 12.計算機アーキテクチャ

### PC(Program Counter)

 - プログラムカウンター（Program Counter, 略して PC）とは、コンピュータのCPU(中央演算処理装置)にあるレジスタの一つで、次に実行する命令のアドレス（メモリ上の位置）を保持しているもの。


## 13.会社の歴史

### Intel
- Integlated Electronicsの略称

- 元々何をしてたの？
  - 元々はSRAMとDRAMを作る会社だった。世界初の商用SRAMとDRAMを作った会社
  - 1969年ごろ、Busicom(ビジコン)がIntelに「電卓用の専用チップセット」を依頼

| 時期            | 主な事業                    | 製品例                                 |
| ------------- | ----------------------- | ----------------------------------- |
| 1968〜1970年代初期 | 半導体メモリ                  | 1103（DRAM）、3101（SRAM）               |
| 1971〜1980年代   | マイクロプロセッサも展開            | 4004 → 8008 → 8086など                |
| 1980年代〜現在     | CPU事業が中核に               | x86系CPU（Pentium、Coreなど）             |
| 現在            | CPU + FPGA + AI半導体など多角化 | Xeon, Core, Gaudi, FPGA (Altera) など |

### AMD
- 元はIntelのセカンドソース（互換CPUを製造）
- 今は独自のx86-64拡張を持ち、Ryzen(CPU)やRadeon(GPU)などを展開
- Intelは製造設備も自社(IDMモデル)、AMDは設計専業(ファブレス)

### ARM

- ARMはイギリスの企業（Arm Ltd.）でCPUの設計専門
- ARM自身は製造せず、ライセンスを提供（Apple, Qualcomm, etc）
- ARMのCPUは低消費電力でスマホや組込み機器に最適

## FPGA

- Gate Array は、あらかじめ多数の基本ゲート（通常はNANDやNORなど）をチップ上に配置し、あとから配線（interconnect）で特定の論理機能を実現するLSI設計手法の一種。
- これを現場(Field)で設計可能(Programmable)にしたのがFPGAである。
