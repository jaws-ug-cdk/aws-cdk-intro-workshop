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
hint: Using 'master' as the name for the initial branch. This default branch name
hint: is subject to change. To configure the initial branch name to use in all
hint: of your new repositories, which will suppress this warning, call:
hint: 
hint:   git config --global init.defaultBranch <name>
hint: 
hint: Names commonly chosen instead of 'master' are 'main', 'trunk' and
hint: 'development'. The just-created branch can be renamed via this command:
hint: 
hint:   git branch -m <name>
Executing npm install...
npm notice 
npm notice New minor version of npm available! 8.15.0 -> 8.19.2
npm notice Changelog: https://github.com/npm/cli/releases/tag/v8.19.2
npm notice Run npm install -g npm@8.19.2 to update!
npm notice 
✅ All done!
****************************************************
*** Newer version of CDK is available [2.43.0]   ***
*** Upgrade recommended (npm install -g aws-cdk) ***
****************************************************
```

上記出力にあるとおり、CDKを開始するための便利なコマンドがたくさんあります。

## See Also

- [AWS CDK Command Line Toolkit (cdk) in the AWS CDK User Guide](https://docs.aws.amazon.com/CDK/latest/userguide/tools.html)
