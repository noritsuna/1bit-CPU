# 1bit CPU made with ASIC (ISHI-Kai's OpenMPW PTC06-1)
[naoto64氏の1bit-CPU](https://naoto64.github.io/1bit-CPU/)を[ISHI会版OpenMPW PTC06-1](https://ishi-kai.org/openmpw/shuttle/ptc06/2024/07/06/shuttle_ISHI-Kai_OpenMPW-PTC06-1_start.html)を利用して、ASIC化しました！  
サイズは、280um x 65um くらいとなりました。元の基板が85mm x 70mmですのでおよそ36万分の1に縮小できました！  

 ![1bit-CPU](https://naoto64.github.io/1bit-CPU/img/implementation-example.jpg)
 ![1bit-CPUの回路図](images/xschem_1bit-CPU.png)
 ![1bit-CPUのレイアウト](images/klayout_1bit-CPU_size.png)


# 論理回路化
まずは、74HC series logic IC部分を論理回路に起こし直して、シミュレーションを行いました。  
シミュレーションには[logicsim](http://www.cburch.com/logisim/)を利用しています。  

 ![1bit-CPUのシミュレーション回路](images/logicsim_1bit-CPU.png)


## 各種回路
本家のサンプル回路をシミュレーションしました。  


-[LED点滅回路](logicsim/LED_brink.circ)
 ![LED点滅回路](images/prog_LED_brink.png)
-[LED点灯回路](logicsim/LED_flash.circ)
 ![LED点灯回路](images/prog_LED_flash.png)
-[NOP回路](logicsim/NOP.circ)
 ![NOP回路](images/prog_NOP.png)


# AISC化
ここから、ASICにするための設計を行なっていきます。  


## 回路図
まずは、回路図を書きます。とは言っても、シミュレーション時に回路としては完成しているので、それをxschemで書き直しただけです。  

-[回路図](xschem/1bit-CPU.sch)
 ![1bit-CPUの回路図](images/xschem_1bit-CPU.png)


## レイアウト
続いて、論理回路のレイアウトをします。  
論理回路は実際には、P-FETやN-FETを組み合わせて機能が実現されています。  

 ![NAND回路のFETによる実装例](https://image.itmedia.co.jp/edn/articles/1012/01/mn_ti_digitalyougo03_05.jpg)


もちろん、P-FETやN-FETを組み合わせて、自分で論理回路のレイアウトをしても良いのですが、通常スタセル（スタンダードセル）という物が用意されています。  
要は、P-FETやN-FETを組み合わせて作られている論理回路ライブラリです。3つ目が拡大図でP-FETとN-FETで構成されているのが確認できるかと思います。  

 ![スタセル](images/klayout_logic_sc_block.png)
 ![スタセルの中身](images/klayout_logic_sc.png)
 ![スタセル拡大](images/klayout_sc_zoomin.png)


回路図そのままで、スタセルを配置してみたパターンです。  

 ![回路図のままの配置](images/klayout_logic_location.png)


### 一列化
このまま配線するだけでも良いのですが、通常のスタセルはVerilogなどから自動的にレイアウトに変換する時に使用するように作成されています。  
そのため、スタセルは自動レイアウトを考慮して、下の図のように縦の長さが固定されて、上部にVDDがあり下部にVSS(GND)があるタイル状の形状をしています。これらを横に並べて利用することで、VSSやVSSが接続されて、自動ルーティングや配置がしやすいように作られています。  

 ![スタセルの中身](images/klayout_logic_sc.png)


そのため、人間が見やすい回路図の形で配置し、配線すると無駄が多くなります。  
そこで、回路図をこのタイル型レイアウトに合わせて、一列に並ぶように書き換えます。  
（通常は、この手の論理回路で設計するロジック回路を手でレイアウトすることはあまり無いと思いますので、今回は結構特殊なシチュエーションとなります。）  

-[1ライン化回路図](xschem/1bit-CPU_1line.sch)
 ![回路図の1ライン化](images/xschem_1line.png)
 ![レイアウトの1ライン化](images/klayout_1line_block.png)
 ![レイアウトの1ライン化の中身](images/klayout_1line.png)


### 最終レイアウト
一列化して、配線したレイアウトです。  

-[レイアウト](klayout/1bit-CPU.gds)
 ![1bit-CPUのレイアウト](images/klayout_1bit-CPU.png)


## LVS
手配線のため回路図とレイアウトが間違っている可能性があるため、LVS（Layout vs Schematic）をかけるのですが、OpenRule1umのスタセルの作りが微妙のため、完全体とはなりませんでした。  
そのため、目LVSで乗り切りました。とは言っても、接続先は下の図（LVSの配線ハイライト）のように強調表示されるためそれほど大変ではありません。  

 ![1bit-CPUのLVS](images/klayout_LVS.png)
 ![LVSの配線ハイライト](images/klayout_LVS_highlight.png)


## パッドへの接続
最後に、実際の半導体上のパッド（ピン）に各ポートを接続します。  
本来ならピンの設定（INやOUT、デジタルかアナログかなどを指定する）があるのですが、そこの設定はNDAの範囲内での設定となるため、ここでは行いません。そのため、これは実質的に仮配線となります。  

 ![1bit-CPUのピンレイアウト](images/pin_layout.png)

-[パッド付きレイアウト](klayout/1bit-CPU_frame.gds)
 ![パッドフレーム付き](images/klayout_frame.png)
 ![パッドフレーム拡大](images/klayout_frame_zoomin.png)


# [本チップ用ボード](kicad/1bit-CPU_board)
メモリや外部クロック、動作確認用のLEDを搭載したボードをKiCADで設計しました。  

[本チップ用ボードディレクトリ：KiCADファイル](kicad/1bit-CPU_board)

 ![ボード：回路図](images/kicad_circuit.png)
 ![ボード：アートワーク](images/kicad_artwork.png)

## 動作確認
ボードが到着したので、部品を実装して動作確認しました。  
無事に、本家の1bit-CPUボードと同じ動きをしております！  
※クロックが微妙にずれているため、同期はしてません。

 ![ボード：実物](images/1bit-CPU_board.jpg)
[ボード：YouTube動画](https://youtube.com/shorts/-G9ZElhzAAo?feature=share)
[ボード：動画ファイル](images/1bit-CPU_board.mp4)



# [本チップ用ラズパイHAT](kicad/1bit-CPU_HAT)
ラズパイHATをKiCADで設計しました。  
ただ、本CPUは5V動作で、ラズパイは3.3V動作のため、レベル変換が必要となります。  

## インバータの応用
レベル変換の一つの手として、バッファ回路を使う手があります。  
バッファ回路＝インバータ回路*2です。そこで、今回は、みんなで実装したインバータ回路をそのまま使うことにしました。  

 ![インバータ動作図](images/HAT_inverter.png)
 ![インバータの接続図](images/HAT_pins.png)
 ![みんなのインバータ](https://github.com/ishi-kai/ISHI-KAI_Multiple_Projects_OpenMPW_PTC06-1/raw/main/Submitted/all_members_layout_using.png)


### インバータの動作確認
- [インバータ動作チェック用AD3設定ファイル](kicad/1bit-CPU_HAT/Inverter_checker.dwf3work)

 ![インバータのバッファ動作 3.3V->5V](images/inverter_3to5.png)
 ![インバータのバッファ動作 5V->3.3V](images/inverter_5to3.png)
 

## クロック源
クロック源として、りょうすさんが実装したVCOを流用しました。  

 ![VCOの動作](images/VCO_5to3.png)


### クロック源の動作確認
- [VCO動作チェック用AD3設定ファイル](kicad/1bit-CPU_HAT/VCO_checker.dwf3work)
 ![VCOの動作図](images/HAT_vco.png)


## HATの実装
チップ内にあるインバータ回路やVCOを利用したため、外部回路は3つのみとなっています。  

- LED点灯用回路
- リセット回路
- VCO用電圧制御回路

 ![ボード：回路図](images/HAT_kicad_circuit.png)
 ![ボード：アートワーク](images/HAT_kicad_artwork.png)

- [本チップ用ラズパイHATディレクトリ：KiCADファイル](kicad/1bit-CPU_HAT)


## 実行用インタープリター
ラズパイ用のプログラム実行用のインタープリターの実装です。

- [本CPU用インタープリター](kicad/1bit-CPU_HAT/program/1bit-CPU_interpreter.py)


## 実際のボード
実際にラズパイZEROに装着した状態と動作状態。

 ![ボード](images/HAT_board.jpg)
 ![ボード：LED](images/HAT_board_LED.jpg)

