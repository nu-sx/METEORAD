<p align="center"><img src="images/meteorad_icon_256.png" width="160" alt="METEORAD icon"></p>

<h1 align="center">METEORAD</h1>

<p align="center"><b>METEOr RADiation spectra</b><br>仕様書・ユーザーマニュアル</p>

<p align="center">SPRADIAN-1.8 (JAXA) の Python 版 — GUI 版 0.1.0<br>原典: SPRADIAN-1.8 — 藤田和央・安部隆士 (ISAS/JAXA)<br>追加原子・分子データベースおよび GUI: 阿部新助 (Shinsuke Abe)<br>文書日付: 2026-10-24</p>

> **使用時は Fujita & Abe (1997)、Fujita & Abe (2003) および Abe et al. (2026) の引用が必須です (1.4 節)。**

[English version](en.md) ・ [Word 版をダウンロード](METEORAD_Manual_ja.docx)

## 目次

- [1. 概要](#sec-1)
  - [1.1 用途](#sec-1-1)
  - [1.2 主な機能](#sec-1-2)
  - [1.3 検証状況](#sec-1-3)
  - [1.4 引用 (必須)](#sec-1-4)
- [2. 動作環境とインストール](#sec-2)
  - [2.1 Windows](#sec-2-1)
  - [2.2 macOS](#sec-2-2)
  - [2.3 データベースの保存場所](#sec-2-3)
  - [2.4 アンインストール](#sec-2-4)
- [3. 起動と画面構成](#sec-3)
- [4. 化学種パネル](#sec-4)
  - [4.1 プリセット](#sec-4-1)
- [5. パラメータパネル](#sec-5)
  - [5.1 圧力 FWHM の自動推定 (Auto)](#sec-5-1)
- [6. 計算の実行と表示](#sec-6)
  - [6.1 基本手順](#sec-6-1)
  - [6.2 プロットパネルの機能](#sec-6-2)
  - [6.3 スラブ強度](#sec-6-3)
- [7. メニュー](#sec-7)
  - [7.1 スペクトルの保存形式](#sec-7-1)
- [8. 温度スイープ](#sec-8)
- [9. 観測スペクトルのフィット](#sec-9)
  - [9.1 設定項目](#sec-9-1)
  - [9.2 探索方法](#sec-9-2)
- [10. VALD 線リストの取込み](#sec-10)
- [11. 制御ファイル (.ctl) の読込み](#sec-11)
- [12. Python API](#sec-12)
- [13. 物理モデルと仕様](#sec-13)
  - [13.1 単位系と規約](#sec-13-1)
  - [13.2 計算の流れ](#sec-13-2)
  - [13.3 線形状](#sec-13-3)
  - [13.4 装置関数の畳み込み](#sec-13-4)
- [14. データベース](#sec-14)
  - [14.1 追加分子の近似](#sec-14-1)
  - [14.2 新しい化学種の追加](#sec-14-2)
- [15. 制約と注意事項](#sec-15)
- [16. クレジットと参考文献](#sec-16)
  - [16.1 引用必須の論文](#sec-16-1)
  - [16.2 その他の参考文献](#sec-16-2)
  - [16.3 利用条件](#sec-16-3)

<a id="sec-1"></a>

## 1. 概要

**METEORAD** (METEOr RADiation spectra) は、高温プラズマの発光・吸収スペクトルを線ごと (line-by-line) に計算するソフトウェアです。JAXA の輻射解析コード **SPRADIAN-1.8** (Structured Package for Radiation Analysis; 藤田和央・安部隆士, ISAS/JAXA) を Python に移植したもので、物理モデル・データベース形式・制御ファイル形式は C 版を踏襲しています。

本ソフトウェアは、JAXA の SPRADIAN に阿部新助 (Shinsuke Abe) が、大気圏突入や流星発光に重要な他の原子・分子を追加収録し、GUI 化したソフトウェアです。

Python 化・追加原子/分子・GUI・温度スイープ・観測スペクトルのフィット・Python API などの METEORAD としての拡張は Abe et al. (2026) に記述されています (1.4 節)。

<a id="sec-1-1"></a>

### 1.1 用途

- 流星・スペースデブリのアブレーション発光の解析 (金属原子・イオン・金属酸化物)
- 再突入カプセル周りの空気プラズマの輻射解析 (N₂, O₂, NO, N, O とそのイオン)
- 燃焼ガス (C₂, CN, CH, OH, CO など) の発光解析
- 観測スペクトルへのフィットによる温度・数密度・装置分解能の推定

<a id="sec-1-2"></a>

### 1.2 主な機能

- 5 温度モデル (並進 Ttr / 回転 Trot / 振動 Tvib / 電子励起 Tex / 電子 Tel) による非平衡計算
- 束縛-束縛 (原子線・二原子分子バンド) と連続スペクトル (束縛-自由・自由-束縛・自由-自由)
- 線形状: Voigt / 矩形 / ローレンツ。圧力広がり FWHM の自動推定 (Stark + van der Waals)
- 一様スラブの視線積分強度 (自己吸収込み) と Tel の黒体放射
- 装置関数 (ガウス) による畳み込み、化学種ごとの色分け表示
- 温度スイープの一括計算と CSV 出力
- 観測スペクトルへのフィット (グリッド探索 + 線形最小二乗 + Nelder–Mead 精密化)
- VALD 線リストの取込み、SPRADIAN-1.8 の制御ファイル (.ctl) の読込み
- Python API (run_spectrum) による GUI なしの計算

<a id="sec-1-3"></a>

### 1.3 検証状況

C 版 SPRADIAN-1.8 の sample1.ctl (空気 10 種, 10000 K, 3000–3500 Å) について、スペクトルとフラックスが出力印字精度 (有効数字 5 桁) で一致することを確認しています。追加したデブリ・金属種は、全種で最強線/バンド頭の波長が文献値と一致することを確認しています (付属の tools/test_debris.py)。

<a id="sec-1-4"></a>

### 1.4 引用 (必須)

METEORAD を使用した成果 (論文・学会発表・報告書など) を公表する際は、謝辞・参考文献に**次の 3 論文の引用を必須**とします。

1. Fujita K, Abe T (1997) SPRADIAN, structured package for radiation analysis: theory and application. ISAS Report No. 669, pp. 1–47. Institute of Space and Astronautical Science. https://jaxa.repo.nii.ac.jp/records/31438
2. Fujita K, Abe T (2003) Development of a radiation analysis code for hypervelocity reentry applications. ISAS Report SP No. 17, pp. 69–82. Institute of Space and Astronautical Science. https://jaxa.repo.nii.ac.jp/records/33242
3. Abe S, et al (2026) C₂, CH and CN emission, surface temperature and luminous efficiency of the Hayabusa2 sample-return capsule from ground-based slitless video spectroscopy. Earth, Planets and Space.

Fujita and Abe (1997) は原典 SPRADIAN の理論論文、Fujita and Abe (2003) は超高速再突入向け輻射解析コードの開発論文、Abe et al. (2026) は、SPRADIAN に対する METEORAD の拡張 (Python 化、大気圏突入・流星発光に重要な原子・分子の追加、GUI、温度スイープ、観測スペクトルのフィット、Python API など) を記述した論文です。

> 謝辞の例: 「本研究のスペクトル計算には METEORAD (Fujita and Abe 1997, 2003; Abe et al. 2026) を使用した。」

<a id="sec-2"></a>

## 2. 動作環境とインストール

METEORAD は Windows 用インストーラと macOS 用ディスクイメージ (dmg) で配布しています。Python などの追加ソフトウェアは不要です。

**表 1.** 動作環境と配布ファイル

| 項目 | Windows | macOS |
|---|---|---|
| OS | Windows 10 / 11 (64 ビット) | macOS 14.4 以降、Apple Silicon (M1 以降) |
| 配布ファイル | `METEORAD-<ver>-Windows-x64-Setup.exe`<br>(ポータブル版: `METEORAD-<ver>-Windows-x64.zip`) | `METEORAD-<ver>-macOS-arm64.dmg` |
| ディスク容量 | 約 200 MB | 約 150 MB |
| コード署名 | なし (初回に SmartScreen の警告) | なし (初回に Gatekeeper の確認) |

<a id="sec-2-1"></a>

### 2.1 Windows

1. `METEORAD-<ver>-Windows-x64-Setup.exe` をダブルクリックします。
2. 「Windows によって PC が保護されました」と表示された場合は、**詳細情報** → **実行** を選びます (署名なしのため)。
3. インストールモード (自分のみ／すべてのユーザー) を選びます。すべてのユーザーを選ぶと管理者権限が必要です。
4. 利用条件を確認して同意し、インストール先 (既定: `C:\Program Files\METEORAD`) とデスクトップアイコンの有無を選んで **インストール** を押します。
5. 完了画面で「METEORAD を実行する」にチェックを入れたまま **完了** を押すと起動します。

> ポータブル版 (zip) を使う場合は、任意のフォルダに展開して `METEORAD\METEORAD.exe` を実行します。インストールは不要です。

<a id="sec-2-2"></a>

### 2.2 macOS

1. `METEORAD-<ver>-macOS-arm64.dmg` をダブルクリックして開きます。
2. **METEORAD.app** を同じウィンドウ内の **Applications** フォルダへドラッグします。
3. アプリケーションフォルダ (または Launchpad) から METEORAD を開きます。初回は「開発元を検証できない」旨のダイアログが表示されるので **完了** (または **OK**) で閉じます。
4. **システム設定** → **プライバシーとセキュリティ** を開き、下部の「“METEORAD”は…ブロックされました」の横にある **このまま開く** を押して、表示される確認で再度 **開く** を選びます (2 回目以降は不要)。
5. dmg は Finder のサイドバーで取り出してかまいません。

<a id="sec-2-3"></a>

### 2.3 データベースの保存場所

起動時に次の 2 つのデータベースが走査され、化学種パネルに表示されます。

**表 2.** データベースの場所

| グループ | 場所 | 内容 |
|---|---|---|
| Air/Combustion | アプリ内 (読み取り専用) | 原典 SPRADIAN DATAPAC の空気・燃焼系 23 種 |
| Debris/Metal | Windows: `%APPDATA%\METEORAD\database_debris`<br>macOS: `~/Library/Application Support/METEORAD/database_debris` | デブリ・金属種 20 種と VALD 取込み (10 章) で作成した種 |

> Debris/Metal 用フォルダは初回起動時に自動で作成され、同梱のデータベースがコピーされます。アプリを新しい版に更新すると、同梱ファイルと同じ名前のファイルは新しい版で上書きされます (それ以外のファイルは残ります)。同梱種を VALD 取込みで置き換えた場合などは、別名で保存するかバックアップを取ってください。フォルダの内容を変更したあとは **Tools → Rescan databases** で一覧を更新できます。

<a id="sec-2-4"></a>

### 2.4 アンインストール

- Windows: **設定 → アプリ → インストールされているアプリ** で METEORAD を選び **アンインストール** します。
- macOS: アプリケーションフォルダの METEORAD.app をゴミ箱に入れます。
- どちらの場合もユーザーデータ (上表の Debris/Metal フォルダと設定) は残ります。不要なら `METEORAD` フォルダごと削除してください。

<a id="sec-3"></a>

## 3. 起動と画面構成

- Windows: スタートメニュー (またはデスクトップ) の **METEORAD** から起動します。
- macOS: アプリケーションフォルダまたは Launchpad の **METEORAD** から起動します。
- SPRADIAN-1.8 の制御ファイル (.ctl) は、起動後に **File → Load control file (.ctl)** で読み込めます (11 章)。

コマンドラインから起動すると、.ctl を読み込んだ状態で開始したり、版を確認したりできます。

```text
# Windows (PowerShell)
& "C:\Program Files\METEORAD\METEORAD.exe" sample1.ctl

# macOS (ターミナル)
/Applications/METEORAD.app/Contents/MacOS/METEORAD sample1.ctl
/Applications/METEORAD.app/Contents/MacOS/METEORAD --version
```

メインウィンドウは次の 4 つの領域で構成されます (図 1)。

<p align="center"><img src="images/01_main_empty.png" width="640" alt="起動直後のメインウィンドウ"></p>

<p align="center"><b>図 1.</b> 起動直後のメインウィンドウ</p>

**表 3.** メインウィンドウの領域

| 領域 | 位置 | 役割 |
|---|---|---|
| 化学種パネル | 左 | 計算に含める化学種と数密度・表示色の設定、プリセット選択 |
| パラメータパネル | 右上 | 波長範囲・格子間隔・線形状・圧力広がり・温度・スラブ長・連続光、Run/Cancel |
| プロットパネル | 右下 | 計算結果の表示 (表示量・対数軸・種別表示・分解能・色) |
| ステータスバー | 下端 | 進行状況、計算時間、電子密度 Ne・全数密度、クリック位置の値 |

<a id="sec-4"></a>

## 4. 化学種パネル

<p align="center"><img src="images/08_species_panel.png" width="330" alt="化学種パネル (Meteor ablation 4500K プリセット適用後)"></p>

<p align="center"><b>図 2.</b> 化学種パネル (Meteor ablation 4500K プリセット適用後)</p>

- 化学種は **Air/Combustion** (原典 DATAPAC) と **Debris/Metal** (database_debris) の 2 グループに分かれて表示されます。
- チェックした化学種が計算に含まれます。
- **Density [m^-3]** 列をダブルクリックすると数密度を編集できます (例: `1.5e18`)。
- **Color** 列をダブルクリックすると、その化学種のプロット色を選べます。自動色に戻すには右クリック →「Reset to auto color」、Delete キー、または **Auto colors** ボタンを使います。色の選択はセッションをまたいで保存されます。
- **Select all** / **Clear** で全化学種の選択・解除ができます。
- チェックや密度を手で変更すると、プリセット表示は自動的に **Custom** になります。

<a id="sec-4-1"></a>

### 4.1 プリセット

Preset ボックスで代表的な組成と温度 (5 温度すべてに同じ値) を一括設定できます。

**表 4.** プリセット

| プリセット | 温度 | 組成 [m⁻³] |
|---|---|---|
| Air plasma 10000K | 10000 K | sample1.ctl の平衡空気: NI 5.511e23, OI 1.476e23, NII 1.462e22, OII 2.534e21, N2 1.976e21, NO+ 6.15e19, NO 5.875e19, N2+ 2.723e19, O2 8.927e17, O2+ 8.847e16 |
| Debris ablation 4000K | 4000 K | NaI 1e19, MgI 5e18, FeI 5e18, AlI 2e18, TiI 1e18, NiI 1e18, CuI 1e18, MgO 5e17, AlO 2e17, TiO 1e17, FeO 1e17 |
| Meteor ablation 4500K | 4500 K | CI コンドライト元素比を Si = 2e19 に規格化: SiI 2e19, MgI 2.1e19, FeI 1.8e19, AlI 1.7e18, NaI 1.1e18, NiI 1e18, CaI 6e17, CaII 6e17, CrI 2.6e17, MnI 1.9e17, KI 7.4e16, TiI 4.8e16 |
| Custom | — | 手動設定 (プリセットを適用しない) |

> プリセットの組成は代表値です。実際の解析では観測条件に合わせて数密度を設定してください。

<a id="sec-5"></a>

## 5. パラメータパネル

<p align="center"><img src="images/07_param_multi_temperature.png" width="640" alt="パラメータパネル (多温度モード・スラブ強度有効)"></p>

<p align="center"><b>図 3.</b> パラメータパネル (多温度モード・スラブ強度有効)</p>

**表 5.** パラメータ一覧

| 項目 | 単位 | 説明 |
|---|---|---|
| Wavelength min / max | Å | 計算する波長範囲。計算済み範囲の内側に変更するとズームのみ、外側に広げると自動で再計算します。 |
| Interval | Å | 波長格子の間隔 |
| Line shape | — | Voigt / Rectangular (矩形) / Lorentzian |
| Pressure FWHM | Å | 圧力広がり (ローレンツ成分) の半値全幅。ドップラー幅は Ttr から自動計算されます。 |
| Auto | — | 選択中の密度・温度から圧力 FWHM を推定 (5.1 節) |
| Temperature | K | 平衡計算の温度 (5 温度すべてに適用) |
| Multi-temperature | K | チェックすると Ttr, Trot, Tvib, Tex, Tel を個別に設定 |
| Slab intensity, length | m | 一様スラブの視線積分強度を計算 (6.3 節) |
| Include continua (bf/fb/ff) | — | 束縛-自由・自由-束縛・自由-自由の連続スペクトルを含める |
| Run / Cancel | — | 計算の開始・中断 (化学種または計算段階の区切りで停止) |

**表 6.** 5 温度モデル

| 温度 | 意味 | 主に支配する量 |
|---|---|---|
| Ttr | 並進温度 | ドップラー幅、van der Waals 幅 |
| Trot | 回転温度 | 分子の回転準位分布 (バンドの形) |
| Tvib | 振動温度 | 分子の振動準位分布 (バンド列の強度比) |
| Tex | 電子励起温度 | 原子・分子の電子準位分布 (線強度) |
| Tel | 電子温度 | 電離・連続光、黒体放射の温度 |

<a id="sec-5-1"></a>

### 5.1 圧力 FWHM の自動推定 (Auto)

Auto ボタンは、波長範囲の中心 λ における Stark 幅と van der Waals 幅の和を推定します。電子密度 Ne はイオン種の密度から電荷中性条件で求め、中性種の密度 N₀ を van der Waals 項に用います。

```text
FWHM = Stark + vdW
Stark ∝ λ² · Ne,      vdW ∝ λ² · (N₀ / N_ref) · (Ttr / 296 K)^0.3
```

> 推定値は桁の目安です。精密な解析では、観測からフィット (第 9 章) するか文献値を用いてください。イオン種を選択していない場合 Stark 項は 0 になります。

<a id="sec-6"></a>

## 6. 計算の実行と表示

<a id="sec-6-1"></a>

### 6.1 基本手順

1. 化学種パネルでプリセットを選ぶか、化学種にチェックを入れて数密度を入力する。
2. パラメータパネルで波長範囲・格子間隔・温度などを設定する。
3. **Run** を押す。計算はバックグラウンドで行われ、データベースは実行間でキャッシュされます。
4. 結果がプロットパネルに表示され、ステータスバーに計算時間、Ne、全数密度、格子点数が表示されます。

<p align="center"><img src="images/02_air_plasma.png" width="640" alt="空気プラズマ 10000 K (3000–4500 Å, 対数表示, 支配的な化学種で色分け)"></p>

<p align="center"><b>図 4.</b> 空気プラズマ 10000 K (3000–4500 Å, 対数表示, 支配的な化学種で色分け)</p>

<a id="sec-6-2"></a>

### 6.2 プロットパネルの機能

**表 7.** プロットパネルのコントロール

| コントロール | 機能 |
|---|---|
| Plot | 表示量: Emission (発光係数) / Absorption (吸収係数) / Slab intensity (スラブ強度) / Blackbody (Tel) |
| Log Y | 縦軸を対数表示 |
| Per-species | 化学種ごとの発光を各色で重ね描き (全体は淡色) |
| Color by species | 各波長で線発光の 50 % 以上を占める化学種の色で全体スペクトルを描く。連続光や混在領域は Color ボタンの色。 |
| Resolution FWHM | 表示スペクトルをガウス型装置関数 (面積保存) で畳み込む。保存 CSV にも畳み込み列が追加されます。 |
| Color | スペクトル線の色。メニューの Reset to default で既定色に戻す。 |
| Observed | フィット後に観測スペクトルを重ね描き (第 9 章) |
| ツールバー | matplotlib 標準: ホーム・戻る/進む・パン・ズーム・余白調整・軸設定・画像保存 |
| クリック | 最寄り格子点の波長・値と支配的な化学種をステータスバーに表示 |

<p align="center"><img src="images/03_debris_color_by_species.png" width="640" alt="Debris ablation 4000K (4000–6500 Å, 分解能 5 Å, 化学種で色分け, 対数表示)"></p>

<p align="center"><b>図 5.</b> Debris ablation 4000K (4000–6500 Å, 分解能 5 Å, 化学種で色分け, 対数表示)</p>

<p align="center"><img src="images/04_debris_per_species.png" width="640" alt="同じ計算の Per-species 表示 (化学種ごとの発光を重ね描き)"></p>

<p align="center"><b>図 6.</b> 同じ計算の Per-species 表示 (化学種ごとの発光を重ね描き)</p>

<a id="sec-6-3"></a>

### 6.3 スラブ強度

Slab intensity をチェックしてスラブ長 L [m] を与えると、一様な厚さ L のプラズマ層から出射する強度を自己吸収込みで計算します (C 版 1D-tool/Self-abs と同じ式)。

```text
I(λ) = ε(λ) / κ(λ) · [1 − exp(−κ(λ) L)]
  光学的に薄い (κL ≪ 1): I → ε L      光学的に厚い (κL ≫ 1): I → ε/κ (= 黒体 B(Tex) で上限)
```

<p align="center"><img src="images/06_meteor_slab_intensity.png" width="640" alt="Meteor ablation 4500K のスラブ強度 (L = 0.1 m, 分解能 5 Å)"></p>

<p align="center"><b>図 7.</b> Meteor ablation 4500K のスラブ強度 (L = 0.1 m, 分解能 5 Å)</p>

<a id="sec-7"></a>

## 7. メニュー

<p align="center"><img src="images/09_menu_0_File.png" width="300" alt="File メニュー"></p>

<p align="center"><b>図 8.</b> File メニュー</p>

**表 8.** メニュー項目

| メニュー | 項目 | 機能 |
|---|---|---|
| File | Load control file (.ctl)... | SPRADIAN-1.8 のジョブ制御ファイルを読み込みフォームに反映 (第 11 章) |
| File | Save spectrum as CSV/DAT... | スペクトルを CSV (カンマ区切り) または DAT (空白区切り) で保存 (7.1 節) |
| File | Save plot as PNG... | 現在のプロットを PNG 画像で保存 |
| File | Temperature sweep → CSV... | 温度を振った一括計算 (第 8 章) |
| File | Fit observed spectrum (parameter search)... | 観測スペクトルへのフィット (第 9 章) |
| Tools | Import VALD line list... | VALD 抽出ファイルを .dat に変換して取込み (第 10 章) |
| Tools | Rescan databases | データベースディレクトリを再走査して種の一覧を更新 |
| Help | About | バージョン・クレジット・単位系の表示 |

<a id="sec-7-1"></a>

### 7.1 スペクトルの保存形式

拡張子 .csv ではカンマ区切りでヘッダ 1 行、それ以外は空白区切りで `#` 付きヘッダになります。数値は `%.6e` 形式です。

**表 9.** スペクトル CSV/DAT の列

| 列名 | 単位 | 出力条件 |
|---|---|---|
| wavelength_A | Å | 常に |
| emission_W/(A.m3.sr) | W Å⁻¹ m⁻³ sr⁻¹ | 常に |
| absorption_1/m | m⁻¹ | 常に |
| intensity_W/(A.m2.sr) | W Å⁻¹ m⁻² sr⁻¹ | スラブ強度を計算したとき |
| blackbody_W/(A.m2.sr) | W Å⁻¹ m⁻² sr⁻¹ | 常に (Tel の黒体) |
| emission_conv&lt;F>A / intensity_conv&lt;F>A | 同上 | Resolution FWHM = F が有効なとき |

<p align="center"><img src="images/12_about.png" width="440" alt="About ダイアログ (クレジットと単位系)"></p>

<p align="center"><b>図 9.</b> About ダイアログ (クレジットと単位系)</p>

<a id="sec-8"></a>

## 8. 温度スイープ

File → Temperature sweep → CSV... は、メインウィンドウの他のパラメータ (波長範囲・格子・線形状・圧力 FWHM・スラブ・連続光) を固定し、チェックした化学種 1 種またはその混合物について温度を振ったスペクトルを一括計算して 1 つの CSV に保存します。

<p align="center"><img src="images/10_sweep_equilibrium.png" width="520" alt="温度スイープ ダイアログ (平衡モード)"></p>

<p align="center"><b>図 10.</b> 温度スイープ ダイアログ (平衡モード)</p>

- **Equilibrium**: 1 つの温度 T を start / end / step で振り、5 温度すべてに適用します (原子種の既定)。
- **Multi-temperature**: Ttr, Trot, Tvib, Tex, Tel を個別に固定または掃引します。掃引した温度の組合せは **Grid** (全組合せ) または **Linked** (同じ番号どうしを同時に進める; 点数が等しい必要あり) を選べます。分子種の既定は Trot + Tvib の連動掃引です。
- **Quantity**: Emission / Absorption coefficient / Slab intensity (スラブ強度はメイン画面で Slab intensity を有効にしたときのみ選択可)。
- **Convolve, FWHM**: 各スペクトルをガウス型装置関数で畳み込んでから保存します。

<p align="center"><img src="images/11_sweep_multi.png" width="520" alt="温度スイープ ダイアログ (多温度モード)"></p>

<p align="center"><b>図 11.</b> 温度スイープ ダイアログ (多温度モード)</p>

出力 CSV は `#` で始まるメタデータ行の後に、`wavelength_A` と温度セットごとの列 (例: `T=5000K`, `Trot=3000K Tvib=4000K`) が並びます。pandas では次のように読めます。

```text
import pandas as pd
df = pd.read_csv("sweep.csv", comment="#")
```

<a id="sec-9"></a>

## 9. 観測スペクトルのフィット

File → Fit observed spectrum... で開くフィット ウィンドウは、観測スペクトルを読み込み、メインウィンドウでチェックした化学種について温度・数密度・圧力 FWHM・装置分解能を探索します。波長間隔・線形状・スラブ長・連続光は Run search 時のメインウィンドウの値を使います。

<p align="center"><img src="images/13_fit_setup.png" width="640" alt="フィット ウィンドウ: 観測スペクトルの読込みと探索条件の設定"></p>

<p align="center"><b>図 12.</b> フィット ウィンドウ: 観測スペクトルの読込みと探索条件の設定</p>

<a id="sec-9-1"></a>

### 9.1 設定項目

**表 10.** フィット ウィンドウの設定

| グループ | 項目 | 説明 |
|---|---|---|
| Observed spectrum | Load... / λ column / value column / λ unit | テキスト/CSV の観測データ。列番号 (1 始まり) と波長単位 (A, nm, um) を指定 |
| Observed spectrum | fit window [A] | フィットに使う波長範囲 |
| Species | Density mode | Fit (linear): 線形最小二乗で決定 (発光モデルのみ) / Grid (log): min–max を対数等間隔で探索 / Fixed: 固定 |
| Temperatures | Equilibrium / Multi-temperature | 温度スイープと同じ指定方法 (第 8 章) |
| Broadening | Pressure FWHM | Fixed / Range (探索) / Auto (各点で物理推定) |
| Broadening | Resolution FWHM | None / Fixed / Range (探索) |
| Model | Model | Emission (光学的に薄い・高速) / Slab intensity (自己吸収込み・点ごとに完全計算) |
| Model | Observed units | Arbitrary: スケール係数を推定 (密度は比のみ決まる) / Absolute: 観測がモデル単位 |
| Model | Baseline | None / Constant / Linear (迷光・モデル外の連続光を同時推定) |
| Model | Refine the best point | グリッド最良点から Nelder–Mead 法で連続最適化 |

<a id="sec-9-2"></a>

### 9.2 探索方法

- **Emission モデル**: (温度セット, 圧力 FWHM) ごとに参照密度で 1 回計算し、各化学種の線発光を基底ベクトルとして密度でスケールします。連続光 (全体 − 線) は非負の振幅をもつ追加基底です。
- **Slab intensity モデル**: (温度セット, 圧力 FWHM, 密度グリッドの組合せ) ごとに完全計算します。
- 評価関数はフィット範囲の残差二乗和で、相対 RMS = √(Σr² / Σobs²) として表示します。
- 計算量の上限: 完全計算 2000 回、線形解 500000 回。上位 50 解がモデル配列付きで保持されます。

<p align="center"><img src="images/14_fit_result.png" width="640" alt="フィット結果: 観測 (黒)・モデル (赤)・成分・残差と評価点の一覧"></p>

<p align="center"><b>図 13.</b> フィット結果: 観測 (黒)・モデル (赤)・成分・残差と評価点の一覧</p>

図 13 の例は、MgI + FeI + NaI (4300 K, Mg/Fe = 1.33, 分解能 2 Å) から合成しノイズを加えた 5100–5300 Å のスペクトルを、MgI と FeI でフィットしたものです。T = 4290 K、Mg/Fe = 1.36、分解能 2.00 Å が復元されています (相対 RMS 0.075 はノイズ由来)。

結果表の行を選ぶとその解が表示されます。**Apply to main window** は選択した解の温度・密度・圧力 FWHM・分解能をメインウィンドウに設定して再計算し、観測スペクトルを重ね描きします (図 14)。**Save results CSV** は評価した全点のパラメータと誤差を、**Save model CSV** は選択解の観測・モデル・残差・各成分を観測波長で保存します。

<p align="center"><img src="images/15_fit_applied_main.png" width="640" alt="Apply to main window の結果 (観測スペクトルを重ね描き)"></p>

<p align="center"><b>図 14.</b> Apply to main window の結果 (観測スペクトルを重ね描き)</p>

> Observed units が Arbitrary の場合、数密度は比のみが意味を持ちます (表示値は参照密度に対する相対値)。絶対密度を求めるには絶対較正した観測と Absolute を使ってください。

<a id="sec-10"></a>

## 10. VALD 線リストの取込み

Tools → Import VALD line list... は、VALD3 (https://www.astro.uu.se/valdwiki/) の長形式抽出ファイルを SPRADIAN 形式の原子 .dat に変換し、Debris/Metal 用フォルダ (2.3 節) に保存します。

1. VALD に登録し、対象元素・電離段階の Extract Element (long format) を取得する。
2. Import VALD line list... で抽出ファイル (.vald, .txt, .gz) を選ぶ。
3. ファイル内の化学種一覧から取り込む種を選ぶ。
4. 準位表と線リストが自動生成され、種の一覧が再走査されます。ステータスバーに準位数・線数が表示されます。

```text
# 変換式 (検証済み)
A_ul [s^-1] = 6.6702e15 · gf / (g_u · λ_Å²)
Ae [W/sr]   = 1e10 · h c · A_ul / (4π λ_Å)
Ak [m² Å]   = A_ul · λ_Å⁴ · 1e-30 / (8π c g_l)
```

> VALD のデータを用いた結果を公表する際は、VALD の引用規定に従ってください。

<a id="sec-11"></a>

## 11. 制御ファイル (.ctl) の読込み

File → Load control file (.ctl)... で SPRADIAN-1.8 のジョブ制御ファイルを読み込み、波長範囲・格子間隔・線形状・圧力 FWHM・温度・化学種の密度をフォームに反映します。フォームで表現できない項目は、読込み後のダイアログで通知されます。

**表 11.** .ctl 読込みの制約

| 制御ファイルの指定 | GUI での扱い |
|---|---|
| 外部ファイルの密度 | 読み込まない (手入力が必要) |
| 外部ファイルの温度 | 読み込まない (手入力が必要) |
| 種ごとの温度セット (multi-control) | 最初のセットを全種に適用 |
| 波長ファイル指定の格子 | min/max と既定の間隔を使用 |
| 圧力 FWHM 'auto' | 0.05 Å に置換 |
| データベースの無い化学種 | スキップ |

<a id="sec-12"></a>

## 12. Python API

計算コア `meteorad` は numpy のみで動作し、GUI なしでスクリプトから利用できます。

```text
from meteorad.runner import run_spectrum
from meteorad.models import Temperature

res = run_spectrum(
    ["NI", "OI"], [5.511e23, 1.476e23],
    Temperature(tr=1e4, ro=1e4, vi=1e4, ex=1e4, el=1e4),
    wl_min=7000.0, wl_max=8000.0, interval=0.5,
    db_dir="../SPRADIAN-1.8/DATAPAC/database",
    slab_length=0.1,          # 任意: 一様スラブ強度
)
# res.wavelengths [Å], res.emission [W/(Å m³ sr)], res.absorption [1/m],
# res.intensity [W/(Å m² sr)], res.blackbody, res.per_species_emis, ...

from meteorad.control import read_control_file   # 旧形式ジョブファイル
```

**表 12.** 計算コアのモジュール

| モジュール | 内容 (C 版の対応) |
|---|---|
| database.py | load_species(): .dat 固定桁パーサ (Read_Data) |
| control.py | read_control_file(): .ctl パーサ (Read_Parm) |
| population.py | 占有数・分配関数 (Gene_Popu, InptThrm) |
| wavegrid.py / lineshape.py | 波長格子、Voigt (Whiting 近似)・矩形・ローレンツ線形状 |
| bound_bound.py | 束縛-束縛 線スペクトル (BounBoun) |
| continuum.py | 束縛-自由・自由-束縛・自由-自由 (BounFree/FreeBoun/FreeFree) |
| spectrum.py | 合成・黒体・スラブ強度・保存 (Gene_Spec, Gene_Blck, Self-abs) |
| runner.py | run_spectrum(): 高水準 API (CalSpec main) |
| vald.py | VALD 長形式 → .dat 変換 |

<a id="sec-13"></a>

## 13. 物理モデルと仕様

<a id="sec-13-1"></a>

### 13.1 単位系と規約

**表 13.** 単位

| 量 | 単位 |
|---|---|
| 波長 | Å |
| 発光係数 ε | W Å⁻¹ m⁻³ sr⁻¹ |
| 吸収係数 κ | m⁻¹ |
| 強度・黒体放射 | W Å⁻¹ m⁻² sr⁻¹ |
| 数密度 | m⁻³ |
| 温度 | K |
| .dat 内のエネルギー | K (= cm⁻¹ × 1.4387687) |

> 波長の規約は原典と同じく混在しています: 原子線リストは 2000 Å 以上で**空気中波長**、分子バンドの線は**真空波長** (λ = K_LAMBDA_CONV / E) です。

<a id="sec-13-2"></a>

### 13.2 計算の流れ

run_spectrum() は C 版 CalSpec の main と同じ順序で処理します。

```text
データベース読込 (.dat) → 占有数 (5 温度) → 束縛-束縛 → 束縛-自由 / 自由-束縛 / 自由-自由
→ スペクトル合成 (ε, κ) → 黒体 B(Tel) → [スラブ強度] → 出力
```

<a id="sec-13-3"></a>

### 13.3 線形状

- **Voigt**: ドップラー幅 (Ttr と質量から) と圧力 FWHM のローレンツ幅を Whiting 近似で合成し、格子点で評価します。
- **Rectangular / Lorentzian**: 格子区間で平均化します (最終要素は未使用)。分子バンドを粗い格子で表示する場合は矩形が適します。
- Voigt 線の幅が格子間隔より狭いと、線の間引き (エイリアシング) が起こります。格子間隔は線幅の 1/2 以下にするか、分解能の畳み込みを併用してください。

<a id="sec-13-4"></a>

### 13.4 装置関数の畳み込み

Resolution FWHM は単位面積のガウス関数による畳み込みで、積分フラックスを保存します。格子端ではカーネルを格子内の部分で再規格化するため、端での落ち込みはありません。

<a id="sec-14"></a>

## 14. データベース

**表 14.** 収録化学種

| グループ | 化学種 |
|---|---|
| Air/Combustion (原典 DATAPAC, 23 種) | ArI, C2, CH, CI, CII, CN, CO, CO+, H2, HI, HII, N2, N2+, NH, NI, NII, NO, NO+, O2, O2+, OH, OI, OII |
| Debris/Metal — 中性原子 (13) | NaI, MgI, AlI, SiI, KI, CaI, TiI, CrI, MnI, FeI, NiI, CuI, LiI |
| Debris/Metal — イオン (3) | CaII, MgII, SiII |
| Debris/Metal — 金属酸化物 (4) | MgO (B–X 緑色系), AlO (B–X 青緑系), TiO (A–X γ 系, C–X α 系), FeO (橙色系, 包絡線近似) |

<a id="sec-14-1"></a>

### 14.1 追加分子の近似

- **AlO B²Σ⁺–X²Σ⁺**: ²Σ 状態を S = 0 の一重項として記述 (スピン分裂 ~0.007 cm⁻¹ は格子より十分小さい)。全状態に同じ因子がかかるため分配関数で相殺されます。
- **TiO**: X³Δ / A³Φ / C³Δ を ³Π (結合ケース 23) で近似。縮退度 6 は実際の値と一致しますが、γ 系の 3 本のバンド頭 (7054/7089/7126 Å) は 1 本 (~7125 Å) にまとまります。
- **FeO 橙色系**: 多状態が重なる複雑な系を、Te = 17050 cm⁻¹ の 1 状態 (S = 2, Λ = 0) で包絡線として表現。回転線の解析には使えません。
- **Franck–Condon 因子**: 各状態の分子定数 (we, wexe, Be) と換算質量から Morse ポテンシャルを作り、振動波動関数の重なりを数値計算 (2026-10-23 改訂)。

**表 15.** Morse 振動子による Franck–Condon 因子 (A_total は v′=0 の全遷移率 1/τ₀)

| バンド系 | rₑ′ / rₑ″ (Å) | q00 | q01 | q10 | A_total (s⁻¹) |
|---|---|---|---|---|---|
| MgO B–X | 1.732 / 1.743 | 0.985 | 0.015 | 0.015 | 2.0e7 |
| AlO B–X | 1.667 / 1.618 | 0.730 | 0.238 | 0.225 | 1.0e7 |
| TiO γ (A–X) | 1.665 / 1.620 | 0.721 | 0.236 | 0.235 | 5.0e6 |
| TiO α (C–X) | 1.694 / 1.620 | 0.407 | 0.378 | 0.348 | 1.6e7 |
| FeO orange | 1.630 / 1.620 | 0.984 | 0.016 | 0.015 | 3.0e6 |

<a id="sec-14-2"></a>

### 14.2 新しい化学種の追加

- 原子: VALD 取込み (第 10 章) または `tools/make_atomic_dat.py` (NIST の準位・A 値を記述)。
- 二原子分子: `tools/make_diatomic_dat.py` に電子状態の分子定数とバンド系を追加。FCF は自動計算されます。
- 生成後は `tools/test_debris.py` で読込み・ピーク位置を確認してください。形式の詳細は docs/DATA_FORMATS_ja.md と notes/PORT_NOTES_database.md を参照。

<a id="sec-15"></a>

## 15. 制約と注意事項

- 追加したデブリ原子の .dat には束縛-自由 / 自由-束縛のブロックが無く、これらの種の再結合連続光は含まれません (自由-自由は有効電荷 EffC2 = 0.01 の概算)。
- 一部の A 値は log gf からの換算で ±10–30 % の不確かさがあります。精密な解析には VALD 取込みでの置換を推奨します。
- 内部分配関数の Qe 表は 1500–7000 K 向けです。約 8000 K を超えると準位の打ち切りにより不完全になります。Tex ≥ 5000 K では Qe 表参照に切り替わります。
- TiO・FeO は第 14.1 節の近似を含みます。FCF の Morse 近似は低い v で数 % の精度、高い v′ ではより大きな誤差を持ちえます。
- 三原子分子 (CO₂ など) の経路は未実装です。
- Python 版は C 版の 1D/2D 輻射輸送ツール (Ray-integ, 1D-slab, S-cap, 3D-integ) を含みません (一様スラブのみ)。

<a id="sec-16"></a>

## 16. クレジットと参考文献

本ソフトウェアは、JAXA の SPRADIAN に阿部新助 (Shinsuke Abe) が、大気圏突入や流星発光に重要な他の原子・分子を追加収録し、GUI 化したソフトウェアです。原典 SPRADIAN-1.8 の著作者は藤田和央・安部隆士 (ISAS/JAXA) です。

<a id="sec-16-1"></a>

### 16.1 引用必須の論文

METEORAD の使用時は、謝辞・参考文献に以下の 3 論文を必ず引用してください。

1. Fujita K, Abe T (1997) SPRADIAN, structured package for radiation analysis: theory and application. ISAS Report No. 669, pp. 1–47. Institute of Space and Astronautical Science. https://jaxa.repo.nii.ac.jp/records/31438
2. Fujita K, Abe T (2003) Development of a radiation analysis code for hypervelocity reentry applications. ISAS Report SP No. 17, pp. 69–82. Institute of Space and Astronautical Science. https://jaxa.repo.nii.ac.jp/records/33242
3. Abe S, et al (2026) C₂, CH and CN emission, surface temperature and luminous efficiency of the Hayabusa2 sample-return capsule from ground-based slitless video spectroscopy. Earth, Planets and Space.

Fujita and Abe (1997) は原典 SPRADIAN の理論論文、Fujita and Abe (2003) は超高速再突入向け輻射解析コードの開発論文、Abe et al. (2026) は、SPRADIAN に対する METEORAD の拡張 (Python 化、大気圏突入・流星発光に重要な原子・分子の追加、GUI、温度スイープ、観測スペクトルのフィット、Python API など) を記述した論文です。

<a id="sec-16-2"></a>

### 16.2 その他の参考文献

- JAXA 航空技術部門「大気突入機の熱空力評価システムの高度化」(SPRADIAN): https://www.aero.jaxa.jp/research/basic/aerodynamic/aerothermodynamic/
- SPRADIAN-1.5 ユーザーズマニュアル (第 2.3 版), ISAS/JAXA
- S. Abe, N. Ebizuka, H. Yano, J. Watanabe and J. Borovička, Detection of the N₂⁺ First Negative System in a Bright Leonid Fireball, ApJ 618, L141 (2005)
- Huber, K. P. & Herzberg, G., Constants of Diatomic Molecules (分子定数)
- NIST Atomic Spectra Database; VALD3 (https://www.astro.uu.se/valdwiki/)

<a id="sec-16-3"></a>

### 16.3 利用条件

SPRADIAN に由来するプログラムと DATAPAC は JAXA の知的財産です。複製・改変・再配布・商用利用など権利者の許諾を要する利用は「JAXA 知的財産利用の手引き」(2024.9 版) に従い、JAXA との利用許諾契約が必要です: https://www.jaxa.jp/about/ip_policy/data/chizairiyou_tebiki_202409.pdf

詳細は配布物に同梱の TERMS_ja.txt / TERMS_en.txt を参照してください。

