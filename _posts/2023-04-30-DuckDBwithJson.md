# Playing with JSON in DuckDB

With all the hype around DuckDB nowadays, just felt like exploring it for fun. This will not be a one off post, I plan to convert this learning with some meaningful exercise.

I have chosen to do some analysis with Indian Mutual Fund data.

I happen to stumble upon [this](https://www.mfapi.in/) free API which claims to cover all the fund houses, fund types and provides NAV data till date since inception along with meta information about the fund.

I downloaded the master list of all the funds from [here](https://api.mfapi.in/mf).

While this is such a simple JSON array of objects (schemeName, schemeCode) atleast is a good starting point to experiment with DuckDB.

I [installed](https://duckdb.org/docs/installation/index) command line utility of DuckDB to begin with.

By the way they have a good [documentation](https://duckdb.org/docs/) and plenty of videos and articles are available online for one to get started.

## Reading JSON
select * from read_json_objects('path_to_file');

I ran into issue and message that I got on the CLI was to increase maximum_object_size. Default it seems to be 1048576 bytes. My JSON file on disk was 4.5MB but somehow setting maximum_object_size to 2326500 seems to have worked. I am still wrapping my head around this.

Tried different commands such as read_json_auto with different parameters columns, auto_detect etc.

Simply reading json doesn't create a permanent table, it is transient in nature.

## Creating a permanent table
create table <table_name> as select * from read_json_auto(....) --> creates a table in the catalog so that you can start querying table than everytime reading json.

## Normal SQL commands against table created from JSON

select * from mf where schemeName like `%Aditya%`; --> for filtering schemeName that has Aditya (Fund House = Aditya Birla) in it.

select * from mf where schemeName like `%Index%`; --> for filtering schemeName that are of index fund type.

select count(*) from mf; -- total row count

select distinct schemeName from mf; --> finding distinct schemename revealed that there are scehmename that has multiple codes

select distinct schemeName, count(schemeCode) as c from mf group by schemeName order by c desc; --> to find out which scheme has more than one code

select * from mf where schemeName = 'JM Large Cap Fund-Growth'; --> for example

likewise one can find out fund type growth or direct, ELSS or index etc.

## Information Schema

select * from information_schema.tables; --> gives table created in catalog

## Exporting table to csv

copy mf to 'mf.csv' (HEADER); --> exports the data into CSV with HEADER information and stores at the location where DuckDB CLI client resides


That is for today. Hope to play around more and share more learning.






