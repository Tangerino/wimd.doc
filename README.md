![alt tag](https://wimd.io/gitlogo.jpg)

# WIMD.IO Data Repository & Services

## What is WIMD?
This is WIMD.IO, an open, fully API driven, IoT data repository that enables low footprint connected devices to do two way data exchange with the cloud in order to store data and retrieve commands.<br>
The platform provides several data services that can be used to do data cleansed, data validation, on the fly aggregation and much more.
The data is stored in a database that can easily accessed by any external system, reporting tools, analytics systems that will focus on the data exploration, leaving all the data acquisition life cycle issues behind.

## Global system architecture
<pre>
+-----------------+       +----------+          +--------------------+
|                 |       |          |          |                    |
|   DATA SOURCE   +------->  rest.c  +---------->  sensor_json_data  <------------------------------+---------------+
|                 |       |          |          |                    |                              |               |
+-----------------+       +----------+          +---------+----------+                              |               |
                                                          |                                         |               |
                                                          |                                         |               |
                       +------------------+      +--------v---------+     +--------------+          |               |
                       |                  |      |                  |     |              |          |               |
                       |  sensor_rule_v2  +------>  validationv2.c  <-----+    sensor    |          |               |
                       |                  |      |                  |     |              |          |               |
                       +------------------+      +--+------------+--+     +--------------+          |               |
                                                    |            |                                  |               |
                                                    |            |                                  |               |
                                                    |            |                                  |               |
                                +-------------------v---+    +---v-------------------+              |               |
                                |                       |    |                       |              |               |
                                |  sensor_history_data  |    |          job          |              |               |
                                |                       |    |                       |              |               |
                                +-----------+-----------+    +---+-------------------+              |               |
                                            |                    |                                  |               |
                                            |                    |                                  |               |
                                +-----------v-------------+------)-------------------+--------------)----------+----)---------------+
                                |                         |      |                   |              |          |    |               |
                                |                         |      |                   |              |          |    |               |
+-----------------+    +--------v----------+     +--------v------v---+   +-----------v-----------+  | +--------v----+---+    +------v-----+
|                 |    |                   |     |                   |   |                       |  | |                 |    |            |
|     alarm.c     <----+ sensor_alarm_data |     |   aggregation.c   |   |  sensor_virtual_data  |  | |  calculation.c  |    |  influx.c  |
|                 |    |                   |     |                   |   |                       |  | |                 |    |            |
+--------+--------+    +-------------------+     +--------+----------+   +-----------+-----------+  | +--------^--------+    +------+-----+
         |                                                |                          |              |          |                    |
         |                                                |                          |              |          |                    |
+--------v--------+                                       |                   +------v------+       |          |             +------v------+
|                 |                                       |                   |             |       |          |             |             |
|  alarm_summary  |                                       |                   |  virtual.c  +-------+          |             |  INFLUX DB  |
|                 |                                       |                   |             |                  |             |             |
+--------+--------+                                       |                   +-------------+                  |             +-------------+
         |                                                |                                                    |
         |                                                |                                                    |
+--------v--------+                             +---------v----------+                                         |
|                 |                             |                    |                                         |
|  alarm_history  |                             | sensor_rollup_data +-----------------------------------------+
|                 |                             |                    |
+-----------------+                             +--------------------+
</pre>

## The back end database
The back end database for WIMD.IO is MariaDB server.
The database schema consists of many tables, no triggers not storage procedures. The main tables in the system are:<br>
<b>Sensor</b><br>
The <code>sensor</code> table holds the basic system element, a sensor.<br>
A sensor is one numerical time series. Each sensor have an associate validation rule that tells to the system the way each sensor behaves, how it is validated and/or transformed.

### Sensor attributes
A data point consists of three attributes as in:
* Time stamp
* Value and
* Status

### Sensor status
After the validation process is executed, a data point will have one of these status code
* valid      - The data point is valid. I.e. did not break any rule
* increment  - The data point is not incremental
* decrement  - The data point is not decremental
* gap        - There is a gap between samples
* maximum    - The data point is above the limit
* minimum    - The data point is below the limit
* estimated  - Tha data is valid but was estimated

<b>Validation rule</b><br>
The <code>validation_rule_v2</code> table holds the rules to import, validate and optionally transform a time series.<br>
Before use a time series the user must be sure that the data is correct, valid, clean. If we manipulate data without any sort of validation we may not be accurate or even lead to bad decisions.

## License
See license file please
