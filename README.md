# Homework16

# aws ec2 create-vpc --cidr-block 10.0.0.0/16 --tag-specifications "ResourceType=vpc,Tags=[{Key=Name,Value=my-vpc}]" > readme.txt

{
    "Vpc": {
        "CidrBlock": "10.0.0.0/16",
        "DhcpOptionsId": "dopt-0e6f1ace6652cd3f5",
        "State": "pending",
        "VpcId": "vpc-056f94061c71b6ae7",
        "OwnerId": "623523139385",
        "InstanceTenancy": "default",
        "Ipv6CidrBlockAssociationSet": [],
        "CidrBlockAssociationSet": [
            {
                "AssociationId": "vpc-cidr-assoc-09dc7e515de1145ee",
                "CidrBlock": "10.0.0.0/16",
                "CidrBlockState": {
                    "State": "associated"
                }
            }
        ],
        "IsDefault": false,
        "Tags": [
            {
                "Key": "Name",
                "Value": "my-vpc"
            }
        ]
    }
}
## VPC_ID=$(aws ec2 describe-vpcs --filters "Name=tag:Name,Values=my-vpc" --query 'Vpcs[*].VpcId' --output text)
echo "VPC ID: $VPC_ID" >> readme.txt

VPC ID: vpc-056f94061c71b6ae7

aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.1.0/24 --availability-zone us-east-1a --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=public}]" >> readme.txt
{
    "Subnet": {
        "AvailabilityZone": "us-east-1a",
        "AvailabilityZoneId": "use1-az4",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.1.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0ad532f3986ea4c0d",
        "VpcId": "vpc-056f94061c71b6ae7",
        "OwnerId": "623523139385",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "public"
            }
        ],
        "SubnetArn": "arn:aws:ec2:us-east-1:623523139385:subnet/subnet-0ad532f3986ea4c0d",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}

aws ec2 create-subnet --vpc-id $VPC_ID --cidr-block 10.0.2.0/24 --availability-zone us-east-1b --tag-specifications "ResourceType=subnet,Tags=[{Key=Name,Value=private}]" >> readme.txt
{
    "Subnet": {
        "AvailabilityZone": "us-east-1b",
        "AvailabilityZoneId": "use1-az6",
        "AvailableIpAddressCount": 251,
        "CidrBlock": "10.0.2.0/24",
        "DefaultForAz": false,
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0dc0b947c2412e113",
        "VpcId": "vpc-056f94061c71b6ae7",
        "OwnerId": "623523139385",
        "AssignIpv6AddressOnCreation": false,
        "Ipv6CidrBlockAssociationSet": [],
        "Tags": [
            {
                "Key": "Name",
                "Value": "private"
            }
        ],
        "SubnetArn": "arn:aws:ec2:us-east-1:623523139385:subnet/subnet-0dc0b947c2412e113",
        "EnableDns64": false,
        "Ipv6Native": false,
        "PrivateDnsNameOptionsOnLaunch": {
            "HostnameType": "ip-name",
            "EnableResourceNameDnsARecord": false,
            "EnableResourceNameDnsAAAARecord": false
        }
    }
}

IGW_ID=$(aws ec2 create-internet-gateway --query 'InternetGateway.InternetGatewayId' --output text)
aws ec2 attach-internet-gateway --internet-gateway-id $IGW_ID --vpc-id $VPC_ID >> readme.txt

RTB_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query 'RouteTable.RouteTableId' --output text)
aws ec2 create-route --route-table-id $RTB_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID >> readme.txt

{
    "Return": true
}


PUBLIC_SUBNET_ID=$(aws ec2 describe-subnets --filters "Name=tag:Name,Values=public" --query "Subnets[0].SubnetId" --output text)
PUBLIC_ROUTE_TABLE_ID=$(aws ec2 create-route-table --vpc-id $VPC_ID --query "RouteTable.RouteTableId" --output text)
aws ec2 create-route --route-table-id $PUBLIC_ROUTE_TABLE_ID --destination-cidr-block 0.0.0.0/0 --gateway-id $IGW_ID >> readme.txt
aws ec2 associate-route-table --route-table-id $PUBLIC_ROUTE_TABLE_ID --subnet-id $PUBLIC_SUBNET_ID >> readme.txt

{
    "Return": true
}
{
    "AssociationId": "rtbassoc-0d5efed4eca47e92b",
    "AssociationState": {
        "State": "associated"
    }
}

aws ec2 describe-subnets --filters "Name=tag:Name,Values=private" --query "Subnets[*].{CIDR:CidrBlock}" >> readme.txt
[
    {
        "CIDR": "10.0.2.0/24"
    }
]


aws ec2 describe-images --owners 'amazon' --region us-west-2 --filters 'Name=architecture,Values=x86_64' 'Name=description,Values=Amazon Linux 2 *' --query "sort_by(Images, &CreationDate)[-1].ImageId"  --output text>>readmi.txt

ami-0720d173d2dc9052b
