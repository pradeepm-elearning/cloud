To change your AWS EC2 instance's network driver from the basic vif (Virtual Interface) to the high-performance ENA (Elastic Network Adapter), you must follow a three-part process: verifying current support, installing the driver, and enabling the ENA attribute via the AWS CLI. 
1. Verify Current Support
Before making changes, confirm if your instance is already using ENA or if the attribute is disabled. 
Check Linux Driver: Run ethtool -i eth0. If the output shows driver: vif, you are using the legacy driver. 

repost.aws
 +2
Check ENA Attribute: From your local machine (with AWS CLI configured), run:
aws ec2 describe-instances --instance-ids <instance-id> --query "Reservations[].Instances[].EnaSupport"
If it returns true, ENA is already enabled at the AWS level. 

docs.aws.amazon.com
 +4
2. Install the ENA Driver
The operating system must have the ENA driver installed before you switch the hardware attribute. 
Amazon Linux 2 / Ubuntu 18.04+: These usually have the ENA driver pre-installed. 
RHEL/CentOS/Legacy Linux:
Update your kernel: sudo yum update -y.
Install development tools: sudo yum install gcc kernel-devel-$(uname -r) git make -y.
Clone and build the Amazon ENA GitHub repo if it's not in your package manager. 

repost.aws
Windows: Download and install the latest driver from the AWS Windows ENA Driver page. 
3. Enable ENA Attribute (Required) 
Even with the driver installed, AWS will continue to present a vif interface unless you explicitly enable the EnaSupport flag on the instance.
Stop the instance: You cannot modify this attribute while the instance is running.
Run the following command from your local terminal:
bash
aws ec2 modify-instance-attribute --instance-id <instance-id> --ena-support
Use code with caution.
Start the instance.
Verify: Run ethtool -i eth0 again. The driver should now list as ena. 

repost.aws
 +4
Troubleshooting
Instance Compatibility: Ensure you are using a supported instance type (e.g., C5, M5, R5, T3). Older types like T2 (standard) or M3 do not support ENA. 

repost.aws
 +4
Networking Issues: If the instance becomes unreachable after the change, the driver was likely not installed correctly in Step 2. Revert by stopping the instance and running:
aws ec2 modify-instance-attribute --instance-id <instance-id> --no-ena-support
