# Define dynamic scaling policy

In this lab you will define a dynamic scaling policy for your autoscaling group based on CPU level.

### Helpful commands

#### Get instance IPs
```
aws ec2 describe-instances --instance-ids <instance_id> --query 'Reservations[0].Instances[0].PrivateIpAddress'
```

### Lab

#### Provision a group
```
ansible-playbook provision-elb-ec2-metric-alarm-playbook.yml
```

#### Verify instance states
```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names workshop-elb-ec2-metric-alarm-asg --query 'AutoScalingGroups[0].Instances'
```

#### Verify ELB state
```
aws elb describe-instance-health --load-balancer-name workshop-elb-ec2-metric-alarm-lb
```

#### Verify alarms
```
aws cloudwatch describe-alarms --alarm-names cpuDown_workshop-elb-ec2-metric-alarm-asg cpuUP_workshop-elb-ec2-metric-alarm-asg
```

#### Copy stress rpm onto any of the instances
```
aws autoscaling describe-auto-scaling-groups --auto-scaling-group-names workshop-elb-ec2-metric-alarm-asg --query 'AutoScalingGroups[0].Instances'
aws ec2 describe-instances --instance-ids <instance_id> --query 'Reservations[0].Instances[0].PrivateIpAddress'
scp -i <ssh_key> rpm/stress.rpm <user>@<instance_ip>:/tmp/
```

#### SSH to the machine, install 'stress' and run it
```
ssh <user>@<instance_ip> -i <path_to_key>
sudo su
rpm -i /tmp/stress.rpm
stress -c 50
```

#### Watch the current CPU level and verify that ASG scales up
```
watch uptime
```

#### Stop 'watch uptime' and verify that ASG scales down 

#### Clean up
```
ansible-playbook provision-elb-ec2-metric-alarm-playbook.yml --extra-vars "state=absent" --tags "asg"
ansible-playbook provision-elb-ec2-metric-alarm-playbook.yml --extra-vars "state=absent" --tags "lc"
```