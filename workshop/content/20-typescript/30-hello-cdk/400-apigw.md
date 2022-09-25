+++
title = "API Gateway"
weight = 400
+++

次のステップでは、API Gatewayを関数の前に追加していきます。
API GatewayはパブリックHTTPエンドポイントを公開します。このエンドポイントは、インターネット上の誰もが
[curl](https://curl.haxx.se/)やウェブブラウザのようなHTTPクライアントでヒットできます。

API Gatewayのルートには、
[Lambda proxy integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/api-gateway-create-api-as-simple-proxy-for-lambda.html)を利用します。
つまり、どのURLパスへのリクエストも、直接Lambda関数にプロキシされ、関数からのレスポンスがユーザーに返されることになります。

## LambdaRestApi コンストラクトを追加する

`lib/cdk-workshop-stack.ts` に戻り、APIエンドポイントを定義してLambda関数に関連付けましょう。

{{<highlight ts "hl_lines=4 17-20">}}
import { Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as lambda from 'aws-cdk-lib/aws-lambda'
import * as apigw from 'aws-cdk-lib/aws-apigateway';

export class CdkWorkshopStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // defines an AWS Lambda resource
    const hello = new lambda.Function(this, 'HelloHandler', {
      runtime: lambda.Runtime.NODEJS_16_X,   // execution environment
      code: lambda.Code.fromAsset('lambda'), // code loaded from "lambda" directory
      handler: 'hello.handler',              // file is "hello", function is "handler"
    });

    // defines an API Gateway REST API resource backed by our "hello" function.
    new apigw.LambdaRestApi(this, 'Endpoint', {
      handler: hello,
    });
  }
}
{{</highlight>}}

以上です。これで、すべてのリクエストをAWS Lambda関数にプロキシするAPI Gatewayが定義されました。

## cdk diff

これをデプロイするとどうなるのか見てみましょう。

```
cdk diff
```

出力は次のようになります。

