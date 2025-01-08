CREATE VPC USING TERRAFORM MODULE- CREATE THE VPC.TF FILE IN THE PROJECT FOLDER

// Specify the provider for aws_availibility_zones
Provider "aws" {
  region = "eu-central-1" - Specify the region we are querying the region
}
// Variables are defined below
variable vpc_cidr_block {}
variable private_subnet_cidr_blocks {}
variable public_subnet_cidr_blocks {}

// data will query AWS to give us all the AZs in the region.
data "aws_availability_zones" "azs" {}
-----------------------------------------------------------------
//Existing ready module to create VPC for the EKS cluster.
module "myapp-vpc" {
  source  = "terraform-aws-modules/vpc/aws"
  version = "5.17.0"

-----------------------------------------------------------------
  name = "myapp-vpc" - Name of the VPC inside AWS
  cidr = var.vpc_cidr_block - IP range specified from tfvars file.
  private_subnets = var.private_subnet_cidr_blocks - Assign IP to private subnets (3 for 3 different Avl zones)
  public_subnets = var.public_subnet_cidr_blocks - Assign IP to public subnets (3 for 3 different Avl zones)
  azs = data.aws_availability_zones.azs.names - will give us the names of AZs (names attribute is specified in resources in Terraform registry) Distributed in 3 AZs

  enable_nat_gateway = true - 
  single_nat_gateway = true - created a shared common gateway b/w subnets to share internet
  enable_dns_hostnames = true - EC2 instances will be assigned private and private DNS names

// these tags are used to detect the resources and also referencing components (Cloud controller manager needs to know which resource it needs to talk to, detect and identify)
  tags = {
    "kubernetes.io/cluster/myapp-eks-cluster" = "shared" (name of the cluster)
  }
// tags are required to identify and help CCM for connecting
  public_subnet_tags = {
    "kubernetes.io/cluster/myapp-eks-cluster" = "shared"
    "kubernetes.io/role/elb" = 1 
  }
// tags are required to identify and help CCM for connecting
  private_subnet_tags = {
    "kubernetes.io/cluster/myapp-eks-cluster" = "shared"
    "kubernetes.io/role/internal-elb" = 1 
  }
}
