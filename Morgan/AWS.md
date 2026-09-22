Summary — networking/VPC in this file

The VPC + subnets/security-groups are created by module.networking. It takes inputs like var.vpc_cidr, var.availability_zone_names and conditionally creates a data subnet via var.create_data_subnet.

Key networking outputs used across the module:

module.networking.vpc_id — VPC id consumed by many modules (EKS, privatelink, step_functions, postgresql_db, etc.).
module.networking.public_subnet_ids_map — public subnets (passed to module.gateway for ALB/ingress).
module.networking.private_app_subnet_ids_map — private app subnets (used by module.eks, module.batch, module.step_functions, privatelink, etc.).
module.networking.private_app_subnet_map — private subnet map (used by Neptune / Sagemaker).
module.networking.app_subnet_security_group_ids — app subnet SGs (used by EKS, Batch, Neptune, Step Functions, privatelink).
[module.networking.data_subnet_ids_map](terraform/modules/exchange/main.tf] and module.networking.data_subnet_security_group_id — data subnet + SG used by module.postgresql_db.
How networking is wired:
Public-facing ingress: module.gateway attaches to module.networking.public_subnet_ids_map and uses the certificate from module.certificate; it publishes ingress IPs used by module.domain_name.
Cluster & private workloads: module.eks and other private services use module.networking.private_app_subnet_ids_map and security groups from module.networking.app_subnet_security_group_ids.
Data plane (databases/private services): module.postgresql_db, module.neptune, module.sagemaker_notebook, and module.privatelink use the VPC, private/data subnets and corresponding SGs from module.networking.
Serverless / step functions: module.step_functions is run inside the VPC via module.networking.vpc_id, module.networking.private_app_subnet_ids_map and module.networking.app_subnet_security_group_ids.