```text
Stack CdkWorkshopStack
IAM Statement Changes
┌───┬─────────────────────┬────────┬───────────────────────┬──────────────────────────────────────────────────────────────────┬───────────────────────────────────────────────────────────────────┐
│   │ Resource            │ Effect │ Action                │ Principal                                                        │ Condition                                                         │
├───┼─────────────────────┼────────┼───────────────────────┼──────────────────────────────────────────────────────────────────┼───────────────────────────────────────────────────────────────────┤
│ + │ ${HelloHandler.Arn} │ Allow  │ lambda:InvokeFunction │ Service:apigateway.amazonaws.com                                 │ "ArnLike": {                                                      │
│   │                     │        │                       │                                                                  │   "AWS:SourceArn": "arn:${AWS::Partition}:execute-api:${AWS::Regi │
│   │                     │        │                       │                                                                  │ on}:${AWS::AccountId}:${EndpointEEF1FD8F}/${Endpoint/DeploymentSt │
│   │                     │        │                       │                                                                  │ age.prod}/*/*"                                                    │
│   │                     │        │                       │                                                                  │ }                                                                 │
│ + │ ${HelloHandler.Arn} │ Allow  │ lambda:InvokeFunction │ Service:apigateway.amazonaws.com                                 │ "ArnLike": {                                                      │
│   │                     │        │                       │                                                                  │   "AWS:SourceArn": "arn:${AWS::Partition}:execute-api:${AWS::Regi │
│   │                     │        │                       │                                                                  │ on}:${AWS::AccountId}:${EndpointEEF1FD8F}/test-invoke-stage/*/*"  │
│   │                     │        │                       │                                                                  │ }                                                                 │
│ + │ ${HelloHandler.Arn} │ Allow  │ lambda:InvokeFunction │ Service:apigateway.amazonaws.com                                 │ "ArnLike": {                                                      │
│   │                     │        │                       │                                                                  │   "AWS:SourceArn": "arn:${AWS::Partition}:execute-api:${AWS::Regi │
│   │                     │        │                       │                                                                  │ on}:${AWS::AccountId}:${EndpointEEF1FD8F}/${Endpoint/DeploymentSt │
│   │                     │        │                       │                                                                  │ age.prod}/*/"                                                     │
│   │                     │        │                       │                                                                  │ }                                                                 │
│ + │ ${HelloHandler.Arn} │ Allow  │ lambda:InvokeFunction │ Service:apigateway.amazonaws.com                                 │ "ArnLike": {                                                      │
│   │                     │        │                       │                                                                  │   "AWS:SourceArn": "arn:${AWS::Partition}:execute-api:${AWS::Regi │
│   │                     │        │                       │                                                                  │ on}:${AWS::AccountId}:${EndpointEEF1FD8F}/test-invoke-stage/*/"   │
│   │                     │        │                       │                                                                  │ }                                                                 │
└───┴─────────────────────┴────────┴───────────────────────┴──────────────────────────────────────────────────────────────────┴───────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[+] AWS::ApiGateway::RestApi Endpoint EndpointEEF1FD8F 
[+] AWS::ApiGateway::Deployment Endpoint/Deployment EndpointDeployment318525DA5f8cdfe532107839d82cbce31f859259 
[+] AWS::ApiGateway::Stage Endpoint/DeploymentStage.prod EndpointDeploymentStageprodB78BEEA0 
[+] AWS::ApiGateway::Resource Endpoint/Default/{proxy+} Endpointproxy39E2174E 
[+] AWS::Lambda::Permission Endpoint/Default/{proxy+}/ANY/ApiPermission.CdkWorkshopStackEndpoint018E8349.ANY..{proxy+} EndpointproxyANYApiPermissionCdkWorkshopStackEndpoint018E8349ANYproxy747DCA52 
[+] AWS::Lambda::Permission Endpoint/Default/{proxy+}/ANY/ApiPermission.Test.CdkWorkshopStackEndpoint018E8349.ANY..{proxy+} EndpointproxyANYApiPermissionTestCdkWorkshopStackEndpoint018E8349ANYproxy41939001 
[+] AWS::ApiGateway::Method Endpoint/Default/{proxy+}/ANY EndpointproxyANYC09721C5 
[+] AWS::Lambda::Permission Endpoint/Default/ANY/ApiPermission.CdkWorkshopStackEndpoint018E8349.ANY.. EndpointANYApiPermissionCdkWorkshopStackEndpoint018E8349ANYE84BEB04 
[+] AWS::Lambda::Permission Endpoint/Default/ANY/ApiPermission.Test.CdkWorkshopStackEndpoint018E8349.ANY.. EndpointANYApiPermissionTestCdkWorkshopStackEndpoint018E8349ANYB6CC1B64 
[+] AWS::ApiGateway::Method Endpoint/Default/ANY EndpointANY485C938B 

Outputs
[+] Output Endpoint/Endpoint Endpoint8024A810: {"Value":{"Fn::Join":["",["https://",{"Ref":"EndpointEEF1FD8F"},".execute-api.",{"Ref":"AWS::Region"},".",{"Ref":"AWS::URLSuffix"},"/",{"Ref":"EndpointDeploymentStageprodB78BEEA0"},"/"]]}}
```

追加したコードにより、10個の新しいリソースがスタックに追加されることがわかります。

## cdk deploy

デプロイする準備が整いました。

```
cdk deploy
```

## Stack outputs

デプロイが完了すると、次の内容が出力されているはずです。

```
CdkWorkshopStack.Endpoint8024A810 = https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

これは、API Gatewayコンストラクトによって自動的に追加される[stack output](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacks.html)であり、API GatewayエンドポイントのURLが含まれます。

## アプリをテストする。

次に、このエンドポイントを `curl` で叩いてみましょう。 URLをコピーして実行します。（プレフィックスとリージョンは異なる可能性があります）

{{% notice info %}}
[curl](https://curl.haxx.se/)をインストールしていない場合は、WebブラウザでこのURLにアクセスしてみてください。
{{% /notice %}}

```
curl https://xxxxxxxxxx.execute-api.us-east-1.amazonaws.com/prod/
```

出力は次のようになります。

```
Hello, CDK! You've hit /
```

Webブラウザでも確認できます。

![](./browser.png)

この出力がされていれば、アプリは正常に動作しています。

## 正常に動作していないとき

API Gatewayから5xxエラーを受け取った場合、次の2つの問題のいずれかが該当しています。

1. lambda関数が返した応答は、API Gatewayが期待するものと一致していません。
   手順を戻って、Lambdaのhandler関数の返り値に`statusCode`, `body`, `header` フィールドが含まれているか確認してください。
   (参照: [Lambdaのhandler関数のコード](./200-lambda.html))
2. 何らかの理由で関数が失敗しています。
   Lambda関数をデバッグするには、[このセクション](../40-hit-counter/500-logs.html)で、Lambdaログをどのように表示するか学べます。

---

お疲れさまでした！ 次の章では再利用可能な独自のコンストラクトを作成します。
