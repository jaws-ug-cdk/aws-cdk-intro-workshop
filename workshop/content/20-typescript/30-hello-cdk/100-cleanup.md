+++
title = "サンプルコードの削除"
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

これでスタックの中身を修正したことになります。
`cdk diff`を実行することで、スタックの修正によってどのような変更が発生するのかを画面で確認できます。
これは `cdk deploy` を実行したときに何が起こるかを確認する安全な方法であり、いつでも使える良いプラクティスです。

```
cdk diff
```

出力は次のようになります。

```
Stack CdkWorkshopStack
IAM Statement Changes
┌───┬─────────────────────────────────┬────────┬─────────────────┬───────────────────────────┬──────────────────────────────────────────────────┐
│   │ Resource                        │ Effect │ Action          │ Principal                 │ Condition                                        │
├───┼─────────────────────────────────┼────────┼─────────────────┼───────────────────────────┼──────────────────────────────────────────────────┤
│ - │ ${CdkWorkshopQueue50D9D426.Arn} │ Allow  │ sqs:SendMessage │ Service:sns.amazonaws.com │ "ArnEquals": {                                   │
│   │                                 │        │                 │                           │   "aws:SourceArn": "${CdkWorkshopTopicD368A42F}" │
│   │                                 │        │                 │                           │ }                                                │
└───┴─────────────────────────────────┴────────┴─────────────────┴───────────────────────────┴──────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[-] AWS::SQS::Queue CdkWorkshopQueue50D9D426 destroy
[-] AWS::SQS::QueuePolicy CdkWorkshopQueuePolicyAF2494A5 destroy
[-] AWS::SNS::Subscription CdkWorkshopQueueCdkWorkshopStackCdkWorkshopTopicD7BE96438B5AD106 destroy
[-] AWS::SNS::Topic CdkWorkshopTopicD368A42F destroy
```

想定の通り、既存のリソースがすっかり削除されることになります。

## cdk deploy

`cdk deploy`を実行したら、**次のセクションに進みます。**

```
cdk deploy
```

リソースが削除されていくのが確認できます。
