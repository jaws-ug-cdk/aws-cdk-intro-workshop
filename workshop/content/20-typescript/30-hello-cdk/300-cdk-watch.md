+++
title = "CDK Watch"
weight = 300
+++

## より高速なデプロイ

{{% notice info %}}
このセクションはワークショップを完了するために必要ではありませんが、
`cdk deploy --hotswap` と `cdk watch` がどのようにデプロイを高速化するのか見てみましょう。
{{% /notice %}}

lambdaが動作するようになりましたね！
しかし、もしlambdaのコードを微調整して正しく動作させたい場合はどうしたらよいでしょうか？
例えば、lambda関数を「`Hello, CDK`」ではなく「`Good Morning, CDK!`」と応答させることに決めたとしましょう。

今のところ、スタックを更新するために使えるツールは `cdk deploy` しかないように思えます。
しかし、`cdk deploy`には時間がかかります。
CloudFormationスタックをデプロイして、`lambda`ディレクトリをbootstrapバケットにアップロードしなければならないからです。
lambdaのコードを変更するだけならCloudFormationスタックを更新する必要はないので、`cdk deploy`の部分は無駄な労力となります。

本当に必要なのは、lambdaコードの更新だけなのです。
それだけを行うための他のメカニズムがあれば最高なのですが...。

## `cdk deploy` にかかる時間を測ってみる

まず、`cdk deploy`を実行するのにかかる時間を計ってみましょう。
これは、CloudFormationのフルデプロイにどれくらい時間がかかるかの基準値になります。
そのために、`lambda/hello.ts` 内のコードを変更します。

{{<highlight ts "hl_lines=6">}}
export const handler: AWSLambda.APIGatewayProxyHandler = async (event) => {
  console.log("request:", JSON.stringify(event, undefined, 2));
  return {
    statusCode: 200,
    headers: { "Content-Type": "text/plain" },
    body: `Good Morning, CDK! You've hit ${event.path}\n`
  };
};
{{</highlight>}}

変更が `cdk deploy` を実行してみましょう。

```
cdk deploy
```

出力は次のようになります。

```
✨  Synthesis time: 42.35s

CdkWorkshopStack: deploying...
CdkWorkshopStack: creating CloudFormation changeset...



 ✅  CdkWorkshopStack

✨  Deployment time: 27.48s

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT-ID:stack/CdkWorkshopStack/STACK-ID

✨  Total time: 69.83s
```

正確な時間にはばらつきがありますが、通常のデプロイにかかる時間については、かなりの目安になるはずです！

## Hotswap deployments

{{% notice info %}}
このコマンドは、デプロイを高速化するために、CloudFormationのスタックに意図的にドリフトを発生させるものです。
このため、開発目的にのみ使用してください。
本番デプロイには絶対にhotswapを使わないでください！
{{% /notice %}}

`CDK deploy --hotswap` を使えばデプロイ時間を短縮することができます。これはCloudFormation のデプロイの代わりにホットスワップデプロイが実行可能かどうかを評価してくれます。
ホットスワップデプロイが可能であれば、CDK CLIはAWSサービスAPIを使用して直接変更を行います。そうでない場合は、CloudFormationのフルデプロイメントを実行します。

ここでは、`cdk deploy --hotswap` を使用して、AWS Lambda のアセットコードにホットスワップ可能な変更をデプロイします。

## `cdk deploy --hotswap` にかかる時間を測ってみる

`lambda/hello.ts`のlambdaコードをもう一度変えてみましょう。

{{<highlight ts "hl_lines=6">}}
export const handler: AWSLambda.APIGatewayProxyHandler = async (event) => {
  console.log("request:", JSON.stringify(event, undefined, 2));
  return {
    statusCode: 200,
    headers: { "Content-Type": "text/plain" },
    body: `Good Afternoon, CDK! You've hit ${event.path}\n`
  };
};
{{</highlight>}}

そして `cdk deploy --hotswap` を実行してみます。

```
cdk deploy --hotswap
```

出力は次のようになります。

```
✨  Synthesis time: 21.51s

⚠️ The --hotswap flag deliberately introduces CloudFormation drift to speed up deployments
⚠️ It should only be used for development - never use it for your production Stacks!

CdkWorkshopStack: deploying...
✨ hotswapping resources:
   ✨ Lambda Function 'CdkWorkshopStack-HelloHandler2E4FBA4D-Cho6nu1hDGKg'
✨ Lambda Function 'CdkWorkshopStack-HelloHandler2E4FBA4D-Cho6nu1hDGKg' hotswapped!

 ✅  CdkWorkshopStack

✨  Deployment time: 2.81s

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT-ID:stack/CdkWorkshopStack/STACK-ID

✨  Total time: 24.31s
```

