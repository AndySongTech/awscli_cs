# awscli
## awscli installation
```shell
brew install awscli
```
- refer: https://aws.amazon.com/cli/
## aws_completer installation
**zshrc**
```shell
which aws_completer
export PATH=/usr/local/bin:$PATH
source ~/.bash_profile
autoload bashcompinit && bashcompinit
complete -C '/usr/local/bin/aws_completer' aws
```
Add the command to ~/.zshrc to run it each time you open a new shell.
```
cat <<EOF >>~/.zshrc
complete -C '/usr/local/bin/aws_completer' aws
EOF
```
- refer: https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-completion.html
## aws configure 
```shell
$ aws configure
AWS Access Key ID: AKIAxxxFODNN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFExxxI/K7MDENG/bPxRfiCYEXAMPLEKEY
Default region name [None]: us-west-2
Default output format [None]: json

$ cd ~/.aws
$ tree                                           
├── config
├── credentials
└── ops-hub
    └── logs
        ├── main.2020-10-25.log
        ├── main.2020-10-28.log
        ├── renderer.2020-10-25.log
        ├── renderer.2020-10-28.log
        ├── worker.2020-10-25.log
        └── worker.2020-10-28.log

2 directories, 8 files
```
AWS prompts you to enter your Access Key ID and Secret Access Key and stores them in ~/.aws/credentials:
```
[default]
aws_access_key_id=AKIAIOSFODNN7EXAdPLE
aws_secret_access_key=wJalrXUtnFEMI/K7sdENG/bPxRfiCYEXAMPLEKEY
```
It also stores the other settings you entered in ~/.aws/config:
```
[default]
region=us-west-2
output=json
```
Check config list 
```
$ aws configure list                                                                                                                                               
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                <not set>             None    None
access_key     ****************ABCD shared-credentials-file
secret_key     ****************EDCS shared-credentials-file
    region                us-west-2      config-file    ~/.aws/config
```
## manage aws cli for multiple accounts
```shell
$ aws configure --profile aws-cn
AWS Access Key ID: AKIAxxx----NN7EXAMPLE
AWS Secret Access Key: wJalrXUtnFExxxI/K7M------RfiCYEXAMPLANDY
Default region name [None]: cn-northwest-1
Default output format [None]: json

# check the new credentials/config 
$ cat ~/.aws/credentials
[default]
aws_access_key_id=YOUR_ACCESS_KEY
aws_secret_access_key=YOUR_SECRET_ACCESS_KEY

[cn-aws]
aws_access_key_id=USER_2_ACCESS_KEY
aws_secret_access_key=USER_2_SECRET_ACCESS_KEY

$ cat ~/.aws/config
[default]
region=YOUR_REGION
output=json

[profile cn-aws]
region=USER_2_REGION
output=json

# check configure list
$ aws configure list --profile cn-aws
      Name                    Value             Type    Location
      ----                    -----             ----    --------
   profile                   aws-cn           manual    --profile
access_key     ****************UFTB shared-credentials-file
secret_key     ****************kJd0 shared-credentials-file
    region           cn-northwest-1      config-file    ~/.aws/config
    
# When you have multiple profiles configured, every time you run a command in the CLI, you have to use the “--profile” flag to specify which profile to use. However, it is really annoying to put this flag with every command. You can easily escape this boring task by exporting an environment variable in your current terminal.
$ aws s3 ls --profile cn-aws    # access cn-aws account by --profile cn-aws

# if you don't want type --profile every time, try below export
$ export AWS_PROFILE=cn-aws

# when you access default profile
$ aws s3 ls --profile default 

# if you want to change the profile to default, check below export
$ export AWS_PROFILE=default

```
## cli reference
- refer: https://docs.aws.amazon.com/cli/latest/reference/#available-services
- cheatsheet: https://www.bluematador.com/learn/aws-cli-cheatsheet
```shell
aws help   # list all the available commands 
```
### ec2
```shell
aws ec2 describe-instances   # output all ec2 instances
aws ec2 describe-instances >~/Downloads/instance.txt  # output the EC2 instances to a file, defualt format is json. Convert to csv by https://json-csv.com/ 
aws ec2 describe-instances --instance-ids "instanceid1" "instanceid2"  # output the specified instance
aws ec2 describe-instances --region us-west-2 --profile default # output the instance in specfied region
aws resourcegroupstaggingapi get-resources
aws configservice get-discovered-resource-counts --resource-types "AWS::EC2::Instance" --region us-west-2
aws ec2 start-instances --instance-ids "instanceid1" "instanceid2"
aws ec2 stop-intances --instance-ids "instanceid1" "instanceid2"
aws ec2 run-instances --image-id ami-b6b62b8f --security-group-ids sg-xxxxxxxx --key-name mykey --block-device-mappings "[{\"DeviceName\": \"/dev/sdh\",\"Ebs\":{\"VolumeSize\":100}}]" --instance-type t2.medium --count 1 --subnet-id subnet-e8330c9c --associate-public-ip-address
(Note: 若不指定subnet-id则会在默认vpc中去选，此时若指定了非默认vpc的安全组会出现请求错误。如无特殊要求，建议安全组和子网都不指定，就不会出现这种问题。)
```
- **check region & AZ**
 ```shell
aws resource-name describe-regions  # list all the regions that supported the specified service, like ec2, rds, workspaces, cloudwatch, s3...
aws ec2 describe-regions   # list all the regions that supported ec2 service
aws s3 describe-regions
aws ec2 describe-availability-zones --region region-name  # list if ec2 service is available in specifed region AZ.
```
- **check metadata & userdata**
```
curl http://169.254.169.254/latest/meta-data／
curl http://169.254.169.254/latest/user-data／
```
- **check ami & key pair**
```shell
aws ec2 describe-images
aws ec2 describe-key-pairs
aws ec2 create-key-pair --key-name mykeyname
```
- **security group**
```shell
aws ec2 create-security-group --group-name mygroupname --description mydescription --vpc-id vpc-id (若不指定vpc，则在默认vpc中创建安全组)
aws ec2 authorize-security-group-ingress --group-id sg-xxxxyyyy --protocol tcp --port 22 --cidr 0.0.0.0/0
aws ec2 authorize-security-group-ingress --group-id sg-xxxxyyyy --protocol tcp --port 9999 --source-group sg-xxxxxxxx
```
### rds
```shell
aws rds describe-db-instances # output the rds info 
aws rds describe-db-instances --region us-west-2  # output the rds in specifed region
aws rds describe-db-instances --region us-west-2 --profile default   # output the rds in specifed region and profile 
aws rds describe-reserved-db-instances  # output reserved rds info
aws rds describe-reserved-db-instances --region us-west-2   # output the reserved rds in specifed region  
aws rds describe-reserved-db-instances --region us-west-2 --profile default   # output the reserved rds in specifed region and profile 
aws rds describe-db-instances | jq -r '.DBInstances[]|{DBInstanceIdentifier}'|grep DBInstanceIdentifier |cut -d: -f 2|tr -d '"" '
aws rds describe-db-instances | jq -r '.DBInstances[]|.DBInstanceIdentifier' # the same function like above command
```
### autoscaling
```shell
aws autoscaling describe-auto-scaling-groups   # output the autoscaling groups
aws autoscaling describe-auto-scaling-instances   # output the autoscaling instances
aws autoscaling describe-auto-scaling-instances --instance-ids [instance-id-1 instance-id-2 ...]  # output specified instance
aws autoscaling detach-instances --auto-scaling-group-name --instance-ids  # remove one instance from autoscaling group
aws autoscaling detach-instances --auto-scaling-group-name myasgroup --instance-ids instanceid1 instanceid2 [--should-decrement-desired-capacity|--no-should-decrement-desired-capacity]   # remove instance from autoscaling group
挂起AS流程
aws autoscaling suspend-process --auto-scaling-group-name mygroupname --scaling-processes AZRebalance|AlarmNotification|...
删除AS组
aws autoscaling delete-auto-scaling-group --auto-scaling-group-name groupname
```
### s3
```
aws s3 ls    # list all buckets under current account
aws s3 ls help  # check help
aws s3 ls s3://bucket   # list objects of specifiy bucket  
aws s3 ls s3://bucket/prefix  # list subfolder objects
aws s3 ls s3://mybucket --recursive   # recursively listing all prefixes and objects
aws s3 ls s3://mybucket --recursive --human-readable --summarize
aws s3 ls s3://andy-terraform-bucket/terraform --recursive  # list prefix files
aws s3 mb s3://andysongbucket  # create a bucket
aws s3 rb s3://andysongbucket   # remove a bucket
aws s3 cp /to/local/path s3://bucket/prefix   # copy local file to s3
aws s3 cp s3://bucket/prefix /to/local/path   # copy s3 file to local 
aws s3 cp s3://bucket1/prefix1 s3://bucket2/prefix2   # copy s3 file to another s3
aws s3 cp s3://andy-terraform-bucket/terraform/training/terraform.tfstate .  # copy s3 file to local current dir
aws sync [--delete] /to/local/dir s3://bucket/prefixdir
aws sync [--delete] s3://bucket/prefixdir /to/local/dir
aws sync [--delete] s3://bucket1/prefixdir1 s3://bucket2/prefixdir2

```

