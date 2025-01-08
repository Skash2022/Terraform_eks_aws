 
// add module from Terraform registry
module "eks" {
  source  = "terraform-aws-modules/eks/aws"
  version = "20.31.6"

  cluster_name = "myapp-eks-cluster" - Name of the cluster
  cluster_version = "1.31" - Version of the cluster
  cluster_endpoint_public_access  = true

  subnet_ids = module.myapp-vpc.private_subnets - list of subnets which we want worker nodes to be started in ( for security reason we need it in Pvt subnet with no internet access)
  vpc_id = module.myapp-vpc.vpc_id - 

// tags for your reference
  tags = {
    environment = "development" 
    application = "myapp"
  }
// enable auto EKS 
  cluster_compute_config = {
    enabled = true
  }
}
