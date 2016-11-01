![alt tag](https://wimd.io/gitlogo.jpg)

# WIMD.IO Data Repository & Services

## What is WIMD?
This is WIMD.IO, an open, fully API driven, IoT data repository that enables low footprint connected devices to do two way data exchange with the cloud in order to store data and retrieve commands.<br>
The platform provides several data services that can be used to do data cleansed, data validation, on the fly aggregation and much more.
The data is stored in a database that can easily accessed by any external system, reporting tools, analytics systems that will focus on the data exploration, leaving all the data acquisition life cycle issues behind.

## System entities

## Data ingestion diagram
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
The back end database for WIMD.IO is MariaDB server.<br>
The database schema consists of many tables, no triggers nor stored procedures.<br>
The main tables in the system are:<br><pre>
| Name                | Description                                                                                                           |
|---------------------|-----------------------------------------------------------------------------------------------------------------------|
| sensor              | A variable in the system that represents a physical or logical quantity                                               |
| sensor_rule         | A set of rules to validate and transform sensor values before they get into the system database                       |
| sensor_json_data    | The 'raw' data sent by clients are stored here before being processed by the validation task                          |
| sensor_history_data | The sensor database in form of time series. Once a sensor data is validated it is stored here                         |
| job                 | A list of tasks to be performed by the system, like aggregation for example                                           |
| sensor_alarm_data   | If a sensor is part of one or more alarm definition, the sensor data is copied here for later handling                |
| sensor_virtual_data | If a sensor is part of a virtual sensor calculation, the sensor data if copied here                                   |
| sensor_rollup_data  | Once the time series are aggregated, its content is stored here in form of hourly, daily, monthly and annually groups |
| alarm_summary       | Active alarms (not finished or terminated) are kept here                                                              |
| alarm_history       | Once a alarm is not active anymore it goes here                                                                       |</pre>
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
