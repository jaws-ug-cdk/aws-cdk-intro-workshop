+++
title = "HitCounterハンドラー"
weight = 100
+++

## HitCounter Lambda ハンドラー

HitCounterのLambdaハンドラーコードを記述しましょう。

`lambda/hitcounter.ts`を作成し、次のコードを追記してください。

```js
import { DynamoDB, Lambda } from 'aws-sdk';

export const handler: AWSLambda.APIGatewayProxyHandler = async (event) => {
  console.log('request:', JSON.stringify(event, undefined, 2));

  // create AWS SDK clients
  const dynamo = new DynamoDB();
  const lambda = new Lambda();

  // update dynamo entry for "path" with hits++
  await dynamo
    .updateItem({
      TableName: process.env.HITS_TABLE_NAME!,
      Key: { path: { S: event.path } },
      UpdateExpression: 'ADD hits :incr',
      ExpressionAttributeValues: { ':incr': { N: '1' } },
    })
    .promise();

  // call downstream function and capture response
  const resp = await lambda
    .invoke({
      FunctionName: process.env.DOWNSTREAM_FUNCTION_NAME!,
      Payload: JSON.stringify(event),
    })
    .promise();

  console.log('downstream response:', JSON.stringify(resp, undefined, 2));

  // return response back to upstream caller
  return JSON.parse(resp.Payload!.toString());
};
```

## 実行時のリソース検出

このコードは、次の2つの環境変数を参照していることがわかります。

 * `HITS_TABLE_NAME` データストアとして使用するDynamoDBのテーブル名
 * `DOWNSTREAM_FUNCTION_NAME` ダウンストリームLambda関数

テーブルとダウンストリーム関数の名前はアプリをデプロイするときに決まるため、これらの値をコンストラクトコードから関連付ける必要があります。
次のセクションでそれを行います。

Lambda関数がさらに別のLambda関数を呼び出すようになっています。
呼び出される側のLambda関数をダウンストリームLambda関数、あるいは単にダウンストリーム関数とここでは呼んでいます。
