# **High-Availability Web Application Hosting on AWS**

This document outlines the architecture for a scalable, resilient, and highly available web application hosted on the AWS Cloud. The design leverages a multi-Availability Zone (AZ) deployment to ensure fault tolerance and uses managed services to automate scaling and monitoring.

## **Table of Contents**

1. [Architecture Overview]
2. [Key Components]
3. [Workflow & Traffic Flow]
4. [Core Features]
5. [Network Configuration]

## **Architecture Overview** 
**![Architecture Diagram][./images/architecture-diagram.png]**

The architecture is designed to provide a robust environment for a web application by distributing resources across two Availability Zones (AZs) within a single Virtual Private Cloud (VPC). This multi-AZ setup is fundamental to achieving high availability, as it protects the application from failure in a single data center.  
Traffic is intelligently routed and balanced across healthy application servers, which can scale automatically based on demand. The database is also configured for high availability with a primary instance and a synchronous standby replica in a separate AZ. Monitoring and alerting are handled by CloudWatch and SNS to ensure operational visibility.

## **Key Components**

* **Amazon Route 53:** Acts as the DNS service. It resolves the application's domain name to the Elastic Load Balancer using an Alias A record. This provides a reliable and cost-effective way to route users to the application endpoint.  
* **Elastic Load Balancer (ELB):** Serves as the single entry point for all incoming application traffic. It automatically distributes requests across the EC2 instances (App Servers) in both Availability Zones, ensuring no single server is overwhelmed.  
* **Auto Scaling Group:** Manages the fleet of application servers. It automatically adjusts the number of EC2 instances up or down based on traffic or predefined schedules. If an instance becomes unhealthy, the Auto Scaling group will terminate it and launch a new one to maintain the desired capacity.  
* **App Servers (EC2 Instances):** These are the virtual servers that run the core application logic. They are placed in private App Subnets and do not have direct access from the internet, enhancing security. An IAM Role is attached to these instances to provide secure permissions to other AWS services without needing to store credentials on the servers.  
* **Amazon RDS (Multi-AZ):** A managed relational database service.  
  * **RDS Primary:** The main database instance located in RDS Subnet 1 (Availability Zone A), handling all read and write operations.  
  * **RDS Multi-AZ Standby:** A synchronous replica of the primary database located in RDS Subnet 2 (Availability Zone B). This replica is not directly accessible but is ready to be promoted to the primary instance in case of a failure, ensuring data durability and high availability.  
* **Amazon CloudWatch:** Monitors the AWS resources and the application in real-time. It collects metrics (like CPU utilization) from the instances. Alarms can be configured to trigger actions when a certain threshold is breached.  
* **Amazon Simple Notification Service (SNS):** A managed messaging service. In this architecture, it is used to send notifications (e.g., via email) when a CloudWatch Alarm is triggered, alerting administrators to potential issues.

## **Workflow & Traffic Flow**

1. A user accesses the application's domain name in their browser.  
2. **Route 53** receives the DNS query and resolves it, returning the IP address of the **Elastic Load Balancer**.  
3. The user's browser sends the HTTP(S) request to the **ELB**.  
4. The **ELB** forwards the request to one of the healthy **App Servers** in either Availability Zone A or B. The choice is based on the load balancing algorithm.  
5. The **App Server** processes the request. If it needs to access the database, it connects to the **RDS Primary** instance.  
6. The **RDS Primary** database processes the query and returns the data to the App Server. All data written to the primary is synchronously replicated to the **RDS Multi-AZ** standby instance.  
7. The App Server sends the response back to the ELB, which then forwards it to the user.  
8. In the background, the **Auto Scaling** group monitors the health and load of the App Servers, scaling them in or out as needed.  
9. **CloudWatch** continuously monitors metrics from the infrastructure. If an alarm threshold is met (e.g., high CPU), it sends a message to an **SNS topic**, which in turn sends an **email notification** to the administrators.

## **Core Features**

* **High Availability:** By deploying across two Availability Zones, the application can withstand the failure of a single AZ. The ELB, Auto Scaling, and RDS Multi-AZ features all contribute to this resilience.  
* **Scalability:** The Auto Scaling group allows the application to handle changes in traffic load automatically, ensuring performance during peak times and cost savings during off-peak hours.  
* **Security:** Placing App Servers and RDS instances in private subnets protects them from direct external access. The ELB acts as a controlled entry point. IAM Roles provide secure, temporary credentials for AWS service access.  
* **Automation & Monitoring:** The combination of CloudWatch and SNS provides automated monitoring and alerting, reducing the need for manual intervention and enabling quick responses to operational events.

## **Network Configuration**

* **VPC:** 10.0.0.0/16 \- A single VPC contains all the resources.  
* **App Subnet 1 (AZ A):** 10.0.2.0/24 \- For App Servers.  
* **App Subnet 2 (AZ B):** 10.0.3.0/24 \- For App Servers.  
* **RDS Subnet 1 (AZ A):** 10.0.4.0/24 \- For the primary RDS instance.  
* **RDS Subnet 2 (AZ B):** 10.0.5.0/24 \- For the standby RDS instance.