### iam
```shell
aws iam list-users  # list all users
aws iam create-role MY-ROLE-NAME --assum-role-policy-document file://path/to/trustpolicy.json
aws iam put-role-policy --role-name MY-ROLE-NAME --policy-name MY-PERM-POLICY --policy-document file://path/to/permissionpolicy.json
aws iam create-instance-profile --instance-profile-name MY-INSTANCE-PROFILE
aws iam add-role-to-instance-profile --instance-profile-name MY-INSTANCE-PROFILE --role-name MY-ROLE-NAME
aws iam list-roles |jq '[.Roles[]|{RoleName: .RoleName}]'| grep RoleName |tr -d '"":' # get roles name list from json output
aws iam list-roles |jq '.Roles[]|{RoleName: .RoleName}'| grep RoleName |tr -d '"":' # the same result without []
```
### ecr
```
aws ecr list-images --repository-name kube-apiserver   # check repo image
docker images  # list all docker images
docker tag hello-world:latest aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest # tag the local image to aws ecr
docker push aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest  # upload the tag the image to aws ecr
```


### Config
Create profiles
```
aws configure --profile profilename
```
Output format
```
aws configure output format {json, yaml, yaml-stream, text, table}
```
Specify your AWS Region
```
aws configure region (region-name)
```
 
