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
aws s3 mb s3://andysongbucket  # create a bucket
aws s3 rb s3://andysongbucket   # remove a bucket
aws s3 cp /to/local/path s3://bucket/prefix   # copy local file to s3
aws s3 cp s3://bucket/prefix /to/local/path   # copy s3 file to local 
aws s3 cp s3://bucket1/prefix1 s3://bucket2/prefix2   # copy s3 file to another s3
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
```
### ecr
```
aws ecr list-images --repository-name kube-apiserver   # check repo image
docker images  # list all docker images
docker tag hello-world:latest aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest # tag the local image to aws ecr
docker push aws_account_id.dkr.ecr.us-east-1.amazonaws.com/hello-world:latest  # upload the tag the image to aws ecr
```
- refer: https://docs.aws.amazon.com/AmazonECR/latest/userguide/getting-started-cli.html
