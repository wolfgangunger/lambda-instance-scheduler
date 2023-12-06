# lambda-instance-scheduler
lambda to stop and start ec2 instances, auto-scaled instances and rds-instances

## installation
deploy the CloudFormation Template Lambda-Instance-Start-Stop.yaml  
The params are:  
  AccountName:  
    Description: Account name, used to prefix resources  
  
  OwnerName:  
    Description: An owner name, used in tags  
  
  ScheduleExpressionStart:    
    Description: Cron Job expression, when to start the EC2 and RDS instances  
    Default: cron(0 6 ? * 2-6 *) # UTC time  
  
  ScheduleExpressionStop:  
    Description: Cron Job expression, when to stop the EC2 and RDS instances  
    Default: cron(0 18 ? * 2-6 *) # UTC time  
  
## created resources

the CloudFormation Template will deploy 2 Lambdas  
[AccountName]-lambda-start-instances  
[AccountName]-lambda-stop-instances  
  
also 2 event rules:  
StartEC2Instances  
StoptEC2Instances  
  
The Rules are scheduled by the cronjob expression in the Cloudformation Template, default is   
Start: cron(0 6 ? * 2-6 *) # UTC time, which is mo-fr 6am  
Stop: cron(0 18 ? * 2-6 *) # UTC time, which is mo-fr 6pm, on weekends instances will remain stopped until next start event on monday  
Change the cronjob expression for the required behaviour  

## logic  
The Lambdas will search for instances & autoscaling groups by tag, 
the following Tag is required to identify an instance to schedule:  
Auto-Start :	true    
  
The lambda will start & stop EC2 instances and RDS instances and will change the minSixe and DesiredCapacity for autoscaling groups.   
  
Instances to be scheduled:
- EC2 Instances
- RDS Instances
- EC2 Instances in a autoscaling group
The autoscaling group must be tagged, not the instance ( stopping the instance would only cause an immediatelly restart by the autoscaling group)  
autoscaling params MinSize,  and DesiredCapacity will be set to either 1 or 0  
Note: For autoscaling groups this solutions currently only supports desired count = 1  
If your desired count is != 1 don't tag them.    
To support different values the lambda logic must be changed, for example storing the current value in a parameter in parameter store   
before stopping the instance and reading this value before starting/reseting the old value.  


