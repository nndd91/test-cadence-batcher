# Readme
For testing on cadence batcher

## Commands


### Testing batch workflow
We will start a couple of workflow then run the batch command. Currently it seem to get stuck and failed with a context error. (See Example error message below)
```
docker-compose up -d 

docker exec -it test-cadence-batcher_cadence_1 /bin/sh

cadence --address=host.docker.internal:7933 --do simple-domain domain register

cadence --address=host.docker.internal:7933 --do simple-domain wf start --tl hello-world --wt SimpleWorkflow --et 600 -i '"cadence"'

cadence --address=host.docker.internal:7933 --do simple-domain wf batch start --query "WorkflowType='SimpleWorkflow' and CloseTime = missing" --reason "test" --bt signal --sig testname --input "some input"

cadence --address=host.docker.internal:7933 --do simple-domain wf list --query "WorkflowType='SimpleWorkflow' AND CloseTime = missing "
```

Visit: http://localhost:8088/domains/cadence-batcher/workflows?range=last-30-days&status=ALL to view progress on batcher.

### Checking Elastic Search is working
Since search attributes only works with es enabled, we try to start a few workflows with search attributes and use the list workflow command to check that elastic search is indeed working.
```
cadence --address=host.docker.internal:7933 --do simple-domain wf start --tl hello-world --wt SimpleWorkflow --et 600 -i '"cadence"' -search_attr_key 'CustomKeywordField' -search_attr_value 'keyword3'

cadence --address=host.docker.internal:7933 --do simple-domain wf list -q "CustomKeywordField in ('keyword3') and CloseTime = missing and WorkflowType='SimpleWorkflow'"

```

## Example Error Message
```
{"level":"error","ts":"2021-02-10T10:05:10.151Z","caller":"internal/internal_task_handlers.go:1829","msg":"Activity error.","Domain":"cadence-batcher","TaskList":"cadence-sys-batcher-tasklist","WorkerID":"1@ebc1d61dfac9@cadence-sys-batcher-tasklist","WorkflowID":"f5750a58-584b-456d-8bdd-dbe4ccbeb063","RunID":"f8392a43-2ebd-4421-899d-467dd8a23b7d","ActivityType":"cadence-sys-batch-activity","error":"code:unavailable message:round-robin peer list timed out waiting for peer: context deadline exceeded","stacktrace":"go.uber.org/cadence/internal.(*activityTaskHandlerImpl).Execute\n\t/go/pkg/mod/go.uber.org/cadence@v0.14.1/internal/internal_task_handlers.go:1829\ngo.uber.org/cadence/internal.(*activityTaskPoller).ProcessTask\n\t/go/pkg/mod/go.uber.org/cadence@v0.14.1/internal/internal_task_pollers.go:888\ngo.uber.org/cadence/internal.(*baseWorker).processTask\n\t/go/pkg/mod/go.uber.org/cadence@v0.14.1/internal/internal_worker_base.go:323"}
{"level":"info","ts":"2021-02-10T10:05:10.154Z","caller":"internal/internal_worker_base.go:328","msg":"Task processing failed with error","Domain":"cadence-batcher","TaskList":"cadence-sys-batcher-tasklist","WorkerID":"1@ebc1d61dfac9@cadence-sys-batcher-tasklist","WorkerType":"ActivityWorker","error":"EntityNotExistsError{Message: activity task not found}"}
```