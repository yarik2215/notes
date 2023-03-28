# Network

## AWS Private Link
AWS PrivateLink provides a private connection between your VPCs and supported AWS services. This AWS service provides secure usage within the AWS network and avoids exposing traffic to the public internet.

![.assets/Network_20230323234312.png](.assets/Network_20230323234312.png)

## VPC Peering
A VPC peering connection is a networking connection between two VPCs that lets you route traffic between them privately. 

### Peering scenarious
- **Full sharing of resources between VPCs**
Each VPC must have a one-to-one connection with each VPC it is approved to communicate with. This is because each VPC peering connection is nontransitive in nature and does not allow network traffic to pass from one peering connection to another.
![.assets/Network_20230323234330.png](.assets/Network_20230323234330.png)
- **Partial sharing**
![ ](.assets/Network_20230323231119.png)


### Non-valid peering configurations
- **Overlaping CIDR blocks**
You cannot create a VPC peering connection between VPCs with matching or overlapping IPv4 
- **Transitive peering**
You have a VPC peering connection between VPC A and VPC B, and between VPC A and VPC C. There is no VPC peering connection between VPC B and VPC C. You cannot route packets directly from VPC B to VPC C through VPC A.
- **Edge-to-edge routing through a gateway or private connection**
If either VPC in a peering relationship has one of the following connections, you cannot extend the peering relationship to that connection:
	- A VPN connection or a Direct Connect connection to a corporate network
	- An internet connection through an internet gateway
	- An internet connection in a private subnet through a NAT device
	- A gateway VPC endpoint to an AWS service, for example, an endpoint to Amazon S3
	
	
## AWS Direct Connect
Direct Connect provides a private, reliable connection to AWS from your physical facility, such as a data center or office. It is a fully integrated and redundant AWS service that provides complete control over the data exchanged between your AWS environment and the physical location of your choice.

## AWS Site-to-Site VPN and AWS Client VPN
AWS VPN is comprised of two services: 
- AWS Site-to-Site VPN enables you to securely connect your on-premises network to Amazon VPC, for example your branch office site. 
- AWS Client VPN enables you to securely connect users to AWS or on-premises networks, for example remote employees. 

### AWS Site-to-Site VPN
![ ](.assets/Network_20230323231729.png)

### Client VPN
Based on OpenVPN technology, Client VPN is a managed client-based VPN service that lets you securely access your AWS resources and resources in your on-premises network.

## AWS Transit Gateway
AWS Transit Gateway is a highly available and scalable service that provides interconnectivity between VPCs and your on-premises network. Within a Region, AWS Transit Gateway provides a method for consolidating and centrally managing routing between VPCs with a hub-and-spoke network architecture.
