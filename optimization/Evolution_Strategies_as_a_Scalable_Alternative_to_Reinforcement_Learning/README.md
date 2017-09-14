# Evolution Starategies as a Scalable Alternative to Reinforcement Learning

## Authors
  * Tim Salimans
  * Jonathan Ho
  * Xi Chen
  * Szymon Sidor
  * Ilya Sutskever


## Abstract

進化戦略（ES)っていうブラックボックス最適化アルゴリズムがあるんですよ。
そいでESが、[マルコフ決定過程](https://ja.wikipedia.org/wiki/%E3%83%9E%E3%83%AB%E3%82%B3%E3%83%95%E6%B1%BA%E5%AE%9A%E9%81%8E%E7%A8%8B)（MDP）を
ベースにしたQ学習や方策勾配法（Policy Gradient）みたいな教化学習の代用として使えんかなってやってみたよ。

[MuJoCO](http://www.mujoco.org/)、Atariみたいな物理シミュレータやゲームで実験してみて、進化戦略がCPUの数に対してだいぶスケーラブルで、かなり良さげなソリューションであることを示すよ。
上記は、よくある乱数使った手法をもとに新たなcommunication starategy(なんだろう、進化的オペレーションのことかな)を使ってやっていく。
今回のESの実装ではcommunicate scalarsのみが必要である（たぶん、並列実行する際に、スカラーな値だけの受け渡しで良い、ってことを言っているっぽい？）,そいで1000並列処理を実現するよ。
このことで3D humanoid walkingを10分で解けたんよ、さらにAtariも1時間の学習で良い感じの競合できる（これまでいろんな論文で出ている結果に対して？）ような結果が得られたよ。
さらに、black box optimizationとしてESの強みはまだまだあって、アクション頻度や遅延報酬に対して不変であり、非常に長いhorizon(?)に耐え、時間的な割引や価値関数の近似を必要としないー。


## Introduction

複雑で不確実なタスクを実現可能であるようなagentの開発は人工知能の目標の一つである。
上記の問題を解決するため、最近ではよくマルコフ決定過程をベースにした強化学習で関数をとくみたいなのが流行っている。
MnihらはpixelからAtariのplayを学習させた来、Ngらはヘリの空力特性を良くしたりいろいろやられている。

RLの問題を解くためにまた別の手法としてblack-box optimizationが使われる。この手法はdirect policy searchとか、neuro evolutionとかが知られていて、ニューラルネットワークに適用されている。
この論文では進化戦略の研究で、特定の最適化アルゴリズムの研究よ(?)。
そんでもって、進化戦略がちゃんとニューラルネットワークのポリシー関数の学習ができ、分散処理とかもスケーラブルに対応できる様をMujoCoやAtariをやっていける様を示していく。

この論文のkey ideaは以下に示すものっす。

1. Salimansらの仮想 batch normalization、nueral network policyの再パラメータ化が進化戦略の確実性を大幅に促進することを見つけた
2. 進化戦略がhighly paralleizeな手法であることを見っけた（せやね）
   普通の乱数に基づいた新しい伝達戦略(あとはabstractの内容といっしょ)
3. 進化戦略のデータ効率（？）もまあまあ案外良いよ。A3CのAtariの結果と同等の結果を得るのに3倍から10倍のデータを用いた。
   データ効率のわずかな低下はバックプロパゲーションやvalue functionを用いないことで削減される3倍くらいの計算リソースの削減で相殺よ！
4. 進化戦略がTRPOといったようなpolicy gradientよりも良い感じに振る舞って探索することを発見した。
   MuJoCoにおいて、進化戦略はめっちゃ広く探索してくれる。
5. 進化戦略はだいぶロバストであり、前述したとおりAtariのhyper parameterの探索にしようした。同上にMujoCoにも

black box 最適化はいくつかのめっちゃ魅力的な点がある。
 * 報酬の分布に無関心（依存しない？）
 * 勾配の逆伝播を必要としない
 * 長い探索時間への耐性がある（？）

だが、Q学習やpolycy gradientみたいな手法と比較してRL問題を解くのは苦手だと知られている。
この論文の貢献は、進化戦略がRLアルゴリズムと比較して競合可能であること、そして並列処理でスケール可能であることを示したことである。

## Evlution Strategies

進化戦略はblack box最適化に分類され、ヒューリスティックな探索が自然の進化に影響されたアルゴリズムである。
例えば、
 * iteration -> 世代(generation)
 * パラメータベクトル-> 遺伝子型（genotypes)
 * perturbed -> 突然変異(mutated)
 * 目的関数 -> 適応度
とか呼んだりする。

