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

RLの問題を解くためにまた別の手法としてblack-box optimizationが使われる。
