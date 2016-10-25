# tf-aws-gitlab

Terraform config to create a single Gitlab host using Ubuntu 16.04 LTS and Omnibus Gitlab package

All actual values have been generalized, or gitignored. 

Steps to provision:

1. Prepare terraform.tfvars file:
   ```
   cp terraform.tfvars.example terraform.tfvars
   vim terraform.tfvars
   ```

   Existing AWS Resources
   ```
   # List existing VPC, IsDefault, CidrBlock  and Subnets
   ec2 describe-vpcs --query "Vpcs[].[VpcId,IsDefault,CidrBlock]" --output=table
   ec2 describe-subnets --filters "Name=vpc-id,Values=<vpc-id>"
   # List existing TLS Certs
   iam list-server-certificates --query "ServerCertificateMetadataList[].[ServerCertificateName,Arn]" --output=table
   # Add your ip to white list:
   curl ifconfig.co
   ```

   Review the gitlab_host_SG cidr block in main.tf 
   (Todo: add variable for vpc cidr block..)

2. set environment variables
   ```
   export TF_VAR_aws_access_key=AKIA...
   export TF_VAR_aws_secret_key=x823dzzawer...
   ```

3. Edit `bucket_policy.json`, follow [AWS steps](http://docs.aws.amazon.com/ElasticLoadBalancing/latest/DeveloperGuide/enable-access- logs.html#attach-bucket-policy)
   update principal and account id:
   ```
   # Get 12 Digit Acount ID
   iam get-user --output text --query='User.Arn' | grep -Eo '[[:digit:]]{12}'
   ```
4. Add domains to route53 hosted zone as CNAME for gitlab-elb

Create stack:

```
terraform plan
```

```
terraform apply
```

```
while true;do aws elb describe-instance-health --load-balancer-name gitlab-elb --query "InstanceStates[].State" --output=text;sleep 1; done
```

```
pbcopy < ~/.ssh/rsa_id.pub
```

```
open http://gitlab.honestbee.com
```

1. Set Password
2. Change Account Name
3. Add public key
4. Disable User Registration

### Gitlab Registry

Once a Repository has been created, docker images can be pushed to it:

```
docker login glr.honestbee.com
Username: <gitlab username>
Password: <gitlab password>
```

```
docker tag ... 
docker push
```

## Optional:

### Update AMIs

To change the Amazon Machine Images: [Ubuntu Cloud Images](https://cloud-images.ubuntu.com/locator/ec2/)

### SMTP setup:

[credentials](https://github.com/comerford/tf-aws-gitlab/blob/master/conf/gitlab.rb#L290) and [region](https://github.com/comerford/tf-aws-gitlab/blob/master/conf/gitlab.rb#L288) settings in gitlab.rb (will start, but mail won't work)
