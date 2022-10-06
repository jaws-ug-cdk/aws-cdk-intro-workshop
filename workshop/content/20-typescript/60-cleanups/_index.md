+++
title = "Clean up"
weight = 60
chapter = true
+++

# リソースのクリーンアップ

## スタックのクリーンアップ

スタックを破棄するとき、リソースはその削除ポリシーに従って「削除」、「保持」、「スナップショット」のいずれかで処理されます。
デフォルトでは、ほとんどのリソースはスタック削除時に削除されますが、すべてのリソースがそうなるわけではありません。
DynamoDBのテーブルは、デフォルトで保持されます。このテーブルを保持したくない場合は、CDKのコードで `RemovalPolicy` を使って設定ができます。

## スタック削除時に削除するDynamoDBテーブルを設定する

`hitcounter.ts` を編集して、テーブルに `removalPolicy` プロパティを追加します。

{{<highlight ts "hl_lines=26">}}
import * as cdk from 'aws-cdk-lib';
import * as lambda from 'aws-cdk-lib/aws-lambda';
import * as dynamodb from 'aws-cdk-lib/aws-dynamodb';
import { Construct } from 'constructs';

export interface HitCounterProps {
  /** the function for which we want to count url hits **/
  downstream: lambda.IFunction;
}

export class HitCounter extends Construct {
  /** allows accessing the counter function */
  public readonly handler: lambda.Function;

  /** the hit counter table */
  public readonly table: dynamodb.Table;

  constructor(scope: Construct, id: string, props: HitCounterProps) {
    super(scope, id);

    const table = new dynamodb.Table(this, "Hits", {
      partitionKey: {
        name: "path",
        type: dynamodb.AttributeType.STRING
      },
      removalPolicy: cdk.RemovalPolicy.DESTROY
    });
    this.table = table;

    this.handler = new lambda.Function(this, 'HitCounterHandler', {
      runtime: lambda.Runtime.NODEJS_14_X,
      handler: 'hitcounter.handler',
      code: lambda.Code.fromAsset('lambda'),
      environment: {
        DOWNSTREAM_FUNCTION_NAME: props.downstream.functionName,
        HITS_TABLE_NAME: table.tableName
      }
    });

    // grant the lambda role read/write permissions to our table
    table.grantReadWriteData(this.handler);

    // grant the lambda role invoke permissions to the downstream function
    props.downstream.grantInvoke(this.handler);
  }
}
{{</highlight>}}

さらに、作成されたLambda関数は、永久に保持されるCloudWatchのログを生成します。
これらはスタックの一部ではないので、CloudFormationでは追跡されず、ログは削除されずに残ります。
CloudWatchのログを削除する場合は、コンソールでこれらを手動で削除する必要があります。

どのリソースが削除されるかがわかったので、スタックの削除を進めましょう。
CloudFormationのコンソールからスタックを削除するか、`cdk destroy`を使用するかのどちらかです。

```
cdk destroy
```

以下のように聞かれるはずです。

```
Are you sure you want to delete: CdkWorkshopStack (y/n)?
```

`y`を押すと、スタックが削除されていく進捗が表示されます。

`cdk bootstrap` によって作成されたブートストラップスタックは削除されません。
**将来的にCDKを使う予定がある場合は、このスタックを削除しないでください**。

`cdk bootstrap` によって作成されたスタックを削除したい場合は、CloudFormationコンソールから行う必要があります。
CloudFormationコンソールから`CDKToolkit`スタックを削除してください。
作成されたS3バケットは、デフォルトで保持されます。
予期せぬ課金を避けたい場合はS3コンソールから、ブートストラップで生成されたバケットを空にして、削除しておいてください。

## Cloud9 のクリーンアップ

CDK の実行環境として使用した Cloud 9 を削除します。AWS マネージメントコンソールから Cloud 9 を開いて、「Delete」ボタンを押します。

![cloud9-delete-button](./60-cleanups/cloud9-delete-1.png)

確認画面で「Delete」と入力すると削除できます。

![cloud9-delete-confirmation-screen](./60-cleanups/cloud9-delete-2.png)