### API Gateway
List API Gateway IDs and Names
```
aws apigateway get-rest-apis | jq -r ‘.items[ ] | .id+” “+.name’
```
List API Gateway keys
```
aws apigateway get-api-keys | jq -r ‘.items[ ] | .id+” “+.name’
```
List API Gateway domain names
```
aws apigateway get-domain-names | jq -r ‘.items[ ] | .domainName+” “+.regionalDomainName’
```
List resources for API Gateway
```
aws apigateway get-resources --rest-api-id ee86b4cde | jq -r ‘.items[ ] | .id+” “+.path’
```
Find Lambda for API Gateway resource
```
aws apigateway get-integration --rest-api-id (id) --resource-id (resource id) --http-method GET | jq -r ‘.uri’
```
### Amplify
List Amplify apps and source repository
```
aws amplify list-apps | jq -r ‘.apps[ ] | .name+” “+.defaultDomain+”
```
### CloudFront
List CloudFront distributions and origins
```
aws cloudfront list-distributions | jq -r ‘.DistributionList.Items[ ] | .DomainName+” “+.Origins.Items[0].DomainName’
```
Create a new invalidation
```
aws cloudfront create-invalidation [distribution-id]
```
### CloudWatch
List information about an alarm
```
aws cloudwatch describe-alarms | jq -r ‘.MetricAlarms[ ] | .AlarmName+” “+.Namespace+” “+.StateValue’
```
Delete an alarm or alarms (you can delete up to 100 at a time)
```
aws cloudwatch delete-alarms --alarm-names (alarmnames)
```
Set a alarm state(Temporarily sets the state of an alarm for testing purposes)
```
aws cloudwatch set-alarm-state --alarm-name "RebootEC2OnHighCPU" --state-value ALARM --state-reason "Testing"
```
Refer: https://docs.aws.amazon.com/cli/latest/reference/cloudwatch/set-alarm-state.html  
### Cognito
List user pool IDs and names
```
aws cognito-idp list-user-pools --max-results 60 | jq -r ‘.UserPools[ ] | .Id+” “+.Name’
```
List phone and email of all users
```
aws cognito-idp list-users --user-pool-id (resource) | jq -r ‘.Users[ ].Attributes | from_entries | .sub + “ “ + .phone_number + “ “ + .email’
```
 
