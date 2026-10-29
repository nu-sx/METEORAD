<p align="center"><img src="manual/images/meteorad_icon_256.png" width="160" alt="METEORAD icon"></p>

# METEORAD — METEOr RADiation spectra

Line-by-line emission/absorption spectra of high-temperature plasmas
(air, combustion gases, metal/oxide vapours) for the analysis of meteor,
space-debris and re-entry emission. **This repository distributes the
manuals only** — the source code is not public.

高温プラズマ（空気・燃焼ガス・金属/酸化物蒸気）の line-by-line 発光・吸収スペクトル
計算ソフトウェアです。流星・スペースデブリ・再突入カプセルの発光解析に使えます。
**このリポジトリではマニュアルのみを配布しています**（ソースコードは非公開）。

## Manual / マニュアル

| | Web | PDF | Word (.docx) |
|---|---|---|---|
| 日本語 | [仕様書・ユーザーマニュアル](manual/ja.md) | [METEORAD_Manual_ja.pdf](manual/METEORAD_Manual_ja.pdf) | [METEORAD_Manual_ja.docx](manual/METEORAD_Manual_ja.docx) |
| English | [Specification and User Manual](manual/en.md) | [METEORAD_Manual_en.pdf](manual/METEORAD_Manual_en.pdf) | [METEORAD_Manual_en.docx](manual/METEORAD_Manual_en.docx) |

Web version / Web 版: https://nu-sx.github.io/METEORAD/

## Credits / クレジット

This software is based on JAXA's SPRADIAN (SPRADIAN-1.8; K. Fujita and T. Abe,
ISAS/JAXA). Shinsuke Abe added further atoms and molecules important for
atmospheric entry and meteor emission and built the GUI.

本ソフトウェアは、JAXA の SPRADIAN に阿部新助 (Shinsuke Abe) が、大気圏突入や流星発光に
重要な他の原子・分子を追加収録し、GUI 化したソフトウェアです。

SPRADIAN at JAXA (大気突入機の熱空力評価システム): https://www.aero.jaxa.jp/research/basic/aerodynamic/aerothermodynamic/

## Citation (required) / 引用（必須）

Any use of METEORAD must cite **all three** papers in the acknowledgements/references.
METEORAD を使用した成果の公表時には、謝辞・参考文献に以下の 3 論文の引用を必須とします。

1. Fujita K, Abe T (1997) SPRADIAN, structured package for radiation analysis: theory and application. ISAS Report No. 669, pp. 1–47. Institute of Space and Astronautical Science. https://jaxa.repo.nii.ac.jp/records/31438
2. Fujita K, Abe T (2003) Development of a radiation analysis code for hypervelocity reentry applications. ISAS Report SP No. 17, pp. 69–82. Institute of Space and Astronautical Science. https://jaxa.repo.nii.ac.jp/records/33242
3. Abe S, et al (2026) C₂, CH and CN emission, surface temperature and luminous efficiency of the Hayabusa2 sample-return capsule from ground-based slitless video spectroscopy. Earth, Planets and Space.

Abe et al. (2026) describes the METEORAD extensions of SPRADIAN: the Python port,
the additional atoms/molecules for atmospheric entry and meteor emission, the GUI,
temperature sweeps, fitting of observed spectra, the Python API, etc.

## Terms / 利用条件

Use of the SPRADIAN-derived parts (program and DATAPAC, JAXA intellectual property) follows JAXA's [知的財産利用の手引き (Guide to the Use of Intellectual Property, 2024.9)](https://www.jaxa.jp/about/ip_policy/data/chizairiyou_tebiki_202409.pdf); acts requiring the right holder's permission (copying, modification, redistribution, commercial use) need a licence agreement with JAXA.
SPRADIAN 由来部分（プログラム・DATAPAC は JAXA の知的財産）の利用は「[JAXA 知的財産利用の手引き](https://www.jaxa.jp/about/ip_policy/data/chizairiyou_tebiki_202409.pdf)」に従います。
