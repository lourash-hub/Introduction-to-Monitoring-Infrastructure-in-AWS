### Introduction to Monitoring Infrastructure in AWS
Monitoring Infrastructure in AWS (Cloudwatch, Cloud trail)
Imagine you've just launched your startup's new application on Amazon Web Services (AWS). Excitedly, you see traffic start to flow in as users begin to interact with your service. However, with success comes responsibility. As yourapplication grows, so does the complexity of managing its underlying infrastructure. How do you ensure that everythingcontinues to run smoothly? How do you detect and address issues before they impact your users? This is where monitoringyour infrastructure becomes crucial.
Monitoring, refers to the process of observing and collecting data about the performance, health, and behavior of systems,applications, networks, or infrastructure components. The primary goal of monitoring is to ensure that these systemsoperate effectively, effi ciently, and securely, while also detecting and addressing any issues or anomalies in a timelymanner.


### **AWS CloudWatch and CloudTrail**

AWS CloudWatch is a monitoring and observability service provided by Amazon Web Services (AWS). It allows users tocollect and track metrics, monitor logs, set alarms, and automatically react to changes in AWS resources and applicationsrunning on the AWS infrastructure. CloudWatch provides insights into the performance, health, and operational status ofAWS resources and applications, helping users to troubleshoot issues, optimize resource utilization, and ensure thereliability of their systems.
AWS CloudTrail on the other hand is also a service provided by Amazon Web Services (AWS) that enables governance,compliance, operational auditing, and risk auditing of your AWS account. CloudTrail records and logs all API activity inyour AWS account, providing a comprehensive trail of events that can be used for security analysis, resource changetracking, troubleshooting, and compliance auditing.


### **CloudWatch Metrics and Alarms**
Amazon CloudWatch Metrics and Alarms are essential components of the Amazon CloudWatch service, which providesmonitoring and observability capabilities for AWS resources and applications. Let's delve into each of these components:

**CloudWatch Metrics**
CloudWatch Metrics are data points representing the behavior of AWS resources and applications over time. Thesemetrics can be collected from various AWS services such as Amazon EC2, Amazon RDS, Amazon S3, AWS Lambda, andmany others. Metrics provide insights into the performance, health, and operational status of these resources, allowingusers to monitor and analyze their behavior.

![Image](./Images/Cloudwatch_001.png)


Key aspects of CloudWatch Metrics include:

Default and Custom Metrics: AWS services automatically publish default metrics to CloudWatch, such as CPU utilization,network traffic, and disk I/O for EC2 instances. Additionally, users can create custom metrics to monitor specific aspects of their applications or services.

Namespace and Dimensions: Metrics are organized into namespaces, which categorize related metrics together. Within each namespace, metrics can have dimensions that further specify the resource or aspect being monitored. For example,an EC2 instance metric might have dimensions such as InstanceId or InstanceType.


Timestamps and Units: Each metric data point includes a timestamp indicating when the measurement was taken, as wellas a unit specifying the measurement's scale (e.g., bytes, percentage, seconds).

![Image](./Images/Cloudwatch_002.png)

Retention and Granularity: CloudWatch retains metric data for different periods depending on the data's age andgranularity. Users can specify the granularity of their metric data, ranging from one-minute to one-day intervals.

![Image](./Images/Cloudwatch_003.png)

CloudWatch Alarms:

CloudWatch Alarms allow users to defi ne thresholds on CloudWatch Metrics and trigger actions when these thresholds are breached. Alarms are used to proactively monitor the health and performance of AWS resources and applications,enabling users to respond promptly to changes in their environment.

Key aspects of CloudWatch Alarms include:
Thresholds and Actions: Users can set thresholds on CloudWatch Metrics, specifying conditions that, when met or exceeded, trigger alarm states. When an alarm enters an alarm state, users can confi gure actions such as sending notifications via Amazon SNS, executing AWS Lambda functions, or auto-scaling resources.
Alarm States: CloudWatch Alarms have three possible states: OK, INSUFFICIENT_DATA, and ALARM. The OK stateindicates that the metric is within the defi ned threshold, while the ALARM state indicates that the threshold has beenbreached. The INSUFFICIENT_DATA state occurs when there is not enough data to evaluate the alarm.
Alarm History: CloudWatch maintains a history of alarm state changes, allowing users to track when alarms transitionbetween states and investigate the circumstances surrounding each state change.
Confi guration and Management: Users can create, modify, and delete alarms through the CloudWatch ManagementConsole, AWS CLI, or SDKs. Alarms can be managed individually or as part of larger monitoring confi gurations, such asCloudFormation templates or AWS Auto Scaling policies.

Monitoring AWS EC2 using CloudWatch
Now that we an idea of what AWS CloudWatch and CloudTrail is all about, let launch an EC2 instance and monitor it
Step 1: Create an IAM Role With CloudWatchFull Access and SSMFullAccess
1.  Navigate to the IAM console.
2.  In the IAM Console navigation click on roles.
3.  Follow the image to create a role with `CloudWatchFullAccess` and `SSMFullAccesspolicy`

![Image](./Images/Cloudwatch_004.png)

Create A Parameter In System Manager
Now that we have created an IAM role, we need to create a paramater in the system manager console. By doing this, wewill be able to defi ne the metrics we want to monitor for our EC2 instance
1.  Navigate to the AWS System Manager Console
2.  In the AWS System Manager navigate menu, select parameter store

