---
title: Bulk Importing Data
---

### Overview

You can use the Sky API to add individual events but if you have your events in a flat file such as a CSV or tab-delimited format, you can use the Sky importer to make the process easier.
The Sky importer allows you to import using a predefined translation format or you can can define your own as described below.

File types are automatically detected by extension.
Tab-delimited formats end in `.tsv` and `.txt`.
All other extensions default to CSV format.


### Using the Importer

The standard Sky import format recognizes `object_id` and `timestamp` as standard header columns.
It will also automatically associate any header column that matches one of the formats:

* `data.<property_name>:<data_type>`
* `action.<property_name>:<data_type>`

The default data type is `string` but you can also use `int`, `float` & `boolean`.
The timestamp field is parsed using the [Chronic](https://github.com/mojombo/chronic) gem so you can generally format your dates how you choose.

Here's an example CSV data import file:

```text
object_id,timestamp,action.name,action.purchase_amount:float,data.gender,data.age:int,data.rewards_member:boolean
100,2012-01-01T00:00:00Z,/index.html,0,male,35,false
100,2012-01-01T00:01:00Z,/signup.html,9.95,male,35,true
100,2012-01-01T00:02:00Z,/buy.html,9.95,male,35,true
101,2012-02-01T07:35:00Z,/buy.html,100.00,female,23,false
```

When you run the loader, you can specify the table name with the command line argument (`--table`) or the loader will ask you when you run.
If the table already exist, the loader will ask if you wish to append the data to the table or overwrite the existing table.
You can specify `--append` and `--overwrite` in the command line as well.

Your import command should look something like this:

```bash
$ sky import --table my_tbl --format sky mydata.csv
```


### The Format File

There are two types of formats in Sky.
First there are named formats.
These are format files built into the gem.
You can find their definitions in the `lib/skydb/import/transforms` folder of the gem.
Currently there is just the standard `sky` format.

The second is a format file.
If you pass a path into the `--format` argument then you can specify a YAML file that describes the mapping from each row of the input file to it's corresponding Sky event.

The format file has three parts: `fields`, `require` & `translate`.
The `fields` section maps an input column (and data type) on the right to an output field on the right.
You can also specify inline ruby code to convert an input field to an output field by enclosing it in curly braces.

The following format file converts a CSV file with headers `Path`, `UserID`, `Timestamp` and `FollowerCount` to a Sky event:

```yaml
fields:
  object_id: UserID:int
  timestamp: Timestamp:date
  action:
    name: Path
    follower_count: "{ input['FollowerCount'].to_i  }"
```

The `translate` section provides you with a free form area to convert input to output:

```yaml
translate: |
  if input['UserID'] != ''
    output['object_id'] = input['UserID'].to_i
  else
    output['object_id'] = input['AlthernateUserIDColumn'].to_i
  end
```

And finally, if you have special libraries you need for your conversion, you can list them in the `require` section like this:

```yaml
require: [uri, active_support]
```

If you write a file format for a common data format, please consider sending the Sky project a pull request on GitHub!


<br/>
<a href="query.html">« Prev <span class="hidden-phone">(Behavioral Query Language)</span></a>
