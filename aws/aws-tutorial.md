Security groups
Inbound rules and outbound rules
protocol: TCP, Port range: 22, Anywhere means 0.0.0.0/0

Webserver
Inbound rules
Type: HTTP, Protocol: TCP, Port Range: 80
Type: Custom TCP, Protocol: TCP, Port Range: 8080

- We cannot assign deny rules, but only allow rules
- We can attach multiple security groups for one instance

### How to add security group in exisiting instance?
- Select the instance
- Click on the Actions dropdown
- From Security dropdown, Change security groups
- Search from the input box and select your desired security group
- And save

## Instance types
To know about instance types here is the link
https://aws.amazon.com/ec2/instance-types/

AWS pricing, on-demand instance, Reserved instances and spot instances, saving plan, dedicated host

https://aws.amazon.com/ec2/pricing/

## Instance metadata with userdata

### Magic IP
```bash
curl 169.254.169.254

curl 169.254.169.254/latest

curl 169.254.169.254/latest/meta-data
```

to get dynamic data

```bash
curl 169.254.169.254/latest/dynamic
```

to fetch document

```bash
curl 169.254.169.254/latest/dynamic/instance-identity/document
```

## Attach Elastic/Static IP to an EC2 instance

Network & Securities >> Elastic IPs


Allocate Ilastic IP address >> Allocate

Associate Elastic IP Address, Select instance >> Click Associate

## Elastic Block Storage or EBS

EBS volume is a hard diskt that you can attach to your EC2 instance.

### Q: Can you attach an EBS to multiple EC2 instances?
The answer could be yes or no. We can create 6 types of EBS volume. And 2 among them can be used in multiple EBS instance and rest 4 volumens only can be used in one EC2 instance.

### Q: I've an EC2 instance which has only 10 GB of volume, in future if want to extend the volume by 120 GB. Is it possible to do that?
Yes we can.

Root device type: _instance-store_

## Detach an EBS volume from one EC2 instance and attach it another one
### Q. How can we check if EBS volume has any data or not by accessing EC2 instance? (Assume Ubuntu 22.02)
```bash
sudo file -s /dev/xvdf
```

### Q. How can we permanently mount an EBS volume to an EC2 instance? When machine stops and starts again it should mounted again.

```bash
mkdir /data
mount /dev/xvdf /data
```

Check if `/dev/xvdf` is mounted or not:

```bash
mountpoint /data
```

Output:

```
/data is a mountpoint
```

## Resize EBS volume and Resize the file System
Volumes => Select a volume you want to resize => Actions => Modify volume => Size (GiB) input (add your desired volume in GiB) => Modify => Confirm modify after reading the confirmation message

Remeber: the volume you can only increase but cannot decrease

You can access the EC2 instance and verify the size you've increased by:

```bash
df -h
```
```
Filesystem      Size  Used Avail Use% Mounted on
/dev/root       7.6G  1.5G  6.1G  20% /
tmpfs           484M     0  484M   0% /dev/shm
tmpfs           194M  836K  193M   1% /run
tmpfs           5.0M  0  	5.0M   1% /run/lock
/dev/xvda15     105M  5.3M  100M   5% /boot/efi
tmpfs            97M  4.0K   97M   1% /run/user/1000
/dev/xvdf		4.9G   28K  4.6G   1% /data
```

We need to grow the file system for `/dev/xvdf`. So run following command:

```bash
resize2fs /dev/xvdf
```

## How to resize Root EBS Volume
```bash
growpart /dev/xvda 1
resize2fs /dev/xvda1
```

## Attach one EBS volume to multiple EC2 instances
Make a volume type as `io2` which will give you the option for Amazon EBS Multi-Attach. But some regions gives you EBS Multi-Attach for `io1` EBS volume

Remember: We cannot attach multiple EBS volume on `xen system` based EC2 instance. But `nitro system` supports it.

### Can we change an instance type for exising instance?
Yes we can, but at first we need to stop the instance. Then Actions => Instance settings => change instance type

Note: Make sure use such a filesystem in your EC2 instance which is cluster aware. `ext4` or `sfx` doesn't support this.

## Types of EBS volumes - Which EBS volume should I use?
Six different volumes are shown:
- General Purpose SSD (gp2)
- General Purpose SSD (gp3)
- Provisioned IOPS SSD (io1)
- Provisioned IOPS SSD (io2)
- Cold HDD (sc1)
- Throughput optimized HDD (st1)
- Magnetic (standard)

IOPS = Input Output Operation per second
Throughput = How much data can be transfered (MiB/s)

When your requirement is I/O speed you can choose `io1` or `io2`.

To know more about ebs: https://aws.amazon.com/ebs/

### Volume types:
**General purpose**: https://aws.amazon.com/ebs/general-purpose/

**Provisioned IOPS**: https://aws.amazon.com/ebs/provisioned-iops/

**Throughput optimized**: https://aws.amazon.com/ebs/throughput-optimized/

**Cold HDD**: https://aws.amazon.com/ebs/cold-hdd/

## Snapshot Overview - AWS Snapshot - EBS Snapshot

If you ever had used virtual machine in your host operating system there is a system called "Take". If you take a snapshot of your currently installed operating system of that given time it will store a restore point on that time. And if anything goes wrong like you accidentlally broke the configuration your OS then you can revert to that restore point in your virtual machine. This is called a `Snapshot`.

So, snapshot happens on point of time.

In amazon EBS you can take a backup anytime. And if you accidentally remove some important data you can restore to that snapshot when you took the snapshot. Lets say you are writing a blog on your web server. And you take snapshot every 3 days interval, in this way you don't need to worry about loss of your data.

When amazon stores your EBS snapshot it stores your snapshot in Simple storage service (S3). The snapshot is region specific, because S3 bucket is region specific. Snapshot happens in incremental order. 

See the image from `Full backup` vs `Incremental backup`

## Create Snapshot

Do following things in aws console:

- Create a new volume
- Create a new Ubuntu instance
- Attach volume to newly created instance

Now access instance using `ssh`.

- List all block storage
	```bash
	lsblk
	```
- Check if newly added ESB has any data or not:
	```bash
	file -s /dev/xvdf
	```
- If not then format to ext4
	```bash
	mkfs.ext4 /dev/xvdf
	```
- make a data directory in root
	```bash
	mkdir /data
	```
- mount the `/dev/xvdf` to `/data` directory
	```bash
	mount /dev/xvdf /data
	```
- check the mountpoint for the `/data` directory
	```bash
	mountpoint /data
	```
- change directory to `/data`
	```bash
	cd /data
	```
- create a file
	```bash
	yes "develper" >> abc.txt
	```
	This `yes` command creates a file and returns the output in a file using a loop until you stop the execution using `ctrl+c`.

- Now check the size of `data` directory using

	```bash
	df -h
	```
Go to your aws `EBS` console. And select the volume, click to **Actions** dropdown and **Create snapshot**.
But before doing that you can click on the **Snapshots** link under **Elastic Block Store** section. There you will see all the available snapshots if you already created any snapshots or not. There will be no snapshots if you didn't created any snapshots earlier.

Now when you follow the **Create snapshot** a new page will open:

**Create snapshot**:

Create a point-in-time snapshot to back up the data on an Amazon EBS volume to Amazon S3

**Details**:

Add a description for your snapshot

The good approach is to use your application name and the curernt date.

## Snapshot and AMI Recycle Bin for Snapshot and AMI AWS
Normally when you delete your data your data is deleted permanently. But for AWS snapshot they gave us an oppurtunity of retention rules. So, it means when you delete a snapshot they goes to the recycel bin.

## Copy Snapshot from one region to another region

If AWS announces any service they first launch their service to **US East (N. Virginia)** or **us-east-1** region. And for this reason it is the default region in AWS. One more thing to mention is that:

If you go to the volume section and create a new volume. Then select the volume type as **Provision IOPS SSD (io1)** you will see they are giving us the oppurtunity to use **Amazon multi attach** option. But **Mumbai** region doesn't have this for `io1` volume type.

Now **Elastic Block Store** >> **Snapshots** >> **Select a snapshot** >> **Actions** >> **Copy Snapshot**.

A new page will open. Give the description and change the **Destination Region** to another region you wan to copy.

When copying is done from one region to another region, go to other region and go to the **Snapshots** section. Select the snapshot and from Actions menu and click to **Create volume from snapshot**. 

## Encrypt the EBS volume

When you create a volume if you click to the **Encrypt this volume** a new input field will appear where you need to provide the **KMS key** where you can pass your own KMS key or use the default key from the selection.

**Note**: When you create a new snapshot for encrypted volume the snapshot will also be encrypted.

**Benefits**:

Encrypted EBS (Elastic Block Store) in Amazon Web Services (AWS) offers several benefits:

1. **Data Security**:
   - **Data at Rest Encryption**: Encrypts the data stored on the EBS volumes, ensuring that any data stored on them is protected from unauthorized access.
   - **Data in Transit Encryption**: Encrypts the data as it moves between the EBS volume and the instance, protecting it during transfer.

2. **Compliance**:
   - Helps meet compliance requirements for various standards like GDPR, HIPAA, and others by providing encryption mechanisms that safeguard sensitive data.

3. **Integrated with AWS Key Management Service (KMS)**:
   - Allows for easy management of encryption keys and offers additional security by integrating with AWS KMS, which provides centralized control over the encryption keys.

4. **Performance**:
   - Encryption is handled transparently, meaning there is no impact on the performance of your EBS volumes. AWS uses hardware acceleration to perform encryption and decryption.

5. **Ease of Use**:
   - Simplifies encryption management since encryption is enabled by default when you create a new EBS volume or snapshot.
   - Snapshots of encrypted volumes are automatically encrypted, and volumes created from these snapshots are also encrypted.

6. **Data Durability**:
   - Encrypted EBS volumes offer the same durability and availability as non-encrypted volumes, ensuring that your data is safe and highly available.

7. **Seamless Integration**:
   - Works seamlessly with other AWS services, such as Amazon EC2, ensuring a smooth workflow and secure data management across your AWS environment.

Overall, encrypted EBS volumes provide a robust security mechanism to protect sensitive data, comply with regulatory requirements, and ensure that encryption management is straightforward and efficient without compromising performance.

**How can we enable `encryption` by default?**

Go to your EC2 console. From the right sidebar, under the **Account attributes** >> **Settings** click on **EBS Encryption**. Then **Manage**, Check on the **Enable** for **Always encrypt new EBS volume**. Then **Update EBS encryption**.


Snapshots >> Actions >> Manage fast snapshot restore
Snapshots stored in S3 bucket. When you create a new volume using a snapshot the snapshot data moves from S3 bucket to the new volume. But when you create a snapshot that has very large size, in that case it takes time to restore the data. But you enable this feature the data transfer will be high compared to nomral data transfer. And they will grow the compute and memory size behind the scene. And the restoration speed will be fast.


## AWS AMI - Amazon Machine Image

Amazon Machine Image (AMI) is a critical concept in Amazon Web Services (AWS) that allows you to create and use virtual servers (instances) in the cloud. Here's a detailed explanation to help you understand and remember AMI:

### What is an Amazon Machine Image (AMI)?

An Amazon Machine Image (AMI) is a template that contains the software configuration (operating system, application server, and applications) required to launch an instance in AWS. Think of it as a snapshot of your system at a point in time, which can be used to create new instances with the same configuration.

### Key Components of an AMI:

1. **Root Volume Template**:
   - Contains an operating system, an application server, and applications.
   - The root volume can be based on Amazon Elastic Block Store (EBS) or an instance store.

2. **Launch Permissions**:
   - Control which AWS accounts can use the AMI to launch instances.

3. **Block Device Mapping**:
   - Specifies the volumes to attach to the instance when it is launched.

### Types of AMIs:

1. **Pre-configured AMIs**:
   - Provided by AWS, community members, or third-party vendors.
   - Include popular operating systems like Amazon Linux, Ubuntu, Windows Server, and more.

2. **Custom AMIs**:
   - Created by users to include specific configurations, software, and settings tailored to their needs.

### Benefits of Using AMIs:

1. **Standardization**:
   - Ensure consistency across multiple instances by using the same AMI to launch them.

2. **Customization**:
   - Create AMIs with pre-installed applications and configurations, saving time on setting up new instances.

3. **Scalability**:
   - Quickly scale out your applications by launching multiple instances from the same AMI.

4. **Backup and Recovery**:
   - Use AMIs to create backups of your instances, allowing for quick recovery in case of failures.

5. **Portability**:
   - Easily migrate applications and services to different regions or accounts by sharing and using AMIs.

### How to Create an AMI:

1. **From an Instance**:
   - Launch an instance, configure it as desired, and then create an AMI from it using the AWS Management Console, AWS CLI, or SDKs.

2. **From a Snapshot**:
   - Create an AMI from an EBS snapshot that contains the desired configuration.

### Using an AMI:

1. **Launch an Instance**:
   - Use the AMI to launch new EC2 instances with the same configuration.

2. **Copy AMIs**:
   - Copy AMIs to different regions for disaster recovery and high availability.

3. **Share AMIs**:
   - Share AMIs with specific AWS accounts or make them public for community use.

### Summary:

An Amazon Machine Image (AMI) is a pre-configured template used to launch virtual server instances in AWS. It includes the operating system, applications, and configurations necessary to create a consistent, repeatable environment. By using AMIs, you can standardize deployments, save time on setup, ensure scalability, and enhance disaster recovery capabilities.

This comprehensive understanding should help you remember the significance and functionality of AMIs in AWS.

To create a new AMI:
- Go to the instances, select an instance you want to create template from. 
- First method:
	1. Right click on that instance, **Image and templates** >> **Create image**
	2. Select the instance, from the actions menu **Image and templates** >> **Create image**.
- Give the image name and then create an image
- Now go to the **Images** >> **AMIs**. You can see the status as **Pending** when you create a new AMI. It will take little bit of time to process this.
- Select the created AMI and click on **Launch instance from AMI**.
- Behind the scene when you create a new AMI from an instance a new snapshot is created

## Share your AMI with other AWS account

To share with other person or make it public
- Go to the **Images**, then **AMIs**. Select an AMI, then from the actions menu **Edit AMI permissions**. From the **Shared Accounts** you can add an account ID. Or we can make this AMI as public from the **AMI availability**.
- You can copy an AMI from one region to other region.
- To delete an AMI:
	- select the AMI and from the actions menu **Deregister AMI**, And click to the **Deregister AMI** button.
	- From the snapshots, select a snapshot that was associated with an AMI. From the actions menu delete the snapshot. Now you've remove your AMI.

## Elastic Load Balancer (ELB)

Types:
- Application Load Balancer
- Network Load Balancer
- Gateway Load Balancer
- Classic Load Balancer

### How can we configure EC2 instance that gives HTTP Port 80 support only to load balancer?

- Go to instances, select the instance, from the below groups, select **Security group**. Click on the link under *Security groups* section that starts like that: `sg-044..758`. 
- **Edit inbound rules**
- **Add rule** button
- Type: HTTP
- Source: The security group of your attached load balancer, so that all the traffic of that EC2 instance will go to load balancer security group
- **Save rules** button

Note:
- High availibility
- Fault tolerance

