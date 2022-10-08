+++
title = "リソースの定義"
weight = 300
+++

## HitCounterコンストラクトにリソースを追加する

次に、Lambda関数とDynamoDBテーブルを`HitCounter`コンストラクトに定義します。

`lib/hitcounter.ts` に戻って、以下のコードを追記しましょう。

{{<highlight ts "hl_lines=1-3 13-14 18-36">}}
import { AttributeType, Table } from 'aws-cdk-lib/aws-dynamodb';
import { IFunction, Runtime } from 'aws-cdk-lib/aws-lambda';
import { NodejsFunction } from 'aws-cdk-lib/aws-lambda-nodejs';
import { Construct } from 'constructs';
import { RemovalPolicy } from 'aws-cdk-lib'

export interface HitCounterProps {
  /** the function for which we want to count url hits **/
  downstream: IFunction;
}

export class HitCounter extends Construct {
  /** allows accessing the counter function */
  public readonly handler: IFunction;
  constructor(scope: Construct, id: string, props: HitCounterProps) {
    super(scope, id);

    const table = new Table(this, 'Hits', {
      partitionKey: { name: 'path', type: AttributeType.STRING },
      // Note: プロダクションではDBテーブルは保持することが多いが今回はハンズオンのためDESTROYを指定
      removalPolicy: RemovalPolicy.DESTROY,
    });

    this.handler = new NodejsFunction(this, 'HitCounterHandler', {
      runtime: Runtime.NODEJS_16_X,
      entry: 'lambda/hitcounter.ts',
      environment: {
        DOWNSTREAM_FUNCTION_NAME: props.downstream.functionName,
        HITS_TABLE_NAME: table.tableName,
      },
    });

    // Lambda関数に対してテーブルを読み書き/別のLambdaを実行する権限を付与
    // CDKでは「grant」を利用してリソースに権限を付与できる
    table.grantReadWriteData(this.handler);
    props.downstream.grantInvoke(this.handler);
  }
}
{{</highlight>}}

## コードの解説

* DynamoDBテーブルに`path`パーティションキーを定義しました。
* `lambda/hitcounter.ts`にバインドされるLambda関数を定義しました。
* Lambdaの環境変数と`downstream.functionName`、`table.tableName`とを紐付けました。

## 遅延バインディング値

`functionName`と`tableName`プロパティは、CloudFormationスタックをデプロイするタイミングで解決される値です
（テーブル/関数を定義した時点では、まだ物理名が決まっていないことに注目してください。論理IDだけが決まっています。）。
CloudFormationテンプレート生成時にそれらの値を表示すると、"TOKEN" という値が得られます。
この値は、CDKがこれらの遅延バインディング値をどのように表現するかを示しています。
CDKはこれらの遅延バインディング値を未定の値として扱う必要があります。
例えば、それらを連結することはできますが、コード内でそれらを解析（splitやsubstringなど）しても正しく動作することはありません。
