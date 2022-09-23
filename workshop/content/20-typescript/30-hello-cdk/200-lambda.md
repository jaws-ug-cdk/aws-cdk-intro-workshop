+++
title = "Hello Lambda"
weight = 200
+++

## Lambda handler code

まずは、Lambda handlerのコードから書いていきます。

1. `cdk-workshop`ディレクトリに`lambda`ディレクトリを作成します。
<!-- TODO: Cloud9はデフォルトで`.`で始まるファイルを非表示にしてる。何かしら対応が必要。tsでhandler書く手順にしちゃだめかなぁ。。 -->
2. TS CDK プロジェクトを `cdk init` で作成すると、デフォルトではすべての `.js` ファイルを無視します。
   これらのファイルをgitで追跡するには、 `.gitignore` ファイルに `!lambda/*.js` を追記してください。
   これにより、このチュートリアルのパイプラインのセクションで、Lambdaアセットを発見することができます。
3. `lambda/hello.js`というファイルを追加し、以下の内容を記述します。

```js
exports.handler = async function(event) {
  console.log("request:", JSON.stringify(event, undefined, 2));
  return {
    statusCode: 200,
    headers: { "Content-Type": "text/plain" },
    body: `Hello, CDK! You've hit ${event.path}\n`
  };
};
```

これは、**「Hello, CDK! You’ve hit [url path]」**というテキストを返す単純なLambda関数です。
HTTPステータスコードとHTTPヘッダーが付加されたHTTPレスポンスとしてユーザーに応答するために、API Gatewayを使用します。

{{% notice info %}}
このLambda関数はJavaScriptで実装されています。
その他の言語での実装については[AWS Lambdaのドキュメント](https://docs.aws.amazon.com/lambda/latest/dg/welcome.html)を参照してください。
{{% /notice %}}

## コピー＆ペーストは使わずにコードを書いてみましょう

このワークショップでは、コピー&ペーストをするのではなく、実際にCDKのコードを入力することを強く推奨します（通常、入力する量は多くありません）。
これにより、CDKの使い方についてより理解していただけます。
IDEがオートコンプリート、インラインドキュメント、およびタイプセーフに対応しているのがご理解いただけるでしょう。

![](./auto-complete.png)

## AWS Lambda関数をスタックに追加する

Add an `import` statement at the beginning of `lib/cdk-workshop-stack.ts`, and a
`lambda.Function` to your stack.

`import`ステートメントを`lib/cdk-workshop-stack.ts`の冒頭に挿入し、`lambda.Function`をスタックに追加します。

{{<highlight ts "hl_lines=3 9-14">}}
import { Stack, StackProps } from 'aws-cdk-lib';
import { Construct } from 'constructs';
import * as lambda from 'aws-cdk-lib/aws-lambda'

export class CdkWorkshopStack extends Stack {
  constructor(scope: Construct, id: string, props?: StackProps) {
    super(scope, id, props);

    // defines an AWS Lambda resource
    const hello = new lambda.Function(this, 'HelloHandler', {
      runtime: lambda.Runtime.NODEJS_16_X,    // execution environment
      code: lambda.Code.fromAsset('lambda'),  // code loaded from "lambda" directory
      handler: 'hello.handler',               // file is "hello", function is "handler"
    });
  }
}
{{</highlight>}}

注目すべきいくつかの点：

- この関数は`NODEJS_16_X`(Nodejs v16.x)ランタイムを使用します。
- ハンドラーコードは、先程作った `lambda` ディレクトリからロードされます。
  パスは、`cdk` コマンドが実行されたディレクトリから相対パスです。
- ハンドラー関数の名前は `hello.handler` （「hello」はファイル名、「handler」は関数名です）

## コンストラクト(constructs) と コンストラクター(constructors) について

ご覧のとおり、`CdkWorkshopStack`と`lambda.Function`の両方のコンストラクタークラス（およびCDKの他の多くのクラス）は`(scope, id, props)`という同じような引数を受け取ります。
これは、これらのクラスがすべて**コンストラクタ**であるためです。
コンストラクトはCDKアプリの基本的な構成要素です。
それらは「クラウドコンポーネント」を表現します。クラウドコンポーネントはスコープを介してより高いレベルの抽象化に構築できます。
スコープにはコンストラクトを含めることができ、そのコンストラクトには他のコンストラクトなどを含めることができます。

コンストラクトは常に別のコンストラクトのスコープ内で作成され、作成されたスコープ内で一意でなければならない識別子（id）を持っている必要があります。
したがって、コンストラクト初期化子（コンストラクター）には常に次のシグネチャが必要です。

1. **`scope：`** : 最初の引数には、この構成が作成されるスコープを必ず指定します。
   ほとんどすべての場合、現在の コンストラクトスコープ内でコンストラクトを定義することになります。
   つまり、通常最初の引数には`this`を渡すだけです。

2. **`id`** ： 2番目の引数は、構造の**ローカルID**です。
   これは、同じスコープ内のコンストラクト間で一意である必要があるIDです。
   CDKはこのIDを使用して、 このスコープ内で定義された各リソースの[CloudFormation 論理ID](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/resources-section-structure.html)を計算します。
   *CDKのIDの詳細*については、[CDKユーザーマニュアル](https://docs.aws.amazon.com/cdk/latest/guide/identifiers.html#identifiers_logical_ids)を参照してください。

3. **`props`** : 最後の（場合によっては不要である場合もある）引数は、初期化プロパティのセットです。
   これらは各コンストラクトで固有です。
   たとえば、`lambda.Function`コンストラクトは`runtime`、`code`、`handler`のようなプロパティを受け取ります。
   IDEのオートコンプリートまたは[オンラインドキュメント](https://docs.aws.amazon.com/cdk/api/latest/docs/aws-lambda-readme.html)を使用して、さまざまなオプションを調べられます。

## Diff

コードを保存し、デプロイする前に差分を見てみましょう。

```
cdk diff
```

出力は次のようになります。

```text
Stack CdkWorkshopStack
IAM Statement Changes
┌───┬─────────────────────────────────┬────────┬────────────────┬──────────────────────────────┬───────────┐
│   │ Resource                        │ Effect │ Action         │ Principal                    │ Condition │
├───┼─────────────────────────────────┼────────┼────────────────┼──────────────────────────────┼───────────┤
│ + │ ${HelloHandler/ServiceRole.Arn} │ Allow  │ sts:AssumeRole │ Service:lambda.amazonaws.com │           │
└───┴─────────────────────────────────┴────────┴────────────────┴──────────────────────────────┴───────────┘
IAM Policy Changes
┌───┬─────────────────────────────┬────────────────────────────────────────────────────────────────────────────────┐
│   │ Resource                    │ Managed Policy ARN                                                             │
├───┼─────────────────────────────┼────────────────────────────────────────────────────────────────────────────────┤
│ + │ ${HelloHandler/ServiceRole} │ arn:${AWS::Partition}:iam::aws:policy/service-role/AWSLambdaBasicExecutionRole │
└───┴─────────────────────────────┴────────────────────────────────────────────────────────────────────────────────┘
(NOTE: There may be security-related changes not in this list. See https://github.com/aws/aws-cdk/issues/1299)

Resources
[+] AWS::IAM::Role HelloHandler/ServiceRole HelloHandlerServiceRole11EF7C63 
[+] AWS::Lambda::Function HelloHandler HelloHandler2E4FBA4D 


NOTICES

21902   apigateway: Unable to serialize value as aws-cdk-lib.aws_apigateway.IModel

        Overview: Users of CDK in any language other than TS/JS cannot use
                  values that return an instance of a deprecated class.

        Affected versions: framework: >=2.41.0, framework: >=1.172.0

        More information at: https://github.com/aws/aws-cdk/issues/21902


If you don’t want to see a notice anymore, use "cdk acknowledge <id>". For example, "cdk acknowledge 21902".
```

上記のとおり、このコードから **AWS::Lambda::Function** リソース用のCloudFormationテンプレートを生成しました。
また、ツールキットがハンドラーコードの場所を伝達するためにいくつかの[CloudFormationパラメーター](https://docs.aws.amazon.com/cdk/latest/guide/get_cfn_param.html)を利用しています。

## Deploy

次にデプロイをします。

```
cdk deploy
```

`cdk deploy` を実行すると、CloudFormationスタックをデプロイするだけでなく、
初期構築したS3バケットに対して、ローカルの `lambda` ディレクトリを圧縮後、アップロードしていることがが分かるでしょう。

## Testing our function

AWS Lambdaコンソールに移動して、Lambda関数をテストしましょう。

1. [AWS Lambdaコンソール](https://console.aws.amazon.com/lambda/home#/functions) を開きます
   （正しいリージョンにいることを確認してください）。

   Lambda関数が表示されます。

   ![](./lambda-1.png)

2. 関数名をクリックして、コンソールを移動します。

3. **テスト**タブをクリックします。

    ![](./lambda-2.png)

5. **イベント名**に`test`を入力します。

4. **テンプレート**リストから**Amazon API Gateway AWS Proxy**を選択します。

    ![](./lambda-3.png)

6. **保存**をクリックします。

7. **テスト**をクリックし、実行が完了するまで待ちます。

8. **実行結果**ペインで**詳細**を展開すると、出力が表示されます。

   ![](./lambda-4.png)

# 👏
