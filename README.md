# PySAT: Python SAT ソルバーツールキット

[![PyPI version](https://img.shields.io/pypi/v/python-sat)](https://pysathq.github.io/)
[![Tests](https://img.shields.io/github/actions/workflow/status/pysathq/pysat/test.yml?label=tests)](https://github.com/pysathq/pysat)
[![Downloads](https://img.shields.io/pypi/dm/python-sat)](https://pypi.org/project/python-sat/)
[![License: MIT](https://img.shields.io/github/license/pysathq/pysat)](https://github.com/pysathq/pysat/blob/master/LICENSE.txt)
[![DOI (SAT'18)](https://img.shields.io/badge/doi-10.1007%2F978--3--319--94144--8__26-blue)](https://doi.org/10.1007/978-3-319-94144-8_26)
[![DOI (SAT'24)](https://img.shields.io/badge/doi-10.4230%2FLIPIcs.SAT.2024.16-blue)](https://doi.org/10.4230/LIPIcs.SAT.2024.16)

PySAT は、最先端の [Boolean 充足可能性問題 (SAT)](https://en.wikipedia.org/wiki/Boolean_satisfiability_problem) ソルバー群への統一的な Python インターフェースを提供するツールキットです。16 以上の SAT ソルバーへのインクリメンタルアクセス、多様なカーディナリティ / 擬似ブール制約エンコーディング、CNF 前処理、ユーザー定義制約プロパゲータ（IPASIR-UP）などを備えています。

- **公式サイト**: https://pysathq.github.io/
- **ソースコード**: https://github.com/pysathq/pysat
- **PyPI**: https://pypi.org/project/python-sat/
- **バージョン**: 1.8.dev30
- **ライセンス**: MIT

---

## 目次

1. [概要](#概要)
2. [ディレクトリ構造](#ディレクトリ構造)
3. [主要モジュール](#主要モジュール)
4. [対応 SAT ソルバー](#対応-sat-ソルバー)
5. [カーディナリティ・擬似ブールエンコーディング](#カーディナリティ擬似ブールエンコーディング)
6. [インストール](#インストール)
7. [使い方](#使い方)
8. [サンプルスクリプト](#サンプルスクリプト)
9. [SAT ソルバー高速化の先行研究分析](#sat-ソルバー高速化の先行研究分析)
10. [新規手法の探索](#新規手法の探索)
11. [PySAT への統合提案](#pysat-への統合提案)
12. [テスト](#テスト)
13. [引用](#引用)
14. [ライセンス](#ライセンス)

---

## 概要

PySAT は、SAT ソルバーを用いたプロトタイピングを Python で容易にするためのツールキットです。主な用途:

- **NP 完全問題の求解**: CNF 式で符号化された問題を SAT ソルバーで解く
- **MaxSAT ソルバーの実装**: RC2、LBX 等のアルゴリズムの実装
- **MUS/MCS の抽出・列挙**: 最小不充足部分集合 / 最小修正部分集合の操作
- **モデル列挙**: 充足可能な割当の全列挙
- **インクリメンタル求解**: 仮定 (assumptions) を用いた逐次的な SAT 呼び出し
- **制約プロパゲーション**: IPASIR-UP インターフェースによるユーザー定義エンジン

すべてのソルバーは C/C++ のオリジナル実装をラップしており、Python の使いやすさと低レベル実装の性能を両立しています。

---

## ディレクトリ構造

```
pysat/
├── pysat/                  # メイン Python パッケージ
│   ├── __init__.py         #   パッケージ初期化・バージョン情報
│   ├── solvers.py          #   SAT ソルバー統一インターフェース (7,591 行)
│   ├── formula.py          #   論理式データ構造 — CNF, WCNF, CNFPlus (5,696 行)
│   ├── integer.py          #   有限定義域整数変数のブール符号化 (1,945 行)
│   ├── engines.py          #   IPASIR-UP 制約プロパゲータ (1,302 行)
│   ├── card.py             #   カーディナリティ制約エンコーディング (802 行)
│   ├── pb.py               #   擬似ブール制約エンコーディング (463 行)
│   ├── process.py          #   CNF 前処理 — CaDiCaL 1.5.3 (363 行)
│   ├── _fileio.py          #   圧縮ファイル I/O (gzip, bzip2, lzma, zstd)
│   └── _utils.py           #   ユーティリティ
│
├── solvers/                # SAT ソルバーソースコード・ビルド
│   ├── pysolvers.cc        #   C++ → Python バインディング
│   ├── prepare.py          #   ソルバーダウンロード・パッチスクリプト
│   ├── patches/            #   各ソルバー用パッチファイル (16 個)
│   └── *.tar.gz / *.zip    #   ソルバーアーカイブ (16 ソルバー)
│
├── cardenc/                # C++ カーディナリティエンコーディング実装
│   ├── pycard.cc           #   Python C 拡張エントリポイント
│   ├── tot.hh              #   Totalizer
│   ├── itot.hh             #   Incremental Totalizer
│   ├── seqcounter.hh       #   Sequential Counter
│   ├── sortcard.hh         #   Sorting Networks
│   ├── cardn.hh            #   Cardinality Networks
│   ├── mto.hh              #   Modulo Totalizer
│   ├── kmtot.hh            #   k-Cardinality Modulo Totalizer
│   ├── pairwise.hh         #   Pairwise (AtMost1)
│   ├── bitwise.hh          #   Bitwise (AtMost1)
│   └── ladder.hh           #   Ladder/Regular (AtMost1)
│
├── allies/                 # サードパーティツール統合
│   ├── approxmc.py         #   近似モデルカウンティング (pyapproxmc)
│   └── unigen.py           #   一様ランダムサンプリング (pyunigen)
│
├── examples/               # 実行可能サンプルスクリプト (15 個)
│   ├── rc2.py              #   MaxSAT ソルバー (RC2 アルゴリズム)
│   ├── lbx.py              #   近似 MCS 抽出・列挙 (LBX)
│   ├── optux.py            #   最適化 + UNSAT コア
│   ├── musx.py             #   MUS 抽出
│   ├── mcsls.py            #   最小修正部分集合 (MCS-LS)
│   ├── hitman.py           #   ヒッティングセット列挙
│   ├── models.py           #   モデル列挙
│   ├── fm.py               #   特徴モデル分析
│   ├── genhard.py          #   困難インスタンス生成
│   ├── lsu.py              #   コア学習・縮小
│   ├── bica.py             #   二基準最適化
│   ├── bbscan.py           #   ブラックボックススキャナ
│   ├── primer.py           #   前処理 + 求解
│   └── usage.py            #   基本使用例
│
├── tests/                  # テストスイート (17 ファイル, pytest)
├── docs/                   # Sphinx ドキュメント
├── web/                    # ウェブサイトアセット
├── .github/                # GitHub Actions CI/CD
├── travis/                 # Travis CI スクリプト
├── appveyor/               # AppVeyor CI スクリプト (Windows)
├── win/                    # Windows 用ビルドファイル
│
├── setup.py                # パッケージビルド設定 (C++ 拡張 2 つ)
├── setup.cfg               # セットアップメタデータ
├── MANIFEST.in             # 配布マニフェスト
├── requirements.txt        # 依存: six
├── README.rst              # 公式ドキュメント (reStructuredText)
├── LICENSE.txt             # MIT ライセンス
└── appveyor.yml            # Windows CI 設定
```

---

## 主要モジュール

### `pysat.solvers` — SAT ソルバー統一インターフェース

MiniSat ライクなインクリメンタル API で 16 以上のソルバーにアクセス。

| クラス | 主な機能 |
|--------|----------|
| `Solver` | ソルバー名による汎用ファクトリ |
| `Cadical103` / `Cadical153` / `Cadical195` | CaDiCaL 各バージョン |
| `Glucose3` / `Glucose4` / `Glucose42` | Glucose 各バージョン |
| `Gluecard3` / `Gluecard4` | Glucose + ネイティブカーディナリティ |
| `Lingeling` | Lingeling |
| `MapleChrono` / `MapleCM` / `Maplesat` | Maple 系列 |
| `Mergesat3` | MergeSAT 3.0 |
| `Minicard` | ネイティブカーディナリティ対応 MiniSat |
| `Minisat22` / `MinisatGH` | MiniSat 各バージョン |
| `CryptoMinisat` | XOR 節サポート (pycryptosat 経由) |

### `pysat.formula` — 論理式データ構造

| クラス | 用途 |
|--------|------|
| `CNF` | 連言標準形 (節のリスト) |
| `CNFPlus` | CNF + ネイティブカーディナリティ制約 |
| `WCNF` / `WCNFPlus` | 重み付き部分 CNF (MaxSAT 用) |
| `IDPool` | 変数 ID 管理 |
| `Atom`, `And`, `Or`, `Neg`, `Implies`, `Equals`, `XOr`, `ITE` | 非節形式の論理式ノード |

### `pysat.card` — カーディナリティエンコーディング

9 種類の制約エンコーディングを C++ 実装で提供 (`CardEnc`, `ITotalizer`)。

### `pysat.pb` — 擬似ブール制約

PyPBLib 経由で BDD、Sequential Weight Counter、Sorting Network、Adder Network、Binary Merge を提供。

### `pysat.engines` — 制約プロパゲータ (IPASIR-UP)

CaDiCaL 1.9.5 のユーザープロパゲータインターフェースにより、線形 (カーディナリティ / PB) 制約やパリティ (XOR) 制約をネイティブに伝搬。

### `pysat.process` — CNF 前処理

CaDiCaL 1.5.3 のプリプロセッサを公開。ブロック節削除、等価リテラル置換、限界変数消去、失敗リテラル探索、節包含、vivification 等を実行。

### `pysat.integer` — 整数変数

有限定義域の整数変数と線形制約をブール変数上にエンコード（実験的機能）。

---

## 対応 SAT ソルバー

| ソルバー | バージョン | 主な特徴 |
|----------|-----------|----------|
| [CaDiCaL](https://github.com/arminbiere/cadical) | 1.0.3 / 1.5.3 / 1.9.5 | 最先端の CDCL ソルバー。1.5.3 で前処理、1.9.5 で IPASIR-UP サポート |
| [Glucose](http://www.labri.fr/perso/lsimon/glucose/) | 3.0 / 4.1 / 4.2.1 | LBD ベースの学習節管理で知られるソルバー |
| Gluecard | 3 / 4 | Glucose + MiniCard のネイティブカーディナリティ |
| [Lingeling](http://fmv.jku.at/lingeling/) | bbc-9230380 | 高度な inprocessing を備えたソルバー |
| [MapleLCMDistChronoBT](http://sat2018.forsyte.tuwien.ac.at/) | SAT Competition 2018 | LRB + 時系列バックトラック |
| MapleCM / Maplesat | SAT Competition 2018 / LRB | Maple 系列の CDCL ソルバー |
| [MergeSAT](https://github.com/conp-solutions/mergesat) | 3.0 | Merge Resolution ベースの最適化 |
| [MiniCard](https://github.com/liffiton/minicard) | 1.2 | ネイティブカーディナリティ制約 |
| [MiniSat](http://minisat.se/MiniSat.html) | 2.2 / GitHub | CDCL の基本実装 |
| [CryptoMiniSat](https://github.com/msoos/cryptominisat) | 5.x (外部) | XOR 節のネイティブサポート |

---

## カーディナリティ・擬似ブールエンコーディング

### カーディナリティエンコーディング (C++ 実装)

| エンコーディング | 型 | 適用範囲 | 参考文献 |
|------------------|----|---------|----|
| Pairwise | `EncType.pairwise` | AtMost1 のみ | Prestwich (2009) |
| Bitwise | `EncType.bitwise` | AtMost1 のみ | Prestwich (2009) |
| Ladder/Regular | `EncType.ladder` | AtMost1 のみ | Ansotegui & Manya (2004) |
| Sequential Counter | `EncType.seqcounter` | 汎用 | Sinz (2005) |
| Sorting Networks | `EncType.sortnetwrk` | 汎用 | Batcher (1968) |
| Cardinality Networks | `EncType.cardnetwrk` | 汎用 | Asin et al. (2009) |
| Totalizer | `EncType.totalizer` | 汎用 | Bailleux & Boufkhad (2003) |
| Modulo Totalizer | `EncType.mtotalizer` | 汎用 | Ogawa et al. (2013) |
| k-Cardinality Modulo Totalizer | `EncType.kmtotalizer` | 汎用 | Ogawa et al. (2013) |

### 擬似ブールエンコーディング (PyPBLib 経由)

BDD、Sequential Weight Counter、Sorting Networks、Adder Networks、Binary Merge

---

## インストール

### PyPI (推奨)

```bash
pip install python-sat[aiger,approxmc,cryptosat,pblib]
```

オプション依存なし:

```bash
pip install python-sat
```

### ソースからビルド

```bash
git clone https://github.com/pysathq/pysat.git
cd pysat
python setup.py install
```

### 必要環境

- Python 2.7 / 3.4+
- C/C++11 対応コンパイラ (GCC / Clang)
- GNU `make`, `patch`
- Python ヘッダ、zlib ヘッダ
- `six` パッケージ

> **macOS**: Clang 推奨 (`export CC=/usr/bin/clang`)

---

## 使い方

### 基本的な SAT 求解

```python
from pysat.solvers import Glucose3

g = Glucose3()
g.add_clause([-1, 2])
g.add_clause([-2, 3])
print(g.solve())       # True
print(g.get_model())   # [-1, -2, -3]
g.delete()
```

### コンテキストマネージャの利用

```python
from pysat.solvers import Minisat22

with Minisat22(bootstrap_with=[[-1, 2], [-2, 3]]) as m:
    print(m.solve(assumptions=[1, -3]))  # False
    print(m.get_core())                  # [-3, 1]
```

### CNF ファイルの読み込み

```python
from pysat.formula import CNF
from pysat.solvers import Cadical195

formula = CNF(from_file='example.cnf')
with Cadical195(bootstrap_with=formula) as solver:
    if solver.solve():
        print(solver.get_model())
```

### カーディナリティ制約

```python
from pysat.card import CardEnc, EncType
from pysat.solvers import Glucose4

# x1 + x2 + x3 + x4 <= 2 (AtMost)
cnf = CardEnc.atmost(lits=[1, 2, 3, 4], bound=2, encoding=EncType.totalizer)
with Glucose4(bootstrap_with=cnf) as solver:
    solver.solve()
    print(solver.get_model())
```

### MaxSAT (RC2)

```python
from pysat.formula import WCNF
from pysat.examples.rc2 import RC2

wcnf = WCNF()
wcnf.append([-1, 2])        # ハード節
wcnf.append([1], weight=1)  # ソフト節 (重み 1)
wcnf.append([-2], weight=3) # ソフト節 (重み 3)

with RC2(wcnf) as rc2:
    model = rc2.compute()
    print(model)             # 最適モデル
    print(rc2.cost)          # 最適コスト
```

### IPASIR-UP ユーザープロパゲータ

```python
from pysat.solvers import Cadical195
from pysat.engines import BooleanEngine, LinearConstraint

# x1 + x2 + x3 >= 2 をプロパゲータで処理
engine = BooleanEngine()
engine.add(LinearConstraint(lits=[1, 2, 3], bound=2, is_atleast=True))

with Cadical195() as solver:
    solver.set_engine(engine)
    solver.add_clause([1, 2, 3])
    solver.solve()
```

---

## サンプルスクリプト

インストール後、Python 内部から、またはコマンドラインから利用可能:

| スクリプト | アルゴリズム | 用途 |
|-----------|-------------|------|
| `rc2.py` | RC2/OLLITI | MaxSAT ソルバー |
| `lbx.py` | LBX | 近似 MCS 抽出・列挙 |
| `optux.py` | OptUx | 最適 UNSAT 部分集合 |
| `musx.py` | Deletion-based | MUS 抽出 |
| `mcsls.py` | CLD-like | MCS 列挙 |
| `hitman.py` | Hitting Set | ヒッティングセット列挙 |
| `models.py` | — | モデル列挙 |
| `fm.py` | Fu & Malik | 特徴モデル分析 |
| `genhard.py` | — | 困難インスタンス生成器 |
| `lsu.py` | LSU | コア学習・縮小 |
| `bica.py` | — | 二基準最適化 |
| `bbscan.py` | — | ブラックボックススキャナ |
| `primer.py` | — | 前処理 + 求解 |

```bash
# コマンドライン例
rc2.py -s g4 -v input.wcnf
lbx.py -e all -d -s g4 another-input.wcnf
```

---

## SAT ソルバー高速化の先行研究分析

SAT ソルバーの性能を向上させるための主要な研究領域と最新動向を以下に整理します。

### 1. CDCL (Conflict-Driven Clause Learning) の改良

現代の SAT ソルバーの中核をなす CDCL アルゴリズムに対する主な改善:

| 技術 | 概要 | 影響度 |
|------|------|--------|
| **LBD ベースの学習節管理** | Glucose が導入。節の「接着度 (LBD)」で品質を測定し、低品質節を積極的に削除 | 高 |
| **LRB (Learning Rate Branching)** | MapleSAT で導入。変数活性度ではなく学習率に基づく分岐ヒューリスティック | 高 |
| **CHB (Conflict History-Based)** | 衝突履歴に基づく分岐。指数移動平均で変数の有用性を追跡 | 中 |
| **時系列バックトラック** | MapleLCMDistChronoBT (SAT Competition 2018 優勝)。非時系列バックジャンプの代替 | 高 |
| **Phase Saving** | 以前の割当の極性を記憶し、バックトラック後の再割当を高速化 | 高 |

### 2. Inprocessing 技術

求解中に定期的にインスタンスを簡略化する技術群:

| 技術 | 説明 | PySAT での対応 |
|------|------|---------------|
| **節包含 (Subsumption)** | 冗長な節の検出・除去 | `process.py` (CaDiCaL 1.5.3) |
| **Vivification** | 節の各リテラルを伝搬テストし、短縮可能な節を発見 | `process.py` |
| **限界変数消去 (BVE)** | 解決可能な変数を消去してインスタンスを縮小 | `process.py` |
| **ブロック節削除 (BCE)** | ブロックされたリテラルを含む節の除去 | `process.py` |
| **等価リテラル置換** | 等価なリテラル対を検出し置換 | `process.py` |
| **失敗リテラル探索** | 単位伝搬で矛盾するリテラルの検出 | `process.py` |

**CaDiCaL 2.0 の改良点** ([Biere et al., CAV 2024](https://link.springer.com/chapter/10.1007/978-3-031-65627-9_7)):
- コード規模が約 46 KLOC → 64 KLOC に拡大
- Kissat から移植された技術群
- Vivification で接頭辞木 (prefix tree) を構築し、判断回数を約 30% 削減
- 学習節の階層 (tier) 別に vivification 予算を配分

### 3. リスタート戦略

| 戦略 | 説明 |
|------|------|
| **Luby リスタート** | Luby 数列に基づく固定パターン |
| **Glucose スタイル** | 最近の LBD 移動平均が全体平均を大幅に上回った場合にリスタート |
| **幾何学的リスタート** | 間隔を幾何的に増加 |
| **適応的リスタート** | ソルバーの状態に応じて動的に調整 |

### 4. エンコーディング選択の影響

カーディナリティ・PB 制約のエンコーディング選択は、ソルバー性能に大きく影響します:

- **Totalizer**: 一般的に最もバランスの取れた性能。変数増加は O(n log n)
- **Sequential Counter**: 変数 O(n·k)、節 O(n·k)。k が小さい場合に有効
- **Sorting Networks**: 大規模制約で安定した性能
- **Pairwise/Ladder**: AtMost1 限定だが、極めて効率的

### 5. 前処理の重要性

SAT 求解前の前処理で、インスタンスサイズを大幅に削減可能:

```python
from pysat.process import Processor
from pysat.formula import CNF

formula = CNF(from_file='large_instance.cnf')
processor = Processor(bootstrap_with=formula)
processed = processor.process()  # 等価な簡略化された CNF
processor.delete()
```

---

## 新規手法の探索

SAT 求解をさらに高速化するための最新・新興アプローチを以下に整理します。

### 1. 機械学習統合アプローチ

#### NeuroBack — GNN ベースの変数フェーズ予測

[NeuroBack](https://openreview.net/pdf?id=samyfu6G93) は、Graph Neural Network (GNN) を用いて CDCL ソルバーの変数フェーズ（極性）を事前予測する手法です。

- **手法**: Weighted Literal-Incidence Graph (WLIG) で式を符号化し、GNN でバックボーン変数を予測
- **性能**: SAT Competition ベンチマークで最大 7.4% 多くの問題を解決
- **特徴**: 求解時に GPU 不要（オフライン推論のみ）
- **成熟度**: 実用段階

#### SATLUTION — LLM によるソルバー自律進化

[SATLUTION](https://arxiv.org/abs/2509.07367) は、LLM ベースのコード進化をリポジトリ全体に拡張した初のフレームワークです。

- **成果**: SAT Competition 2025 の上位2ソルバーを上回る PAR-2 スコアを達成
- **手法**: 2024 年のコードベースとベンチマークのみで学習
- **意義**: 人間が設計したソルバーを LLM 進化が凌駕した初の事例
- **成熟度**: 研究段階

#### アルゴリズム選択 (Algorithm Selection)

- **SATzilla / AutoFolio**: インスタンス特徴量に基づいて最適ソルバーを自動選択
- **PySAT での応用**: 複数ソルバーからインスタンスごとに最適なものを選択するポートフォリオ戦略

### 2. 並列・分散 SAT 求解

#### Cube-and-Conquer

困難な SAT インスタンスに対する主要な並列化手法:

1. **Cube フェーズ**: Look-ahead ソルバーで探索空間を小さな部分問題 (cube) に分割
2. **Conquer フェーズ**: 各 cube を独立した CDCL ソルバーで並列求解

- **[AlphaMapleSAT](https://arxiv.org/abs/2401.13770)** (2024): Monte Carlo Tree Search (MCTS) で cube 生成を最適化。128 コアで 1.61x〜7.57x の高速化
- **利点**: 本質的に並列化可能（各 cube は独立）
- **課題**: cube 分割の品質が性能を大きく左右

#### GPU 加速

- **[ParaFROST](https://github.com/muhos/ParaFROST)**: NVIDIA CUDA GPU で inprocessing を並列実行
- **[TurboSAT](https://arxiv.org/html/2511.07737)**: 行列演算として SAT を定式化し、GPU-CPU ハイブリッドシステムで求解。勾配ベースの確率的探索

### 3. ハイブリッドアプローチ

| 手法 | 説明 | 成熟度 |
|------|------|--------|
| **Local Search + CDCL** | Stochastic Local Search (SLS) と CDCL の組み合わせ。CCSat 等 | 実用段階 |
| **Look-ahead + CDCL** | Cube-and-Conquer の基盤技術 | 実用段階 |
| **SMT 統合** | SAT Modulo Theories — 理論特化の推論と SAT の統合 | 広く利用 |

### 4. 次世代ソルバーアーキテクチャ

#### Kissat (CaDiCaL の C 移植版)

- CaDiCaL を C 言語に書き直し、データ構造を最適化
- Inprocessing のスケジューリングを改善
- SAT Competition で継続的に上位を獲得
- **PySAT には未統合** → 統合候補

#### CaDiCaL 2.0 ([SAT Competition 2024/2025 参加](https://cca.informatik.uni-freiburg.de/papers/BiereFallerFleuryFroleyksPollitt-SAT-Competition-2025-solvers.pdf))

- Kissat からの技術バックポート
- Vivification の大幅改善
- 証明生成機能の強化

---

## PySAT への統合提案

リサーチ結果に基づく、PySAT の性能向上のための具体的な提案:

### 短期 (すぐに実装可能)

| 提案 | 期待効果 | 実装難易度 |
|------|---------|----------|
| **Kissat の統合** | 最新の inprocessing、最高水準の性能 | 中 — C ソルバーのバインディング追加 |
| **CaDiCaL 2.0 へのアップデート** | vivification 改良等の恩恵 | 低 — 既存バインディングの更新 |
| **ソルバーポートフォリオ機能** | インスタンス特徴量に基づく自動ソルバー選択 | 中 — Python レベルの実装 |
| **前処理パイプライン** | 複数の前処理技術の自動チェーン | 低 — 既存 `process.py` の拡張 |

### 中期 (設計が必要)

| 提案 | 期待効果 | 実装難易度 |
|------|---------|----------|
| **並列ポートフォリオ** | 複数ソルバーの並列実行、最速の結果を採用 | 中 — `multiprocessing` 活用 |
| **Cube-and-Conquer 統合** | 困難インスタンスの並列分割求解 | 高 — look-ahead ソルバー統合が必要 |
| **Intel SAT Solver 統合** | 追加のソルバー選択肢 | 中 — C++ バインディング追加 |
| **学習節フェーズ予測** | NeuroBack 的なオフラインフェーズ最適化 | 中 — Python + GNN 推論 |

### 長期 (研究段階)

| 提案 | 期待効果 | 実装難易度 |
|------|---------|----------|
| **GPU 加速 inprocessing** | ParaFROST 的な GPU 並列前処理 | 高 — CUDA 統合 |
| **GNN ベースの分岐ヒューリスティック** | 学習ベースの変数選択 | 高 — モデル学習インフラが必要 |
| **LLM ベースのソルバー進化** | SATLUTION 的な自動改良 | 実験段階 |

### 使い方の最適化ガイド

PySAT を使って SAT 問題をより高速に解くための実践的アドバイス:

1. **ソルバー選択**: 問題の性質に応じたソルバー選択が最も効果的
   - 一般的な CNF → `Cadical195` (最新 CDCL + IPASIR-UP)
   - XOR 制約あり → `CryptoMinisat`
   - カーディナリティ制約あり → `Minicard` / `Gluecard4`
   - インクリメンタル求解 → `Glucose42` / `Cadical195`

2. **前処理の活用**: 大規模インスタンスでは `Processor` による前処理が効果的

3. **エンコーディング選択**: カーディナリティ制約には Totalizer が安定、AtMost1 なら Pairwise/Ladder

4. **インクリメンタル API の活用**: 仮定 (assumptions) を使い、ソルバーの学習済み情報を再利用

---

## テスト

```bash
# 全テスト実行
pytest tests/

# 特定モジュールのテスト
pytest tests/test_cnf.py
pytest tests/test_process.py
```

| テストファイル | 対象 |
|--------------|------|
| `test_cnf.py` | CNF/CNFPlus 構造とソルバー |
| `test_integer.py` | 整数制約ソルバー |
| `test_process.py` | 前処理パイプライン |
| `test_propagate.py` | プロパゲータ |
| `test_boolengine.py` | BooleanEngine |
| `test_clausification.py` | 論理式の節化 |
| `test_atmost.py` / `test_atmost1.py` / `test_atmostk.py` | カーディナリティ制約 |
| `test_encode_pb_conditional.py` | 擬似ブールエンコーディング |
| `test_idpool.py` | ID プール管理 |
| `test_warmstart.py` | ウォームスタート |
| `test_accum_stats.py` | 統計蓄積 |
| `test_formula_unique.py` / `test_unique_model.py` / `test_unique_mus.py` | ユニーク制約 |
| `test_equals1.py` | Equals1 制約 |

---

## 引用

PySAT を利用した研究成果を公開する場合は、以下の引用をお願いします:

```bibtex
@inproceedings{imms-sat18,
  author    = {Alexey Ignatiev and Antonio Morgado and Joao Marques{-}Silva},
  title     = {{PySAT:} {A} {Python} Toolkit for Prototyping with {SAT} Oracles},
  booktitle = {SAT},
  pages     = {428--437},
  year      = {2018},
  doi       = {10.1007/978-3-319-94144-8_26}
}

@inproceedings{itk-sat24,
  author    = {Alexey Ignatiev and Zi Li Tan and Christos Karamanos},
  title     = {Towards Universally Accessible {SAT} Technology},
  booktitle = {SAT},
  pages     = {4:1--4:11},
  year      = {2024},
  doi       = {10.4230/LIPICS.SAT.2024.16}
}
```

---

## ライセンス

MIT License. Copyright (c) 2018 Alexey Ignatiev, Joao Marques-Silva, Antonio Morgado.

詳細は [LICENSE.txt](LICENSE.txt) を参照してください。

---

## 参考文献・情報源

### SAT ソルバー技術

- [SAT Competitions](https://satcompetition.github.io/) — SAT ソルバー競技会
- [CaDiCaL 2.0 (CAV 2024)](https://link.springer.com/chapter/10.1007/978-3-031-65627-9_7) — CaDiCaL の最新改良
- [CaDiCaL, Kissat — SAT Competition 2025](https://cca.informatik.uni-freiburg.de/papers/BiereFallerFleuryFroleyksPollitt-SAT-Competition-2025-solvers.pdf)
- [MaxSAT Evaluations](https://maxsat-evaluations.github.io/) — MaxSAT 評価

### 機械学習 + SAT

- [SATLUTION: Autonomous SAT Solver Evolution (2025)](https://arxiv.org/abs/2509.07367)
- [NeuroBack: GNN-guided CDCL (ICLR 2024)](https://openreview.net/pdf?id=samyfu6G93)
- [ML for Solvers Tutorial (AAAI 2025)](https://ml-for-solvers.github.io/)
- [SAT-GATv2: Dynamic Attention-Based GNN](https://www.mdpi.com/2079-9292/14/3/423)

### 並列・GPU SAT

- [AlphaMapleSAT: MCTS-based Cube-and-Conquer (2024)](https://arxiv.org/abs/2401.13770)
- [ParaFROST: GPU Accelerated Inprocessing](https://github.com/muhos/ParaFROST)
- [TurboSAT: GPU-CPU Hybrid (2025)](https://arxiv.org/html/2511.07737)
