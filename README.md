![alt tag](https://wimd.io/gitlogo.jpg)

# WIMD.IO Data Repository & Services

## What is WIMD?
This is WIMD.IO, an open, fully API driven, IoT data repository that enables low footprint connected devices to do two way data exchange with the cloud in order to store data and retrieve commands.<br>
The platform provides several data services that can be used to do data cleansed, data validation, on the fly aggregation and much more.
The data is stored in a database that can easily accessed by any external system, reporting tools, analytics systems that will focus on the data exploration, leaving all the data acquisition life cycle issues behind.

## System services
![alt tag](https://wimd.io/images/services.jpg)

## System entities
![alt tag](https://wimd.io/images/entities.jpg)

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
                       |   sensor_rule    +------>      vee.c       <-----+    sensor    |          |               |
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
                                +-----------v-------------+--------------------------+-------------------------+--------------------+
                                |                         |      |                   |              |          |    |               |
                                |                         |      |                   |              |          |    |               |
+-----------------+    +--------v----------+     +--------v------v---+   +-----------v-----------+  | +--------v----+---+    +------v-----+
|                 |    |                   |     |                   |   |                       |  | |                 |    |            |
|     alarm.c     <----+ sensor_alarm_data |     |      rollup.c     |   |  sensor_virtual_data  |  | |  calculation.c[_mod_calculation_](#calculation)  |    |    etl.c   |
|                 |    |                   |     |                   |   |                       |  | |                 |    |            |
+--------+--------+    +-------------------+     +--------+----------+   +-----------+-----------+  | +--^-------+------+    +------+-----+
         |                                                |                          |              |    |       |                  |
         |                                                |                          |              |    |       |                  |
+--------v--------+                                       |                   +------v------+       |    | +-----v------+    +------v------+
|                 |                                       |                   |             |       |    | |            |    |             |
|  alarm_summary  |                                       |                   |  virtual.c  +-------+    | |   report   |    |     TSDB    |
|                 |                                       |                   |             |            | |            |    |             |
+--------+--------+                                       |                   +-------------+            | +------------+    +-------------+
         |                                                |                                              |
         |                                                |                                              |
+--------v--------+                             +---------v----------+                                   |
|                 |                             |                    |                                   |
|  alarm_history  |                             | sensor_rollup_data +-----------------------------------+
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
<pre>
| Attribute  | Description                                    |
|------------|------------------------------------------------|
| Time Stamp | A time stamp in in the form YYY-MM-DD HH:MM:SS |
| Value      | A double precision numerical value             |
| Status     | The data points status as listed below         |
</pre>

### Sensor status
After the validation process is executed, a data point will have one of these status code
<pre>
| Status    | Value | Description                                          |
|-----------|-------|------------------------------------------------------|
| Valid     | 1     | The data point is valid. I.e. did not break any rule |
| Increment | 2     | The data point is not incremental                    |
| Decrement | 4     | The data point is not decremental                    |
| gap       | 8     | There is a gap between samples                       |
| Maximum   | 16    | The data point is above the limit                    |
| Minimum   | 32    | The data point is below the limit                    |
| Estimated | 64    | The data is valid but was estimated                  |
| Null      | 128   | The data point was set to NULL                       |
| Constant  | 256   | The data point is set to a constant                  |
| Last      | 512   | The value was set as its previous value              |
</pre>

## Validation rules
The rules have the following atributes:

### Global rule
```sh
enabled [true | false]
```
Indicates that the rule is active, otherwise no results are returned

```sh
expectedinterval [seconds]
```
The expected time span from one data point to another. This might be used as a reference by onter rules.

### Pre validation
```sh
increment [true | false]
```
Time series must receive consecutive incremental data.

```sh
decrement [true | false]
```
Time series must receive consecutive decremental data.

```sh
checkgap [true | false]
```
The time series time stamp data points will be checked over its time span. If the time difference between two consecutive samples are bigger than ``` expectedinterval ``` parameter the sample will be flagged as invalid. If the value is zero no check is made.

```sh
fillgaps [true | false]
```
The gap(s) found between two data points will be filled by using the ```fillfunction ``` and will follow the ``` expectedinterval ``` parameter in order to create the data points.

```sh
fillfunction [null | last | linear | number]
``` 
Fill method.
 
### Transformation
```sh
applydelta [true | false]
```
This is an index and the returning time series will be the delta  calculation from adjacent values.

```sh
scalefactor [number]
```
Will apply an multiplicator factor if different of zero.

```sh
addOffset [number]
```
Add and offset to the resulting value.

```sh
absolute [true | false]
```
Apply ABS function to the result.

```sh
downsampling [true | false]
```
Downsample data points to the ``` expectedinterval ```

```sh
downsamplingfunction [min | max | avg | sum | count]
```
Downsample statistical function

```sh
hastimezone [true | false]
```
Indicated it the a time zone converstion must be applied to the time series before being used

```sh
tzname [text]
```
The time zone mane as defined. Example: Africa/Abidjan

### Data transformation
If the validation process transforms the original time series values, this time series is stored in the system as a second version, along with its original values. This is important mostly because with the original time series the system can always change the rules and rebuild the calculations. In order to store this derived time series the system needs  to know at least:
```sh
newsensorname [text]
```
The name of the new sensor

```sh
newsensorunit [text]
```
The new sensor unit

```sh
newsensorunitname [text]
```
The new sensor unit name

Some sensor table fields are cloned automatically by the system like classification, zone, usage, time zone, etc.

### Post validation
```sh
maximum [number]
```
Check if the values is under of a maximum value.

```sh
minimum [number]
```
Check if the values is above of a maximum value.

```sh
tseoi [true | false]
```
For the resulting value, consider the time stamp for the end of interval for aggregations purposes.

## System APIs
The whole system can be controlled by using the RESTful API web service described in [here](https://wimd.io/api/ "WIMD.IO API").

## Calculation module
This module loads scripts from the database in form of libraries and programs and execute them based on the customer needs. 
Every program must be lunched by an external entity like a cron job instance by running the 'wimd' program with <code>--do-calculation [script name]</code> option. In this case any other job will be ignored and only the script will be executed.
The calculation has access to all system entities and historical data via specific
functions (Lua-C binding, not APIs). You can check the available functions here.
Although the result might be sent back to a new time series, the module has access to the report table via an API.
The report table consists basically in a JSON document that have all necessary information for a report generation toll.
The report must be consumed or sent to an external application in order to be generated, as an example, we can send new charts to an Excel application or Tableau in a form of CSV content.
The repost structure has the following format
<pre>
{
    "name": "The report name here",
    "type": "The report type",
    "created": "ISO date creation time UTC",
    "content-type": ["csv" | "json" | "xml"],
    "content-encoding": ["none" | "base64"],
    "compression": ["plain" | "gzip" | "defalte"],
    "body": {
        [your content here]
    }
}
</pre>
The script can optionally choose to push the report result to an URL via a POST command. The report will be available via API anytime.




## License
See license file please
