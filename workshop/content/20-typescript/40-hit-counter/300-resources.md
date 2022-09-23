+++
title = "Define resources"
weight = 300
+++

## HitCounterコンストラクトにリソースを追加する

次に、AWS Lambda関数とDynamoDBテーブルをHitCounter コンストラクトに定義します。

`lib/hitcounter.ts` に戻って、以下のコードを追記しましょう。

{{<highlight ts "hl_lines=3 13-14 19-31">}}
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

  constructor(scope: Construct, id: string, props: HitCounterProps) {
    super(scope, id);

    const table = new dynamodb.Table(this, 'Hits', {
      partitionKey: { name: 'path', type: dynamodb.AttributeType.STRING }
    });

    this.handler = new lambda.Function(this, 'HitCounterHandler', {
      runtime: lambda.Runtime.NODEJS_16_X,
      handler: 'hitcounter.handler',
      code: lambda.Code.fromAsset('lambda'),
      environment: {
        DOWNSTREAM_FUNCTION_NAME: props.downstream.functionName,
        HITS_TABLE_NAME: table.tableName
      }
    });
  }
}
{{</highlight>}}

## コードの解説

* DynamoDBテーブルに`path`パーティションキーを定義しました。
* `lambda/hitcounter.handler`にバインドされるLambda関数を定義しました。
* Lambdaの環境変数と`downstream.functionName`、`table.tableName`とを紐付けました。

## 遅延バインディング値

`functionName`と`tableName`プロパティは、CloudFormationスタックをデプロイするタイミングで解決される値です
（テーブル/関数を定義した時点では、まだ物理名が決まっていないことに注目してください。論理IDだけが決まっています。）。
CloudFormationテンプレート生成時にそれらの値を表示すると、"TOKEN" という値が得られます。
この値は、CDKがこれらの遅延バインディング値をどのように表現するかを示しています。
CDKはこれらの遅延バインディング値を未定の値として扱う必要があります。
つまり、例えばそれらを連結することはできますが、コード内でそれらを解析（splitやsubstringなど）しても正しく動作することはありません。
