
# **附录A SLO文档示例**

<br/>
<br/>

本文介绍了示例游戏服务的SLO。


| **Status** | **Published** |
| :--- | :--- |
| Author|Steven Thurgood|
| Date| 2018-02-19 |
| Reviewers|David Ferguson|
|Approvers|Betsy Beyer|
| Approval Date| 2018-02-20 |
| Revisit Date| 2019-02-01 |


**服务概述**

*示例游戏服务*允许Android和iPhone用户彼此玩游戏。该应用程序在用户的手机上运行，并且移动通过REST API发送回*API*。*数据存储区*包含所有当前和先前游戏的状态。*得分管道*读取该表并生成今天，本周和所有时间的最新联赛表。排行榜结果可在应用程序中通过API获得，也可以在公共*HTTP服务器*上获得。

SLO使用四个星期的滚动窗口。


**SLI和SLO **


| **Category** | **SLI** | **SLO** |
| :--- | :--- | :--- |
| **API** |  |  |
|Availability|The proportion of successful requests, as measured from the load balancer metrics. Any HTTP status other than 500–599 is considered successful. <br/> ``` count of "api" http_requests which do not have a 5XX status code divided by count of all "api" http_requests  ```| 97％ success|
|延迟|The proportion of sufficiently fast requests, as measured from the load balancer metrics.<br/>```"Sufficiently fast" is defined as < 400 ms, or < 850 ms. count of "api" http_requests with a duration less than or equal to "0.4" seconds divided by count of all "api" http_requests```<br/> <br/> ```count of "api" http_requests with a duration less than or equal to "0.85" seconds divided by count of all "api" http_requests```| 90% of requests < 400 ms <br/> 99% of requests < 850 ms |
| **HTTP server** | | |
|可用性|根据负载平衡器指标衡量的成功请求的比例。除500--599以外的任何HTTP状态均被视为成功。<br/> ```count of "web" http_requests which do not have a 5XX status code divided by count of all "web" http_requests```| 99％|
|Latency | The proportion of sufficiently fast requests, as measured from the load balancer metrics.<br/>“Sufficiently fast” is defined as < 200 ms, or < 1,000 ms.<br/>``` count of "web" http_requests with a duration less than or equal to "0.2" seconds divided by count of all "web" http_requests ```<br/><br/> ``` count of "web" http_requests with a duration less than or equal to "1.0" seconds divided by count of all "web" http_requests ```|90% of requests < 200 ms<br/> 99% of requests < 1,000 ms|
| **Score pipeline** |||
| Freshness | The proportion of records read from the league table that were updated recently. “Recently” is defined as within 1 minute, or within 10 minutes.<br/> Uses metrics from the API and HTTP server: ``` count of all data_requests for "api" and "web" with freshness less than or equal to 1 minute divided by count of all data_requests ``` <br/><br/> ``` count of all data_requests for "api" and "web" with freshness less than or equal to 10 minutes divided by count of all data_requests ```| 90% of reads use data written within the previous 1 minute. 99% of reads use data written within the previous 10 minutes. |
| Correctness|The proportion of records injected into the state table by a correctness prober that result in the correct data being read from the league table.<br/> A correctness prober injects synthetic data, with known correct outcomes, and exports a success metric: ``` count of all data_requests which were correct divided by count of all data_requests ``` |99.99999% of records injected by the prober result in the correct output.|
| Completeness|The proportion of hours in which 100% of the games in the data store were processed (no records were skipped).<br/>Uses metrics exported by the score pipeline:``` count of all pipeline runs that processed 100% of the records divided by count of all pipeline runs```| 99% of pipeline runs cover 100% of the data. |


**理论**

可用性和延迟SLI基于2018年1月1日至2018年1月28日之间的测量。可用性SLO向下舍入到最接近的1％，而延迟SLO时序被舍入到最接近的50 ms。作者选择了所有其他数字，并验证了这些服务正在或高于这些级别运行。

尚未尝试验证这些数字是否与用户体验密切相关。[^115]

**错误预算**

每个目标都有一个单独的错误预算，定义为该目标的100％减去（-）目标。例如，如果在前四周中有1,000,000个请求发送到API服务器，则API可用性错误预算为1,000,000中的3％（100％-97％）：30,000个错误。

当我们的任何目标用尽错误预算时，我们将制定错误预算政策（请参阅附录B）。

**说明和警告**

- 请求指标是在负载均衡器上测量的。此度量可能无法准确度量用户请求未到达负载均衡器的情况。

- 我们仅将HTTP 5XX状态消息视为错误代码；其他一切都算成功。

- 正确性探测器使用的测试数据包含大约200个测试，每1秒注入一次。我们的错误预算是每四周48个错误。


<br/>
<br/>

[^115]: 即使SLO中的数字不是严格依据证据的，也有必要对此进行记录，以使将来的读者可以理解这一事实，并做出适当的决定。他们可能会决定值得收集更多证据的投资。