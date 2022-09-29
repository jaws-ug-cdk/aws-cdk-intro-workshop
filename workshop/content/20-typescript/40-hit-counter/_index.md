+++
title = "コンストラクトの作成"
chapter = true
weight = 40
+++

# コンストラクトの作成

この章では、`HitCounter`と呼ばれる新しいコンストラクトを定義します。
このコンストラクトは、API Gatewayバックエンドとして使用されるLambda関数にアタッチでき、
各URLのパスに発行されたリクエストの数をカウントします。
この結果をDynamoDBテーブルに保存します。

![](/aws-cdk-intro-workshop/images/hit-counter.png)
