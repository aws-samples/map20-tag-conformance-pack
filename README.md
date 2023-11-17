# AWS Config Conformance Pack for MAP 2.0 Tags

The Migration Acceleration Program (MAP 2.0) is a comprehensive and proven cloud migration program that’s the result of AWS’ experience migrating hundreds of enterprise customers, to help customers manage migration double bubble costs.
As customers move  existing on-premises workloads to AWS, the migrated workloads are identified through a tagging mechanism. Tagging is required as it is used to report the migrated workloads’ spend and generate appropriate credits.

MAP 2.0 eligible services list can be found here: https://s3-us-west-2.amazonaws.com/map-2.0-customer-documentation/included-services/MAP_Included_Services_List.pdf

Tagging key combinations are described here: https://s3.us-west-2.amazonaws.com/map-2.0-customer-documentation/html/AWSMapDocs/setting-up.html

A conformance pack is a collection of AWS Config rules and remediation actions that can be easily deployed as a single entity in an account and a Region or across an organization in AWS Organizations

The objective of this Conformance Pack is to identify MAP 2.0 eligible resources, created after MAP agreement was signed, residing in customer's account(s), that are not tagged or wrongly tagged, as per MAP 2.0 expectations. It contains the remediation action which properly tags identified resources.
However, only the customer can decide to apply the rectification action as only they know the details of their MAP 2.0 migration scope. 

This Conformance Pack is applicable only to customers that have signed a MAP deal from January 2023 onwards.

The AWS Config rules in this conformance pack will check compliance based on the following rules:
1. **Not Applicable**: When the resource does not qualify for MAP 2.0 credits or is created before the MAP signing date.
2. **Compliant**: The resource qualifies for MAP 2.0 credits, is created after the MAP signing date and has a map-migrated tag with the correct value.
3. **Noncompliant**: in all other cases.

This conformance pack checks for compliance on the following resource types:
* EC2 Instance
* EC2 Volume
* Backup Plan
* Backup Vault
* Elastic Load Balancer
* S3 Bucket
* RDS Instance
* RDS Cluster
* DynamoDB Table
* API Gateway

We are working to include more resources to this list in the future.

## Prerequisite

Before you deploy the solutions:
* The MAP 2.0 should be signed.
* The customer should be familiar with MAP 2.0 signature date, MPE number and tag values.They will be needed to initially configure the Conformance Pack.
* Initial [MAP 2.0 account setup](https://s3.us-west-2.amazonaws.com/map-2.0-customer-documentation/html/AWSMapDocs/getting-started-step1.html) should be completetd.
* AWS Config Conformace Pack [should be supported](https://docs.aws.amazon.com/config/latest/developerguide/conformance-packs.html#conformance-packs-regions) in the required region.
* [Setup the AWS Config](https://docs.aws.amazon.com/config/latest/developerguide/gs-console.html#gs-console-setting-up.title).
* [Create an S3 Bucket](https://docs.aws.amazon.com/AmazonS3/latest/userguide/creating-bucket.html) for the conformance pack and the code zip file.

## Deploying the solution

### Cost

The total cost of running this solution is dependent on the number of resource configuration items in your account and the number of rule evaluations that the conformance pack evaluates. The example below assumes you have a 1000 resource changes (creations, modification, deletions) per month with one rule evaluation each for a total cost of approximately $5 per month (in ap-south-1 region) with the breakdown as follows:

| Item      | Quantity | Approximate Price |
| ----------- | ----------- | --- |
| [AWS Config Configuration Items](https://aws.amazon.com/config/pricing/)      | 1000       | ~ $3 |
| [AWS Config Rule Evaluations](https://aws.amazon.com/config/pricing/)   | 1000        | ~ $1 |
| [AWS Lambda](https://aws.amazon.com/lambda/pricing/) | 1000 | < $1 | 

### Setup

Before you deploy the conformance pack:

* Zip the source file under `src/check_map_tags.py` and upload it to an S3 bucket. Note the bucket name and full path of the uploaded zip file.

*Note: The source file should be at the root of the zip folder.*

### Deployment

You can deploy the solution using AWS CLI or the AWS Console.

### CLI

Run the following command. Note:
1. Replace the `Value*` with the correct parameter values for your deployment
2. Replace the `ap-south-1` region with the one you require.
```
aws cloudformation deploy --stack-name map-tag --template-file map-tag.yml --capabilities CAPABILITY_IAM --parameter-overrides ZipBucketName=Value1 ZipFilePath=Value2 ConformancePackDeliveryBucket=Value3 TagValue=Value4 MapSignDate=Value5 --region ap-south-1
```

### AWS Console

1. Navigate to CloudFormation > Stack
2. Click on **Create Stack** > **With new resources (standard)**
3. Select **Template is ready**
4. Select **Upload a template file**
5. Choose the map-tag.yml and click on **Next**
6. On the **Specify Stack Details** screen, enter `map-tag` as the stack name. Fill out rest of the detailed asked on the screen and click on **Next**
7. Click on **Next** in the **Configure stack options screen**.
8. Review the configuration in this screen, then tick the **I acknowledge that AWS CloudFormation might create IAM resources.** box and click on **Submit** to start deployment.

CloudFormation can take anywhere from 3-10 minutes to deploy the template.

### Viewing the results

Navigate to AWS Config > Conformance Packs > Map20-Tag-Conformance-Pack to view the compliance score and the compliance status of your resources.

### Remediation

When your resources are marked as Noncompliant, you should add the necessary **map-migration** tags so that they will be counted towards your MAP 2.0 credits. If you are using Infrastructure as Code (IaaC) tools such as AWS CloudFormation or AWS CDK then make the necessary script/code changes to add those tags and update your resources. Otherwise, you can add tags using AWS Console, or using inbuilt remediation option in the AWS Config as described below:

1. Navigate to Config > Conformance Packs > Map20-Tag-Conformance-Pack
2. Click on the rule name which has Compliance status as *Noncompliance*
3. Select the radio button for the required noncompliant resource in the *Resources in Scope* list.
4. Click on the **Remediate** button to start the remediation process.

Remediation is done using AWS System Manager Automation Document. 

## Additional considerations

To deploy this custom rule to multiple accounts in organisation, you can also leverage AWS Cloudformation Stacksets.  Please be aware that this will deploy all the resources (Lambda functions, roles, automation documents, etc) in EACH account to which you deploy the stackset.
Follow the following links to learn how to deploy CloudFormation stacksets across multiple accounts in AWS Organizations:
https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/stacksets-getting-started-create.html
https://docs.aws.amazon.com/organizations/latest/userguide/services-that-can-integrate-cloudformation.html

You will also have to take the following considerations into account:
1. Each account to which you deploy the stack set must have read access to the S3 bucket which contains the Lambda function zip file
2. Each account to which you deploy the stack set must have read access to the conformance pack delivery S3 bucket
3. The name for the lambda execution IAM role for each stack you deploy must be unique as IAM roles are global
4. You must deploy the stackset from an account which has permissions to deploy to all accounts in your organization.

## Cleaning up

To clean up the resources:

### CLI

```
aws cloudformation delete-stack --stack-name map-tag --region ap-south-1
```

### AWS Console

Navigate to CloudFormation > Stacks. Select the **map-tag** stack and click on **Delete**

## Security

See [CONTRIBUTING](CONTRIBUTING.md#security-issue-notifications) for more information.

## License

This library is licensed under the MIT-0 License. See the LICENSE file.
