# Readme
For testing on cadence batcher

## Commands

```
docker-compose up -d 

docker exec -it test-cadence-batcher_cadence_1 /bin/sh

cadence --address=host.docker.internal:7933 --do simple-domain domain register

cadence --address=host.docker.internal:7933 --do simple-domain wf start --tl hello-world --wt SimpleWorkflow --et 600 -i '"cadence"'

cadence --address=host.docker.internal:7933 --do simple-domain wf batch start --query "WorkflowType='SimpleWorkflow' and CloseTime = missing" --reason "test" --bt signal --sig testname --input "some input"
```

Visit: http://localhost:8088/domains/cadence-batcher/workflows?range=last-30-days&status=ALL to view progress on batcher.
