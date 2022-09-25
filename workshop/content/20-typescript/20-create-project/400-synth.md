+++
title = "cdk synth"
weight = 400
+++

## アプリからテンプレートを生成する

AWS CDKアプリ自体はコードを使用したインフラストラクチャの **定義** にすぎません。
CDKアプリが実行されると、アプリケーションで定義された各スタックのAWS CloudFormationテンプレートが生成（CDKの用語では `"synthesize"` ）されます。

CDKアプリを生成するには、 `cdk synth` コマンドを実行してください。
サンプルアプリから生成されたCloudFormationテンプレートを確認してみましょう。

{{% notice info %}}
**CDK CLI** は`cdk.json`ファイルが配置されているプロジェクトのルートディレクトリで実行する必要があります。
ディレクトリを移動している場合はプロジェクトのルートディレクトリに戻ってからCDKコマンドを実行してください。
{{% /notice %}}

```
cdk synth
```

上記コマンドを実行すると、次のCloudFormationテンプレートを出力します。

```yaml
Resources:
  CdkWorkshopQueue50D9D426:
    Type: AWS::SQS::Queue
    Properties:
      VisibilityTimeout: 300
    UpdateReplacePolicy: Delete
    DeletionPolicy: Delete
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopQueue/Resource
  CdkWorkshopQueuePolicyAF2494A5:
    Type: AWS::SQS::QueuePolicy
    Properties:
      PolicyDocument:
        Statement:
          - Action: sqs:SendMessage
            Condition:
              ArnEquals:
                aws:SourceArn:
                  Ref: CdkWorkshopTopicD368A42F
            Effect: Allow
            Principal:
              Service: sns.amazonaws.com
            Resource:
              Fn::GetAtt:
                - CdkWorkshopQueue50D9D426
                - Arn
        Version: "2012-10-17"
      Queues:
        - Ref: CdkWorkshopQueue50D9D426
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopQueue/Policy/Resource
  CdkWorkshopQueueCdkWorkshopStackCdkWorkshopTopicD7BE96438B5AD106:
    Type: AWS::SNS::Subscription
    Properties:
      Protocol: sqs
      TopicArn:
        Ref: CdkWorkshopTopicD368A42F
      Endpoint:
        Fn::GetAtt:
          - CdkWorkshopQueue50D9D426
          - Arn
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopQueue/CdkWorkshopStackCdkWorkshopTopicD7BE9643/Resource
  CdkWorkshopTopicD368A42F:
    Type: AWS::SNS::Topic
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CdkWorkshopTopic/Resource
  CDKMetadata:
    Type: AWS::CDK::Metadata
    Properties:
      Analytics: v2:deflate64:H4sIAAAAAAAA/1WNQQrCMBBFz9J9OhoFxXUvoK17aZMI09akZhJFQu5uk4DgZv7/jwezg/0JeNW/qRZyqmccIHSuFxNb0S3QkyBcvPKKNXddSr5nM6P4/GCZkZFe/c4PJCwuDo1Oxt++mgVFornEmGqryHgr8o/GaInJjEwbqWCkzYsfgB9hW42EWFuvHT4UtCW/VHqIZsEAAAA=
    Metadata:
      aws:cdk:path: CdkWorkshopStack/CDKMetadata/Default
    Condition: CDKMetadataAvailable
Conditions:
  CDKMetadataAvailable:
    Fn::Or:
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - af-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-northeast-2
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-southeast-1
          - Fn::Equals:
              - Ref: AWS::Region
              - ap-southeast-2
          - Fn::Equals:
              - Ref: AWS::Region
              - ca-central-1
          - Fn::Equals:
              - Ref: AWS::Region
              - cn-north-1
          - Fn::Equals:
              - Ref: AWS::Region
              - cn-northwest-1
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-central-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-north-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-1
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-2
          - Fn::Equals:
              - Ref: AWS::Region
              - eu-west-3
          - Fn::Equals:
              - Ref: AWS::Region
              - me-south-1
          - Fn::Equals:
              - Ref: AWS::Region
              - sa-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-east-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-east-2
      - Fn::Or:
          - Fn::Equals:
              - Ref: AWS::Region
              - us-west-1
          - Fn::Equals:
              - Ref: AWS::Region
              - us-west-2
Parameters:
  BootstrapVersion:
    Type: AWS::SSM::Parameter::Value<String>
    Default: /cdk-bootstrap/hnb659fds/version
    Description: Version of the CDK Bootstrap resources in this environment, automatically retrieved from SSM Parameter Store. [cdk:skip]
Rules:
  CheckBootstrapVersion:
    Assertions:
      - Assert:
          Fn::Not:
            - Fn::Contains:
                - - "1"
                  - "2"
                  - "3"
                  - "4"
                  - "5"
                - Ref: BootstrapVersion
        AssertDescription: CDK bootstrap stack version 6 required. Please run 'cdk bootstrap' with a recent version of the CDK CLI.
```

ご覧のとおり、このテンプレートには4つのリソースが含まれています。

- **AWS::SQS::Queue** - キュー
- **AWS::SNS::Topic** - トピック
- **AWS::SNS::Subscription** - キューとトピックの間のサブスクリプション
- **AWS::SQS::QueuePolicy** - このトピックがメッセージをキューに送信できるようにするリソースポリシー

{{% notice info %}} 
**AWS::CDK::Metadata** リソースは、ツールキットによって自動的に各スタックに追加されます。
AWS CDKチームはこれを分析のために使用し、セキュリティ上の問題があるバージョンを特定できるようにします。
詳細については、AW​​S CDKユーザーガイドの[Version Reporting](https://docs.aws.amazon.com/cdk/latest/guide/tools.html)を参照してください。
このワークショップの残りの説明では、メタデータリソースに関する詳細は省略します。
{{% /notice %}}
