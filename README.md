# Jamulus Docker Cloudformation Template

Use at your own risk!

## Requirements

The template relies on a registered domain that is configured in the AWS account.
The VPC must [support IPv6](https://docs.aws.amazon.com/vpc/latest/userguide/vpc-migrate-ipv6.html)

## Template Features

### EC2 Instance

The template is limited to a few small low-cost general purpose Amazon EC2 instances.
Using the default `t3.nano` instance should be sufficient for most use cases.
A `t2.micro` instance can be used [for free for 12 month](https://aws.amazon.com/free/), but it hast just 1 vCPU.

### Jamulus Service

Jamulus does not get installed on the server directly, but is launched as a docker container using the [grundic/jamulus-docker](https://github.com/grundic/jamulus-docker) image.  
A sysvinit script is created to start and stop the docker container. That script is important so Cloudformation can restart the service when you change the configuration.

### Cloudwatch Metrics

The Server create a Cloudwatch Metric "JamulusUserCount" with the number of users connected to the Jamulus Server.

### DNS Record

A DNS record with both the IPv4 and IPv6 addresses of the server is added.

## Resource Usage

### CPU

A `t3.nano` instance with 2 vCPUs gets ~ 25 % of CPU utilization with 10 clients connected. So it should be able to handle quite a few more.  
Just be aware that when exceeding the CPU credits included with your instance, you have to pay for the additional ones or the CPUs get throttled if you don't run the instance in `unlimited` mode.

### Memory

The Jamulus docker container uses only between 40 and 50 MB of memory, depending on the number of channels configured.  
Using a `t3.nano` instance with only 0.5 GiB of memory is sufficient by far.

## Costs

Below calculations are based on prices for the AWS Region eu-central-1 (Frankfurt).  
For other Regions, see pricing for [EC2](https://aws.amazon.com/ec2/pricing/on-demand) and [EBS](https://aws.amazon.com/ebs/pricing/)

### Fixed Costs

You pay a fixed price for the EC2 Instance and the EBS Volume attached to it:

| | Unit Price | Quantity | Price/Month |
|-|-|-|-|
| EC2-Instance: `t3.nano` | $0.006 per Hour | 720 Hours / Month | $4,32 |
| EBS-Volume: `gp3` | $0.0952 per GB-month | 8 GB / Month | $0,76 |

### Data Transfer Costs

You need about 200 Kbps per user up and down (https://jamulus.io/wiki/Running-a-Server), so one user will transfer ~90 MB of data per hour.

10 users, using the server for 1 hour in average every day in a month would transfer ~27 GB up and down. For that the costs would be:

| | Unit Price | Quantity | Price/Month |
|-|-|-|-|
| Data Transfer OUT Up to 1 GB / Month | $0.00 per GB | 1 GB / Month | $0.00 |
| Data Transfer OUT Next 9.999 TB / Month | $0.09 per GB | 26 GB / Month | $2.34 |

### Costs for extra CPU credits

The Baseline Performance of a `t3.nano` instance is 5 %. That corresponds to ~ 2 users being connected all the time. So 10 users could use the server for ~ 4-5 hours per day with no additional cost:

| | Unit Price | Quantity | Price/Month |
|-|-|-|-|
| EC2-Instance extra CPU credits (Linux, RHEL and SLES) | $0.05 per vCPU-Hour | 0 vCPU-Hours / Month | $0.00 |

### Other costs

If you shutdown an instance without removing the [Elastic IP](https://aws.amazon.com/ec2/pricing/on-demand/#Elastic_IP_Addresses) assigned to it, you get charged for that.
