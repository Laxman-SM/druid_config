**Table of Contents**

- [Overview](#overview)
- [Issues](#issues)
  - [Index Task Fails](#index-task-fails)
  - [Coordinator Console Problems](#coordinator-console-problems)
- [Resolved Issues](#resolved-issues)
  - [No Task Logs](#no-task-logs)

Overview
===

Please see the included `.properties` files for the druid configuration of each node type. At the top of each file is a summary of system resources such as the number of processors the machine has, the size of main memory in bytes, etc...

My druid cluster consists of the following nodes:

* [3 historical](historical.properties)
* [1 overlord](overlord.properties)
* [1 middleManager](middleManager.properties)
* [1 broker](broker.properties)
* [1 coordinator](coordinator.properties)
* 1 mysql
* 1 zookeeper

**Each node is a seperate AWS EC2 instance**

Issues
===

Index Task Fails
---
I am trying to reindex my `click_conversion` dataset so that segments are indexed by `WEEK` rather than by `DAY`.

Here is my indexing task specification:

```json
{
    "type": "index",
    "dataSource": "click_conversion_weekly",
    "granularitySpec":
    {
        "type": "uniform",
        "gran": "WEEK",
        "intervals": ["2014-04-06T00:00:00.000Z/2014-04-13T00:00:00.000Z"]
    },
    "indexGranularity": "minute",
    "aggregators": [
        {"type": "count", "name": "count"},
        {"type": "doubleSum", "name": "commissions", "fieldName": "commissions"},
        {"type": "doubleSum", "name": "sales", "fieldName": "sales"},
        {"type": "doubleSum", "name": "orders", "fieldName": "orders"}
    ],
    "firehose": {
        "type": "ingestSegment",
        "dataSource": "click_conversion",
        "interval": "2014-04-06T00:00:00.000Z/2014-04-13T00:00:00.000Z"
    }
}
```

The task fails but I am unable to determine why because there is no task log.

Coordinator Console Problems
----

The Coordniator Console looks like this:
![alt tag](https://raw.github.com/lexicalunit/druid_config/master/images/console.png)

When I go to `http://coordinator-ip:8080`, the Coordinator Console comes up and works perfectly. For example, the dropdowns are properly populated:
![alt tag](https://raw.github.com/lexicalunit/druid_config/master/images/working_console.png)

However, when I go to, for example, `http://overlord-ip:8080` instead, the console will come up but nothing works properly. For example, dropdowns not working anymore:
![alt tag](https://raw.github.com/lexicalunit/druid_config/master/images/broken_console.png)

Resolved Issues
===

No Task Logs
---
After kicking off a Indexing Task I can see that it is running in the console:
![alt tag](https://raw.github.com/lexicalunit/druid_config/master/images/task_running.png)

Eventually, the task fails:
![alt tag](https://raw.github.com/lexicalunit/druid_config/master/images/task_fail.png)

When I click on the link `log (all)` I am taken to a page that just says:

```
No log was found for this task. The task may not exist, or it may not have begun running yet.
```

Given the following properties in my overlord configuration:

```properties
druid.indexer.logs.type=s3
druid.indexer.logs.s3Bucket=s3-bucket
druid.indexer.logs.s3Prefix=logs
```

And the following properties in my middleManager configuration:

```properties
druid.selectors.indexing.serviceName=druid:overlord
```

The logs should be available on S3, however they are not:

```bash
$ s3cmd ls s3://s3-bucket/
	DIR   s3://s3-bucket/click_conversion/
	DIR   s3://s3-bucket/click_conversion_weekly/
```

**Resolution:**

The following needed to be added to the middleManager configuration:

```properties
druid.indexer.logs.type=s3
druid.indexer.logs.s3Bucket=s3-bucket
druid.indexer.logs.s3Prefix=logs
```
