+++
title = "cdk init"
weight = 100
+++

## プロジェクトディレクトリを作成する

空のディレクトリを作成します。

```
mkdir cdk-workshop && cd cdk-workshop
```

## cdk init

`cdk init`コマンドを実行して、TypeScript製の新しいCDKプロジェクトを作成します。

```
cdk init sample-app --language typescript
```

出力は次のようになります。

```
# Welcome to your CDK TypeScript project

You should explore the contents of this project. It demonstrates a CDK app with an instance of a stack (`CdkWorkshopStack`)
which contains an Amazon SQS queue that is subscribed to an Amazon SNS topic.

The `cdk.json` file tells the CDK Toolkit how to execute your app.

## Useful commands

* `npm run build`   compile typescript to js
* `npm run watch`   watch for changes and compile
* `npm run test`    perform the jest unit tests
* `cdk deploy`      deploy this stack to your default AWS account/region
* `cdk diff`        compare deployed stack with current state
* `cdk synth`       emits the synthesized CloudFormation template

Initializing a new git repository...
Executing npm install...
✅ All done!
```

上記出力にあるとおり、CDKを開始するための便利なコマンドがたくさんあります。

## See Also

- [AWS CDK Command Line Toolkit (cdk) in the AWS CDK User Guide](https://docs.aws.amazon.com/CDK/latest/userguide/tools.html)
