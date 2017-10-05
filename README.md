Infrastructure for ACC created with Terraform
=====================================

This is the infrastructure as code provided by Terraform. Please read the documentation of Terraform in https://www.terraform.io/docs/index.html

About Provisioned Infrastructure
--------------------------------

1. All components created and versioned were tested on a demo AWS account and it worked as expected
2. We are using custom values for some params, but these depends on the output of the common provisioning or from the preexisting components in the target AWS Account.
3. Check the related graph in the common/files/common.png and prod/files/prod.png

Check current values of Infrastructure components
-------------------------------------------------
1. Go to common and environment(prod) directory and execute `terraform output` this shows the current IDs for common components. You can also execute `terraform show`
2. According to common infrastructure components IDs, you can use the related values to provision an specific environment infrastructure.
3. please NOTE that the terraform.tfstate is NOT versioned as long as this is a basis for provisioning the final and stable Architecture version, but as soon as this is complete, you SHOULD enable versioning of this file editing the .gitignore file. The terraform.tfstate is IMPORTANT to track changes.

How to custom the environment params params.tf
----------------------------------------------
1. Get the output from the common infrastructure `terraform output` i.e
```
main_vpc_id = vpc-123abcd
network_acl_web_access_id = acl-123abcd
role_policy_cloudwatchlogs_arn = arn:aws:iam::123456789:policy/CloudWatchLogsACCMain
route_table_public_id = rtb-123abcd
security_group_ssh_access_id = sg-123abcd
security_group_web_access_id = sg-456abcd
zone_id = Z322RSAQ9C5XJ5
```

2. In the prod/stage/dev environment you should replace the following values in this way:

*General Parameters:*

```

variable "params" {
  type = "map"

  default = {
    main_vpc_id = "<output of common plan: main_vpc_id>"
    public_key = "<public-key to access instances>"
    route_table = "<output of common plan: route_table_public_id>"
    security_group_ssh_access_id = "<output of common plan: security_group_ssh_access_id>"
    security_group_web_access_id = "<output of common plan: security_group_web_access_id>"
    public_key_suffix = "web(you can custom this value)"
    instance_type = "t2.micro(You can custom this according to required instance type)"
    instance_ami_id = "<After provisioning instance with Ansible use this AMI ID>"
    role_name = "webMainACC(You can choose another name for the role)"
    instance_root_block_size = <Use an integer value for the size of the volume, e.g 100>
    instance_volume_device = "/dev/sdf"
    cloudwatchlogs_arn = "arn:aws:iam::<account_id>:policy/CloudWatchLogsACCMain"
    acm_certificate_domain = "acc-aws.cloud" (This should be the name of the production domain)
  }
}
```

*Database Parameters*

```

variable "database" {
  type = "map"

  default = {
    #These parameters are for a demo account fore reference while provisioning in demo account, please DON'T version the prod data.
    user = "<project_name>"
    dbname = "<project_name_environment>"
    password = "<Master Database Password>"
    dbtype = "<Instance type>"
    cluster_identifier = "aurora-cluster-<project name>"
    subnet_group_name = "<project_name_environment>"
  }
}
```

3. If we got the following requirements:

```

- EBS Volume Size: 100Gb
- Production Domain: acc-aws.cloud
- Instance Type: t2.large
- Database Instance Type: db.t2.large
- AWS Account ID: 123456789
- DNS of Project: acc-aws.cloud
- Project Name: ACC
```
Taking the example for the common infrastructure Output in step 1, then the custom parameters should be like this:

*General Parameters:*

```

variable "params" {
  type = "map"

  default = {
    main_vpc_id = "vpc-123abcd"
    #For the public_key you must create a new key pair and copy here.
    public_key = "ssh-rsa ***********== user@domain"
    route_table = "rtb-123abcd"
    security_group_ssh_access_id = "sg-123abcd"
    security_group_web_access_id = "sg-456abcd"
    public_key_suffix = "web"
    instance_type = "t2.micro"
    instance_ami_id = "ami-123abc"
    role_name = "webMainACC"
    instance_root_block_size = 100
    instance_volume_device = "/dev/sdf"
    cloudwatchlogs_arn = "arn:aws:iam::123456789:policy/CloudWatchLogsACCMain"
    acm_certificate_domain = "acc-aws.cloud"
  }
}
```

*Database Parameters*

```
variable "database" {
  type = "map"

  default = {
    user = "acc"
    dbname = "acc_prod"
    #Create a strong password, share with lastpass, DON'T version password here.
    password = "***********"
    dbtype = "db.t2.large"
    cluster_identifier = "aurora-cluster-acc"
    subnet_group_name = "acc_prod"
  }
}
```

4. Execute the plan and apply changes in the target infrastructure. Test the created components, update code if required and then test again.

5. Please NOTE that the *terraform.tfstate* is NOT currently versioned as long as this is a basis for provisioning the final and stable Architecture version, but as soon as this is complete, you *SHOULD enable* versioning of this file editing the .gitignore file. The *terraform.tfstate is IMPORTANT* to track changes.
