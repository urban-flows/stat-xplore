# DWP Harvester

This is a data harvester for the Department for Work and Pensions (DWP) [Stat-Explore](https://stat-xplore.dwp.gov.uk/) system. See [Stat-Xplore : Open Data API](https://stat-xplore.dwp.gov.uk/webapi/online-help/Open-Data-API.html) documentation. This API is based on the [SuperSTAR 9.5 Open Data API](https://docs.wingarc.com.au/superstar95/9.5/open-data-api) by [WingArc1st](https://wingarc.com.au/).

# Installation

## Authentication

The harvester authenticates against the remote server using an access token.

To use the harvester you must register an account on [Stat-Xplore](https://stat-xplore.dwp.gov.uk). When you're logged in, click the menu button (the three dots in the top-right corner) and click "Account". The string under "Open Data API Access" which is your API access key/token. 

This token must be input to the harvester either in the command line or from a file (the default path is `~/configs/stat_explore.txt`) as shown below:

```bash
python . --api_key <my_access_token>
# or
python . --api_key_path ~/configs/stat_explore.txt
```

# Usage

To view the available commands and options, run the following command:

```bash
$ python . --help
```

## Querying

To get a query specification in JSON format, visit [Stat-Xplore](https://stat-xplore.dwp.gov.uk), log in and select a data set. Choose a table and click "Open Table." You may customise the rows and columns as needed. Next, click the "Download Table" field and select "Open Data API Query (.json)" This JSON file can be used to define a query as shown below.

Run the following command to execute a query and output the result to a CSV file:

```bash
$ python . -o test.csv -q queries\relative-low-income-by-year-sheffield.json
```

Where `-o` (`--output`) is the output CSV file path, `-q` (`--query`) is the query JSON file.

To generate the CSV headers that will result from a particular query, use the `--csv` (`-c`) flag:

```bash
python . -o test.csv -q queries\relative-low-income-by-year-sheffield.json -c
```

# Code documentation

An authenticated HTTP session is required to communicate with the API.

```python
import http_session

session = http_session.StatSession(api_key='<access_token>')
```

## API objects

The subclasses of `objects.StatObject` are thin wrappers around the API endpoints. Please refer to the [API documentation](https://stat-xplore.dwp.gov.uk/webapi/online-help/Open-Data-API.html).

### Schema

The [/schema endpoint](/schema endpoint) returns information about the Stat-Xplore datasets that are available to you, and their fields and measures.

The root endpoint, `/schema`, returns details of all datasets and folders at the root level of Stat-Xplore.

```python
import objects

# List all data schemas
objects.Schema.list(session)
# Get info about a schema
objects.Schema('str:folder:fuc').get(session)
# Get the schema of a specific table
objects.Schema('str:database:UC_Monthly').get(session)
```

### Table examples

The `/table` [endpoint](https://stat-xplore.dwp.gov.uk/webapi/online-help/Open-Data-API-Table.html) allows you to submit table queries and receive the results. The body of the request contains your query.

```python
# Retrieve the number of people on Universal Credit broken down by month
objects.Table('str:database:UC_Monthly').run_query(session,
    measures=['str:count:UC_Monthly:V_F_UC_CASELOAD_FULL'],
    dimensions=[['str:field:UC_Monthly:F_UC_DATE:DATE_NAME']],
)
```

It's also possible to use JSON to define a query. This is useful for replicating queries generated by the Stat-Xplore graphical user interface. (In Table View, go to Download Table and select "Open Data API Query (.json)" then click Go.)

```python
query = """{
  "database" : "str:database:DLA_In_Payment_New",
  "measures" : [
    "str:count:DLA_In_Payment_New:V_F_DLA_In_Payment_New",  
    "str:statfn:DLA_In_Payment_New:V_F_DLA_In_Payment_New:CAWKLYAMT:MEAN" ],
  "dimensions" : [
    [ "str:field:DLA_In_Payment_New:V_F_DLA_In_Payment_New:COA_CODE" ],
    [ "str:field:DLA_In_Payment_New:F_DLA_QTR_New:DATE_NAME" ]
  ]
}"""
data = objects.Table.query_json(session, query)
```

# Resources

* Stack Overflow [How to easily generate the query JSON for the Stat-Xplore API](https://stackoverflow.com/a/65341265)