### DynamoDB
List DynamoDB tables
```
aws dynamodb list-tables | jq -r .TableNames [ ]
```
Get all items from a table
```
aws dynamodb scan --table-name events
```
Get item count from a table
```
aws dynamodb scan --table-name events --select count | jq .ScannedCount
```
Get item using key
```
aws dynamodb get-item --table-name events --key ‘{“email””"email@example.com”}}’
```
Get specific fields from an item
```
aws dynamodb get-item --table-name events --key ‘{“email””"email@example.com"}}’ --attributes-to-get event_type
```
Delete item using key
```
aws dynamodb delete-item --table-name events --key ‘{“email””email@domain.com”}}’
```
 
### EBS
Complete a Snapshot
```
aws ebs complete-snapshot (snapshot-id)
```
Start a Snapshot
```
aws ebs start-snapshot --volume-size (value)
```
Get a Snapshot block
```
aws ebs get-snapshot-block
--snapshot-id (value)
--block-index (value)
--block-token (value)
```
 
### EC2
List Instance ID, Type and Name
```
aws ec2 describe-instances | jq -r '.Reservations[].Instances[]|.InstanceId+" "+.InstanceType+" "+(.Tags[] | select(.Key == "Name").Value)'
```
List Instances with public IP address and Name
```
aws ec2 describe-instances --query 'Reservations[*].Instances[?not_null(PublicIpAddress)]' | jq -r '.[][]|.PublicIpAddress+" "+(.Tags[]|select(.Key=="Name").Value)'
```
List VPCs and CIDR IP Block
```
aws ec2 describe-vpcs | jq -r '.Vpcs[]|.VpcId+" "+(.Tags[]|select(.Key=="Name").Value)+" "+.CidrBlock'
```
List Subnets for a VPC
```
aws ec2 describe-subnets --filter Name=vpc-id,Values=vpc-0d1c1cf4e980ac593 | jq -r '.Subnets[]|.SubnetId+" "+.CidrBlock+" "+(.Tags[]|select(.Key=="Name").Value)'
```
List Security Groups
```
aws ec2 describe-security-groups | jq -r '.SecurityGroups[]|.GroupId+" "+.GroupName'
```
Print Security Groups for an Instance
```
aws ec2 describe-instances --instance-ids i-0dae5d4daa47fe4a2 | jq -r '.Reservations[].Instances[].SecurityGroups[]|.GroupId+" "+.GroupName'
```
Edit Security Groups of an Instance
```
aws ec2 modify-instance-attribute --instance-id i-0dae5d4daa47fe4a2 --groups sg-02a63c67684d8deed sg-0dae5d4daa47fe4a2
```
Print Security Group Rules as FromAddress and ToPort
```
aws ec2 describe-security-groups --group-ids sg-02a63c67684d8deed | jq -r '.SecurityGroups[].IpPermissions[]|. as $parent|(.IpRanges[].CidrIp+" "+($parent.ToPort|tostring))'
```
Add Rule to Security Group
```
aws ec2 authorize-security-group-ingress --group-id sg-02a63c67684d8deed --protocol tcp --port 443 --cidr 35.0.0.1
```
Delete Rule from Security Group
```
aws ec2 revoke-security-group-ingress --group-id sg-02a63c67684d8deed --protocol tcp --port 443 --cidr 35.0.0.1
```
Edit Rules of Security Group
```
aws ec2 update-security-group-rule-descriptions-ingress --group-id sg-02a63c67684d8deed --ip-permissions 'ToPort=443,IpProtocol=tcp,IpRanges=[{CidrIp=202.171.186.133/32,Description=Home}]'
```
Delete Security Group
```
aws ec2 delete-security-group --group-id sg-02a63c67684d8deed
```
### ECS
Create an ECS cluster
```
aws ecs create-cluster --cluster-name=NAME --generate-cli-skeleton
```
Create an ECS service
```
aws ecs create-service
```
 
