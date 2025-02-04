# Creating Tables in Athena<a name="creating-tables"></a>

You can run DDL statements in the Athena console, using a JDBC or an ODBC driver, or using the Athena [**Add table** wizard](#to-create-a-table-using-the-wizard)\.

When you create a new table schema in Athena, Athena stores the schema in a data catalog and uses it when you run queries\.

Athena uses an approach known as *schema\-on\-read*, which means a schema is projected on to your data at the time you run a query\. This eliminates the need for data loading or transformation\.

 Athena does not modify your data in Amazon S3\.

Athena uses Apache Hive to define tables and create databases, which are essentially a logical namespace of tables\. 

When you create a database and table in Athena, you are simply describing the schema and the location where the table data are located in Amazon S3 for read\-time querying\. Database and table, therefore, have a slightly different meaning than they do for traditional relational database systems because the data isn't stored along with the schema definition for the database and table\. 

When you query, you query the table using standard SQL and the data is read at that time\. You can find guidance for how to create databases and tables using [Apache Hive documentation](https://cwiki.apache.org/confluence/display/Hive/LanguageManual+DDL), but the following provides guidance specifically for Athena\.

The maximum query string length is 256 KB\.

Hive supports multiple data formats through the use of serializer\-deserializer \(SerDe\) libraries\. You can also define complex schemas using regular expressions\. For a list of supported SerDe libraries, see [Supported SerDes and Data Formats](supported-serdes.md)\.

## Considerations and Limitations<a name="creating-tables-considerations-and-limitations"></a>

Following are some important limitations and considerations for tables in Athena\.

### Requirements for Tables in Athena and Data in Amazon S3<a name="s3-considerations"></a>

When you create a table, you specify an Amazon S3 bucket location for the underlying data using the `LOCATION` clause\. Consider the following:
+ Athena can only query the latest version of data on a versioned Amazon S3 bucket, and cannot query previous versions of the data\.
+ You must have the appropriate permissions to work with data in the Amazon S3 location\. For more information, see [Access to Amazon S3](s3-permissions.md)\.
+ Athena supports querying objects that are stored with multiple storage classes in the same bucket specified by the `LOCATION` clause\. For example, you can query data in objects that are stored in different Storage classes \(Standard, Standard\-IA and Intelligent\-Tiering\) in Amazon S3\.
+  Athena supports [Requester Pays Buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/RequesterPaysBuckets.html)\. For information how to enable Requester Pays for buckets with source data you intend to query in Athena, see [Creating a Workgroup](workgroups-create-update-delete.md#creating-workgroups)\.
+ Athena does not support querying the data in the [S3 Glacier](https://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html#sc-glacier) or S3 Glacier Deep Archive storage classes\. Objects in the S3 Glacier storage class are ignored\. Objects in the S3 Glacier Deep Archive storage class that are queried result in the error message The operation is not valid for the object's storage class\. Data that is moved or transitioned to one of these classes are no longer readable or queryable by Athena even after storage class objects are restored\. To make the restored objects that you want to query readable by Athena, copy the restored objects back into Amazon S3 to change their storage class\.

  For information about storage classes, see [Storage Classes](https://docs.aws.amazon.com/AmazonS3/latest/dev/storage-class-intro.html), [Changing the Storage Class of an Object in Amazon S3](https://docs.aws.amazon.com/AmazonS3/latest/dev/ChgStoClsOfObj.html), [Transitioning to the GLACIER Storage Class \(Object Archival\) ](https://docs.aws.amazon.com/AmazonS3/latest/dev/lifecycle-transition-general-considerations.html#before-deciding-to-archive-objects), and [Requester Pays Buckets](https://docs.aws.amazon.com/AmazonS3/latest/dev/RequesterPaysBuckets.html) in the *Amazon Simple Storage Service Developer Guide*\.
+ If you issue queries against Amazon S3 buckets with a large number of objects and the data is not partitioned, such queries may affect the Get request rate limits in Amazon S3 and lead to Amazon S3 exceptions\. To prevent errors, partition your data\. Additionally, consider tuning your Amazon S3 request rates\. For more information, see [Request Rate and Performance Considerations](https://docs.aws.amazon.com/AmazonS3/latest/dev/request-rate-perf-considerations.html)\.

### Functions Supported<a name="hive-ddl-functions-supported"></a>

The functions supported in Athena queries are those found within Presto\. For more information, see the documentation for Presto versions [0\.172](https://prestodb.io/docs/0.172/functions.html) and [0\.217](https://prestodb.io/docs/0.217/functions.html), which correspond to Athena engine versions 1 and 2\.

### Transactional Data Transformations Are Not Supported<a name="transactional-data-transformations-are-not-supported"></a>

Athena does not support transaction\-based operations \(such as the ones found in Hive or Presto\) on table data\. For a full list of keywords not supported, see [Unsupported DDL](unsupported-ddl.md)\.

### Operations That Change Table States Are ACID<a name="operations-that-change-table-states-are-acid"></a>

When you create, update, or delete tables, those operations are guaranteed ACID\-compliant\. For example, if multiple users or clients attempt to create or alter an existing table at the same time, only one will be successful\.

### All Tables Are EXTERNAL<a name="all-tables-are-external"></a>

If you use `CREATE TABLE` without the `EXTERNAL` keyword, Athena issues an error; only tables with the `EXTERNAL` keyword can be created\. We recommend that you always use the `EXTERNAL` keyword\. When you drop a table in Athena, only the table metadata is removed; the data remains in Amazon S3\.

## Creating Tables Using AWS Glue or the Athena Console<a name="creating-tables-how-to"></a>

You can create tables in Athena by using AWS Glue, the add table wizard, or by running a DDL statement in the Athena Query Editor\.

### To create a table using the AWS Glue Data Catalog<a name="to-create-a-table-using-the-aws-glue-data-catalog"></a>

1. Open the Athena console at [https://console\.aws\.amazon\.com/athena/](https://console.aws.amazon.com/athena/home)\.

1. In the Query Editor, under **Database**, choose **Create table**, and then choose **from AWS Glue crawler**\.   
![\[Choosing to create a table by using AWS Glue.\]](http://docs.aws.amazon.com/athena/latest/ug/images/create-table-choices-glue.png)

1. In the **Go to AWS Glue to set up a crawler** dialog box, choose **Continue**\.  
![\[Choose Continue to go to the AWS Glue console and set up a crawler.\]](http://docs.aws.amazon.com/athena/latest/ug/images/go-to-glue-crawler.png)

1. Follow the steps in the AWS Glue console to add a crawler\. 

   For more information, see [Using AWS Glue Crawlers](glue-best-practices.md#schema-crawlers)\.

### To create a table using the Athena add table wizard<a name="to-create-a-table-using-the-wizard"></a>

1. Open the Athena console at [https://console\.aws\.amazon\.com/athena/](https://console.aws.amazon.com/athena/home)\.

1. Under the database display in the Query Editor, choose **Create table**, and then choose **from S3 bucket data\.**  
![\[Choosing to create a table from S3 bucket data.\]](http://docs.aws.amazon.com/athena/latest/ug/images/create-table-choices-s3.png)

1. in the **Add table** wizard, follow the steps to create your table\.  
![\[Follow the steps in the Add table wizard to create a table in Athena from Amazon S3 data.\]](http://docs.aws.amazon.com/athena/latest/ug/images/add-table-wizard.png)

### To create a table using Hive DDL<a name="to-create-a-table-using-hive-ddl"></a>

1. From the **Database** menu, choose the database for which you want to create a table\. If you don't specify a database in your `CREATE TABLE` statement, the table is created in the database that is currently selected in the Query Editor\.  
![\[Choosing a database in the Athena Query Editor.\]](http://docs.aws.amazon.com/athena/latest/ug/images/catalogdashboard.png)

1. Enter a statement like the following, and then choose **Run Query**, or press **Ctrl\+ENTER**\.

   ```
   CREATE EXTERNAL TABLE IF NOT EXISTS cloudfront_logs (
       `Date` Date,
       Time STRING,
       Location STRING,
       Bytes INT,
       RequestIP STRING,
       Method STRING,
       Host STRING,
       Uri STRING,
       Status INT,
       Referrer STRING,
       OS String,
       Browser String,
       BrowserVersion String
   ) ROW FORMAT SERDE 'org.apache.hadoop.hive.serde2.RegexSerDe'
   WITH SERDEPROPERTIES (
   "input.regex" = "^(?!#)([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+([^ ]+)\\s+[^\(]+[\(]([^\;]+).*\%20([^\/]+)[\/](.*)$"
   ) LOCATION 's3://athena-examples-MyRegion/cloudfront/plaintext/';
   ```

   After the table is created, you can run queries against your data\.

### Sample Database and Table<a name="creating-tables-sampledb-and-table"></a>

In the old Athena console experience, Athena creates a sample database and table for you when you use Athena in a Region for the first time\. The `sampledb` database and `elb_logs` table are created from sample data in the Athena examples folder for the Region \(for example, `s3://athena-examples-eu-west-1/elb/plaintext/`\)\. The sample database and table support the interactive tutorial available from the menu of the old console\. 

![\[Sample database and table\]](http://docs.aws.amazon.com/athena/latest/ug/images/old-console-sampledb-table-tutorial.png)

## Showing Table Information<a name="creating-tables-showing-table-information"></a>

 After you have created a table in Athena, you can choose the vertical three dots next to the table name in the Athena console to show information about the table and manage it\. 

![\[Choose the vertical three dots next to the table name.\]](http://docs.aws.amazon.com/athena/latest/ug/images/creating-tables-kebab-menu.png)
+ **Preview table** – Shows the first 10 rows of all columns by running the `SELECT * FROM "database_name"."table_name" LIMIT 10` statement in the Athena Query Editor\.
+ **Show properties** – Displays table properties and, if the table has partitions, partition information on a second tab\.
  + **Table properties** – Shows the table name, database name, time created, and whether the table has encrypted data\.
  + **Partitions** – Lists the names of the table's partitions and their corresponding data types\. This tab displays only if the table has partitions\. 
+ **Delete table** – Displays a confirmation dialog box asking if you want to delete the table\. If you agree, runs the `DROP TABLE table_name` statement in the Athena Query Editor\.
+ **Load partitions** – Runs the `MSCK REPAIR TABLE table_name` statement in the Athena Query Editor\. This option is available only if the table has partitions\. 
+ **Generate Create Table DDL** – Generates a DDL statement that you can use to re\-create the table by running the `SHOW CREATE TABLE` *table\_name* statement in the Athena Query Editor\.
**Note**  
If you use the AWS Glue [CreateTable](https://docs.aws.amazon.com/glue/latest/webapi/API_CreateTable.html) API operation or the AWS CloudFormation [https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-glue-table.html](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-resource-glue-table.html) template to create a table for use in Athena without specifying the `TableType` property and then run a DDL query like `SHOW CREATE TABLE` or `MSCK REPAIR TABLE`, you can receive the error message FAILED: NullPointerException Name is null\.   
To resolve the error, specify a value for the [TableInput](https://docs.aws.amazon.com/glue/latest/webapi/API_TableInput.html) `TableType` attribute as part of the AWS Glue `CreateTable` API call or [AWS CloudFormation template](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-glue-table-tableinput.html)\. Possible values for `TableType` include `EXTERNAL_TABLE` or `VIRTUAL_VIEW`\.  
This requirement applies only when you create a table using the AWS Glue `CreateTable` API operation or the `AWS::Glue::Table` template\. If you create a table for Athena by using a DDL statement or an AWS Glue crawler, the `TableType` property is defined for you automatically\. 