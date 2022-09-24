+++
title = "Granting permissions"
weight = 600
+++

## LambdaがDynamoDBテーブルを読み書きできるようにする

Let's give our Lambda's execution role permissions to read/write from our table.

Lambdaの実行ロールに、DynamoDBテーブルの読み取り/書き込み権限を与えましょう。

hitcounter.ts に戻り、次の強調表示された行を追加します。

{{<highlight ts "hl_lines=33-34">}}
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
  }
}
{{</highlight>}}

## デプロイ

保存してデプロイしてみましょう。

```
cdk deploy
```

## 再テスト

さて、デプロイが完了しました。テストをもう一度実行してみましょう（`curl`かWebブラウザのいずれかを使用してください）

```
curl -i https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

また？

```
HTTP/2 502 Bad Gateway
...

{"message": "Internal server error"}
```

# 😢

まだこの厄介な5xxエラーが発生しています！CloudWatchログをもう一度見てみましょう（「更新」をクリックします）。

```json
{
    "errorType": "AccessDeniedException",
    "errorMessage": "User: arn:aws:sts::123456789012:assumed-role/CdkWorkshopStack-HelloHitCounterHitCounterHandlerS-1234567890123/CdkWorkshopStack-HelloHitCounterHitCounterHandlerD-123456789012 is not authorized to perform: lambda:InvokeFunction on resource: arn:aws:lambda:ap-northeast-1:123456789012:function:CdkWorkshopStack-HelloHandler2E4FBA4D-123456789012 because no identity-based policy allows the lambda:InvokeFunction action",
    "code": "AccessDeniedException",
    "message": "User: arn:aws:sts::123456789012:assumed-role/CdkWorkshopStack-HelloHitCounterHitCounterHandlerS-1234567890123/CdkWorkshopStack-HelloHitCounterHitCounterHandlerD-123456789012 is not authorized to perform: lambda:InvokeFunction on resource: arn:aws:lambda:ap-northeast-1:123456789012:function:CdkWorkshopStack-HelloHandler2E4FBA4D-123456789012 because no identity-based policy allows the lambda:InvokeFunction action",
    "time": "2022-09-24T14:02:52.516Z",
    "requestId": "2484f30b-70f7-42df-b9ac-134d995c1631",
    "statusCode": 403,
    "retryable": false,
    "retryDelay": 4.445030300047104,
    "stack": [
        "AccessDeniedException: User: arn:aws:sts::123456789012:assumed-role/CdkWorkshopStack-HelloHitCounterHitCounterHandlerS-1234567890123/CdkWorkshopStack-HelloHitCounterHitCounterHandlerD-123456789012 is not authorized to perform: lambda:InvokeFunction on resource: arn:aws:lambda:ap-northeast-1:123456789012:function:CdkWorkshopStack-HelloHandler2E4FBA4D-123456789012 because no identity-based policy allows the lambda:InvokeFunction action",
        "    at Object.extractError (/var/runtime/node_modules/aws-sdk/lib/protocol/json.js:52:27)",
        "    at Request.extractError (/var/runtime/node_modules/aws-sdk/lib/protocol/rest_json.js:49:8)",
        "    at Request.callListeners (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:106:20)",
        "    at Request.emit (/var/runtime/node_modules/aws-sdk/lib/sequential_executor.js:78:10)",
        "    at Request.emit (/var/runtime/node_modules/aws-sdk/lib/request.js:686:14)",
        "    at Request.transition (/var/runtime/node_modules/aws-sdk/lib/request.js:22:10)",
        "    at AcceptorStateMachine.runTo (/var/runtime/node_modules/aws-sdk/lib/state_machine.js:14:12)",
        "    at /var/runtime/node_modules/aws-sdk/lib/state_machine.js:26:10",
        "    at Request.<anonymous> (/var/runtime/node_modules/aws-sdk/lib/request.js:38:9)",
        "    at Request.<anonymous> (/var/runtime/node_modules/aws-sdk/lib/request.js:688:12)"
    ]
}
```

Lambda関数の呼び出しエラーが発生していますが、今回は先ほどのDynamoDBへの書き込みエラーは出力されていません。

```
User: <VERY-LONG-STRING> is not authorized to perform: lambda:InvokeFunction on resource: <VERY-LONG-STRING>"
```

ヒットカウンタはどうにかデータベースへの書き込みは行えたようです。
[DynamoDBコンソール](https://console.aws.amazon.com/dynamodb/home)に移動して確認できます。

![](./logs5.png)

Lambda関数にDynamoDBのアクセス許可を与えたように、ヒットカウンタにダウンストリームのLambda関数を呼び出すための権限を与える必要があります。

## 呼び出し許可権限を付与する

次のとおり、`lib/hitcounter.ts`に強調表示された行を追加します。

{{<highlight ts "hl_lines=36-37">}}
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

## 差分確認

`cdk diff`を使用して、何が変更されたかをチェックします。

```
cdk diff
```

**Resource** セクションにヒットカウンタのロールに追加されたIAMポリシーが表示されます。

```
Resources
[~] AWS::IAM::Policy HelloHitCounter/HitCounterHandler/ServiceRole/DefaultPolicy HelloHitCounterHitCounterHandlerServiceRoleDefaultPolicy1487A60A
 └─ [~] PolicyDocument
     └─ [~] .Statement:
         └─ @@ -19,5 +19,15 @@
            [ ]         "Arn"
            [ ]       ]
            [ ]     }
            [+]   },
            [+]   {
            [+]     "Action": "lambda:InvokeFunction",
            [+]     "Effect": "Allow",
            [+]     "Resource": {
            [+]       "Fn::GetAtt": [
            [+]         "HelloHandler2E4FBA4D",
            [+]         "Arn"
            [+]       ]
            [+]     }
            [ ]   }
            [ ] ]
```

意図した通りの変更となっています。

## デプロイ

再度デプロイしてみましょう。

```
cdk deploy
```

デプロイが完了したら`curl`コマンド、またはWebブラウザでエンドポイントを呼び出してみましょう。

```
curl -i https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

出力は次のようになります。

```
HTTP/2 200 OK
...

Hello, CDK! You've hit /
```

> もし、5XXエラーとなる場合は、数秒待ってからもう一度試してください。
> API Gatewayのエンドポイントに新しいデプロイを適用するのに少し時間がかかることがあります。

# 😲
