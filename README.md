# usergroup_splunk_containerimage

## start AWS demo instance

```bash
aws ec2 run-instances --cli-input-yaml file://demohost-aws/demohost.yml --user-data file://demohost-aws/demohost-cloudinit.yml --output yaml
```

- get external name
```bash
aws ec2 describe-instances \
  --filters Name=instance-state-name,Values=running \
  --query 'Reservations[*].Instances[*].[InstanceId, Tags[?Key==`Name`]|[0].Value, PublicIpAddress, PublicDnsName,State.Name]' \
  --output table
```

- terminate instance
```bash
aws ec2 terminate-instances --instance-ids XXX
```