### EKS
Create a cluster
```
aws eks create-cluster --name (cluster name)
```
Delete a cluster
```
aws eks delete-cluster --name (cluster name)
```
List descriptive information about a cluster
```
aws eks describe-cluster --name (cluster name)
```
List clusters in your default region
```
aws eks list-clusters
```
Tag a resource
```
aws eks tag-resource --resource-arn (resource_ARN) --tags (tags)
```
Untag a resource
```
aws eks untag-resource --resource-arn (resource_ARN) --tag-keys (tag-key)
```
### ElastiCache
Get information about a specific cache cluster
```
aws elasticache describe-cache-clusters | jq -r ‘.CacheClusters[ ] | .CacheNodeType+” “+.CacheClusterId’
```
List ElastiCache replication groups
```
aws elasticache describe-replication-groups | jq -r ‘.ReplicationGroups [ ] | .ReplicationGroupId+” “+.NodeGroups[ ].PrimaryEndpoint.Address’
```
List ElastiCache snapshots
```
aws elasticache describe-snapshots | jq -r ‘.Snapshots[ ] | .SnapshotName’
```
Create ElastiCache snapshot
```
aws elasticache create-snapshot --snapshot-name backend-login-hk-snap-1 --replication-group-id backend-login-hk --cache-cluster-id backend-login-hk
```
Delete ElastiCache snapshot
```
aws elasticache delete-snapshot --snapshot-name login-snap-1
```
Scale up/down ElastiCache replica
```
aws elasticache increase-replica-count --replication-group-id backend-login --apply-immediately
aws elasticache decrease-replica-count --replication-group-id backend-login --apply-immediately
```
### ELB
List ELB Hostnames
```
aws elbv2 describe-load-balancers --query ‘LoadBalancers[*].DNSName’ | jq -r ‘to_entries[ ] | .value’
```
List ELB ARNs
```
aws elbv2 describe-load-balancers | jq -r ‘.LoadBalancers[ ] | .LoadBalancerArn’
```
List of ELB target group ARNs
```
aws elbv2 describe-target-groups | jq -r ‘.TargetGroups[ ] | .TargetGroupArn’
```
Find instances for a target group
```
aws elbv2 describe-target-health --target-group-arn arn:aws:elasticloadbalancing:ap-northwest-1:20394823094:targetgroup/wordpress-ph/203942b32a23 | jq -r ‘.TargetHealthDescriptions[ ] | .Target.Id’
```
 