![Image](./Images/Cloudwatch_005.png)

3.Create a new parameter and paste the code snippet below

```bash
{
	"metrics": {
		"append_dimensions": {
			"InstanceId": "${aws:InstanceId}"
		},
		"metrics_collected": {
			"mem": {
				"measurement": [
					"mem_used_percent"
				],
				"metrics_collection_interval": 180
			},
            "disk": {
				"measurement": [
                     "disk_used_percent"
				],
				"metrics_collection_interval": 180
			}
		}
	}
}

```

![Image](./Images/Cloudwatch_006.png)


The parameters above are a configuration file for the CloudWatch agent, which defines the metrics that will be collected from your EC2 instance and sent to CloudWatch.

1.  "metrics": This is the top-level key in the configuration file, indicating that it contains the definitions for the metrics to be collected.

2.  "append_dimensions": This section specifi es dimensions to be appended to all collected metrics. Dimensions are key-value pairs that help identify the source of the data in CloudWatch. In this case, the dimension "InstanceId" isappended, and its value is populated dynamically with the instance ID of the EC2 instance where the CloudWatchagent is installed.

"InstanceId": "${aws:InstanceId}": This line specifies that the value of the "InstanceId" dimension should be dynamically populated with the instance ID of the EC2 instance.


3."metrics_collected": This section defi nes the specifi c metrics to be collected from the EC2 instance.
"mem": This subsection specifi es memory-related metrics to be collected.
"measurement": This is an array of specifi c memory metrics to collect. In this case, only
"mem_used_percent"is specifi ed, which represents the percentage of memory used on the instance.
"metrics_collection_interval": This parameter specifi es how frequently (in seconds) the metrics should be collected. Here, memory metrics will be collected every 60 seconds.
"disk": This subsection specifies disk-related metrics to be collected.

*   "measurement": This is an array of specific disk metrics to collect. Only "disk_used_percent" is specified, representing the percentage of disk space used on the instance.
*   "metrics_collection_interval": Similar to the memory section, this parameter specifies how frequently disk metrics will be collected, which is every 60 seconds.

### Create an EC2 Instance, Attach the role created in Step 1
Now that we have created an IAM Role and also created a parameter in the Account System Manager Console, let's create an EC2 instance and that roles we created earlier. But note that SSM
will have access to the parameter we created and by attaching the role to the EC2 instance, EC2 will also have access to the parameters
1.Navigate to the EC2 console, select instances. Click on launch instance on the top right
2.Now we will need to launch an Amazon linux 2 instance and attach the role we created in step 1. Follow the images below to attach IAM role to your instance


![Image](./Images/Cloudwatch_007.png)
![Image](./Images/Cloudwatch_008.png)

Install CloudWatch agent. Create a file name script.sh and past the shell script below.
Create a file

```bash
sudo nano script.sh
```

paste the shell script below

```bash
#!/bin/bash
#!/bin/bash

# Download the CloudWatch Agent
wget https://s3.amazonaws.com/amazoncloudwatch-agent/linux/amd64/latest/AmazonCloudWatchAgent.zip

# Unzip the downloaded file
unzip AmazonCloudWatchAgent.zip

# Install the CloudWatch Agent
sudo ./install.sh

# Fetch the CloudWatch Agent configuration from SSM Parameter Store
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -a fetch-config -m ec2 -c ssm:EC2-parameter -s

```

Make the file executable

```bash
sudo chmod +x script.sh
```

Save and run the file

```bash
./script.sh
```

![Image](./Images/Screenshot%202024-05-31%20121019.png)

Start the CloudWatch agent

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a start
```

Verify if CloudWatch is installed and successfully running

```bash
sudo /opt/aws/amazon-cloudwatch-agent/bin/amazon-cloudwatch-agent-ctl -m ec2 -a status
```
![Image](./Images/Screenshot%202024-05-31%20121100.png)


Step 4: Monitor Your Metric In CloudWatch
Before we can monitor our EC2 instance metrics, create a new policy and attach it to our IAM role so that the role does notlack permissions to perform the ec2:DescribeTags action, which is necessary for the CloudWatch agent to retrieve EC2instance tags.

1.  Create a new Policy
*   In the IAM console navigation menu, click on policy and on the top right, select create policy. Follow the image below to create a new policy for the IAM role. Use the Json code snippet below for your policy

```bash
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeTags"
            ],
            "Resource": "*"
        }
    ]
}


```


![Image](./Images/Screenshot%202024-05-31%20121225.png)


![Image](./Images/Screenshot%202024-05-31%20121847.png)

2.  Let's recall the parameters we created for our EC2 metric, now let's view the metric on CLoudWatch console.
Navigate to the CloudWatch console . In the navigation menu, select all metrics.

![Image](./Images/Screenshot%202024-05-31%20121939.png)

Select the browser tab and. Search and click on CWAgent

![Image](./Images/Screenshot%202024-05-31%20122152.png)

We can view our metric the memory percent of our EC2 instance


You have successfuully installed and cofi gured CloudWatch to monitor your EC2 instance.
To monitor more metrics, you can go to the parameter store and edit the parameter we created then add more parametersto it. Follow the
AWS official documentation
to read more on parameters syntax for metrics.