![Classic load balancer](https://i.imgur.com/81SqdQk.png)

The classic load balancer works on application layer as well as on transport layer. Layer 4 and Layer 7. 

**Classic Load Balancer Protocols**:
- HTTP
- HTTPS
- TCP
- SSL (Secure TCP)

## Application Load Balancer (Layer 7 Load balancer)

![application load balancer](https://i.imgur.com/uJyw8Qe.png)

**How to access target group in aws**?
From **Load Balancing**, you will see **Target groups**.

We can specify the target group by Instance/IP address/Lambda function/Application Load Balancer.

## Path base routing in Application Load Balancer

How many types we can use for IF condition in application load balancer?

- Host header
- Path
- HTTP Header
- HTTP Request method
- Query string
- Source IP

## How to get client IP address on Application server?

How to access **Load balancer attributes**?

From Load balancers, Actions menu >> Edit attributes

## Stickiness and custom page routing in Application load balancer

Adding SSL certificate
Cross-zone load balancing

## Network Load Balancer (Layer 4 Load Balancer)

A **Network Load Balancer (NLB)** in AWS operates at the transport layer (Layer 4) of the [OSI model](https://www.geeksforgeeks.org/open-systems-interconnection-model-osi/#what-is-osi-model). It is designed to handle high volumes of traffic with low latency and is ideal for load balancing TCP, UDP, and TLS traffic. 

#### Key Features:

1. **High Performance and Low Latency**:
   - NLB can handle millions of requests per second while maintaining ultra-low latencies. This makes it suitable for latency-sensitive applications.

2. **Static IP Support**:
   - Provides a single, static IP per Availability Zone for the load balancer. This is useful for applications that require whitelisted IP addresses.

3. **Elasticity and Scalability**:
   - Automatically scales to handle changes in traffic load, ensuring consistent performance without manual intervention.

4. **TLS Termination**:
   - Offloads the decryption work from the application servers by terminating TLS (Transport Layer Security) connections at the load balancer. This enhances security and reduces the computational burden on backend instances.

5. **Health Checks**:
   - Continuously monitors the health of registered targets (e.g., EC2 instances, IP addresses) and routes traffic only to healthy targets. It uses health check endpoints and protocols specified during configuration.

6. **Cross-Zone Load Balancing**:
   - Distributes incoming traffic evenly across all registered targets in multiple Availability Zones, ensuring high availability and fault tolerance.

7. **Target Groups**:
   - Targets are grouped into target groups, allowing you to apply health checks and routing policies to specific groups. Each target group routes requests to one or more registered targets using specified ports.

#### Use Cases:

1. **High Throughput and Low Latency Applications**:
   - Ideal for real-time applications like gaming, media streaming, and financial services that require quick response times and can handle high traffic volumes.

2. **TCP/UDP Traffic Management**:
   - Useful for applications that need to balance TCP and UDP traffic such as VoIP, chat applications, and certain types of web applications.

3. **Microservices Architectures**:
   - Helps in efficiently distributing traffic to microservices running on containers or serverless functions.

4. **Static IP Requirements**:
   - Suitable for applications needing a static IP for incoming requests, such as those requiring IP whitelisting for security purposes.

#### How It Works:

1. **Listeners**:
   - A listener checks for connection requests from clients using the protocol and port you configure, such as TCP port 80.

2. **Target Groups**:
   - Targets (EC2 instances, IP addresses, Lambda functions) are registered in target groups. The load balancer routes requests to these targets based on listener rules.

3. **Health Checks**:
   - NLB uses health checks to monitor the status of each target. It routes traffic only to healthy targets, improving reliability and availability.

4. **Routing**:
   - Based on listener rules and health check status, NLB forwards incoming traffic to targets in a target group, distributing the load according to configured policies.

#### Benefits:

1. **High Availability**:
   - By distributing traffic across multiple Availability Zones, NLB enhances the availability and resilience of your applications.

2. **Improved Security**:
   - With TLS termination, NLB secures traffic by encrypting data in transit and offloading the decryption process from backend servers.

3. **Efficient Scaling**:
   - NLB’s ability to scale automatically in response to traffic load ensures your application can handle sudden spikes in traffic without performance degradation.

4. **Cost-Effective**:
   - Pay-as-you-go pricing and automatic scaling make NLB a cost-effective solution for balancing high volumes of network traffic.

### Summary:

The AWS Network Load Balancer (NLB) is a powerful and efficient Layer 4 load balancer designed to handle high traffic volumes with low latency. It provides static IP support, TLS termination, and robust health checks, making it ideal for high-performance, latency-sensitive applications and services that require reliable and secure traffic distribution. By leveraging NLB, organizations can ensure their applications are highly available, scalable, and secure.

## AWS Launch template (Auto Scaling Group - ASG)

### AWS Launch Template (Auto Scaling Group - ASG)

An Auto Scaling Group (ASG) in AWS is a collection of EC2 instances that are managed as a group. ASGs allow you to automatically adjust the number of instances in response to changes in demand, ensuring that you have the right amount of resources at any time.

#### Key Features of ASGs:
1. **Automatic Scaling**:
   - Adjusts the number of instances up or down based on policies you define, responding to changes in demand.
2. **Health Checks and Replacement**:
   - Monitors the health of instances and automatically replaces unhealthy instances.
3. **Elasticity**:
   - Scales your fleet of instances based on criteria like CPU utilization, request count, or custom CloudWatch metrics.
4. **Load Balancing Integration**:
   - Works with Elastic Load Balancing (ELB) to distribute incoming traffic across the instances in the ASG.

#### Benefits:
- **Cost Management**: Optimizes costs by scaling in (terminating instances) during low demand periods and scaling out (adding instances) during high demand.
- **High Availability**: Maintains application availability by ensuring a minimum number of instances are always running.
- **Fault Tolerance**: Automatically replaces failed instances, maintaining the health and performance of your applications.

### How Launch Templates and ASGs Work Together:

1. **Create a Launch Template**:
   - Define the configuration for your EC2 instances, such as AMI, instance type, security groups, and key pairs.
2. **Create an Auto Scaling Group**:
   - Specify the launch template to use, along with other parameters like minimum and maximum instance counts, desired capacity, and scaling policies.
3. **Define Scaling Policies**:
   - Set policies to automatically scale the number of instances based on metrics like CPU utilization or custom CloudWatch metrics.
4. **Monitor and Adjust**:
   - ASG continuously monitors the health and performance of the instances, making adjustments as needed to maintain the desired state.

### Summary:

**AWS Launch Templates** provide a reusable and versioned way to define configurations for EC2 instances, ensuring consistency and efficiency. **Auto Scaling Groups (ASGs)** leverage these templates to dynamically manage the number of instances based on demand, ensuring high availability, fault tolerance, and cost optimization. Together, they automate the process of scaling applications, making it easier to handle variable workloads and maintain application performance.

## How to enable automatic scaling?

Enabling automatic scaling in AWS involves setting up Auto Scaling Groups (ASGs) and defining policies that adjust the number of EC2 instances based on demand. Here's a step-by-step guide to enable automatic scaling:

### Step 1: Create a Launch Template
A launch template specifies the configuration for the EC2 instances, including the AMI, instance type, key pair, security groups, and other settings.

1. **Open the EC2 Dashboard**:
   - Go to the AWS Management Console and navigate to the EC2 dashboard.

2. **Create a Launch Template**:
   - Click on "Launch Templates" under "Instances."
   - Click "Create launch template."
   - Enter a name and description for the launch template.
   - Configure the instance settings (AMI ID, instance type, key pair, security groups, etc.).
   - Click "Create launch template."

### Step 2: Create an Auto Scaling Group (ASG)
An Auto Scaling Group uses the launch template to create and manage a group of instances that can scale automatically.

1. **Open the EC2 Dashboard**:
   - From the EC2 dashboard, click on "Auto Scaling Groups" under "Auto Scaling."

2. **Create Auto Scaling Group**:
   - Click "Create Auto Scaling group."
   - Enter a name for the Auto Scaling group.
   - Select the launch template created in Step 1.
   - Choose the desired instance settings and click "Next."

3. **Configure Group Size and Scaling Policies**:
   - Set the minimum, maximum, and desired number of instances.
   - Minimum capacity: The minimum number of instances that should always be running.
   - Maximum capacity: The maximum number of instances that can be running.
   - Desired capacity: The initial number of instances to start with.

4. **Configure Network Settings**:
   - Select the VPC and subnets where you want the instances to run.
   - Enable "Receive traffic from Elastic Load Balancer(s)" if you use a load balancer.

5. **Set Scaling Policies**:
   - **Dynamic Scaling**:
     - Create scaling policies based on CloudWatch alarms. For example, you can create an alarm to scale out when CPU utilization exceeds 70% and scale in when it drops below 30%.
     - Click "Add policy" and configure the policy details.
   - **Predictive Scaling**:
     - Predictive scaling uses machine learning to predict future traffic and scale your instances accordingly.
     - Select "Predictive scaling" and configure the settings.

6. **Add Notifications (Optional)**:
   - Set up notifications for scaling events using SNS topics.

7. **Configure Tags (Optional)**:
   - Add tags to help identify and manage your resources.

8. **Review and Create**:
   - Review your settings and click "Create Auto Scaling group."

### Step 3: Monitor and Adjust
Once your Auto Scaling Group is set up, monitor the performance and adjust scaling policies as needed.

1. **Monitor with CloudWatch**:
   - Use Amazon CloudWatch to monitor the performance metrics of your instances and Auto Scaling Group.
   - Adjust the CloudWatch alarms and scaling policies based on the observed performance.

2. **Adjust Scaling Policies**:
   - Modify the scaling policies to optimize performance and cost. For example, you might need to adjust the thresholds for scaling out and scaling in based on traffic patterns.

### Summary
By creating a launch template and configuring an Auto Scaling Group with appropriate scaling policies, you can enable automatic scaling for your applications in AWS. This setup ensures that your application can handle varying loads efficiently by automatically adjusting the number of running instances based on demand. Monitoring and adjusting the scaling policies helps maintain optimal performance and cost efficiency.

**How many policies are there for automatic scaling?**

3 policies:
- Dynamic Scaling Policies
- Predictive Scaling Policies
- Scheduled actions

**What are the metrics or target utilization used during predictive scaling policies?**

- **CPU utilization**: Uses the total CPU to create the load forecast and the average CPU to define your target utilization.
- **Network in (bytes)**: Uses the total number of bytes received to create the load forecast and the average number of bytes recieved to define your target utilization.
- **Network out (bytes)**: Uses the total number of bytes sent to create the load forecast and the average number of bytes sent to define your target utilization.
- **Application load balancer request count**: Uses the total request count to create the load forecast and the average number of requests per target to define your target utilization.
- **Custom metric pair**: Choose your own load and scaling metrics. For predictive scaling to work effectively, the scaling metric must be strongly correlated.

**How many policy type are there for creating dynamic scaling policy?**

There are 3 types of dynamic scaling policy:
- Target tracking scaling
- Step scaling
- Simple scaling

**Which package can you use for load CPU testing when accessing an instance with SSH?**

We can use `stress` package. Using following command will install this package in ubuntu instance.

```bash
sudo apt install stress
```

**Usage**:

```bash
stress -c 5
```

## What is cooling period?

The **cooldown period** in AWS Auto Scaling is a setting that helps prevent your Auto Scaling group from launching or terminating additional instances before previous scaling activities have had time to take effect. This helps to avoid unnecessary scaling actions and stabilizes the system.

### Key Concepts of Cooldown Period:

1. **Definition**:
   - The cooldown period is a configurable setting that specifies the amount of time (in seconds) to wait after a scaling activity completes before initiating another scaling activity.

2. **Purpose**:
   - To provide time for the newly launched instances to start up and begin handling the load.
   - To allow metrics (like CPU utilization) to stabilize before making further scaling decisions.
   - To avoid rapid, successive scaling actions that could result in oscillations or "flapping" (frequent scaling in and out).

3. **Default Cooldown**:
   - By default, the cooldown period is set to 300 seconds (5 minutes) for Auto Scaling groups, but this can be customized based on your application's needs.

### How Cooldown Period Works:

1. **Scaling Activity Initiated**:
   - When an Auto Scaling group scales out (adds instances) or scales in (removes instances), the cooldown period starts once the activity is complete.

2. **Cooldown Period Active**:
   - During the cooldown period, the Auto Scaling group temporarily suspends scaling activities. This allows the newly launched instances to boot up and start handling the load, or allows the system to adjust to the reduced capacity after a scale-in.

3. **Monitoring and Metrics Stabilization**:
   - CloudWatch continues to monitor the performance metrics, but scaling actions are paused until the cooldown period ends. This helps to ensure that the metrics accurately reflect the new state of the system without the influence of the recent scaling activity.

4. **Cooldown Period Ends**:
   - Once the cooldown period expires, the Auto Scaling group can initiate new scaling activities if the CloudWatch metrics indicate that scaling is necessary.

### Customizing the Cooldown Period:

1. **Application-Specific Needs**:
   - Adjust the cooldown period based on how quickly your instances become operational and begin handling the load. For example, if your instances take longer to start and become productive, you might increase the cooldown period.

2. **Granular Control with Scaling Policies**:
   - AWS allows you to set different cooldown periods for different scaling policies. For example, you might have a shorter cooldown period for scaling in and a longer one for scaling out.

### Setting the Cooldown Period:

1. **Default Cooldown**:
   - You can set a default cooldown period for the Auto Scaling group when you create or update it.
   - Example (AWS CLI): 
     ```bash
     aws autoscaling update-auto-scaling-group --auto-scaling-group-name my-asg --default-cooldown 300
     ```

2. **Policy-Specific Cooldown**:
   - You can set a specific cooldown period for individual scaling policies.
   - Example (AWS CLI):
     ```bash
     aws autoscaling put-scaling-policy --policy-name scale-out-policy --auto-scaling-group-name my-asg --scaling-adjustment 1 --cooldown 180
     ```

### Summary:
The cooldown period in AWS Auto Scaling is a crucial mechanism that prevents rapid, successive scaling actions by introducing a waiting period after each scaling activity. This ensures that the system has time to stabilize and that scaling decisions are based on accurate metrics, ultimately leading to more efficient and effective scaling. By customizing the cooldown period, you can tailor it to the specific needs of your application and ensure optimal performance and resource utilization.

## Auto scaling with load balancer

Combining AWS Auto Scaling with a load balancer, such as an Elastic Load Balancer (ELB), provides a powerful solution for automatically managing the availability and scalability of your applications. Here’s a detailed explanation of how these components work together:

### Key Components:

1. **Auto Scaling Group (ASG)**:
   - Manages a group of EC2 instances and automatically adjusts the number of instances to handle the load on your application.
   - Ensures the desired number of instances are running, scales out (adds instances) during high demand, and scales in (removes instances) during low demand.

2. **Elastic Load Balancer (ELB)**:
   - Distributes incoming application traffic across multiple targets, such as EC2 instances, in one or more Availability Zones.
   - Ensures high availability and fault tolerance by balancing the load and routing traffic only to healthy instances.

### Types of Load Balancers in AWS:
1. **Application Load Balancer (ALB)**:
   - Operates at the application layer (Layer 7) and is suitable for web applications, microservices, and containerized applications. Supports HTTP/HTTPS traffic and advanced routing features like path-based and host-based routing.

2. **Network Load Balancer (NLB)**:
   - Operates at the transport layer (Layer 4) and is designed for high-performance applications that require low latency and handle TCP/UDP traffic.

3. **Classic Load Balancer (CLB)**:
   - Operates at both the application and transport layers. Suitable for simple load balancing of HTTP/HTTPS and TCP/SSL traffic. CLB is older and less feature-rich compared to ALB and NLB.

### How Auto Scaling with Load Balancer Works:

1. **Create a Launch Template or Configuration**:
   - Define the configuration for your EC2 instances, including the AMI, instance type, security groups, key pairs, etc.

2. **Create an Auto Scaling Group (ASG)**:
   - Specify the launch template or configuration, set the minimum, maximum, and desired number of instances, and configure other settings such as VPC and subnets.

3. **Attach the Load Balancer to the ASG**:
   - When creating or updating the ASG, specify the load balancer to attach to the group. The ASG will automatically register new instances with the load balancer and deregister terminated instances.
   - For ALB or NLB: Attach the target group of the load balancer to the ASG.
   - For CLB: Directly attach the load balancer to the ASG.

4. **Configure Health Checks**:
   - Set up health checks to monitor the status of instances. The load balancer periodically checks the health of instances and routes traffic only to healthy instances.
   - Health checks can be based on HTTP/HTTPS endpoints (for ALB) or TCP connections (for NLB).

5. **Define Scaling Policies**:
   - Create scaling policies based on CloudWatch metrics (e.g., CPU utilization, request count) to automatically adjust the number of instances in the ASG.
   - Policies can include dynamic scaling (based on real-time metrics) and scheduled scaling (based on specific times).

### Benefits of Auto Scaling with Load Balancer:

1. **High Availability**:
   - Ensures that your application remains available and can handle traffic spikes by automatically adding or removing instances.
   - Distributes traffic across multiple instances and Availability Zones, improving fault tolerance.

2. **Improved Performance**:
   - Balances the load evenly across instances, preventing any single instance from becoming a bottleneck.
   - Allows scaling out to meet increased demand, ensuring that performance remains optimal.

3. **Cost Efficiency**:
   - Scales in during low demand periods to reduce costs by terminating unnecessary instances.
   - Uses pay-as-you-go pricing, so you only pay for the resources you use.

4. **Simplified Management**:
   - Automatically handles instance registration and deregistration with the load balancer.
   - Reduces manual intervention and allows you to focus on other aspects of your application.

### Example Workflow:

1. **Traffic Increase**:
   - An increase in traffic to your application triggers a CloudWatch alarm based on metrics like CPU utilization or request count.

2. **Scale Out**:
   - The ASG responds by launching additional EC2 instances based on the configured scaling policy.
   - The new instances are automatically registered with the load balancer and start receiving traffic.

3. **Health Checks**:
   - The load balancer continuously performs health checks on the instances. If any instance is deemed unhealthy, it is removed from the rotation, and the ASG can terminate it and launch a replacement.

4. **Traffic Decrease**:
   - When traffic decreases, another CloudWatch alarm triggers a scale-in action.
   - The ASG terminates excess instances, which are automatically deregistered from the load balancer.

### Summary:

Combining AWS Auto Scaling with a load balancer provides a robust solution for automatically managing the scalability and availability of your applications. Auto Scaling adjusts the number of EC2 instances based on demand, while the load balancer distributes incoming traffic across these instances, ensuring high performance and fault tolerance. This setup helps you efficiently handle traffic fluctuations, maintain optimal performance, and reduce costs by scaling resources up and down as needed.

## Enable Termination Protection

**Termination Protection** is a feature in AWS EC2 that helps prevent accidental termination of an instance. When enabled, it adds an extra layer of protection by requiring an additional step to terminate the instance. This is particularly useful for critical instances that should not be inadvertently stopped.

### Key Concepts:

1. **Purpose**:
   - Prevents accidental termination of important instances.
   - Requires explicit action to disable protection before an instance can be terminated.

2. **Applicability**:
   - Can be applied to any EC2 instance, including those part of an Auto Scaling group.

### How Termination Protection Works:

1. **Enabling Termination Protection**:
   - When you enable termination protection, any attempt to terminate the instance via the AWS Management Console, CLI, or API will fail unless the protection is first disabled.

2. **Disabling Termination Protection**:
   - To terminate an instance with termination protection enabled, you must first disable the protection, then proceed with the termination.

### Enabling Termination Protection:

1. **During Instance Launch**:
   - When launching a new EC2 instance, you can enable termination protection in the instance settings.
   - **AWS Management Console**:
     - In the "Configure Instance Details" step, check the box "Enable termination protection."
   - **AWS CLI**:
     ```bash
     aws ec2 run-instances --image-id ami-xxxxxxxx --count 1 --instance-type t2.micro --key-name MyKeyPair --disable-api-termination
     ```

2. **For an Existing Instance**:
   - You can enable termination protection for an already running instance.
   - **AWS Management Console**:
     - Select the instance in the EC2 dashboard.
     - Click on "Actions" -> "Instance Settings" -> "Change Termination Protection."
     - Check "Enable" and click "Yes, Change."
   - **AWS CLI**:
     ```bash
     aws ec2 modify-instance-attribute --instance-id i-xxxxxxxx --no-disable-api-termination
     ```

### Disabling Termination Protection:

1. **AWS Management Console**:
   - Select the instance in the EC2 dashboard.
   - Click on "Actions" -> "Instance Settings" -> "Change Termination Protection."
   - Uncheck "Enable" and click "Yes, Change."

2. **AWS CLI**:
   ```bash
   aws ec2 modify-instance-attribute --instance-id i-xxxxxxxx --disable-api-termination
   ```

### Example Workflow:

1. **Launch an Instance with Termination Protection**:
   - Launch a new EC2 instance and enable termination protection during the launch process.
   
2. **Attempt to Terminate the Instance**:
   - Try to terminate the instance using the AWS Management Console, CLI, or API.
   - The termination attempt will fail with an error message indicating that termination protection is enabled.

3. **Disable Termination Protection**:
   - Disable termination protection using the Management Console or CLI.
   
4. **Terminate the Instance**:
   - After disabling termination protection, proceed to terminate the instance.

### Summary:

**Termination Protection** in AWS EC2 is a valuable feature to safeguard critical instances from accidental termination. It requires an explicit action to disable protection before an instance can be terminated, adding an extra layer of security. Enabling and disabling termination protection can be done easily via the AWS Management Console or CLI, ensuring that important instances remain running until you are certain they can be safely terminated.

## What is the difference among Hibernate, Sleep and Poweroff

Here are the differences among hibernate, sleep, and poweroff modes for a computer:

### Hibernate:

1. **Purpose**: 
   - Hibernate mode saves the current state of your computer (open documents, running applications, etc.) to the hard drive and then completely powers off the computer.
   
2. **Power Usage**: 
   - Minimal power is used, just enough to keep the system in a state where it can be powered back on and restored.
   
3. **Speed**:
   - Slower to resume than sleep mode because the computer has to read the saved state from the hard drive.
   
4. **Use Case**:
   - Best used when you won't be using your computer for an extended period but want to resume your previous session quickly without rebooting.

5. **Data Safety**:
   - Safe during power outages since the state is saved to disk.

### Sleep:

1. **Purpose**:
   - Sleep mode saves the current state to RAM and puts the computer into a low-power state. This keeps the session active but minimizes power usage.
   
2. **Power Usage**:
   - Low power consumption is necessary to keep the RAM refreshed and maintain the state.
   
3. **Speed**:
   - Fast to resume because the state is maintained in RAM, which is much faster than reading from a hard drive.
   
4. **Use Case**:
   - Ideal for short breaks or when you want to quickly resume your work.
   
5. **Data Safety**:
   - Not safe during power outages unless you have a laptop with a battery, as the session can be lost if the power is completely cut.

### Poweroff (Shutdown):

1. **Purpose**:
   - Poweroff completely shuts down the computer, closing all applications and turning off the hardware.
   
2. **Power Usage**:
   - No power usage because the computer is entirely off.
   
3. **Speed**:
   - Slowest to resume because it requires a full reboot and restarting all applications.
   
4. **Use Case**:
   - Best when you are done using the computer for the day or won't be using it for an extended period and don't need to save the current state.
   
5. **Data Safety**:
   - Safe during power outages since the computer is already off and no data is being held in memory.

### Summary:

- **Hibernate**: Saves state to disk, uses minimal power, slower to resume, safe during power outages.
- **Sleep**: Saves state to RAM, uses low power, fast to resume, not safe during power outages unless on battery.
- **Poweroff**: Completely turns off, uses no power, slowest to resume, safe during power outages.

Each mode has its specific use cases, and choosing between them depends on the duration of inactivity and the importance of preserving the session state versus saving power.

## How to attach multiple NIC (Network Interface Card)

**How to create network interface?**

From EC2, Network interfaces >> create network interface
Fill the details:
- Description (optional)
- Subnet: The subnet in which to create the network interface
- Private IPv4 address
	- Auto assign
	- Custom
- Elastic Fabric Adapter
	- Enable
- Security Groups

**How to attach that network instance in an EC2 instance**?
- Select the interface, from actions menu click attach.
- Select the instance from instance dropdown.
- And then attach

## How to create reserve instance
- In the Instances section on your sidebar click on the **Reserved instances**.
- Purchase Reserved Instances button click
- Fill these:
	- Platform
	- Tenancy
	- Offering class
	- Instance type
	- Term
	- Payment option:
		- Any
		- No Upfront
		- Partial upfront
		- All upfront
- Search

A table will appear upon search:

![reserve instance table](https://i.imgur.com/0xeHmjM.png)

## AWS IAM (Identity Access Management) Service

**AWS Identity and Access Management (IAM)** is a web service that helps you securely control access to AWS services and resources for your users. With IAM, you can create and manage AWS users and groups, and use permissions to allow and deny their access to AWS resources.

### Key Concepts of IAM:

1. **Users**:
   - **Definition**: An IAM user is an entity you create in AWS to represent a person or application that interacts with AWS.
   - **Use Case**: Users are given unique credentials and permissions to access AWS services and resources.
   - **Types of Credentials**: Users can have two types of credentials: password for the AWS Management Console and access keys for programmatic access.

2. **Groups**:
   - **Definition**: An IAM group is a collection of IAM users. You can manage permissions for multiple users by adding them to groups.
   - **Use Case**: Simplifies the management of permissions for users with similar access requirements.

3. **Roles**:
   - **Definition**: An IAM role is an IAM identity with specific permissions. Roles are intended to be assumed by users, applications, or services that need temporary access to AWS resources.
   - **Use Case**: Used for granting permissions to AWS services, federated users, or applications without long-term credentials.

4. **Policies**:
   - **Definition**: IAM policies are JSON documents that define permissions. Policies specify what actions are allowed or denied for which resources.
   - **Types of Policies**: Managed policies (AWS managed and customer managed) and inline policies (embedded directly within a user, group, or role).

5. **Permissions**:
   - **Definition**: Permissions define what actions a user, group, or role can perform on which resources.
   - **Structure**: Defined in policies using statements that include actions, resources, and effect (allow or deny).

### Key Features of IAM:

1. **Fine-Grained Access Control**:
   - Provides detailed control over who can access which resources and what actions they can perform.

2. **Centralized Management**:
   - Allows centralized management of access permissions for multiple AWS accounts and resources.

3. **Multi-Factor Authentication (MFA)**:
   - Enhances security by requiring users to provide additional verification (something they know and something they have).

4. **Federated Access**:
   - Supports integration with external identity providers (e.g., Active Directory, SAML) to allow single sign-on (SSO) and federated access.

5. **Temporary Security Credentials**:
   - Issues temporary security credentials for roles, reducing the risk associated with long-term credentials.

### IAM Best Practices:

1. **Least Privilege Principle**:
   - Grant the minimum permissions required for users to perform their tasks.

2. **Use Groups to Assign Permissions**:
   - Assign permissions to groups instead of individual users to simplify management.

3. **Enable MFA**:
   - Enable Multi-Factor Authentication for all users, especially those with privileged access.

4. **Use Roles for Applications and Services**:
   - Assign roles to applications and services instead of using access keys, to improve security and manageability.

5. **Regularly Review Permissions**:
   - Regularly audit and review permissions to ensure they align with the current requirements and security policies.

### Example Workflow:

1. **Create an IAM User**:
   - Open the IAM console, navigate to "Users," and click "Add user."
   - Enter the username and select the access type (programmatic access, AWS Management Console access).
   - Attach policies directly or add the user to a group with the necessary permissions.
   - Review and create the user. Download the credentials for safekeeping.

2. **Create an IAM Role**:
   - Open the IAM console, navigate to "Roles," and click "Create role."
   - Select the type of trusted entity (e.g., AWS service, another AWS account, or web identity).
   - Attach policies that define the permissions for the role.
   - Name the role and create it. Use this role in your applications or services to grant the necessary access.

3. **Attach Policies to a Group**:
   - Open the IAM console, navigate to "Groups," and click "Create New Group."
   - Name the group and attach the required policies.
   - Add users to the group to grant them the permissions defined in the policies.

### Summary:

AWS Identity and Access Management (IAM) is a fundamental service for managing access to AWS resources. It provides robust mechanisms to create and manage users, groups, roles, and policies, ensuring fine-grained control over permissions. By following IAM best practices, organizations can enhance security, simplify access management, and ensure compliance with organizational policies.

## Groups in AWS IAM

**Groups** in AWS Identity and Access Management (IAM) are a collection of IAM users. Groups make it easier to manage permissions for multiple users who share similar access needs. By using groups, you can assign permissions to a group and have those permissions apply to all users that are members of the group. This simplifies the management of user permissions.

### Key Concepts of IAM Groups:

1. **Definition**:
   - An IAM group is a container for IAM users. You can use groups to assign permissions to multiple users at once.

2. **Permissions**:
   - Permissions are granted to groups by attaching policies. Policies define what actions users in the group can perform on which resources.
   - Any user that is added to the group inherits the permissions assigned to that group.

3. **Policies**:
   - You can attach managed policies or inline policies to a group.
   - Managed policies can be AWS-managed (provided by AWS) or customer-managed (created by your organization).
   - Inline policies are policies that are embedded directly within a group.

4. **User Management**:
   - Users can be added to or removed from groups as needed. When a user is added to a group, they inherit the permissions assigned to that group. When they are removed, they lose those permissions.

### Benefits of Using IAM Groups:

1. **Simplified Permission Management**:
   - Assign permissions to a group rather than individually to each user, which reduces complexity and administrative overhead.

2. **Consistency**:
   - Ensures that users with similar roles have consistent permissions, reducing the risk of permission misconfiguration.

3. **Scalability**:
   - Makes it easy to manage permissions as the number of users grows. You can simply add new users to the appropriate groups.

4. **Best Practices Alignment**:
   - Aligns with the principle of least privilege by allowing you to manage and review permissions at the group level.

### Example Use Cases:

1. **Departmental Groups**:
   - Create groups based on departments, such as `Developers`, `Admins`, `Finance`, etc. Assign permissions relevant to each department.

2. **Project-Based Groups**:
   - Create groups for specific projects. For example, a group for `ProjectA-Team` that has access only to resources related to Project A.

3. **Role-Based Groups**:
   - Create groups based on roles, such as `ReadOnlyAccess`, `PowerUsers`, `SystemAdministrators`, etc., to control access based on the user’s role in the organization.

### Steps to Manage IAM Groups:

1. **Create a Group**:
   - **AWS Management Console**:
     - Open the IAM console.
     - In the navigation pane, choose "Groups" and then "Create New Group."
     - Enter a group name and click "Next Step."
     - Attach policies to the group and click "Next Step."
     - Review and create the group.
   - **AWS CLI**:
     ```bash
     aws iam create-group --group-name Developers
     ```

2. **Attach a Policy to a Group**:
   - **AWS Management Console**:
     - Select the group from the "Groups" list.
     - In the "Permissions" tab, click "Attach Policy."
     - Choose the policies to attach and click "Attach Policy."
   - **AWS CLI**:
     ```bash
     aws iam attach-group-policy --group-name Developers --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
     ```

3. **Add Users to a Group**:
   - **AWS Management Console**:
     - Select the group from the "Groups" list.
     - In the "Users" tab, click "Add Users to Group."
     - Select the users to add and click "Add Users."
   - **AWS CLI**:
     ```bash
     aws iam add-user-to-group --user-name Alice --group-name Developers
     ```

4. **Remove Users from a Group**:
   - **AWS Management Console**:
     - Select the group from the "Groups" list.
     - In the "Users" tab, select the users to remove and click "Remove User from Group."
   - **AWS CLI**:
     ```bash
     aws iam remove-user-from-group --user-name Alice --group-name Developers
     ```

### Summary:

**IAM Groups** in AWS are a powerful tool for managing permissions across multiple users efficiently. By organizing users into groups and assigning permissions at the group level, you can simplify administration, ensure consistency, and maintain security best practices. This approach makes it easier to scale your permission management as your organization grows and evolves.

## Difference Between Root User and Admin User in AWS

In AWS, both the root user and admin users have significant privileges, but they differ in their intended use, level of access, and best practices for usage. Here’s a detailed comparison:

### Root User

1. **Definition**:
   - The root user is the account that is created when you first set up an AWS account. It has complete, unrestricted access to all AWS resources and services.

2. **Access Level**:
   - The root user has full access to all AWS resources and account settings. This includes access to billing information, service control policies, and the ability to close the AWS account.

3. **Use Cases**:
   - Initial account setup.
   - Account and billing management.
   - Performing tasks that require root privileges (e.g., changing account settings, managing IAM roles and users at the highest level).

4. **Security**:
   - Highly sensitive and should be protected with strong security measures, including Multi-Factor Authentication (MFA).
   - AWS best practices recommend minimizing the use of the root user to reduce the risk of accidental or malicious changes.

5. **Best Practices**:
   - Enable MFA.
   - Create individual IAM users with administrative privileges and use these accounts for everyday tasks.
   - Securely store root user credentials and use them only for necessary administrative tasks.

### Admin User

1. **Definition**:
   - An admin user is an IAM user with administrative privileges. This user can perform most tasks in AWS but does not have the inherent full account control that the root user has.

2. **Access Level**:
   - Admin users typically have permissions granted through policies that allow them to manage AWS services and resources. They can create and manage other IAM users and roles.
   - Admin users do not have access to certain sensitive account and billing information by default, unless explicitly granted.

3. **Use Cases**:
   - Day-to-day administration and management of AWS services and resources.
   - Managing IAM users and permissions.
   - Deploying and maintaining applications and infrastructure.

4. **Security**:
   - Should be protected with strong passwords and MFA.
   - Permissions should be managed carefully to adhere to the principle of least privilege.

5. **Best Practices**:
   - Use groups to assign admin permissions to IAM users.
   - Enable MFA for admin users.
   - Regularly review permissions and access levels to ensure they align with current needs and security policies.
   - Avoid using the root account for daily tasks; use admin users instead.

### Key Differences

1. **Access Scope**:
   - **Root User**: Unrestricted access to all account resources and settings.
   - **Admin User**: Access to resources and services as defined by attached policies, typically broad but can be limited.

2. **Usage**:
   - **Root User**: Reserved for critical tasks and initial setup.
   - **Admin User**: Used for regular administrative tasks and resource management.

3. **Security**:
   - **Root User**: Requires highest level of security due to its unrestricted access.
   - **Admin User**: Requires strong security but can have more granular permissions and restrictions.

4. **Best Practices**:
   - **Root User**: Minimal use, enable MFA, create admin users for regular use.
   - **Admin User**: Use for everyday administration, enable MFA, adhere to least privilege principle.

### Summary

- **Root User**: The root user is the AWS account owner with unrestricted access to all resources and settings. It should be used sparingly and protected with strong security measures.
- **Admin User**: Admin users are IAM users with permissions to manage AWS resources and services. They are used for daily administrative tasks and should have MFA enabled and permissions managed according to best practices.

By distinguishing between the root user and admin users and adhering to best practices, you can enhance the security and manageability of your AWS environment.

**Q: We have a group name "S3-allow" that has a policy attached `S3FullAccess` which gives that group read, write, update and delete permission. And we have another group name "s3-deny" which doesn't have any access to S3 related tasks. And let's say we have added an user with those two groups. Now the question is: what user can do?**

In AWS IAM, when a user is a member of multiple groups, the effective permissions for that user are the combination of all permissions granted by all the policies attached to those groups. However, **explicit deny** permissions take precedence over allow permissions. 

Given the scenario:

- The **`S3-allow`** group has a policy (`S3FullAccess`) attached that grants full access (read, write, update, delete) to S3.
- The **`s3-deny`** group has a policy that explicitly denies access to S3.

If a user is added to both the `S3-allow` and `s3-deny` groups, the user will inherit the policies from both groups. The explicit deny in the `s3-deny` group will override the allow permissions granted by the `S3-allow` group. 

**Resulting Permissions**:

- The user will be denied access to any S3 operations.

**Detailed Explanation**:

1. **Allow Permissions**:
   - From the `S3-allow` group, the user would have permissions to perform all actions on S3 resources.

2. **Deny Permissions**:
   - From the `s3-deny` group, the user would have permissions explicitly denying any S3 operations.

3. **Precedence of Deny**:
   - In IAM, explicit deny always overrides any allow permissions. Therefore, even though the `S3-allow` group grants full access to S3, the explicit deny from the `s3-deny` group will take precedence.

**Example Policy Scenarios**:

#### `S3-allow` Group Policy (S3FullAccess):
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```

#### `s3-deny` Group Policy:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Deny",
            "Action": "s3:*",
            "Resource": "*"
        }
    ]
}
```

**Effective Permissions**:

- The user will not be able to perform any S3 operations due to the explicit deny in the `s3-deny` group policy.

**Summary**:

When a user is a member of both `S3-allow` and `s3-deny` groups:
- The user will be denied all access to S3 resources.
- This is because the explicit deny in the `s3-deny` group policy takes precedence over the allow permissions from the `S3-allow` group policy.


## AWS Password Policy

**Process to access Password Policy**:

- Access Managment >> Account Settings
- Edit
- Two sections to choose:
	- IAM default
	- Custom
		- Password minimum length
		- Password strength
			- Require at least one uppercase letter from the Lation alphabet (A-Z)
			- Require at least one lowercase letter from the Lation alphabet (a-z)
			- Require at least one number 
			- Require at least one non-alphanumeric character (`!@#$%^&*()_+-=[]{}|'`)
		- Other requirements
			- Turn on password expiration
				- Expirate password `how many` day(s)
			- Password expiration requires administrator reset
			- Allow users to change their own password
			- Prevent password reuse
				- Remeber `how many` password(s)

## AWS CLI and configure user account to use AWS CLI

Steps to create access key:

- Create an user
- From the users page, click on the user
- Go to the **Security credentials**
- There is a **Access keys** section
- Create access key
- For **Access key best practices & alternatives**
	- Command Line Interface (CLI)
	- Local code
	- Application running on an AWS compute service
	- Third party service
	- Application running on outside AWS
	- Other
- Click on the **Command Line Interface (CLI)** radio button
- Check on *I understand the above recommendation and want to proceed to create an access key*
- Next
- Set description tag - optional
	- Description tag value
- Create access key

Then download the csv file in **Retrieve access keys** page

Search on google for **aws cli latest version**, or at the moment here is the [link for downloading aws cli](https://docs.aws.amazon.com/cli/latest/userguide/getting-started-install.html).

On your local machine, run the command

```bash
aws configure
```

It will ask for `access key id` and `secret key` that you acquired when you retrieved access keys using csv download.

More prompts are:
- Default region name
- Default output format

Press enter if you do not want to specify default region and output format.

Now your aws cli has been configured. You can now use `aws` command to run any task related to aws services.

For example, if you want to get help on ec2:
```bash
aws ec2 help
```

It will give output available commands for ec2.
More example on getting all the available instances in your AWS account for `ap-south-1`:

```bash
aws ec2 describe-instances --region=ap-south-1
```

**To run an instance**:

At first we need to know which options we need to provide to run an instance:

```bash
aws ec2 run-instances help
```

```bash
aws ec2 run-instances --image-id=ami-0f...278e --instance-type=t2.micro --region=ap-south-1
```

**How to create a profile for different region?**

```bash
aws configure --profile account1
```

It will ask same input you specified above:

```txt
AWS Access Key ID [None]:
AWS Secret Access Key [None]: 
Default Region Name [None]: us-east-1
Default Output Format [None]:
```

Now for example if you want to get all the instances from `us-east-1` which you have used in `account1` profile:

```bash
aws describe-instances --profile=account1
```

**How to make one of your profile as default profile?**

With this bash command:

```bash
export AWS_DEFAULT_PROFILE=account1
```

## What is AWS Roles and How to use AWS CLI without access id and session key

### AWS IAM Roles

**AWS IAM Roles** are a secure way to grant permissions to entities such as AWS services, applications, or users from other accounts. Unlike IAM users, roles do not have permanent credentials. Instead, they have temporary security credentials that can be assumed by trusted entities.

### Key Concepts of IAM Roles:

1. **Definition**:
   - An IAM role is an IAM identity that you can create in your account that has specific permissions.
   - Roles are intended to be assumable by trusted entities like AWS services (e.g., EC2 instances, Lambda functions), other AWS accounts, or federated users.

2. **Temporary Security Credentials**:
   - Roles provide temporary security credentials (access key ID, secret access key, and session token) for short-term use.

3. **Use Cases**:
   - **Cross-account access**: Allowing access to resources in another AWS account.
   - **AWS services**: Granting permissions to AWS services like EC2 instances or Lambda functions.
   - **Federated users**: Allowing access for users authenticated through an external identity provider.

4. **Trust Relationships**:
   - Roles have trust policies that define which entities (principals) are allowed to assume the role.

### Creating and Using an IAM Role:

1. **Create a Role**:
   - **AWS Management Console**:
     - Open the IAM console.
     - In the navigation pane, choose "Roles" and then "Create role."
     - Choose the type of trusted entity (e.g., AWS service, another AWS account, web identity).
     - Attach the necessary permissions policies to the role.
     - Define the trust relationship by specifying the trusted entities.
     - Name the role and complete the creation process.

2. **Assume a Role**:
   - When a role is assumed, it provides temporary security credentials that can be used to access AWS resources.
   - **AWS CLI**:
     ```bash
     aws sts assume-role --role-arn arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME --role-session-name SESSION_NAME
     ```
   - This command returns temporary security credentials (access key ID, secret access key, and session token) that you can use to authenticate AWS CLI commands.

### Using AWS CLI Without Access Key and Secret Key

If you don't want to use long-term access keys for AWS CLI, you can use IAM roles in conjunction with the AWS CLI. This is typically done in two scenarios: on an EC2 instance with an instance profile or by assuming a role using an existing set of temporary credentials.

#### Using IAM Roles with EC2 Instances:

1. **Attach an IAM Role to an EC2 Instance**:
   - Create an IAM role with the necessary permissions.
   - Attach the role to your EC2 instance as an instance profile.
   - When the instance is launched, it will automatically retrieve temporary credentials from the instance metadata service.

2. **Configure AWS CLI**:
   - No need to configure access keys on the instance. The AWS CLI will automatically use the temporary credentials provided by the instance profile.
   - Example command:
     ```bash
     aws s3 ls
     ```

#### Assuming a Role Using AWS CLI:

1. **Assume the Role**:
   - Use the `aws sts assume-role` command to get temporary security credentials.
   ```bash
   aws sts assume-role --role-arn arn:aws:iam::ACCOUNT_ID:role/ROLE_NAME --role-session-name SESSION_NAME
   ```
   - Save the output, which includes the access key ID, secret access key, and session token.

2. **Configure AWS CLI with Temporary Credentials**:
   - Export the temporary credentials as environment variables.
   ```bash
   export AWS_ACCESS_KEY_ID=YOUR_ACCESS_KEY_ID
   export AWS_SECRET_ACCESS_KEY=YOUR_SECRET_ACCESS_KEY
   export AWS_SESSION_TOKEN=YOUR_SESSION_TOKEN
   ```
   - Run your AWS CLI commands using these temporary credentials.
   - Example command:
     ```bash
     aws s3 ls
     ```

### Summary:

- **IAM Roles** provide a secure way to grant temporary access to AWS resources for various entities like AWS services, other AWS accounts, or federated users.
- **Using AWS CLI Without Access Keys**: 
  - On EC2 instances, attach an IAM role, and the CLI will automatically use the temporary credentials.
  - For other cases, use the `aws sts assume-role` command to obtain temporary credentials and set them as environment variables for CLI usage.

These methods improve security by reducing the reliance on long-term access keys and using short-lived, automatically rotated credentials.

**Q: You want to create three different accounts in your aws account. The first user will have `EC2FullAccess` but for only one specific region. One user will have the access to modify instances that has `development` tag. What will be the process?**

**Creating AWS IAM Users with Specific Permissions**

To create three different users with specific permissions in your AWS account, follow these steps:

1. **Create IAM Policies**:
   - Define custom policies that specify the necessary permissions for each user.
   
2. **Create IAM Users**:
   - Create IAM users and attach the respective policies to these users.

**User 1: EC2FullAccess for a Specific Region**

1. **Create a Custom Policy for EC2FullAccess in a Specific Region**:
   - Use the AWS Management Console or AWS CLI to create a policy that grants full EC2 access but restricts it to a specific region.

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": "ec2:*",
               "Resource": "*",
               "Condition": {
                   "StringEquals": {
                       "aws:RequestedRegion": "us-west-2"
                   }
               }
           }
       ]
   }
   ```

   Replace `"us-west-2"` with the desired region.

2. **Create the User and Attach the Policy**:

   ```bash
   aws iam create-user --user-name EC2FullAccessUser
   aws iam attach-user-policy --user-name EC2FullAccessUser --policy-arn arn:aws:iam::aws:policy/AmazonEC2FullAccess
   ```

   Then, create and attach the custom policy:

   ```bash
   aws iam put-user-policy --user-name EC2FullAccessUser --policy-name EC2FullAccessSpecificRegion --policy-document file://ec2-full-access-specific-region.json
   ```

**User 2: Modify Instances with `development` Tag**

1. **Create a Custom Policy for Modifying Instances with `development` Tag**:

   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Effect": "Allow",
               "Action": [
                   "ec2:StartInstances",
                   "ec2:StopInstances",
                   "ec2:RebootInstances",
                   "ec2:TerminateInstances"
               ],
               "Resource": "*",
               "Condition": {
                   "StringEquals": {
                       "ec2:ResourceTag/Environment": "development"
                   }
               }
           }
       ]
   }
   ```

2. **Create the User and Attach the Policy**:

   ```bash
   aws iam create-user --user-name EC2DevTagUser
   aws iam put-user-policy --user-name EC2DevTagUser --policy-name EC2ModifyDevInstances --policy-document file://ec2-modify-dev-instances.json
   ```

**User 3: Read-Only Access to S3**

1. **Attach an Existing Read-Only Policy to the User**:

   ```bash
   aws iam create-user --user-name S3ReadOnlyUser
   aws iam attach-user-policy --user-name S3ReadOnlyUser --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
   ```

**Summary of Steps**:

1. **Create Custom Policies**:
   - Define and save the JSON policies for each specific permission.

2. **Create Users**:
   - Use the AWS Management Console or AWS CLI to create the users.

3. **Attach Policies to Users**:
   - Attach the appropriate policies to each user using either managed policies or inline policies.

**Example Commands**:

**EC2FullAccessUser**:

```bash
aws iam create-user --user-name EC2FullAccessUser
aws iam put-user-policy --user-name EC2FullAccessUser --policy-name EC2FullAccessSpecificRegion --policy-document file://ec2-full-access-specific-region.json
```

**EC2DevTagUser**:

```bash
aws iam create-user --user-name EC2DevTagUser
aws iam put-user-policy --user-name EC2DevTagUser --policy-name EC2ModifyDevInstances --policy-document file://ec2-modify-dev-instances.json
```

**S3ReadOnlyUser**:

```bash
aws iam create-user --user-name S3ReadOnlyUser
aws iam attach-user-policy --user-name S3ReadOnlyUser --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
```

By following these steps, you'll create three users with the specified permissions tailored to your requirements.

## What/Why is the CloudShell

**AWS CloudShell** is a browser-based shell that allows you to securely manage, explore, and interact with your AWS resources. It provides a pre-configured environment with the AWS Command Line Interface (CLI) and other tools, enabling you to perform various tasks without needing to install or configure anything locally.

### Key Features of AWS CloudShell:

1. **Pre-Configured Environment**:
   - AWS CloudShell comes pre-installed with the latest version of the AWS CLI, as well as other development tools like Python, Node.js, Git, and more.

2. **Secure Access**:
   - CloudShell uses the same authentication and authorization mechanisms as the AWS Management Console. You can leverage IAM roles and policies to control access.

3. **Persistent Home Directory**:
   - The home directory in CloudShell is persistent, meaning that your files and settings are preserved between sessions. Each region provides up to 1GB of persistent storage.

4. **Integrated with AWS Console**:
   - CloudShell is integrated directly into the AWS Management Console, making it easy to access and use alongside other AWS services.

5. **No Cost**:
   - There is no additional charge for using CloudShell. You only incur costs for the underlying AWS resources that you use, such as S3 storage or EC2 instances.

### Use Cases for AWS CloudShell:

1. **AWS Resource Management**:
   - Perform common tasks such as managing EC2 instances, configuring S3 buckets, and running AWS Lambda functions using the CLI.

2. **Scripting and Automation**:
   - Develop and run scripts to automate AWS resource management, deployment, and other operational tasks.

3. **Development and Testing**:
   - Use CloudShell for quick development and testing of applications and scripts without needing to set up a local environment.

4. **Learning and Experimentation**:
   - Experiment with AWS services and learn AWS CLI commands in a safe and convenient environment.

### How to Use AWS CloudShell:

1. **Accessing CloudShell**:
   - Log in to the AWS Management Console.
   - Click on the CloudShell icon in the console navigation bar, or navigate to it through the "Services" menu.

2. **Using the Shell**:
   - Once CloudShell launches, you’ll have a command-line interface where you can start running AWS CLI commands and other tools.

3. **Managing Files**:
   - Use standard Linux commands to create, edit, and manage files in your home directory.
   - You can also upload and download files between your local machine and CloudShell using the built-in file transfer capabilities.

4. **Running Commands**:
   - Run AWS CLI commands directly from CloudShell to interact with AWS services.
   - Example:
     ```bash
     aws s3 ls
     ```

### Example Commands:

**List S3 Buckets**:
```bash
aws s3 ls
```

**Describe EC2 Instances**:
```bash
aws ec2 describe-instances
```

**Create a New S3 Bucket**:
```bash
aws s3 mb s3://my-new-bucket
```

**Upload a File to S3**:
```bash
aws s3 cp my-file.txt s3://my-new-bucket/my-file.txt
```

**Deploy a Lambda Function**:
```bash
aws lambda create-function --function-name my-function --runtime python3.8 --role arn:aws:iam::123456789012:role/lambda-ex --handler lambda_function.lambda_handler --zip-file fileb://function.zip
```

### Summary:

AWS CloudShell provides a convenient, secure, and fully-managed shell environment for interacting with AWS resources. It eliminates the need for local setup and configuration, making it easier to manage and automate your AWS environment directly from the browser. This tool is particularly useful for developers, administrators, and anyone needing quick access to AWS CLI and related tools without the hassle of environment setup.

## IAM Access Advisor

**IAM Access Advisor** is a feature in AWS Identity and Access Management (IAM) that helps you identify unused permissions associated with IAM roles and users. By providing data on the last accessed time for services, Access Advisor assists in refining permissions to adhere to the principle of least privilege.

### Key Features of IAM Access Advisor:

1. **Service Last Accessed Data**:
   - Access Advisor shows the last time a service was accessed using the permissions granted by an IAM policy attached to a user or role.

2. **Identify Unused Permissions**:
   - Helps you identify which permissions have not been used recently or ever. This is useful for tightening security by removing unnecessary permissions.

3. **Supports Multiple IAM Entities**:
   - Provides data for IAM users, groups, roles, and policies.

4. **Integration with IAM Console**:
   - Available directly within the IAM console, making it easy to access and use.

### Benefits of IAM Access Advisor:

1. **Enhanced Security**:
   - By identifying and removing unused permissions, you can reduce the attack surface and potential for misuse.

2. **Compliance and Governance**:
   - Helps meet compliance requirements by ensuring that permissions are aligned with actual usage patterns.

3. **Simplified Permissions Management**:
   - Facilitates the process of refining and optimizing IAM policies, making permissions management more straightforward.

### How to Use IAM Access Advisor:

#### Viewing Access Advisor Data for a User or Role:

1. **Sign in to the AWS Management Console**:
   - Navigate to the IAM console.

2. **Select the User or Role**:
   - In the navigation pane, choose "Users" or "Roles" and then select the specific user or role you want to analyze.

3. **Access the Access Advisor Tab**:
   - For the selected user or role, choose the "Access Advisor" tab.
   - The Access Advisor tab displays a list of services to which the user or role has access and the last accessed time for each service.

#### Example Steps to View Access Advisor Data for a User:

1. **Open the IAM Console**:
   - Go to the [IAM Management Console](https://console.aws.amazon.com/iam/).

2. **Choose Users**:
   - In the navigation pane, select "Users."

3. **Select a User**:
   - Click on the name of the user you want to view.

4. **Choose the Access Advisor Tab**:
   - Click on the "Access Advisor" tab to see the last accessed data for various services.

### Example Output of Access Advisor Data:

The Access Advisor tab might show a table with columns such as:

- **Service Name**: The name of the AWS service (e.g., Amazon S3, EC2).
- **Last Accessed**: The last time the service was accessed using the permissions associated with the user or role.

### Using Access Advisor Data:

1. **Review the Last Accessed Times**:
   - Check the "Last Accessed" column to see when the services were last used.

2. **Identify Unused Permissions**:
   - Look for services that have not been accessed recently or at all.

3. **Refine IAM Policies**:
   - Based on the data, modify the IAM policies to remove permissions for unused services. This helps implement the principle of least privilege.

### Example: Refining Permissions Based on Access Advisor Data:

Suppose you have a user with the following permissions for services and last accessed data:

| Service Name | Last Accessed       |
|--------------|---------------------|
| Amazon S3    | 2024-06-20 12:34:56 |
| Amazon EC2   | Never               |
| Lambda       | 2024-06-15 08:45:00 |

- **Amazon EC2**: If the user has never accessed EC2, consider removing EC2 permissions from the user's policy.
- **Amazon S3 and Lambda**: Permissions for these services can be retained or reviewed for more granular adjustments.

### Summary:

IAM Access Advisor is a valuable tool for managing IAM permissions by providing insights into service usage. It helps enhance security, ensure compliance, and simplify the management of IAM policies. By regularly reviewing Access Advisor data, you can refine IAM policies to grant only the permissions necessary for users and roles to perform their tasks.

## IAM Credential report

An **IAM Credential Report** is a comprehensive report generated by AWS that provides details about the status of various credentials associated with IAM users in your AWS account. The report includes information on password status, access keys, MFA devices, and more. This report is crucial for auditing and managing IAM users' credentials, helping ensure that security best practices are followed.

### Key Features of IAM Credential Report:

1. **Detailed Information**:
   - The report provides detailed information about each IAM user's credentials, including the age of the password, access keys, and whether MFA is enabled.

2. **Security Auditing**:
   - Helps identify potential security issues, such as unused or old access keys, users without MFA, and users who haven't rotated their passwords recently.

3. **CSV Format**:
   - The report is generated in CSV format, making it easy to analyze using spreadsheets or other tools.

### Contents of the IAM Credential Report:

The IAM Credential Report includes the following columns:

- **user**: The name of the IAM user.
- **arn**: The Amazon Resource Name (ARN) of the IAM user.
- **user_creation_time**: The date and time when the IAM user was created.
- **password_enabled**: Indicates whether the user has a password for accessing the AWS Management Console.
- **password_last_used**: The last time the user signed in to the AWS Management Console with their password.
- **password_last_changed**: The last time the user's password was changed.
- **password_next_rotation**: The date and time when the user's password is scheduled to be rotated (if applicable).
- **mfa_active**: Indicates whether the user has MFA enabled.
- **access_key_1_active**: Indicates whether the user's first access key is active.
- **access_key_1_last_rotated**: The last time the user's first access key was rotated.
- **access_key_1_last_used_date**: The last time the user's first access key was used.
- **access_key_2_active**: Indicates whether the user's second access key is active.
- **access_key_2_last_rotated**: The last time the user's second access key was rotated.
- **access_key_2_last_used_date**: The last time the user's second access key was used.
- **cert_1_active**: Indicates whether the user's first signing certificate is active.
- **cert_2_active**: Indicates whether the user's second signing certificate is active.

### How to Generate and Download the IAM Credential Report:

#### Using the AWS Management Console:

1. **Sign in to the AWS Management Console**:
   - Open the IAM console at [https://console.aws.amazon.com/iam/](https://console.aws.amazon.com/iam/).

2. **Generate the Report**:
   - In the navigation pane, choose "Credential report."
   - Click on the "Download Report" button to generate and download the report.

#### Using the AWS CLI:

1. **Generate the Report**:
   - Run the following command to generate the IAM Credential Report:
     ```bash
     aws iam generate-credential-report
     ```

2. **Download the Report**:
   - Once the report is generated, you can download it using the following command:
     ```bash
     aws iam get-credential-report --query 'Content' --output text | base64 -d > credential_report.csv
     ```

### Example Analysis of IAM Credential Report:

#### Sample CSV Output:
```csv
user,arn,user_creation_time,password_enabled,password_last_used,password_last_changed,password_next_rotation,mfa_active,access_key_1_active,access_key_1_last_rotated,access_key_1_last_used_date,access_key_2_active,access_key_2_last_rotated,access_key_2_last_used_date,cert_1_active,cert_2_active
alice,arn:aws:iam::123456789012:user/alice,2024-01-01T12:00:00Z,true,2024-06-20T12:34:56Z,2024-05-01T12:00:00Z,,true,true,2024-05-01T12:00:00Z,2024-06-20T12:34:56Z,false,,,true,false
bob,arn:aws:iam::123456789012:user/bob,2024-02-01T12:00:00Z,false,,,,false,false,,,false,,,false,false
```

#### Key Insights:
- **Alice**:
  - Password enabled and last used recently.
  - MFA is active.
  - Access key 1 is active and was last used recently.
  - No second access key or signing certificates.

- **Bob**:
  - No password set.
  - No MFA.
  - No access keys or signing certificates.

### Using the Report for Security Auditing:

1. **Identify Users Without MFA**:
   - Check the `mfa_active` column to ensure all users have MFA enabled.

2. **Review Password and Access Key Usage**:
   - Look at the `password_last_used` and `access_key_last_used_date` columns to identify unused or stale credentials.

3. **Monitor Credential Rotation**:
   - Ensure passwords and access keys are rotated regularly by checking the `password_last_changed` and `access_key_last_rotated` columns.

4. **Audit Inactive Credentials**:
   - Identify and deactivate any inactive access keys or signing certificates.

### Summary:

The IAM Credential Report is an essential tool for auditing and managing IAM user credentials in AWS. By providing detailed information on the status and usage of passwords, access keys, and MFA, it helps ensure that security best practices are followed, and potential security risks are mitigated. Regularly generating and reviewing this report is a key aspect of maintaining a secure AWS environment.

## S3 (Simple Storage Service)

As per AWS we can store infinite data in S3 bucket. It is a more important core service. If you're a developer or dev-ops it is crucial to learn this service. For an example when you build an application, if a user upload their files where those files need to be stored? In your server right? But what will happen if your server crashes? So, when your application stores this uploaded data it is benefecial to upload in S3 which reliable. 


Amazon S3 (Simple Storage Service) is a scalable, high-speed, web-based cloud storage service designed for online backup and archiving of data and applications. It is a highly durable and available object storage service that provides secure, flexible, and cost-effective storage for any amount of data.

### Key Features of Amazon S3:

1. **Scalability**:
   - S3 automatically scales storage capacity to handle growing amounts of data.

2. **Durability and Availability**:
   - Designed for 99.999999999% (11 9's) of durability.
   - High availability with multiple storage classes to suit different use cases.

3. **Security and Compliance**:
   - Supports data encryption at rest and in transit.
   - Fine-grained access control using AWS IAM, bucket policies, and ACLs.
   - Complies with various industry standards and regulations.

4. **Cost-Effective**:
   - Pay-as-you-go pricing model.
   - Various storage classes for different use cases and cost requirements (e.g., S3 Standard, S3 Intelligent-Tiering, S3 Glacier).

5. **Performance**:
   - Low latency and high throughput performance.
   - Supports multipart uploads and data transfer acceleration.

6. **Data Management Features**:
   - Versioning, lifecycle policies, replication, and event notifications.
   - Integrates with AWS Lambda for serverless data processing.

### Common Use Cases for Amazon S3:

1. **Backup and Restore**:
   - Store backup data for on-premises or cloud applications.

2. **Archiving**:
   - Archive older data cost-effectively with S3 Glacier.

3. **Content Storage and Distribution**:
   - Store and distribute static content such as images, videos, and documents.

4. **Big Data Analytics**:
   - Store and analyze large datasets with integration to AWS analytics services.

5. **Disaster Recovery**:
   - Use cross-region replication for disaster recovery.

### Managing Objects in S3

#### Uploading an Object:

##### Using AWS Management Console:

1. **Navigate to Your Bucket**:
   - Open the S3 console and select your bucket.
2. **Upload Files**:
   - Click the "Upload" button and follow the prompts to select and upload files.

##### Using AWS CLI:

```bash
aws s3 cp my-file.txt s3://my-unique-bucket-name/my-file.txt
```

#### Downloading an Object:

##### Using AWS Management Console:

1. **Navigate to Your Bucket**:
   - Open the S3 console and select your bucket.
2. **Download Files**:
   - Find the file you want to download, select it, and click the "Download" button.

##### Using AWS CLI:

```bash
aws s3 cp s3://my-unique-bucket-name/my-file.txt my-file.txt
```

### Access Control in S3

#### Bucket Policies:
- JSON-based policies attached to buckets that define permissions for the entire bucket.
- Example:
```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::my-unique-bucket-name/*"
        }
    ]
}
```

#### Access Control Lists (ACLs):
- Define permissions for individual objects or buckets.
- Can specify permissions for AWS accounts and predefined groups.

#### IAM Policies:
- Attach IAM policies to users, groups, or roles to control access to S3 resources.

### Summary

Amazon S3 is a versatile and robust cloud storage service offering high durability, availability, and security. It supports a wide range of use cases, from simple storage to complex data processing and analytics. You can manage S3 using the AWS Management Console, CLI, or SDKs, and control access with bucket policies, ACLs, and IAM policies.

## S3 - How to create bucket in AWS

To create an S3 bucket in AWS, you can use either the AWS Management Console, the AWS CLI (Command Line Interface), or the AWS SDKs. Below are the steps for each method:

### Using AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to [https://aws.amazon.com/](https://aws.amazon.com/) and sign in with your credentials.

2. **Navigate to Amazon S3**:
   - In the AWS Management Console, type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Create a New Bucket**:
   - Click on the "Create bucket" button.

4. **Configure Bucket Settings**:
   - **Bucket name**: Enter a unique name for your bucket. Bucket names must be globally unique and comply with certain naming conventions.
   - **Region**: Select the AWS region where you want the bucket to reside. It’s generally best to choose a region that is geographically close to you or your users to minimize latency.

5. **Configure Options** (Optional):
   - **Object Ownership**: Choose whether to disable or enable ACLs.
   - **Bucket Versioning**: Enable or disable versioning for objects in the bucket.
   - **Default encryption**: Enable default encryption if you want objects in the bucket to be encrypted automatically.
   - **Advanced settings**: Configure additional settings like tags, logging, and Object Lock.

6. **Set Permissions**:
   - Configure permissions for the bucket, such as blocking public access and setting up access control lists (ACLs) or bucket policies.

7. **Review and Create**:
   - Review your settings and click the "Create bucket" button to create the bucket.

### Using AWS CLI

1. **Install AWS CLI**:
   - If you haven't already, install the AWS CLI by following the instructions [here](https://docs.aws.amazon.com/cli/latest/userguide/install-cliv2.html).

2. **Configure AWS CLI**:
   - Run `aws configure` and provide your AWS access key, secret key, default region, and output format.
     ```bash
     aws configure
     ```

3. **Create the Bucket**:
   - Use the `aws s3api create-bucket` command to create the bucket. Specify the bucket name and region.
     ```bash
     aws s3api create-bucket --bucket my-unique-bucket-name --region us-west-2 --create-bucket-configuration LocationConstraint=us-west-2
     ```

### Using AWS SDK (Python Example with Boto3)

1. **Install Boto3**:
   - If you haven’t already, install Boto3 using pip.
     ```bash
     pip install boto3
     ```

2. **Create the Bucket**:
   - Use the following Python script to create an S3 bucket.
     ```python
     import boto3

     s3 = boto3.client('s3')

     bucket_name = 'my-unique-bucket-name'
     region = 'us-west-2'

     s3.create_bucket(
         Bucket=bucket_name,
         CreateBucketConfiguration={'LocationConstraint': region}
     )

     print(f'Bucket {bucket_name} created successfully.')
     ```

### Summary

To create an S3 bucket in AWS, you can use the AWS Management Console for a graphical interface, the AWS CLI for command-line operations, or AWS SDKs like Boto3 for programmatic access. Each method requires specifying a unique bucket name and selecting the desired AWS region. Additional settings and permissions can be configured according to your needs.

## S3 - Public your bucket object

Making an S3 bucket object public involves several steps to ensure that the objects within your S3 bucket are accessible by anyone over the internet. Below are the steps to achieve this using the AWS Management Console, AWS CLI, and an example of setting a bucket policy.

### Using AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to [AWS Management Console](https://aws.amazon.com/console/) and sign in with your credentials.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket name that contains the object you want to make public.

4. **Select the Object**:
   - Find and select the object you want to make public.

5. **Object Actions**:
   - Click on the "Actions" button and select "Make public".

6. **Confirm**:
   - Confirm the action. The object will now be publicly accessible.

### Using AWS CLI

1. **Install and Configure AWS CLI**:
   - If you haven’t already, install the AWS CLI and configure it with your credentials.
     ```bash
     aws configure
     ```

2. **Make an Object Public**:
   - Use the `aws s3api put-object-acl` command to make an object public.
     ```bash
     aws s3api put-object-acl --bucket my-unique-bucket-name --key my-object-key --acl public-read
     ```
   - Alternatively, you can use the `aws s3 cp` command with the `--acl public-read` option when uploading the object.
     ```bash
     aws s3 cp my-file.txt s3://my-unique-bucket-name/my-file.txt --acl public-read
     ```

### Using a Bucket Policy

To make all objects in a bucket public, you can attach a bucket policy that grants public read access to all objects in the bucket.

1. **Open the S3 Console**:
   - Navigate to the S3 console and select your bucket.

2. **Go to Permissions**:
   - Click on the "Permissions" tab and then "Bucket Policy".

3. **Add Bucket Policy**:
   - Copy and paste the following JSON policy into the bucket policy editor:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Sid": "PublicReadGetObject",
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::my-unique-bucket-name/*"
             }
         ]
     }
     ```
   - Replace `my-unique-bucket-name` with the name of your bucket.

4. **Save Changes**:
   - Click "Save" to apply the policy.

### Example: Step-by-Step Process

#### Using AWS CLI:

1. **Upload and Make Public**:
   ```bash
   aws s3 cp my-file.txt s3://my-unique-bucket-name/my-file.txt --acl public-read
   ```

2. **Make an Existing Object Public**:
   ```bash
   aws s3api put-object-acl --bucket my-unique-bucket-name --key my-object-key --acl public-read
   ```

#### Using a Bucket Policy:

1. **Navigate to Bucket Policy Editor**:

2. **Add and Save Bucket Policy**:
   ```json
   {
       "Version": "2012-10-17",
       "Statement": [
           {
               "Sid": "PublicReadGetObject",
               "Effect": "Allow",
               "Principal": "*",
               "Action": "s3:GetObject",
               "Resource": "arn:aws:s3:::my-unique-bucket-name/*"
           }
       ]
   }
   ```

### Summary

Making an S3 bucket object public can be done through the AWS Management Console, AWS CLI, or by applying a bucket policy. Each method requires specific steps to ensure the object or all objects within a bucket are accessible to the public. Always be cautious when making objects public, as it means anyone on the internet can access them.

## S3 Objects - S3 Object Properties - S3 Object Metadata - S3 Object Key

The maximum size of uploaded file size is 5TB or 5000GB.

### S3 Objects

An **S3 object** is the fundamental entity stored in Amazon S3. Objects consist of the following components:
1. **Object Data**: The actual data (e.g., a file).
2. **Metadata**: Information about the data.
3. **Key**: The unique identifier for the object within a bucket.
4. **Version ID**: If versioning is enabled, each version of the object will have a unique version ID.
5. **Access Control Information**: Permissions that determine who can access the object.

### S3 Object Properties

**S3 object properties** provide additional information about the object, including:
1. **Storage Class**: Defines the storage tier of the object (e.g., Standard, Intelligent-Tiering, Glacier).
2. **Last Modified**: The date and time when the object was last modified.
3. **Size**: The size of the object in bytes.
4. **ETag**: The entity tag, which is an MD5 hash of the object data used for data integrity verification.
5. **Version ID**: If versioning is enabled, this is the ID of the object's version.
6. **Server-Side Encryption**: Indicates if and how the object is encrypted (e.g., SSE-S3, SSE-KMS, SSE-C).
7. **Storage Class Analysis**: Provides recommendations for transitioning objects to different storage classes based on access patterns.

### S3 Object Metadata

**S3 object metadata** is a set of name-value pairs that describe the object. Metadata is divided into two categories:
1. **System Metadata**: Metadata set by Amazon S3, such as `Last-Modified`, `Content-Length`, `Content-Type`, `ETag`, and `x-amz-meta`.
2. **User Metadata**: Metadata set by the user. This metadata is prefixed with `x-amz-meta-` (e.g., `x-amz-meta-my-custom-meta`).

Example of adding user metadata using AWS CLI:
```bash
aws s3 cp my-file.txt s3://my-bucket/my-file.txt --metadata x-amz-meta-mykey=myvalue
```

### S3 Object Key

An **S3 object key** is the unique identifier for an object within a bucket. The combination of a bucket name and the object key uniquely identifies the object within the entire Amazon S3 namespace. Object keys are similar to file paths in a file system.

- **Object Key Structure**: S3 object keys can include slashes (`/`), which allows you to simulate a hierarchical folder structure.
- **Example**:
  - Bucket: `my-bucket`
  - Object Key: `folder/subfolder/my-file.txt`

### Example: Working with S3 Objects

#### Uploading an Object with Metadata

Using AWS CLI:
```bash
aws s3 cp my-file.txt s3://my-bucket/my-file.txt --metadata x-amz-meta-mykey=myvalue
```

#### Retrieving Object Metadata

Using AWS CLI:
```bash
aws s3api head-object --bucket my-bucket --key my-file.txt
```

#### Example Response:
```json
{
    "AcceptRanges": "bytes",
    "LastModified": "2024-06-26T00:00:00.000Z",
    "ContentLength": 12345,
    "ETag": "\"d41d8cd98f00b204e9800998ecf8427e\"",
    "ContentType": "text/plain",
    "Metadata": {
        "x-amz-meta-mykey": "myvalue"
    }
}
```

### Summary

- **S3 Objects** are the core entities stored in Amazon S3, consisting of data, metadata, keys, and version IDs.
- **Object Properties** provide additional details like storage class, size, and encryption status.
- **Object Metadata** includes system metadata set by Amazon S3 and user metadata defined by the user.
- **Object Keys** are unique identifiers for objects within a bucket, enabling a structured organization similar to a file system.

Understanding these components helps you effectively manage and utilize your data stored in Amazon S3.

## S3 Versioning - Prevent Object from deletion

### Amazon S3 Versioning

**Amazon S3 Versioning** is a feature that allows you to keep multiple versions of an object in the same bucket. It helps in preserving, retrieving, and restoring every version of every object stored in an Amazon S3 bucket. Versioning is useful for protecting data against accidental or malicious deletions and overwrites.

#### Key Features of S3 Versioning

1. **Multiple Versions**:
   - Store multiple versions of an object to recover from unintended user actions and application failures.

2. **Object Recovery**:
   - Restore previous versions of an object.
   
3. **Enhanced Data Protection**:
   - Prevents data loss by enabling you to recover deleted or overwritten objects.

4. **Compatibility**:
   - Works seamlessly with S3 lifecycle policies to manage versions automatically.

### Enabling Versioning on a Bucket

#### Using AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to [AWS Management Console](https://aws.amazon.com/console/) and sign in with your credentials.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket name where you want to enable versioning.

4. **Go to Properties**:
   - Click on the "Properties" tab.

5. **Enable Versioning**:
   - Under the "Versioning" section, click "Edit".
   - Select "Enable" and click "Save changes".

#### Using AWS CLI

1. **Enable Versioning**:
   ```bash
   aws s3api put-bucket-versioning --bucket my-bucket --versioning-configuration Status=Enabled
   ```

2. **Check Versioning Status**:
   ```bash
   aws s3api get-bucket-versioning --bucket my-bucket
   ```

### Preventing Object Deletion with Versioning

#### Overview

When versioning is enabled, deleting an object does not actually remove it; instead, Amazon S3 adds a delete marker. The delete marker becomes the current version of the object, and the previous versions remain in the bucket and can be retrieved or restored.

### Using Bucket Policies and IAM Policies

To prevent deletion of objects, you can use bucket policies and IAM policies to control access to S3 actions. Here's how you can prevent the deletion of objects:

#### Bucket Policy to Deny Delete Actions

1. **Create a Bucket Policy**:
   - Navigate to the "Permissions" tab of your bucket.
   - Click "Bucket Policy" and add the following policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Deny",
                 "Principal": "*",
                 "Action": "s3:DeleteObject",
                 "Resource": "arn:aws:s3:::my-bucket/*"
             },
             {
                 "Effect": "Deny",
                 "Principal": "*",
                 "Action": "s3:DeleteObjectVersion",
                 "Resource": "arn:aws:s3:::my-bucket/*"
             }
         ]
     }
     ```

#### IAM Policy to Deny Delete Actions

1. **Create an IAM Policy**:
   - Go to the IAM console, select "Policies", and create a new policy with the following JSON:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Deny",
                 "Action": [
                     "s3:DeleteObject",
                     "s3:DeleteObjectVersion"
                 ],
                 "Resource": [
                     "arn:aws:s3:::my-bucket/*"
                 ]
             }
         ]
     }
     ```
   - Attach this policy to the IAM users or roles that should be restricted from deleting objects.

### Recovering Deleted Objects

To recover an object that has been deleted (i.e., has a delete marker), you can list all versions of the object and delete the delete marker.

#### Using AWS Management Console

1. **Navigate to the Object**:
   - Go to the bucket and navigate to the object.

2. **Show Versions**:
   - Enable "Show versions" to list all versions and delete markers of the object.

3. **Delete the Delete Marker**:
   - Select the delete marker and delete it to restore the object.

#### Using AWS CLI

1. **List Object Versions**:
   ```bash
   aws s3api list-object-versions --bucket my-bucket --prefix my-object
   ```

2. **Delete the Delete Marker**:
   ```bash
   aws s3api delete-object --bucket my-bucket --key my-object --version-id <delete-marker-version-id>
   ```

### Summary

- **Amazon S3 Versioning** enables you to keep multiple versions of objects, providing a way to recover from deletions and overwrites.
- **Preventing Deletions** can be achieved by using bucket policies and IAM policies to deny delete actions.
- **Recovering Deleted Objects** involves listing object versions and removing the delete marker to restore the object.

By using these techniques, you can ensure your data in Amazon S3 is protected against accidental or malicious deletions.

## Host static website in S3

Hosting a static website on Amazon S3 is a cost-effective and scalable way to serve web content. Here’s a step-by-step guide on how to set up a static website on S3:

### Step-by-Step Guide to Host a Static Website on S3

#### Step 1: Create an S3 Bucket

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in with your credentials.

2. **Navigate to Amazon S3**:
   - In the AWS Management Console, type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Create a Bucket**:
   - Click on the "Create bucket" button.
   - **Bucket Name**: Enter a unique bucket name that matches your website domain (e.g., `example.com`).
   - **Region**: Select a region.
   - Click "Create bucket".

#### Step 2: Upload Your Website Content

1. **Select Your Bucket**:
   - Click on the bucket name you just created.

2. **Upload Files**:
   - Click the "Upload" button.
   - Add your website files (e.g., `index.html`, `styles.css`, `images/`).
   - Click "Upload".

#### Step 3: Configure the Bucket for Website Hosting

1. **Enable Static Website Hosting**:
   - Go to the "Properties" tab of your bucket.
   - Scroll down to the "Static website hosting" section.
   - Click "Edit".
   - Select "Enable".
   - **Index document**: Enter `index.html`.
   - **Error document**: Enter `error.html` (optional).
   - Click "Save changes".

#### Step 4: Set Bucket Policy for Public Access

1. **Go to Permissions**:
   - Click on the "Permissions" tab.

2. **Edit Bucket Policy**:
   - Scroll down to the "Bucket policy" section and click "Edit".
   - Add the following bucket policy to allow public read access to your website:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::your-bucket-name/*"
             }
         ]
     }
     ```
   - Replace `your-bucket-name` with the name of your S3 bucket.
   - Click "Save changes".

#### Step 5: Access Your Static Website

1. **Obtain the Endpoint URL**:
   - Go back to the "Properties" tab.
   - In the "Static website hosting" section, you will see the endpoint URL for your website.

2. **Visit Your Website**:
   - Open the endpoint URL in your web browser to see your static website live.

### Optional: Use a Custom Domain with Amazon Route 53

If you want to use a custom domain for your website, follow these additional steps:

1. **Register a Domain**:
   - If you don't already have a domain, you can register one using [Amazon Route 53](https://aws.amazon.com/route53/).

2. **Create a Hosted Zone**:
   - In the Route 53 console, create a hosted zone for your domain.

3. **Update DNS Records**:
   - Add an alias record in Route 53 that points your domain to the S3 bucket website endpoint.

4. **Configure Bucket for Custom Domain**:
   - Ensure your bucket name matches your custom domain name (e.g., `example.com`).
   - Update the static website hosting settings if needed.

### Example of Bucket Policy for Public Access

Here is an example of a bucket policy to allow public read access to all objects in the bucket:

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::example.com/*"
        }
    ]
}
```

### Summary

Hosting a static website on Amazon S3 involves creating an S3 bucket, uploading your website files, configuring the bucket for static website hosting, setting the appropriate permissions, and optionally setting up a custom domain using Amazon Route 53. This setup allows you to serve web content efficiently and cost-effectively, leveraging the scalability and durability of Amazon S3.

## How to redirect in S3 static website from one to another

Amazon S3 provides functionality to redirect requests from one object to another within the same bucket or to an external URL. This is useful for managing changes in your website's structure or for redirecting traffic from an old domain to a new one. Here’s how you can set up these redirects:

### Redirecting Using Static Website Hosting Configuration

#### Redirect All Requests to Another Host Name

If you want to redirect all requests to another host name (e.g., redirect `example.com` to `www.example.com`):

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in with your credentials.

2. **Navigate to Amazon S3**:
   - In the AWS Management Console, type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket name that you want to redirect.

4. **Go to Properties**:
   - Click on the "Properties" tab.

5. **Edit Static Website Hosting**:
   - Scroll down to the "Static website hosting" section and click "Edit".

6. **Enable Redirect All Requests**:
   - Select "Enable" for "Static website hosting".
   - Choose "Redirect requests" option.
   - **Host name**: Enter the target host name (e.g., `www.example.com`).
   - **Protocol**: Specify `http` or `https` as needed.
   - Click "Save changes".

#### Redirect Individual Objects

If you want to redirect specific objects to other locations, you can set metadata on each object to perform the redirect.

1. **Upload or Select the Object**:
   - Navigate to the bucket and either upload a new object or select an existing object.

2. **Set Object Metadata for Redirection**:
   - Click on the object name to open its details.
   - Go to the "Properties" tab.
   - In the "Metadata" section, click "Edit" and add the following metadata:
     - **Key**: `x-amz-website-redirect-location`
     - **Value**: The destination URL (e.g., `/new-path/`, `/another-object.html`, or `https://www.example.com/`).

3. **Save Changes**:
   - Click "Save changes" to apply the metadata.

### Using AWS CLI for Redirects

#### Redirecting All Requests to Another Host Name

To configure bucket-level redirects using the AWS CLI:

```bash
aws s3api put-bucket-website --bucket my-bucket --website-configuration '{
  "RedirectAllRequestsTo": {
    "HostName": "www.example.com",
    "Protocol": "https"
  }
}'
```

#### Redirecting Individual Objects

To set the redirect location for an individual object using the AWS CLI:

```bash
aws s3api put-object --bucket my-bucket --key my-object-key --website-redirect-location "/new-path/"
```

### Example of JSON Configuration for Bucket Redirect

If you are using the AWS Management Console to configure static website hosting redirects, you can directly enter JSON configuration for more complex setups. Here’s an example:

```json
{
    "RedirectAllRequestsTo": {
        "HostName": "www.example.com",
        "Protocol": "https"
    }
}
```

### Using Bucket Policies for Redirection (Advanced)

For more advanced use cases, you can create a bucket policy that includes redirects. This is typically done in combination with Route 53 or other DNS services to ensure seamless redirection.

### Summary

Amazon S3 allows you to set up redirects using its static website hosting feature. You can redirect all requests to a new host or set up individual object redirects by configuring the appropriate metadata. These redirects can be managed through the AWS Management Console or using the AWS CLI. By setting up these redirects, you can efficiently manage changes in your website’s structure and direct traffic appropriately.

## How to change prefix in static website S3

Changing the prefix in an S3 static website typically means that you want to move or rename the directory structure of your site within the S3 bucket. For instance, you might want to change URLs from `example.com/old-prefix/page.html` to `example.com/new-prefix/page.html`.

Here’s a step-by-step guide on how to change the prefix for a static website hosted on Amazon S3:

### Step 1: Copy Objects to the New Prefix

To change the prefix, you need to copy objects from the old prefix to the new prefix. This can be done using the AWS Management Console or the AWS CLI.

#### Using the AWS CLI

1. **Copy Objects to the New Prefix**:
   ```bash
   aws s3 cp s3://your-bucket/old-prefix/ s3://your-bucket/new-prefix/ --recursive
   ```

2. **Verify the Copy**:
   Ensure that all files have been copied to the new prefix.

### Step 2: Update Your Website Configuration

If your website configuration, such as HTML files, references the old prefix, you need to update these references to the new prefix.

1. **Update HTML Files**:
   - Open your HTML files and update any URLs that reference the old prefix to use the new prefix instead.

2. **Upload Updated Files**:
   - Upload the updated HTML files back to the new prefix in the S3 bucket.

### Step 3: Delete Objects from the Old Prefix (Optional)

After you have successfully copied your objects and updated your website configuration, you may want to delete the old prefix to avoid confusion and reduce storage costs.

1. **Delete Objects from the Old Prefix**:
   ```bash
   aws s3 rm s3://your-bucket/old-prefix/ --recursive
   ```

### Step 4: Set Up Redirects (Optional)

To ensure that visitors who try to access the old URLs are redirected to the new URLs, you can set up redirects.

#### Using Object Metadata for Redirects

1. **Set Redirect Metadata**:
   - For each object in the old prefix, set the `x-amz-website-redirect-location` metadata to point to the new location.

   ```bash
   aws s3 cp s3://your-bucket/old-prefix/ s3://your-bucket/old-prefix/ --recursive --metadata-directive REPLACE --website-redirect-location /new-prefix/
   ```

#### Using Bucket Policy for Redirects

You can also use a bucket policy to redirect all traffic from the old prefix to the new prefix.

1. **Create or Update the Bucket Policy**:
   - Go to the "Permissions" tab of your bucket.
   - Edit the "Bucket Policy" to include the following policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::your-bucket/old-prefix/*",
                 "Condition": {
                     "StringLike": {
                         "aws:Referer": "http://example.com/*"
                     }
                 }
             },
             {
                 "Effect": "Deny",
                 "Principal": "*",
                 "Action": "s3:GetObject",
                 "Resource": "arn:aws:s3:::your-bucket/old-prefix/*"
             }
         ]
     }
     ```

### Summary

Changing the prefix for a static website hosted on S3 involves copying the content to the new prefix, updating your website configuration to reflect the new URLs, and optionally setting up redirects from the old prefix to the new prefix to ensure seamless user experience. Using the AWS CLI can streamline the process of copying and deleting objects, and updating the bucket policy or object metadata can help manage redirects efficiently.

## How to enable accelarated transfer

### Amazon S3 Transfer Acceleration

Amazon S3 Transfer Acceleration enables fast, easy, and secure transfers of files over long distances between your client and an S3 bucket. By utilizing the globally distributed edge locations of Amazon CloudFront, S3 Transfer Acceleration improves transfer speeds and reduces latency for uploads and downloads.

### Enabling Transfer Acceleration

To enable Transfer Acceleration for an S3 bucket, you can use the AWS Management Console, AWS CLI, or AWS SDKs.

#### Using AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in with your credentials.

2. **Navigate to Amazon S3**:
   - In the AWS Management Console, type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket name for which you want to enable transfer acceleration.

4. **Go to Properties**:
   - Click on the "Properties" tab.

5. **Enable Transfer Acceleration**:
   - Scroll down to the "Transfer acceleration" section and click "Edit".
   - Select "Enabled".
   - Click "Save changes".

#### Using AWS CLI

1. **Enable Transfer Acceleration**:
   ```bash
   aws s3api put-bucket-accelerate-configuration --bucket your-bucket-name --accelerate-configuration Status=Enabled
   ```

2. **Verify Transfer Acceleration Status**:
   ```bash
   aws s3api get-bucket-accelerate-configuration --bucket your-bucket-name
   ```

### Use Case and Benefits

#### Use Case

S3 Transfer Acceleration is particularly useful for:
- Applications where users are located in geographically dispersed locations and need to upload or download large files quickly.
- Data transfer scenarios where low latency and high throughput are critical.
- Use cases like media uploads, backups, and large data migrations where transfer speed is essential.

#### Benefits

1. **Faster Uploads and Downloads**:
   - Transfer Acceleration leverages Amazon CloudFront’s globally distributed edge locations to accelerate data transfer by using optimized network paths.

2. **Improved Performance**:
   - It reduces the time taken for data to travel over long distances, significantly improving performance for transfers between clients and S3 buckets.

3. **Ease of Use**:
   - It requires minimal configuration changes to existing S3 buckets and uses the same security settings and policies as regular S3 data transfers.

4. **Cost-Effective**:
   - While there is an additional cost for using Transfer Acceleration, the improved speed and efficiency can lead to cost savings by reducing the operational time and improving user experience.

5. **Global Access**:
   - It is beneficial for users across the globe, ensuring that data transfer speeds are consistent and reliable regardless of the user's physical location.

### How It Works

When Transfer Acceleration is enabled, the data is routed to the nearest AWS edge location and then transferred over optimized network paths to the S3 bucket. The following diagram illustrates this process:

1. **Client Upload**: The client initiates an upload to the S3 bucket.
2. **Edge Location**: The upload request is routed to the nearest CloudFront edge location.
3. **Optimized Network Path**: The data is transferred over Amazon’s optimized network paths to the destination S3 bucket.
4. **S3 Bucket**: The data is stored in the specified S3 bucket.

### Example of Using Transfer Acceleration

#### Uploading an Object with Transfer Acceleration Using AWS CLI

1. **Specify the Accelerated Endpoint**:
   - Use the bucket name with the `s3-accelerate.amazonaws.com` endpoint.
   ```bash
   aws s3 cp my-file.txt s3://your-bucket-name/my-file.txt --endpoint-url https://s3-accelerate.amazonaws.com
   ```

### Summary

Amazon S3 Transfer Acceleration improves the speed and efficiency of data transfers to and from S3 buckets, particularly beneficial for applications with global users and large data transfer requirements. Enabling it can lead to better performance, reduced latency, and a more seamless experience for end-users, justifying the additional costs associated with its use.

## Same/Cross origin replication 

Amazon S3 Replication enables automatic, asynchronous copying of objects across S3 buckets in the same or different AWS regions. This feature is beneficial for data redundancy, compliance, disaster recovery, and minimizing latency.

### Types of Replication

1. **Same-Region Replication (SRR)**:
   - Replicates objects between buckets in the same AWS region.
   - Use cases: compliance, data protection, and keeping data closer to applications.

2. **Cross-Region Replication (CRR)**:
   - Replicates objects between buckets in different AWS regions.
   - Use cases: disaster recovery, low-latency data access in different geographical regions, and compliance.

### Setting Up S3 Replication

Replication requires a source bucket and a destination bucket. Here's how to configure replication using the AWS Management Console and AWS CLI.

### Using the AWS Management Console

#### Prerequisites
- Both the source and destination buckets must have versioning enabled.
- You must have the necessary permissions to set up replication rules.

#### Steps to Set Up Replication

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select the Source Bucket**:
   - Click on the bucket name that you want to replicate from.

4. **Enable Versioning** (if not already enabled):
   - Go to the "Properties" tab.
   - Scroll down to "Bucket Versioning" and click "Edit".
   - Enable versioning and save changes.

5. **Go to Management Tab**:
   - Click on the "Management" tab.

6. **Create a Replication Rule**:
   - Scroll down to the "Replication rules" section and click "Create replication rule".

7. **Configure the Replication Rule**:
   - **Rule Name**: Enter a name for your replication rule.
   - **Status**: Enable the rule.
   - **Source**: Define the source bucket and specify whether to replicate the entire bucket or specific prefixes or tags.

8. **Destination**:
   - **Destination Bucket**: Specify the ARN of the destination bucket.
   - **Destination Storage Class**: Choose the storage class for the replicated objects.

9. **IAM Role**:
   - **Create a New Role** or **Choose an Existing Role**: The role grants S3 permission to replicate objects on your behalf.
   
10. **Advanced Options (Optional)**:
    - **Replication Time Control (RTC)**: Enable RTC for predictable replication times.
    - **Delete Marker Replication**: Choose whether to replicate delete markers.

11. **Review and Create**:
    - Review your settings and click "Save".

### Using the AWS CLI

#### Prerequisites
- Both the source and destination buckets must have versioning enabled.
- You must have the necessary permissions to set up replication rules.

#### Steps to Set Up Replication

1. **Enable Versioning**:
   ```bash
   aws s3api put-bucket-versioning --bucket source-bucket-name --versioning-configuration Status=Enabled
   aws s3api put-bucket-versioning --bucket destination-bucket-name --versioning-configuration Status=Enabled
   ```

2. **Create an IAM Role for Replication**:
   - Create a JSON file (e.g., `replication-policy.json`) with the following policy:
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": {"Service": "s3.amazonaws.com"},
                 "Action": "sts:AssumeRole",
                 "Resource": "arn:aws:iam::account-id:role/replication-role"
             }
         ]
     }
     ```
   - Create the role:
     ```bash
     aws iam create-role --role-name replication-role --assume-role-policy-document file://replication-policy.json
     ```
   - Attach the necessary policies to the role.

3. **Set Up the Replication Configuration**:
   - Create a JSON file (e.g., `replication-config.json`) with the following configuration:
     ```json
     {
         "Role": "arn:aws:iam::account-id:role/replication-role",
         "Rules": [
             {
                 "ID": "ReplicationRule1",
                 "Status": "Enabled",
                 "Prefix": "",
                 "Destination": {
                     "Bucket": "arn:aws:s3:::destination-bucket-name",
                     "StorageClass": "STANDARD"
                 }
             }
         ]
     }
     ```
   - Apply the replication configuration:
     ```bash
     aws s3api put-bucket-replication --bucket source-bucket-name --replication-configuration file://replication-config.json
     ```

### Benefits of S3 Replication

1. **Data Redundancy**:
   - Ensures that your data is duplicated in another bucket or region, providing high availability and durability.

2. **Disaster Recovery**:
   - In case of a regional outage or disaster, you can access replicated data in another region.

3. **Compliance**:
   - Helps meet regulatory requirements by maintaining copies of data in different locations.

4. **Low-Latency Access**:
   - Users in different geographic locations can access data from the nearest region, reducing latency.

5. **Automated Management**:
   - Once set up, replication runs automatically without manual intervention.

### Summary

Amazon S3 Replication (SRR and CRR) allows you to automatically copy objects across S3 buckets in the same or different regions. This feature is useful for data redundancy, compliance, disaster recovery, and minimizing latency. You can configure replication using the AWS Management Console, AWS CLI, or AWS SDKs, and benefit from improved data availability and reduced access times for global users.

## Use of SRR/CRR

### Use Cases for Same-Region Replication (SRR) and Cross-Region Replication (CRR)

#### Same-Region Replication (SRR)

**1. Data Redundancy and Resilience**
- **Use Case**: Replicate data within the same region to create additional copies for higher availability and resilience.
- **Benefit**: Protects against data loss and enables quick recovery from accidental deletions or corruption.

**2. Compliance Requirements**
- **Use Case**: Satisfy internal or external compliance requirements for data redundancy and backup within the same geographic location.
- **Benefit**: Ensures data is stored in multiple locations within the same region to meet compliance and regulatory needs.

**3. Data Processing Optimization**
- **Use Case**: Create copies of data for processing by different applications or teams.
- **Benefit**: Enables multiple teams to work on the same data simultaneously without affecting the original dataset, improving productivity and data analysis speed.

**4. Performance Optimization**
- **Use Case**: Replicate frequently accessed data to other buckets within the same region for performance optimization.
- **Benefit**: Reduces latency for accessing data by distributing traffic across multiple buckets.

#### Cross-Region Replication (CRR)

**1. Disaster Recovery and Business Continuity**
- **Use Case**: Replicate data to a different geographic region to ensure availability in case of a regional outage or disaster.
- **Benefit**: Provides a robust disaster recovery solution, enabling quick data recovery and business continuity in the event of regional failures.

**2. Compliance and Data Sovereignty**
- **Use Case**: Meet compliance and regulatory requirements by storing data in multiple regions.
- **Benefit**: Ensures data residency and sovereignty by keeping copies of data in specific regions as required by laws and regulations.

**3. Global Content Distribution**
- **Use Case**: Distribute content closer to end users around the world by replicating data to regions closer to them.
- **Benefit**: Reduces latency and improves download/upload speeds for global users, enhancing the user experience.

**4. Backup and Archiving**
- **Use Case**: Create off-site backups for critical data by replicating it to a different region.
- **Benefit**: Provides an additional layer of protection by keeping backups in geographically separated locations, ensuring data durability and integrity.

**5. Data Analytics and Reporting**
- **Use Case**: Replicate data to regions where data analytics and reporting tools are located.
- **Benefit**: Enables localized data processing and analytics, reducing data transfer costs and latency, and improving overall efficiency.

### Key Considerations

**1. Cost**
- **SRR**: Generally lower costs compared to CRR since data is replicated within the same region.
- **CRR**: Higher costs due to data transfer charges between regions.

**2. Latency**
- **SRR**: Lower latency for data replication as data stays within the same region.
- **CRR**: Higher latency due to the distance between regions.

**3. Use Cases and Requirements**
- **SRR**: Suitable for use cases focused on data redundancy, resilience, and compliance within the same region.
- **CRR**: Ideal for use cases requiring disaster recovery, global distribution, compliance across regions, and business continuity.

### Implementing SRR and CRR

Both SRR and CRR can be implemented using the AWS Management Console, AWS CLI, or AWS SDKs. Key steps include enabling versioning on source and destination buckets, setting up replication rules, and configuring IAM roles for permissions.

#### Example for SRR using AWS CLI
```bash
aws s3api put-bucket-versioning --bucket source-bucket-name --versioning-configuration Status=Enabled
aws s3api put-bucket-versioning --bucket destination-bucket-name --versioning-configuration Status=Enabled

aws s3api put-bucket-replication --bucket source-bucket-name --replication-configuration file://replication-config.json
```

#### Example for CRR using AWS CLI
```bash
aws s3api put-bucket-versioning --bucket source-bucket-name --versioning-configuration Status=Enabled
aws s3api put-bucket-versioning --bucket destination-bucket-name --versioning-configuration Status=Enabled

aws s3api put-bucket-replication --bucket source-bucket-name --replication-configuration file://replication-config.json
```

### Summary

Same-Region Replication (SRR) and Cross-Region Replication (CRR) offer versatile solutions for different data replication needs. SRR focuses on redundancy and compliance within the same region, while CRR provides robust disaster recovery, compliance, and global content distribution. Implementing these replication strategies ensures data availability, resilience, and performance tailored to specific business requirements.

## Configure logging in S3 bucket

Configuring logging for an S3 bucket involves setting up Amazon S3 Server Access Logging. This feature provides detailed records for the requests made to your bucket. You can use these logs for security and access audits, compliance, and various other purposes.

### Steps to Configure Server Access Logging in S3

#### Using the AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select the Source Bucket**:
   - Click on the bucket for which you want to enable server access logging (the source bucket).

4. **Go to Properties**:
   - Click on the "Properties" tab.

5. **Select Server Access Logging**:
   - Scroll down to the "Server access logging" section and click "Edit".

6. **Enable Server Access Logging**:
   - Check the box to enable server access logging.

7. **Specify the Target Bucket**:
   - **Target Bucket**: Enter the name of the target bucket where you want the logs to be stored. This bucket must be in the same AWS account.
   - **Target Prefix** (optional): Specify a prefix for the log file names to organize logs better.

8. **Save Changes**:
   - Click "Save changes".

#### Using the AWS CLI

1. **Enable Versioning on the Target Bucket** (if not already enabled):
   ```bash
   aws s3api put-bucket-versioning --bucket target-bucket-name --versioning-configuration Status=Enabled
   ```

2. **Grant the Source Bucket Permissions to Write Logs**:
   - Create a bucket policy and save it to a JSON file (e.g., `bucket-policy.json`):
     ```json
     {
         "Version": "2012-10-17",
         "Statement": [
             {
                 "Effect": "Allow",
                 "Principal": {"AWS": "*"},
                 "Action": "s3:PutObject",
                 "Resource": "arn:aws:s3:::target-bucket-name/logs/*",
                 "Condition": {
                     "StringEquals": {
                         "s3:x-amz-acl": "bucket-owner-full-control"
                     }
                 }
             }
         ]
     }
     ```
   - Apply the policy to the target bucket:
     ```bash
     aws s3api put-bucket-policy --bucket target-bucket-name --policy file://bucket-policy.json
     ```

3. **Enable Logging on the Source Bucket**:
   - Create a logging configuration JSON file (e.g., `logging-config.json`):
     ```json
     {
         "LoggingEnabled": {
             "TargetBucket": "target-bucket-name",
             "TargetPrefix": "logs/"
         }
     }
     ```
   - Apply the logging configuration:
     ```bash
     aws s3api put-bucket-logging --bucket source-bucket-name --bucket-logging-status file://logging-config.json
     ```

### Verifying the Configuration

After setting up logging, it's essential to verify that logs are being delivered to the target bucket.

1. **Check for Log Files**:
   - Navigate to the target bucket in the AWS Management Console or use the AWS CLI to list objects in the target bucket:
     ```bash
     aws s3 ls s3://target-bucket-name/logs/
     ```

2. **Review Log Contents**:
   - Download a log file to inspect its contents:
     ```bash
     aws s3 cp s3://target-bucket-name/logs/some-log-file.log .
     cat some-log-file.log
     ```

### Best Practices for S3 Server Access Logging

1. **Separate Target Bucket**:
   - Use a separate bucket for logs to avoid cluttering your main data buckets.

2. **Enable Versioning on the Target Bucket**:
   - This provides protection against accidental deletions.

3. **Use Lifecycle Policies**:
   - Set up lifecycle policies to manage log file storage, such as transitioning older logs to cheaper storage classes or deleting them after a certain period.

4. **Monitor and Analyze Logs**:
   - Regularly monitor and analyze logs for security audits, compliance, and troubleshooting access issues.

### Summary

Configuring logging for an S3 bucket allows you to track access requests to your bucket, which is useful for security audits and compliance. You can set up server access logging through the AWS Management Console or the AWS CLI, ensuring that logs are stored in a specified target bucket. Following best practices, such as using a separate target bucket and enabling lifecycle policies, helps manage and protect your log data effectively.

## AWS S3 - Storage Classes

**Availability**: Whenever you want to retrieve your desired object it will be fetched faster

**Durability**: The file you stored should always be available for you.

Amazon S3 offers various storage classes to help optimize cost and performance based on access patterns and storage needs. Here are the different S3 storage classes and their use cases:

### 1. S3 Standard
- **Description**: The default storage class for frequently accessed data.
- **Use Cases**: Ideal for primary or frequently accessed data, such as active content, big data analytics, mobile and gaming applications.
- **Durability and Availability**: 
  - Durability: 99.999999999% (11 9's)
  - Availability: 99.99%
- **Features**: 
  - Low latency and high throughput performance.
  - Designed for durability of 99.999999999% (11 9's) of objects across multiple Availability Zones (AZs).

### 2. S3 Intelligent-Tiering
- **Description**: Automatically moves data between two access tiers (frequent and infrequent access) when access patterns change.
- **Use Cases**: Ideal for data with unknown or changing access patterns.
- **Durability and Availability**: 
  - Durability: 99.999999999% (11 9's)
  - Availability: 99.9% and 99% (frequent and infrequent tiers respectively)
- **Features**: 
  - Automatically moves objects between frequent and infrequent access tiers.
  - Monitoring and automation fees apply.
  - Suitable for long-lived data with changing or unknown access patterns.

### 3. S3 Standard-IA (Infrequent Access)
- **Description**: For data that is accessed less frequently but requires rapid access when needed.
- **Use Cases**: Suitable for backups, disaster recovery, and long-term data storage.
- **Durability and Availability**: 
  - Durability: 99.999999999% (11 9's)
  - Availability: 99.9%
- **Features**: 
  - Lower storage cost compared to S3 Standard.
  - Retrieval fees apply.

### 4. S3 One Zone-IA
- **Description**: For infrequently accessed data that does not require multiple Availability Zone (AZ) resilience.
- **Use Cases**: Suitable for secondary backups or data that can be recreated easily.
- **Durability and Availability**: 
  - Durability: 99.999999999% (11 9's)
  - Availability: 99.5%
- **Features**: 
  - Lower cost option for infrequently accessed data.
  - Data is stored in a single AZ.
  - Retrieval fees apply.

### 5. S3 Glacier
- **Description**: Low-cost storage class for data archiving where retrieval times of minutes to hours are acceptable.
- **Use Cases**: Suitable for long-term data archiving, digital preservation, and backup.
- **Durability and Availability**: 
  - Durability: 99.999999999% (11 9's)
- **Features**: 
  - Low storage cost.
  - Retrieval times range from minutes to hours.
  - Suitable for data that is rarely accessed.

### 6. S3 Glacier Deep Archive
- **Description**: Lowest-cost storage class for long-term data archiving where retrieval times of hours are acceptable.
- **Use Cases**: Suitable for data archiving, long-term backup, and digital preservation.
- **Durability and Availability**: 
  - Durability: 99.999999999% (11 9's)
- **Features**: 
  - Lowest storage cost.
  - Retrieval times up to 12 hours.
  - Designed for data that is accessed once or twice a year.

### 7. S3 Reduced Redundancy Storage (RRS) (deprecated)
- **Description**: Designed to provide 99.99% durability and 99.99% availability of objects over a given year.
- **Use Cases**: Suitable for non-critical, reproducible data.
- **Durability and Availability**: 
  - Durability: 99.99%
  - Availability: 99.99%
- **Features**: 
  - Lower durability compared to other storage classes.
  - Cost savings compared to S3 Standard.

### Choosing the Right Storage Class

#### Factors to Consider:
1. **Access Frequency**: Determine how often the data will be accessed. For frequently accessed data, use S3 Standard or S3 Intelligent-Tiering.
2. **Durability and Availability**: Evaluate the durability and availability requirements for your data.
3. **Cost**: Consider the cost of storage, retrieval, and data transfer.
4. **Latency**: Decide on acceptable retrieval times for your data.
5. **Redundancy**: Determine if your data needs to be stored across multiple Availability Zones.

### Summary

Amazon S3 storage classes offer flexibility to manage costs and performance based on your data’s access patterns and storage needs. By selecting the appropriate storage class, you can optimize storage costs while meeting your application requirements for durability, availability, and latency.

For more information: https://aws.amazon.com/s3/storage-classes/

## How to configure storage class

Configuring the storage class for your objects in Amazon S3 can be done through the AWS Management Console, AWS CLI, or AWS SDKs. Here’s how you can set the storage class for your objects in different scenarios:

### Using the AWS Management Console

#### When Uploading a New Object

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket where you want to upload the object.

4. **Upload File**:
   - Click the "Upload" button.
   - Drag and drop or select the files you want to upload.
   - Click "Next".

5. **Set Properties**:
   - In the "Set permissions" step, click "Next".
   - In the "Set properties" step, scroll down to "Storage class".
   - Select the desired storage class (e.g., Standard, Intelligent-Tiering, Standard-IA, One Zone-IA, Glacier, Glacier Deep Archive).

6. **Complete the Upload**:
   - Click "Next" and review your settings.
   - Click "Upload" to complete the process.

#### For Existing Objects

1. **Select Your Bucket**:
   - Click on the bucket where your objects are stored.

2. **Select Objects**:
   - Select the objects you want to change the storage class for.

3. **Change Storage Class**:
   - Click on the "Actions" button.
   - Select "Change storage class".
   - Choose the desired storage class.
   - Click "Save".

### Using the AWS CLI

#### Uploading a New Object with a Specific Storage Class

```bash
aws s3 cp my-file.txt s3://your-bucket-name/my-file.txt --storage-class STANDARD_IA
```

Replace `STANDARD_IA` with the desired storage class (`STANDARD`, `INTELLIGENT_TIERING`, `ONEZONE_IA`, `GLACIER`, `DEEP_ARCHIVE`).

#### Changing the Storage Class of an Existing Object

```bash
aws s3 cp s3://your-bucket-name/old-file.txt s3://your-bucket-name/old-file.txt --storage-class STANDARD_IA --metadata-directive REPLACE
```

### Using the AWS SDK

Here's an example using the AWS SDK for Python (Boto3) to upload a file with a specific storage class:

#### Python (Boto3)

```python
import boto3

s3 = boto3.client('s3')

s3.upload_file(
    'my-file.txt', 
    'your-bucket-name', 
    'my-file.txt', 
    ExtraArgs={'StorageClass': 'STANDARD_IA'}
)
```

#### Changing the Storage Class of an Existing Object

```python
import boto3

s3 = boto3.client('s3')

copy_source = {
    'Bucket': 'your-bucket-name',
    'Key': 'old-file.txt'
}

s3.copy_object(
    CopySource=copy_source, 
    Bucket='your-bucket-name', 
    Key='old-file.txt', 
    StorageClass='STANDARD_IA', 
    MetadataDirective='REPLACE'
)
```

### Use Lifecycle Policies for Automatic Storage Class Transitions

For managing large numbers of objects, you can use lifecycle policies to automatically transition objects to different storage classes based on their age or access patterns.

#### Creating a Lifecycle Policy

1. **Navigate to Your Bucket**:
   - Open the bucket for which you want to create a lifecycle policy.

2. **Open the Management Tab**:
   - Click on the "Management" tab.

3. **Add Lifecycle Rule**:
   - Click on "Create lifecycle rule".
   - Enter a rule name and scope.

4. **Configure Transitions**:
   - Add transitions to specify when objects should move to different storage classes.
   - For example, move objects to Standard-IA after 30 days, Glacier after 90 days.

5. **Add Expiration (Optional)**:
   - Configure when objects should be deleted.

6. **Review and Create**:
   - Review your settings and click "Create rule".

### Summary

Configuring the storage class for your Amazon S3 objects can be done through the AWS Management Console, AWS CLI, or AWS SDKs. By selecting the appropriate storage class, you can optimize costs based on access patterns and storage needs. For large-scale management, using lifecycle policies can automate transitions between storage classes, further optimizing storage costs and performance.

## Simplify data lifecycle management with S3

Amazon S3 offers a feature called Lifecycle Policies that help you manage your objects throughout their lifecycle, simplifying data management and optimizing costs. These policies enable you to automatically transition objects to different storage classes, archive them, or delete them after a specified period.

### Key Concepts

- **Lifecycle Rule**: A set of actions that defines how Amazon S3 manages your objects during their lifetime.
- **Transition Actions**: Automatically move objects to different storage classes.
- **Expiration Actions**: Define when objects expire and should be permanently deleted.
- **Noncurrent Version Actions**: Manage the lifecycle of previous versions of objects.

### Lifecycle Policies Use Cases

1. **Cost Optimization**: Transition objects to less expensive storage classes as they become less frequently accessed.
2. **Compliance and Retention**: Ensure objects are retained for a specific period to meet compliance requirements and automatically delete them afterward.
3. **Data Archiving**: Automatically archive older data that is not frequently accessed but needs to be retained for long-term storage.

### How to Set Up Lifecycle Policies

#### Using the AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket for which you want to create a lifecycle policy.

4. **Go to the Management Tab**:
   - Click on the "Management" tab.

5. **Add Lifecycle Rule**:
   - Scroll down to the "Lifecycle rules" section and click "Create lifecycle rule".

6. **Define Rule Scope**:
   - **Rule Name**: Enter a name for your rule.
   - **Scope**: Choose whether the rule applies to the entire bucket, a prefix, or specific object tags.

7. **Set Transition Actions**:
   - Add a transition rule to move objects to a different storage class after a specified number of days.
   - For example, move objects to Standard-IA after 30 days and to Glacier after 365 days.

8. **Set Expiration Actions**:
   - Add an expiration rule to permanently delete objects after a specified number of days.
   - For example, delete objects 365 days after creation.

9. **Set Noncurrent Version Actions (if versioning is enabled)**:
   - Add actions to transition or expire noncurrent versions of objects.

10. **Review and Create**:
    - Review your settings and click "Create rule".

#### Using the AWS CLI

1. **Create a Lifecycle Policy JSON File**:
   - Create a JSON file (e.g., `lifecycle-policy.json`) with the following structure:
     ```json
     {
         "Rules": [
             {
                 "ID": "ExampleRule",
                 "Status": "Enabled",
                 "Filter": {
                     "Prefix": "documents/"
                 },
                 "Transitions": [
                     {
                         "Days": 30,
                         "StorageClass": "STANDARD_IA"
                     },
                     {
                         "Days": 365,
                         "StorageClass": "GLACIER"
                     }
                 ],
                 "Expiration": {
                     "Days": 3650
                 }
             }
         ]
     }
     ```

2. **Apply the Lifecycle Policy to a Bucket**:
   ```bash
   aws s3api put-bucket-lifecycle-configuration --bucket your-bucket-name --lifecycle-configuration file://lifecycle-policy.json
   ```

### Example Lifecycle Policy Scenarios

#### Scenario 1: Transition to Cheaper Storage Classes

- **30 Days**: Move to Standard-IA.
- **90 Days**: Move to Glacier.
- **365 Days**: Move to Glacier Deep Archive.

```json
{
    "Rules": [
        {
            "ID": "CostOptimization",
            "Status": "Enabled",
            "Filter": {},
            "Transitions": [
                {
                    "Days": 30,
                    "StorageClass": "STANDARD_IA"
                },
                {
                    "Days": 90,
                    "StorageClass": "GLACIER"
                },
                {
                    "Days": 365,
                    "StorageClass": "DEEP_ARCHIVE"
                }
            ]
        }
    ]
}
```

#### Scenario 2: Expire Old Objects

- **365 Days**: Delete objects.

```json
{
    "Rules": [
        {
            "ID": "ExpireOldObjects",
            "Status": "Enabled",
            "Filter": {},
            "Expiration": {
                "Days": 365
            }
        }
    ]
}
```

### Best Practices

1. **Plan Storage Classes**:
   - Understand the access patterns and choose appropriate storage classes to optimize costs.

2. **Combine Rules**:
   - Combine transition and expiration rules to manage the full lifecycle of objects efficiently.

3. **Test and Monitor**:
   - Test lifecycle policies on a smaller set of objects to ensure they work as expected before applying them broadly.
   - Monitor the lifecycle actions and costs to ensure they align with your goals.

### Summary

Amazon S3 Lifecycle Policies simplify data lifecycle management by automating the transition of objects to different storage classes and their expiration. By using the AWS Management Console or AWS CLI, you can create and manage lifecycle rules to optimize storage costs, ensure compliance, and manage data retention effectively.

## Cost efficiency and performance with AWS intelligent tiering for S3

Amazon S3 Intelligent-Tiering is a storage class designed to optimize storage costs automatically when data access patterns change. This storage class moves objects between two access tiers—frequent access and infrequent access—based on changing access patterns, without performance impact or operational overhead.

### Key Features of S3 Intelligent-Tiering

1. **Automatic Cost Optimization**: Moves objects between frequent and infrequent access tiers based on their access patterns.
2. **No Retrieval Fees**: No additional charges for data retrieval, making it ideal for unpredictable or unknown access patterns.
3. **Monitoring and Automation**: Uses automation and monitoring to move objects without manual intervention.
4. **Performance**: Delivers low latency and high throughput performance, similar to S3 Standard.

### Cost Efficiency

1. **Automatic Tiering**: 
   - Objects that haven’t been accessed for 30 consecutive days are automatically moved to the infrequent access tier, which has lower storage costs.
   - If an object in the infrequent access tier is accessed, it is automatically moved back to the frequent access tier.

2. **No Retrieval Fees**:
   - There are no retrieval fees when accessing data, making it cost-effective for workloads with unpredictable access patterns.

3. **No Minimum Storage Duration**:
   - There is no minimum storage duration for objects stored in the S3 Intelligent-Tiering storage class. This means you only pay for the storage you use without having to worry about minimum storage duration fees.

4. **Low Monitoring and Automation Charges**:
   - A small monthly monitoring and automation charge is applied per object, but this is typically outweighed by the savings from automatic tiering.

### Performance

1. **Low Latency and High Throughput**:
   - Provides similar performance to the S3 Standard storage class, ensuring fast access to data when needed.
   
2. **Suitable for a Wide Range of Use Cases**:
   - Ideal for data lakes, user-generated content, analytics, and any other workload where data access patterns are unpredictable or unknown.

### Using S3 Intelligent-Tiering

#### Configuring S3 Intelligent-Tiering in the AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket where you want to enable S3 Intelligent-Tiering.

4. **Upload or Transition Objects**:
   - When uploading new objects, you can select S3 Intelligent-Tiering as the storage class.
   - For existing objects, you can create a lifecycle policy to transition them to S3 Intelligent-Tiering.

5. **Creating a Lifecycle Policy**:
   - Go to the "Management" tab and select "Lifecycle rules".
   - Click "Create lifecycle rule" and define the scope (entire bucket, prefix, or tags).
   - Add a "Transition action" to move objects to S3 Intelligent-Tiering after a specified number of days.
   - Save the lifecycle rule.

#### Using the AWS CLI

1. **Upload New Objects with Intelligent-Tiering**:
   ```bash
   aws s3 cp myfile.txt s3://my-bucket/myfile.txt --storage-class INTELLIGENT_TIERING
   ```

2. **Transition Existing Objects with a Lifecycle Policy**:
   - Create a JSON file (e.g., `lifecycle-policy.json`):
     ```json
     {
         "Rules": [
             {
                 "ID": "TransitionToIntelligentTiering",
                 "Status": "Enabled",
                 "Filter": {},
                 "Transitions": [
                     {
                         "Days": 30,
                         "StorageClass": "INTELLIGENT_TIERING"
                     }
                 ]
             }
         ]
     }
     ```
   - Apply the lifecycle policy:
     ```bash
     aws s3api put-bucket-lifecycle-configuration --bucket my-bucket --lifecycle-configuration file://lifecycle-policy.json
     ```

### Benefits of Using S3 Intelligent-Tiering

1. **Cost Savings**:
   - Automatically saves money by moving data to lower-cost storage tiers when not frequently accessed.
   
2. **Operational Simplicity**:
   - Reduces the need for manual monitoring and management of storage classes, allowing you to focus on other tasks.

3. **Predictable Performance**:
   - Ensures consistent, low-latency access to data without the need for retrieval fees, providing predictable performance.

4. **Flexibility**:
   - Ideal for datasets with unknown or changing access patterns, as it adjusts to the workload's needs dynamically.

### Summary

Amazon S3 Intelligent-Tiering is a powerful storage class that combines cost efficiency with performance. By automatically moving data between frequent and infrequent access tiers based on access patterns, it optimizes costs while maintaining low latency and high throughput. This storage class is particularly beneficial for workloads with unpredictable access patterns, providing a seamless and cost-effective storage solution without additional operational overhead.

## What is CORS and how to enable it in S3

### Cross-Origin Resource Sharing (CORS)

**Cross-Origin Resource Sharing (CORS)** is a mechanism that allows web pages to make requests to a different domain than the one that served the web page. This is useful for web applications that need to interact with resources hosted on different origins (domains). Without CORS, web pages are restricted by the same-origin policy, which prevents requests to different domains for security reasons.

### CORS Configuration for Amazon S3

Amazon S3 supports CORS, allowing you to configure your S3 buckets to accept cross-origin requests. To enable CORS for your S3 bucket, you need to add a CORS configuration to the bucket.

### Steps to Enable CORS on an S3 Bucket

#### Using the AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket**:
   - Click on the bucket for which you want to enable CORS.

4. **Go to Permissions**:
   - Click on the "Permissions" tab.

5. **Edit CORS Configuration**:
   - Scroll down to the "Cross-origin resource sharing (CORS)" section and click "Edit".

6. **Add CORS Rules**:
   - Enter your CORS configuration in JSON format. Below is an example configuration:

     ```json
     [
         {
             "AllowedHeaders": [
                 "*"
             ],
             "AllowedMethods": [
                 "GET",
                 "PUT",
                 "POST",
                 "DELETE"
             ],
             "AllowedOrigins": [
                 "*"
             ],
             "ExposeHeaders": [
                 "x-amz-server-side-encryption",
                 "x-amz-request-id",
                 "x-amz-id-2"
             ],
             "MaxAgeSeconds": 3000
         }
     ]
     ```

   - This example allows all origins to send GET, PUT, POST, and DELETE requests to the bucket, exposes specific headers, and caches the preflight response for 3000 seconds.

7. **Save Changes**:
   - Click "Save changes" to apply the CORS configuration to the bucket.

#### Using the AWS CLI

1. **Create a CORS Configuration JSON File**:
   - Create a file named `cors-config.json` with the following content:

     ```json
     [
         {
             "AllowedHeaders": [
                 "*"
             ],
             "AllowedMethods": [
                 "GET",
                 "PUT",
                 "POST",
                 "DELETE"
             ],
             "AllowedOrigins": [
                 "*"
             ],
             "ExposeHeaders": [
                 "x-amz-server-side-encryption",
                 "x-amz-request-id",
                 "x-amz-id-2"
             ],
             "MaxAgeSeconds": 3000
         }
     ]
     ```

2. **Apply the CORS Configuration to Your Bucket**:
   - Use the AWS CLI to apply the CORS configuration to your bucket:

     ```bash
     aws s3api put-bucket-cors --bucket your-bucket-name --cors-configuration file://cors-config.json
     ```

### Example CORS Configuration Scenarios

#### Scenario 1: Allow Specific Origin

- Allow only requests from `https://example.com` to access your bucket.

```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET"
        ],
        "AllowedOrigins": [
            "https://example.com"
        ],
        "MaxAgeSeconds": 3000
    }
]
```

#### Scenario 2: Allow Multiple Origins and Methods

- Allow requests from multiple origins and various HTTP methods.

```json
[
    {
        "AllowedHeaders": [
            "*"
        ],
        "AllowedMethods": [
            "GET",
            "POST"
        ],
        "AllowedOrigins": [
            "https://example1.com",
            "https://example2.com"
        ],
        "MaxAgeSeconds": 3000
    }
]
```

### Testing Your CORS Configuration

1. **Create an HTML File**:
   - Create a simple HTML file to test your CORS settings:

     ```html
     <!DOCTYPE html>
     <html>
     <head>
         <title>CORS Test</title>
         <script>
             function testCORS() {
                 fetch('https://your-bucket-name.s3.amazonaws.com/your-object-key')
                     .then(response => response.text())
                     .then(data => console.log(data))
                     .catch(error => console.error('Error:', error));
             }
         </script>
     </head>
     <body>
         <button onclick="testCORS()">Test CORS</button>
     </body>
     </html>
     ```

2. **Open the HTML File**:
   - Open the HTML file in a browser and click the "Test CORS" button to verify that your CORS configuration is working correctly.

### Summary

Enabling CORS on an S3 bucket allows your web applications to interact with resources stored in S3 from different origins. By configuring CORS rules, you can specify which HTTP methods, origins, and headers are allowed, ensuring secure and controlled access to your S3 resources. This setup is essential for modern web applications that need to leverage cloud storage while adhering to the same-origin policy.

## Presigned URL in S3

A presigned URL in Amazon S3 allows you to grant temporary access to objects in your S3 bucket without needing to provide permanent access permissions. These URLs are useful for allowing users to download or upload objects securely without requiring AWS credentials.

### Key Features of Presigned URLs

1. **Temporary Access**: The URL is valid only for a specified period.
2. **Secure**: Access is granted through a signature included in the URL, ensuring only the intended user can access the object.
3. **Controlled Access**: You can specify the HTTP method (GET, PUT, etc.) and the expiration time for the URL.

### Use Cases

- **Temporary File Sharing**: Share files with users without changing the bucket policy or object permissions.
- **Secure File Uploads**: Allow users to upload files directly to your S3 bucket without exposing your AWS credentials.
- **Time-Limited Access**: Provide temporary access to an object, which automatically expires after a specified time.

### Creating a Presigned URL

#### Using the AWS Management Console

1. **Sign in to the AWS Management Console**:
   - Go to the [AWS Management Console](https://aws.amazon.com/console/) and sign in.

2. **Navigate to Amazon S3**:
   - Type "S3" in the search bar and select "S3" to open the Amazon S3 console.

3. **Select Your Bucket and Object**:
   - Click on the bucket containing the object for which you want to generate a presigned URL.
   - Select the object.

4. **Generate Presigned URL**:
   - Click on the "Actions" button and select "Share with presigned URL".
   - Specify the expiration time for the URL (e.g., 1 hour, 1 day).
   - Click "Create URL" and copy the generated URL.

#### Using the AWS CLI

1. **Generate a Presigned URL for Download**:
   - Use the following command to generate a presigned URL for downloading an object:

     ```bash
     aws s3 presign s3://your-bucket-name/your-object-key --expires-in 3600
     ```

   - The `--expires-in` parameter specifies the expiration time in seconds (e.g., 3600 seconds for 1 hour).

2. **Generate a Presigned URL for Upload**:
   - To generate a presigned URL for uploading an object, you need to use a script or a programming language with AWS SDK support.

#### Using AWS SDK for Python (Boto3)

1. **Install Boto3**:
   - Ensure Boto3 is installed in your Python environment:

     ```bash
     pip install boto3
     ```

2. **Generate a Presigned URL for Download**:
   - Use the following Python script to generate a presigned URL for downloading an object:

     ```python
     import boto3
     from botocore.exceptions import NoCredentialsError

     s3_client = boto3.client('s3')

     try:
         response = s3_client.generate_presigned_url('get_object',
                                                     Params={'Bucket': 'your-bucket-name',
                                                             'Key': 'your-object-key'},
                                                     ExpiresIn=3600)
         print("Presigned URL:", response)
     except NoCredentialsError:
         print("Credentials not available")
     ```

3. **Generate a Presigned URL for Upload**:
   - Use the following Python script to generate a presigned URL for uploading an object:

     ```python
     import boto3
     from botocore.exceptions import NoCredentialsError

     s3_client = boto3.client('s3')

     try:
         response = s3_client.generate_presigned_url('put_object',
                                                     Params={'Bucket': 'your-bucket-name',
                                                             'Key': 'your-object-key'},
                                                     ExpiresIn=3600)
         print("Presigned URL for Upload:", response)
     except NoCredentialsError:
         print("Credentials not available")
     ```

### Summary

Presigned URLs in Amazon S3 provide a secure and temporary way to grant access to objects in your S3 buckets. They are useful for sharing files, allowing secure uploads, and controlling access without changing bucket policies or object permissions. You can generate presigned URLs using the AWS Management Console, AWS CLI, or AWS SDKs, specifying the HTTP method and expiration time to tailor the access to your needs.

## Securing your AWS S3 Data: at REST, server-side and client side encryption

Securing data in Amazon S3 (Simple Storage Service) involves protecting data at rest and in transit. AWS provides several encryption options to ensure your data remains secure, including server-side encryption (SSE) and client-side encryption (CSE). Here's an overview of these options and how to implement them:

### Data at Rest Encryption

#### 1. Server-Side Encryption (SSE)

**Server-side encryption** protects data at rest by encrypting it as it is written to storage and decrypting it as it is accessed.

##### Types of Server-Side Encryption

1. **SSE-S3 (Server-Side Encryption with S3 Managed Keys)**:
   - AWS manages the encryption keys.
   - Encryption is handled using 256-bit Advanced Encryption Standard (AES-256).
   - Simple to use, no additional setup required.

   **How to Enable SSE-S3**:
   - **AWS Management Console**:
     - When uploading an object, select "Encrypt this object" and choose "Amazon S3 key (SSE-S3)".
   - **AWS CLI**:
     ```bash
     aws s3 cp yourfile.txt s3://your-bucket-name/yourfile.txt --sse
     ```

2. **SSE-KMS (Server-Side Encryption with AWS Key Management Service)**:
   - Uses AWS Key Management Service (KMS) to manage encryption keys.
   - Provides more control over key management and access.

   **How to Enable SSE-KMS**:
   - **AWS Management Console**:
     - When uploading an object, select "Encrypt this object" and choose "AWS KMS key (SSE-KMS)".
     - Specify the KMS key you want to use.
   - **AWS CLI**:
     ```bash
     aws s3 cp yourfile.txt s3://your-bucket-name/yourfile.txt --sse aws:kms --sse-kms-key-id your-kms-key-id
     ```

3. **SSE-C (Server-Side Encryption with Customer-Provided Keys)**:
   - You provide your own encryption keys.
   - AWS does not store the encryption keys.

   **How to Enable SSE-C**:
   - **AWS CLI**:
     ```bash
     aws s3 cp yourfile.txt s3://your-bucket-name/yourfile.txt --sse-c --sse-c-key "your-base64-encoded-key"
     ```

#### 2. Client-Side Encryption (CSE)

**Client-side encryption** involves encrypting data before uploading it to S3 and decrypting it after downloading.

1. **Using AWS SDK for JavaScript (Example)**:
   - You can use AWS Encryption SDK or other libraries to handle encryption and decryption.
   - The AWS SDK provides methods to encrypt data client-side before uploading to S3.

   **Example Code (JavaScript)**:
   ```javascript
   const AWS = require('aws-sdk');
   const s3 = new AWS.S3();
   const crypto = require('crypto');
   const fs = require('fs');

   const key = crypto.randomBytes(32); // Your encryption key

   // Encrypt the data before uploading
   function encryptData(data) {
       const cipher = crypto.createCipheriv('aes-256-cbc', key, crypto.randomBytes(16));
       let encrypted = cipher.update(data, 'utf8', 'hex');
       encrypted += cipher.final('hex');
       return encrypted;
   }

   const filePath = 'yourfile.txt';
   const fileData = fs.readFileSync(filePath, 'utf8');
   const encryptedData = encryptData(fileData);

   // Upload encrypted data to S3
   const params = {
       Bucket: 'your-bucket-name',
       Key: 'yourfile.txt',
       Body: encryptedData
   };

   s3.putObject(params, (err, data) => {
       if (err) {
           console.log('Error uploading data: ', err);
       } else {
           console.log('Successfully uploaded encrypted data to S3');
       }
   });
   ```

### Data in Transit Encryption

- **HTTPS/TLS**: Ensure that data in transit between your application and S3 is encrypted using HTTPS/TLS. By default, AWS supports HTTPS for secure data transfer.

### Summary

To secure your AWS S3 data, you can use a combination of server-side and client-side encryption techniques:

- **Server-Side Encryption (SSE)**:
  - SSE-S3: S3 Managed Keys.
  - SSE-KMS: AWS KMS Managed Keys.
  - SSE-C: Customer-Provided Keys.
  
- **Client-Side Encryption (CSE)**:
  - Encrypt data before uploading it to S3 and decrypt it after downloading.

- **Data in Transit Encryption**:
  - Use HTTPS to ensure secure data transfer.

Implementing these encryption methods ensures that your data remains secure both at rest and in transit, providing robust protection against unauthorized access.
