+++
title = "Cleanup sample"
weight = 100
+++

## スタックからサンプルコードを削除する

`cdk init sample-app` によって作成されたプロジェクトには、SQSキューとSNSトピックが含まれます。
このプロジェクトではそれらを使用する予定はないので、 `CdkWorkshopStack` コンストラクタから削除しましょう。

`lib/cdk-workshop-stack.ts` を開き、削除します。
最終的には次のようになります。

```ts
import { Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';

export class CdkWorkshopStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // nothing here!
  }
}
```

## cdk diff

Now that we modified our stack's contents, we can ask the toolkit to show us the difference between our CDK app and
what's currently deployed. This is a safe way to check what will happen once we run `cdk deploy` and is always good practice:

これでスタックの中身を修正したことになります。
`cdk diff`を実行することで、スタックの修正によってどのような変更が発生するのかをツールキットで確認できます。
これは `cdk deploy` を実行したときに何が起こるかを確認する安全な方法であり、いつでも使える良いプラクティスです。

```
cdk diff
```

出力は次のようになります。

```
Stack CdkWorkshopStack
IAM Statement Changes
┌───┬─────────────────────────────────┬────────┬─────────────────┬───────────────────────────┬─────────────────────────────────────────────────────────────────┐
│   │ Resource                        │ Effect │ Action          │ Principal                 │ Condition                                                       │
├───┼─────────────────────────────────┼────────┼─────────────────┼───────────────────────────┼─────────────────────────────────────────────────────────────────┤
│ - │ ${CdkWorkshopQueue50D9D426.Arn} │ Allow  │ sqs:SendMessage │ Service:sns.amazonaws.com │ "ArnEquals": {                                                  │
│   │                                 │        │                 │                           │   "aws:SourceArn": "${CdkWorkshopTopicD368A42F}"                │
│   │                                 │        │                 │                           │ }                                                               │
└───┴─────────────────────────────────┴────────┴─────────────────┴───────────────────────────┴─────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[-] AWS::SQS::Queue CdkWorkshopQueue50D9D426 destroy
[-] AWS::SQS::QueuePolicy CdkWorkshopQueuePolicyAF2494A5 destroy
[-] AWS::SNS::Subscription CdkWorkshopQueueCdkWorkshopStackCdkWorkshopTopicD7BE96438B5AD106 destroy
[-] AWS::SNS::Topic CdkWorkshopTopicD368A42F destroy


NOTICES

21902   apigateway: Unable to serialize value as aws-cdk-lib.aws_apigateway.IModel

        Overview: Users of CDK in any language other than TS/JS cannot use
                  values that return an instance of a deprecated class.

        Affected versions: framework: >=2.41.0, framework: >=1.172.0

        More information at: https://github.com/aws/aws-cdk/issues/21902


If you don’t want to see a notice anymore, use "cdk acknowledge <id>". For example, "cdk acknowledge 21902".
```

想定の通り、既存のリソースがすっかり削除されることになります。

## cdk deploy

Run `cdk deploy` and __proceed to the next section__ (no need to wait):

`cdk deploy`を実行したら、**次のセクションに進みます。**（デプロイの完了を見届けなくても良いです）

```
cdk deploy
```

リソースが削除されていくのが確認できます。
