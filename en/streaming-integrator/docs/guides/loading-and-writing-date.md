# Loading and Writing Data

Loading and writing data involves publishing the data in a destination where it can be extracted again at any given time for further processing. WSO2 Streaming Integrator supports loading and writing data to databases, files, and cloud storages.

## Loading data to databases

WSO2 Streaming allows you to load data into databases so that the data can be available in a static manner for further processing. You can load the data received from another source unchanged or after processing it. This is achieved by defining [Siddhi tables](https://siddhi.io/en/v5.1/docs/query-guide/#table) that are connected to database tables, and then writing queries to publish data into those tables so that it can be transferred to the connected database tables.

![Loading Data to Databases](../images/loading-and-writing-data/load-data-to_database.png)

To understand this, consider an example of an sweet shop that needs to save all its sales records in a database table. To address this, you can write a Siddhi application as follows:

- **Define a table**

    In this example, let's define a table named `SalesRecords` as follows:
    
    ```
    @primaryKey('ref')
    @index('name')
    @store(type='rdbms', jdbc.url="jdbc:mysql://localhost:3306/SalesRecordsDB", username="root", password="root" , jdbc.driver.name="com.mysql.jdbc.Driver")
    define table SalesRecords(ref string, name string, amount int);
    ```
  The above table definition captures sales records. The details captured in each sales record includes the transaction reference (`ref`), the name of the product (`name`), and sales amount (`amount`). The `ref` attribute is the primary key because there cannot be two or more records with the same transaction reference. The `name` attribute is an index attribute.
  
  A data store named `SalesRecordsDB` is connected to this table definition via the `@store` annotation. This is the data store to which you are loading data.

- **Add a query**

    You need to add a query to specify how events are selected to be inserted into the table you defined. You can add this query as shown below. 
    
    ```
    from SalesRecordsStream
    select *
    insert into SalesRecords;
    ``` 
    The above query gets all the input events from the `SalesRecordsStream` stream and loads them to the `SalesRecords` table.
    
### Try it out

To try out the example given above, follow the steps below:

1. Set up MySQL as follows:

    1. Download and install MySQL.    
    
    2. Start the MySQL server, create the database and the database table you require as follows:
    
        1. To create a new database, issue the following MySQL command.
        
            ```
            CREATE SCHEMA sales;
            ```
                            
        2. Switch to the `sales` database by issuing the following command.
            
           ```
           use sales;
           ```
           
        3. Create a new table, by issuing the following command.        
            
            ```
            CREATE TABLE sales.SalesRecords (
              ref INT NOT NULL AUTO_INCREMENT,
              name VARCHAR(45) NULL,
              amount INT NULL,
              PRIMARY KEY (ref));
            ```
        
2. [Start and access WSO2 Streaming Integrator Tooling](../develop/streaming-integrator-studio-overview.md/#starting-streaming-integrator-tooling).

3. Install the `RDBMS - MySQL` extension in Streaming Integrator tooling. For instructions to install an extension, see [Installing Siddhi Extensions](../develop/installing-siddhi-extensions.md).

4. Open a new file in WSO2 Streaming Integrator Tooling and add the following Siddhi content to it.

    ```
    @App:name('SalesApp')
    
    define stream SalesRecordsStream (name string, amount int);
    
    @store(type = 'rdbms', jdbc.url = "jdbc:mysql://localhost:3306/sales?useSSL=false", username = "root", password = "root", jdbc.driver.name = "com.mysql.jdbc.Driver")
    @primaryKey('ref' )
    @index('name' )
    define table SalesRecords (name string, amount int);
    
    from SalesRecordsStream 
    select * 
    insert into SalesRecords;
    ```
   
   Save the Siddhi application.   
    
5. To simulate an event to the `SalesApp` Siddhi application, click the **Event Simulator** icon in the side panel. 

    ![simulate event](../images/loading-and-writing-data/event-simulator-icon.png)

    Then simulate an event as follows:
    
    1. In the **Siddhi App Name** field, select **SalesApp**.
    
    2. In the **Stream Name** field, select **SalesRecordsStream**.
    
    3. In the **Attributes** section, enter values for the attributes as follows:
    
        ![Simulate Single Event](../images/loading-and-writing-data/simulate-single-event.png)
    
        | **Attribute** | **Value**        |
        |---------------|------------------|
        | **ref**       | `AA000000000001` |
        | **name**      | `fruit cake`     |
        | **amount**    | `100`            |
        
    4. Click **Start and Send**.

6. To check whether the `sales` mysql table is updated, issue the following command in the MySQL server.

    `select * from SalesRecords;`
    
    The table is displayed as follows, indicating that the event you generated is added as a record.
    
    ![Updated MySQL Table](../images/loading-and-writing-data/updated-mysql-table.png)
    
### Publishing data on demand via store queries

To understand how to publish data on demand, see [Processing Data - Aggregating Data](processing-data.md#aggregating-data)

### Supported databases

WSO2 Streaming supports the following database types via Siddhi extensions:

| **Database Type** | **Siddhi Extension**                                                                |
|-------------------|-------------------------------------------------------------------------------------|
| RDBMS             | [rdbms](https://siddhi-io.github.io/siddhi-store-rdbms/api/latest/#store)           |
| MongoDB           | [mongodb](https://siddhi-io.github.io/siddhi-store-mongodb/api/latest/#store)       |
| Redis             | [redis](https://siddhi-io.github.io/siddhi-store-redis/api/latest/#store)           |
| elasticsearch     | [elasticsearch](https://siddhi-io.github.io/siddhi-store-elasticsearch/api/latest/) |

## Writing data to files

WSO2 Streaming allows you to write data into files so that the data can be available in a static manner for further processing. You can write the data received from another source unchanged or after processing it. This is achieved by defining an output [stream](https://ei.docs.wso2.com/en/7.2.0/streaming-integrator/guides/loading-and-writing-date/) and then connecting a [sink](https://siddhi.io/en/v5.1/docs/query-guide/#sink) of the [file]()

![Loading Data to Databases](../images/loading-and-writing-data/load-data-to-file.png)

To understand this, consider the example of a lab with a sensor that reports the temperature at different times. These temperature readings need to be saved in a file for reference when carrying out further analysis.  

To address the above requirement via WSO2 Streaming Integrator, define an output stream and connect a file source to it as shown below.

```
@sink(type = 'file', 
    file.uri = "/users/temperature/temperature.csv",
	@map(type = 'passThrough'))
define stream TemperatureLogStream (timestamp long, temperature int);
```
Here, any event directed to the `TemperatureLogStream` is written into the `/users/temperature/temperature.csv` file.

### Try it out

To try out the above example by including the given output stream and the sink configuration in a complete Siddhi application, follow the steps below:

1. [Start and access WSO2 Streaming Integrator Tooling](../develop/streaming-integrator-studio-overview.md/#starting-streaming-integrator-tooling).

2. Open a new file and copy the following Siddhi Application to it.

    ```
    @App:name('LabTemperatureApp')
    
    define stream LabTemperatureStream (timestamp long, temperature int);
    
    @sink(type = 'file', 
        file.uri = "/users/temperature/temperature.csv",
    	@map(type = 'passThrough'))
    define stream TemperatureLogStream (timestamp long, temperature int);
    
    from LabTemperatureStream 
    select * 
    insert into TemperatureLogStream;
    ```
   This Siddhi application includes the file sink from the previous example.
   
   !!! tip
       If required, you can replace the value for the `file.uri` parameter to a preferred location in your machine.
   
3. To simulate an event to the `LabTemperatureApp` Siddhi application, click the **Event Simulator** icon in the side panel. 

    ![simulate event](../images/loading-and-writing-data/event-simulator-icon.png)

    Then simulate an event as follows:
    
    ![Simulate Single Event](../images/loading-and-writing-data/simulate-single-event-for-file.png)
    
    1. In the **Siddhi App Name** field, select **LabTemperatureApp**.
    
    2. In the **Stream Name** field, select **LabTemperatureStream**.
    
    3. In the **Attributes** section, enter values for the attributes as follows:    
    
        | **Attribute**   | **Value**       |
        |-----------------|-----------------|
        | **timestamp**   | `1603461542000` |
        | **temperature** | `27`            |
       
    4. Click **Start and Send**.
    
4. Open the `/users/temperature/temperature.csv` file. It contains a line as shown below.

    ![Updated File](../images/loading-and-writing-data/updated-file.png)

    This is the event that you simulated that has been written into the file by WSO2 Streaming Integrator.
    
## Storing data in Cloud storage