### IAM Group
List groups
```
aws iam list-groups | jq -r .Groups[ ].GroupName
```
Add/Delete groups
```
aws iam create-group --group-name (groupName)
```
List policies and ARNs
```
aws iam list-policies | jq -r ‘.Policies[ ]|.PolicyName+” “+.Arn’
aws iam list-policies --scope AWS | jq -r ‘.Policies[ ]|.PolicyName+” “+.Arn’
aws iam list-policies --scope Local | jq -r ‘.Policies[ ]|.PolicyName+” “+.Arn’
```
List user/group/roles for a policy
```
aws iam list-entities-for-policy --policy-arn arn:aws:iam:2308345:policy/example-ReadOnly
```
List policies for a group
```
aws iam list-attached-group-policies --group-name (groupname)
```
Add policy to a group
```
aws iam attach-group-policy --group-name (groupname) --policy-arn arn:aws:iam::aws:policy/exampleReadOnlyAccess
```
Add user to a group
```
aws iam add-user-to-group --group-name (groupname) --user-name (username)
```
Remove user from a group
```
aws iam remove-user-from-group --group-name (groupname) --user-name (username)
``` 
List users in a group
```
aws iam get-group --group-name (groupname)
```
List groups for a user
```
aws iam list-groups-for-user --user-name (username)
```
Attach/detach policy to a group
```
aws iam attach-group-policy --group-name (groupname) --policy-arn arn:aws:iam::aws:policy/DynamoDBFullAccess
aws iam detach-group-policy --group-name (groupname) --policy-arn arn:aws:iam::aws:policy/DynamoDBFullAccess
```
### IAM User
List userId and UserName
```
aws iam list-users | jq -r ‘.Users[ ]|.UserId+” “+.UserName’
```
Get single user
```
aws iam get-user --user-name (username)
```
Add user
```
aws iam create-user --user-name (username)
```
Delete user
```
aws iam delete-user --user-name (username)
```
List access keys for user
```
aws iam list-access-keys --user-name (username) | jq -r .AccessKeyMetadata[ ].AccessKeyId
```
Delete access key for user
```
aws iam delete-access-key --user-name (username) --access-key-id (accessKeyID)
```
Activate/deactivate access key for user
```
aws iam update-access-key --status Active --user-name (username) --access-key-id (access key)
aws iam update-access-key --status Inactive --user-name (username) --access-key-id (access key)
```
Generate new access key for user
```
aws iam create-access-key --user-name (username) | jq -r ‘.AccessKey | .AccessKeyId+” “+.SecretAccessKey’
```

### Lambda
List Lambda functions, runtime, and memory
```
aws lambda list-functions | jq -r ‘.Functions[ ] | .FunctionName+” “+.Runtime+” “+(.MemorySize|tostring)’
```
List Lambda layers
```
aws lambda list-layers | jq -r ‘.Layers[ ] | .LayerName’
```
List source event for Lambda
```
aws lambda list-event-source-mappings | jq -r ‘.EventSourceMappings[ ] | .FunctionArn+” “+.EventSourceArn’
```
Download Lambda code
```
aws lambda get-function --function-name DynamoToSQS | jq -r .Code.Location
```
 
### RDS
List DB clusters
```
aws rds describe-db-clusters | jq -r ‘.DBClusters[ ] | .DBClusterIdentifier+” “+.Endpoint’
```
List DB instances
```
aws rds describe-db-instances | jq -r ‘.DBInstances[ ] | .DBInstanceIdentifier+” “+.DBInstanceClass+” “+.Endpoint.Address’
```
Take DB Instance Snapshot
```
aws rds create-db-snapshot --db-snapshot-identifier snapshot-1 --db-instance-identifier dev-1
aws rds describe-db-snapshots --db-snapshot-identifier snapshot-1 --db-instance-identifier general
```
Take DB cluster snapshot
```
aws rds create-db-cluster-snapshot --db-cluster-snapshot-identifier
```

### Route53
Create hosted zone
```
aws route53 create-hosted-zone --name exampledomain.com
```
Delete hosted zone
```
aws route53 delete-hosted-zone --id example
```
Get hosted zone
```
aws route53 get-hosted-zone --id example
```
List hosted zones
```
aws route53 list-hosted-zones
```
Create a record set       
To do this you’ll first need to create a JSON file with a list of change items in the body and use the CREATE action. For example the JSON file would look like this.
```
{
     "Comment": "CREATE/DELETE/UPSERT a record",
     "Changes": [{
     "Action": "CREATE",
          "ResourceRecordSet":{
               "Name": "a.example.com",
               "Type": "A",
               "TTL": 300,
          "ResourceRecords":[{"Value":"4.4.4.4"}]
}}]
}
```
Once you have a JSON file with the correct information like above you will be able to enter the command
```
aws route53 change-resource-record-sets --hosted-zone-id (zone-id) --change-batch file://exampleabove.json
```
Update a record set       

