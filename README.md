# ec2-check-reserved-instances

Compares EC2 instance reservations with running instances.

Amazon's reserved instances (ec2-describe-reserved-instances, ec2-describe-reserved-instances-offerings) are a great way to save money when using EC2. An EC2 instance reservation is specified by an availability zone, instance type, and quantity. Correlating the reservations you currently have active with your running instances is a manual, time-consuming, and error-prone process.

This quick little Python script uses boto3 to inspect your reserved instances and running instances to determine if you currently have any reserved instances which are not being used. Additionally, it will give you a list of non-reserved instances which could benefit from additional reserved instance allocations.

To use the program, make sure you have boto3 installed. If you don't already have it, run:

`$ easy_install boto3`

or

`$ pip install boto3`


### Example Output

```
$ ./ec2-check-reserved-instances.py -r us-east-1
>>> Connecting to EC2...
UNUSED RESERVATION!	(1)	m1.small	us-east-1b
UNUSED RESERVATION!	(1)	m2.2xlarge	us-east-1a

Instance not reserved:	(1)	t1.micro	us-east-1c
Instance not reserved:	(2)	m1.small	us-east-1d
Instance not reserved:	(3)	m1.medium	us-east-1d
Instance not reserved:	(1)	m2.2xlarge	us-east-1b

(23) running on-demand instances
(18) reservations
```


# Command-line options

## -r or --region (required)

AWS region to check. Example: 

```
./ec2-check-reserved-instances.py -r us-east-1
```

## -c or --config_file (default: ~/.s3cfg)

Your AWS credentials should go in a JSON-format configuration file as shown below:

```
{
	"aws_access_key_id": "XXXXX",
	"aws_secret_access_key": "YYYYY"
}
```

Example:

```
./ec2-check-reserved-instances.py -r us-east-1 -c path/to/s3config.json
```

If not explicitly specified, this script will assume the credentials are stored in `~/.s3cfg`.


## -i or --ignore_file

You may have instances or reservations you don't want to see in the results - maybe you've put some unused reservations up for sale, or maybe you have some micro instances you don't care all that much about. You can specify the types of reservations and/or running instances you want to ignore in another JSON file:

```
{ 
	"reserved": [
		{ "type": "hi1.4xlarge", "zone": "us-east-1b", "count": 1 },
		{ "type": "hi1.4xlarge", "zone": "us-east-1a", "count": 1 },
	],
	"running": [
		{ "type": "t2.micro", "zone": "us-east-1e", "count": 5 }
	]
}
```

Then, pass the name of that file on the command line:

```
./ec2-check-reserved-instances.py -r us-east-1 -i ignore.json
```

This is particularly useful if you're running this script in a cron job and want to send out an alert when unexpected results show up.


## -v or --vpc_sensitive (default false)

When you create a new reservation, one of the choices you make is whether it will be a Classic or VPC reservation. This guarantees you resources in the environment specified, which is an important consideration for critical parts of your infrastructure.

However, this distinction is irrelevant if you're primarily concerned about the cost savings that come with reserved instances. This script assumes that you're mainly concerned about cost, so by default, the VPC/Classic distinction is ignored.

If you would like to see VPC/Classic mismatches, specify the -v flag on the command line, as in the example below.

#### Output with -v option

```
$ ./ec2-check-reserved-instances.py -r us-east-1 -v
>>> Connecting to EC2...
UNUSED RESERVATION!	(1)	m3.large	us-east-1b_vpc
UNUSED RESERVATION!	(1)	m3.medium	us-east-1c_vpc
UNUSED RESERVATION!	(2)	c3.large	us-east-1a_vpc
UNUSED RESERVATION!	(1)	m1.small	us-east-1c
UNUSED RESERVATION!	(1)	i2.xlarge	us-east-1a_vpc
...
UNUSED RESERVATION!	(1)	c3.large	us-east-1c_vpc
UNUSED RESERVATION!	(1)	c3.2xlarge	us-east-1a_vpc
UNUSED RESERVATION!	(6)	m2.xlarge	us-east-1c_vpc
UNUSED RESERVATION!	(1)	t2.micro	us-east-1a_vpc

Instance not reserved:	(1)	c3.large	us-east-1b
Instance not reserved:	(1)	c3.2xlarge	us-east-1b
Instance not reserved:	(1)	i2.xlarge	us-east-1a
Instance not reserved:	(1)	c3.large	us-east-1c
...
Instance not reserved:	(1)	c3.2xlarge	us-east-1c
Instance not reserved:	(1)	c3.2xlarge	us-east-1a
Instance not reserved:	(6)	m2.xlarge	us-east-1c
Instance not reserved:	(1)	i2.xlarge	us-east-1b

(232) running on-demand instances
(238) reservations
```

#### Equivalent output without -v 

```
$ ./ec2-check-reserved-instances.py -r us-east-1
>>> Connecting to EC2...
UNUSED RESERVATION!	(1)	t2.micro	us-east-1a
UNUSED RESERVATION!	(1)	m3.medium	us-east-1c
UNUSED RESERVATION!	(2)	t1.micro	us-east-1c
UNUSED RESERVATION!	(1)	m1.small	us-east-1c
UNUSED RESERVATION!	(1)	t1.micro	us-east-1a

Congratulations, you have no unreserved instances

(232) running on-demand instances
(238) reservations
```