さきほどのフルデプロイメントには69秒かかりましたが、ホットスワップデプロイは24秒でデプロイが完了しました！
しかし、警告メッセージが出ていますね。`--hotswap`フラグの使用上の注意ですのでよく読んでください。

```
⚠️ The --hotswap flag deliberately introduces CloudFormation drift to speed up deployments
⚠️ It should only be used for development - never use it for your production Stacks!
```

> `--hotswap` フラグは、デプロイメントを高速化するために意図的に CloudFormation のドリフトを導入しています。
> これは開発のみに使用すべきです - 決して本番のStacksには使用しないでください！

## 本当にデプロイできたのでしょうか？

高速にデプロイされることが確認できたと思います。
実際にコードは変更されたのでしょうか？
AWS Lambda Consoleで再確認してみましょう!

1. [AWS Lambda Console](https://console.aws.amazon.com/lambda/home#/functions?fo=and&o0=%3A&v0=CdkWorkshop)を開きます。 (正しいリージョンにいることを確認してください)

    デプロイした関数を見つけてください。

    ![](./lambda-1.png)

2. 関数の名前をクリックします。

3. デプロイされたコードが表示されます。変更されていることは確認できましたでしょうか？

    ![](./lambda-5.png)

## CDK Watch

毎回 `cdk deploy` や `cdk deploy --hotswap` を呼び出すよりも、もっと良い方法があるはずです。
`cdk watch` は `cdk deploy` と似ていますが、ワンショットの操作ではなく、
コードやアセットの変更を監視して、変更が検出されたときに自動的にデプロイを実行してくれます。
デフォルトでは、`cdk watch` は `--hotswap` フラグを使用し、変更を検査し、その変更がホットスワップ可能かどうかを決定します。
`cdk watch --no-hotswap`を呼び出すと、ホットスワップの動作が無効化されます。

一度設定すれば、`cdk watch`を使用して、ホットスワップ可能な変更とCloudFormationのフルデプロイを必要とする変更の両方を検出することができます。

## `cdk watch` にかかる時間を測ってみる

まず `cdk watch` を実行してみます。

```
cdk watch
```

これで初期デプロイが始まり、すぐに `cdk.json` で指定したファイルの監視を開始します。

もう一度、`lambda/hello.js` を変更してみましょう。

{{<highlight js "hl_lines=6">}}
export const handler: AWSLambda.APIGatewayProxyHandler = async (event) => {
  console.log("request:", JSON.stringify(event, undefined, 2));
  return {
    statusCode: 200,
    headers: { "Content-Type": "text/plain" },
    body: `Good Night, CDK! You've hit ${event.path}\n`
  };
};
{{</highlight>}}

Lambda コードファイルの変更を保存すると、`cdk watch` がファイルが変更されたことを認識し、新しいデプロイメントが開始されます。
今回は、lambdaアセットコードをホットスワップできることを認識し、CloudFormation のデプロイを回避して、代わりに Lambda サービスに直接デプロイされます。

デプロイはどれくらいの速度で行われたのでしょうか？

```
Detected change to 'lambda/hello.ts' (type: change) while 'cdk deploy' is still running. Will queue for another deployment after this one finishes

✨  Synthesis time: 18.63s

⚠️ The --hotswap flag deliberately introduces CloudFormation drift to speed up deployments
⚠️ It should only be used for development - never use it for your production Stacks!

CdkWorkshopStack: deploying...
✨ hotswapping resources:
   ✨ Lambda Function 'CdkWorkshopStack-HelloHandler2E4FBA4D-Cho6nu1hDGKg'
✨ Lambda Function 'CdkWorkshopStack-HelloHandler2E4FBA4D-Cho6nu1hDGKg' hotswapped!

 ✅  CdkWorkshopStack

✨  Deployment time: 2.93s

Stack ARN:
arn:aws:cloudformation:REGION:ACCOUNT-ID:stack/CdkWorkshopStack/STACK-ID

✨  Total time: 21.56s
```

## まとめ

このチュートリアルの残りの部分では、`cdk watch`の代わりに`cdk deploy`を使い続けます。
しかし、必要であれば、単に `cdk watch` をオンにしておくこともできます。
もし、完全なデプロイが必要な場合は、`cdk watch` が `cdk deploy` を呼び出します。

`cdk watch` の使用例についての詳細は、[Increasing Development Speed with CDK Watch](https://aws.amazon.com/blogs/developer/increasing-development-speed-with-cdk-watch/)を参照してください。
