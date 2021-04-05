---
title: "UTS Namespace"
---

# UTS Namespace - UNIXマシンの情報を分離する

まず始めに解説するのは、UTS Namespaceです。UTS Namespaceは「分離された環境を作る」手順がわかりやすく、挙動も理解しやすいため、Namespaceの解説でしょっちゅう登場する、というのはコンテナ業界では「あるある」の話だと思います。pingのfile capabilityぐらいのあるあるです。

さっそくコマンドラインで実験を行ってみます。

## コマンドラインによる実験


## UTS Namespaceを隔離する実装

### clone(2) を用いる

### unshare(2) を用いる

### setns(2) を用いる

## UTS Namespaceに影響されるシステムコール


## カーネルコード解説: TBD
