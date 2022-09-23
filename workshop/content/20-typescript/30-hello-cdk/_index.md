+++
title = "Hello, CDK!"
chapter = true
weight = 30
+++

# Hello, CDK!

この章では、CDKコードを書いていきます。
サンプルアプリにあるSNS / SQSの代わりに、API Gatewayのエンドポイントを持つLambda関数を追加します。

エンドポイントの任意のパスにアクセスするとレスポンスを受け取ることができるようになります。

![](/images/hello-arch.png)

はじめに、サンプルコードをクリーンアップしましょう。
