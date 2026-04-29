# Secure SSH Access to Private EKS Worker Nodes

## Objective
To provide secure administrative access to private EKS worker nodes without exposing them to the public internet via public IPs or direct inbound SSH access.

## Proposed Solution: AWS Systems Manager (SSM) Session Manager
Instead of deploying a traditional Bastion Host (which requires managing SSH keys, security groups with port 22 open, and an additional EC2 instance), the preferred and more secure approach is to use **AWS Systems Manager (SSM) Session Manager**.

### Why SSM Session Manager?
1. **No Public IPs Required**: The instances reside completely within the private subnets.
2. **No Inbound Ports**: Session Manager establishes outbound connections from the SSM Agent on the worker nodes to the SSM service, meaning no inbound rules (e.g., port 22) need to be opened in Security Groups.
3. **No SSH Key Management**: Access is controlled entirely by AWS Identity and Access Management (IAM), removing the need to manage and distribute static SSH keys.
4. **Audit and Logging**: All session activity can be logged centrally in AWS CloudTrail and Amazon S3 or CloudWatch logs.

### Implementation Details
The Terraform code provided in this assessment implements the prerequisites for this access strategy:

1. **SSM Agent**: Modern EKS optimized AMIs come with the SSM Agent pre-installed.
2. **IAM Permissions**: The EKS Node Group IAM role (`aws_iam_role.node`) has the `AmazonSSMManagedInstanceCore` managed policy attached. This grants the SSM Agent the necessary permissions to communicate with the Systems Manager service.
3. **Outbound Internet Access**: Since the worker nodes are in private subnets, they use the configured NAT Gateway in the VPC to route outbound traffic to the Systems Manager endpoints.

### How to Connect (The Workflow)
To connect to a private worker node securely, an authorized user (e.g., a DevOps engineer) would perform the following steps:

1. **Authenticate**: The user authenticates against the AWS account using their IAM credentials (CLI or AWS Management Console).
2. **Identify Instance**: Identify the specific instance ID of the worker node they need to access.
3. **Start Session**:
   - **Via AWS CLI**: Run the command `aws ssm start-session --target <instance-id>`. (Requires the Session Manager plugin to be installed locally).
   - **Via AWS Console**: Navigate to Systems Manager > Session Manager > Start a Session, select the desired worker node, and click "Start session" to open a secure shell directly in the browser.

### Conclusion
By leveraging SSM Session Manager, we achieve a robust, fully private, and auditable method for accessing EKS worker nodes, perfectly aligning with the assessment's strict security constraints.
