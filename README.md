# usergroup_splunk_containerimage

## marp

- start marp webserver
```bash
docker run --rm --init -v $PWD/marp:/home/marp/app -e LANG=$LANG -p 8080:8080 -p 37717:37717 marpteam/marp-cli --allow-local-files --html -s .
```

- convert to PDF

```bash
docker run --rm --init -v $PWD/marp:/home/marp/app/ -e LANG=$LANG marpteam/marp-cli slides.md --allow-local-files --html  --pdf
```


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