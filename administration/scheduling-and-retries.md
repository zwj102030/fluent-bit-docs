# Scheduling and Retries

[Fluent Bit](https://fluentbit.io) has an Engine that helps to coordinate the data ingestion from input plugins and calls the _Scheduler_ to decide when it is time to flush the data through one or multiple output plugins. The Scheduler flushes new data at a fixed time of seconds and the _Scheduler_ retries when asked.

Once an output plugin gets called to flush some data, after processing that data it can notify the Engine three possible return statuses:

* OK
* Retry
* Error

If the return status was **OK**, it means it was successfully able to process and flush the data, if it returned an **Error** status, means that an unrecoverable error happened and the engine should not try to flush that data again. If a **Retry** was requested, the _Engine_ will ask the _Scheduler_ to retry to flush that data, the Scheduler will decide how many seconds to wait before that happens.

## Configuring Retries

The Scheduler provides a simple configuration option called **Retry\_Limit** which can be set independently on each output section. This option allows to disable retries or impose a limit to try N times and then discard the data after reaching that limit:

|  | Value | Description |
| :--- | :--- | :--- |
| Retry\_Limit | N | Integer value to set the maximum number of retries allowed. N must be &gt;= 1 \(default: 1\) |
| Retry\_Limit | `no_limits` or `False` | When Retry\_Limit is set to `no_limits` or`False`, means that there is not limit for the number of retries that the Scheduler can do. |
| Retry\_Limit | no\_retries | When Retry\_Limit is set to no\_retries, means that retries are disabled and Scheduler would not try to send data to the destination if it failed the first time. |

### Example

The following example configures two outputs where the HTTP plugin has an unlimited number of while the Elasticsearch plugin have a limit of 5 retries:

```text
[OUTPUT]
    Name        http
    Host        192.168.5.6
    Port        8080
    Retry_Limit False

[OUTPUT]
    Name            es
    Host            192.168.5.20
    Port            9200
    Logstash_Format On
    Retry_Limit     5
```

