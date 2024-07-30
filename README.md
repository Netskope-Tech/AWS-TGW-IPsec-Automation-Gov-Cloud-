# Netskope-TGW-management - Failover automation solution for AWS TGW - Netskope IPsec tunnels

The solution deploys Lambda function that's been triggered by the CloudWatch event rule for IPsec tunnel status change. When called by the CloudWatch event rule, the function checks if the both tunnel to the Netskope PoP are down, and if so, it scans all the TGW route tables for the static route pointing to the TGW VPN attachments for the corresponding Site-to-Site VPN connection, and replaces them with the alternative static route to the failover PoP.

You must deploy this solution in any of the  AWS region and this template will also create AWS Global Network with Transit Gateway Network Manager activated. AWS Transit Gateway Network Manager is the global AWS service that monitors the status of IPsec Site-to-Site VPN connections and uses Amazon CloudWatch in the same region for alerting and logging. This solution used AWS Transit Gateway events in the same region to monitor your TGW in any region on the same account. Cross-account monitoring is not currently supported.

You need to deploy one instance of the solution per TGW. You also can customize the lambda function to work with multiple TGWs or to treat a group of TGW attachments differently.
The solution assumes all TGW VPC attachments have the static route pointing to the same TGW VPN attachments. Therefore, all VPCs connected to this TGW will be utilizing the same IPsec tunnel between AWS TGW and Netskope PoP. The single tunnel bandwidth is limited to 250 Mbps. You may scale this solution by spliting your VPCs to a number of groups and to route traffic for each group for its own redundant IPsec Site-to-Site VPN connections. The Lambda function can be customized to support this approach.

You may enable of disable fallback functionality. If enabled, the Lambda function will revert static routes to the primary Netskope PoP if both of its IPsec tunnels are up.
In addition to checking and updating routes when IPsec tunnel status changes, the same Lambda function being triggered every 10 minutes to check that there are no routes left pointing to the IPsec connection which is currently down. This is to prevent the unlikely situation when IPsec connections were intensively bouncing, and the this caused a race condition between Lambda function executions which caused the last execution time out. Note, that only one Lambda function execution can run at any point of time to avoid inconsistent results. Concurrency has been controlled using DynamoDB table also being created by this solution.

The CloudFormation stack creates the IAM role used by the Lambda function. This role implemented based on the least privilege access control model. To limit access to only TGW Attachments, therefore to the Route Tables that belong to this specific TGW, it uses IAM policy condition checking the Tags on the TGW Attachments. You must tag each of your TGW attachments with the tag "Key"="TGWName", "Value"="Your TGW Name". For example, "Key"="TGWName", "Value"="MyProdTGW-us-east-1".

Usage:
1. Download the yaml file from the github.
2. Do the required changes in the template and update all the below default parameters in the field according to your environment
   TGWRegion - The AWS region where your TGW is deployed
   TGWName - TGW name that will be used for access control. Your all TGW attachments must have an attribute "Key"="TGWName", "Value"="This parameter". For example, "MyProdTGW-us-east-1"
   TGWID - TGW ID. For example, tgw-01234567890123456
   TGWAttachmentID1 - TGW attachment ID for the first (primary) VPN. For example, tgw-attach-01234567890123456
   TGWAttachmentID2 - TGW attachment ID for the second (failover) VPN. For example, tgw-attach-01234567890123456
   TransitGatewayArn - TGW Arn which will be used while registering in the Network Manager with with Transit Gateway Network Manager
   Fallback - Yes/No for the route fallback support to the TGWAttachmentID1 if both of this IPsec tunnels became active.
3. Before executing the CloudFormation template also make sure to create the S3 bucket and folder in the same bucket containing the lambda in zip format as downloaded from the github: https://github.com/dyuriaws/Netskope-TGW- 
   management/blob/main/Lambda/IPsecManagementLambda.zip
   Update the below fields in the CloudFormation Template according to your values
   S3Bucket
   S3Prefix
4. After doing the required changes in CloudFormation Template and upload the same.
5. Optionally, enter the Tags for your CloudFormation stack and click Next.
6. Acknowledge creating IAM resources and click Create stack.
7. Additional link: https://docs.netskope.com/en/netskope-help/integrations-439794/ipsec-and-gre/netskope-ipsec-with-amazon-web-services/configure-an-ipsec-tunnel-for-an-aws-transit-gateway/#configure-failover-automation-for-the-aws-transit-gateway-1

Costing related to the Cloud Resources in AWS for the IPSec Failover Automation Solution.
AWS Network Manager: 720*0.5$ = 360$
AWS Lambda: 0.5$
DynamoDB: 0.25$
S3 Bucket: 0.15$

Total cost will be around 360.9$ +_5$