To do this you’ll first need to create a JSON file with a list of change items in the body and use the UPSERT action. This will either create a new record set with the specified value, or updates a record set if it already exists. For example the JSON file would look like this.
```
{
     "Comment": "CREATE/DELETE/UPSERT a record",
     "Changes": [{
     "Action": "UPSERT",
          "ResourceRecordSet":{
               "Name": "a.example.com",
               "Type": "A",
               "TTL": 300,
          "ResourceRecords": [{"Value":"4.4.4.4"}]
}}]
}
```
Once you have a JSON file with the correct information like above you will be able to enter the command
```
aws route53 change-resource-record-sets --hosted-zone-id (zone-id) --change-batch file://exampleabove.json
``` 
Delete a record set       

To do this you’ll first need to create a JSON file with a list of the record set values you want to delete in the body and use the DELETE action. For example the JSON file would look like this.
```
{
     "Comment": "CREATE/DELETE/UPSERT a record",
     "Changes": [{
     "Action": "DELETE",
          "ResourceRecordSet": {
               "Name": "a.example.com",
               "Type": "A",
               "TTL": 300,
          "ResourceRecords": [{"Value":"4.4.4.4"}]
}}]
}
```
Once you have a JSON file with the correct information like above you will be able to enter the following command.
```
aws route53 change-resource-record-sets --hosted-zone-id (zone-id) --change-batch file://exampleabove.json
``` 
 
### S3
List Buckets
```
aws s3 ls
```
List files in a Bucket
```
aws s3 ls s3://mybucket
``` 
Create Bucket
```
aws s3 mb s3://bucket-name
make_bucket: bucket-name
```
Delete Bucket
```
aws s3 rb s3://bucket-name --force
```
Download S3 object to local
```
aws s3 cp s3://bucket-name
download: ./backup.tar from s3://bucket-name/backup.tar
```
Upload local file as S3 object
```
aws s3 cp backup.tar s3://bucket-name
upload: ./backup.tar to s3://bucket-name/backup.tar
```
Delete S3 object
```
aws s3 rm s3://bucket-name/secret-file.gz .
delete: s3://bucket-name/secret-file.gz
```
Download bucket to local
```
aws s3 sync s3://bucket-name/ /media/pasport-ultra/backup
```
Upload local directory to bucket
```
aws s3 sync (directory) s3://bucket-name/
```
Share S3 object without public access
```
aws s3 presign s3://bucket-name/file-name --expires-in (time value)
https://bucket-name.s3.amazonaws.com/file-name.pdf?AWSAccessKeyId=(key)&Expires=(value)&Signature=(value)
```
 
### SNS
List SNS topics
```
aws sns list-topics | jq -r ‘.Topics[ ] | .TopicArn’
```
List SNS topic and related subscriptions
```
aws sns list-subscriptions | jq -r ‘.Subscriptions[ ] | .TopicArn+” “+.Protocol+” “+.Endpoint’
```
Publish to SNS topic
```
aws sns publish --topic-arn arn:aws:sns:ap-southeast-1:232398:backend-api-monitoring
```

### SQS
List queues
```
aws sqs list-queues | jq -r ‘.QueueUrls[ ]’
```
Create queue
```
aws sqs create-queue --queue-name public-events.fifo | jq -r .queueURL
```
Send message
```
aws sqs send-message --queue-url (url) --message-body (message)
```
Receive message
```
aws sqs receive-message --queue-url (url) | jq -r ‘.Messages[ ] | .Body’
```
Delete message
```
aws sqs delete-message --queue url (url) --receipt-handle (receipt handle)
```
Purge queue
```
aws sqs purge-queue --queue-url (url)
```
Delete queue
```
aws sqs delete-queue --queue-url (url)
```

- refer: https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html
