# UPDATE

This repo is for an outdated version of Zoho API.

# Zoho Reports PHP SDK

This is a PHP SDK that provides integration with [Zoho Reports](https://www.zoho.com/reports/) through their API.

## v0.1 - Features
- Import csv to Zoho Reports

## Require
The only requirement is PHP >=5.5.0

## Installing via Composer

The raccomended way is to install ZohoReports-SDK is through [Composer](https://getcomposer.org/).

Please refer to [Getting Started](https://getcomposer.org/doc/00-intro.md) on how to download and install Composer.

After you have downloaded/installed Composer, run
````
composer.phar require freemancontingent/zohoreports-sdk
````

or add it in your `composer.json`
````
{
    "require": {
        "freemancontingent/zohoreports-sdk": "dev-master"
    }
}
````

## Prerequisites

> You should have a valid Zoho login email address to use the API. If you do not have one, please Sign up into Zoho Reports and create a login.

- Email - the Zoho Account email address
- Authtoken  - [How to generate a Zoho Authtoken for Zoho Reports](https://zohoreportsapi.wiki.zoho.com/Prerequisites.html#browser-mode)
- Database - a database needs to be created in Zoho Reports


### How to

#### INIT

You must provide your Zoho email, your Database name (the database in Zoho Reports), and your Zoho Authtoken
````php
$zoho = new ZohoReports\ZohoReports('my-zoho-email', 'my-db', 'my-authtoken');
````

#### Import data to your Zoho Reports Database
The import is highly customizable, it supports almost all the different configurations provided by the Zoho Reports API.

In most of the cases this will suits your need.

````php
$res = $zoho->import('somefile.csv', 'My-Table');
````

The first parameter is the full path to your csv file, the second parameter is the table in your Zoho Reports Database where your data will be imported.

This will truncate `My-Table` in your Zoho Reports Database and import all the data from `somefile.csv`. If `My-Table` doesn't exists, it will be created.

You can pass parameters (an array of parameters) to the import function that will override the default settings.

````php
$res = $zoho->import('somefile.csv', 'My-Table', ['param1' => 'my-value', ...]);
````

#### Parameters

Param Name | Description | Values | Default
------------ | ------------- | ------------- | -------------
format [string] | Format of the file to be imported. | CSV, JSON | CSV
create [string] | This will create the table if doesn't exist (if 'true'). | 'true', 'false' | 'true'
type [string] | Define the import type | APPEND, TRUNCATEADD, UPDATEADD | TRUNCATEADD
dateFormat [string] | Format of the dates | | 'yyyy-MM-dd HH:mm:ss'
autoIdentify [string] | Specify if auto identify the CSV format | 'true', 'false' | 'true'
skip [integer] | The number of columns to skip from the top of the CSV file | | 0
onError [string] | Define the action to be taken in case there is an error during the import | 'ABORT', 'SKIPROW', 'SETCOLUMNEMPTY' | 'SETCOLUMNEMPTY'


#### Constraints

When defining your own parameters, in some case you need to provide more informations.

- If `type` is 'UPDATEADD', you must provide a `pk` param, that defines your primary key, or your matching column used for comparison
- If `format` is `CSV` and `autoIdentify` is `'false'`, you must provide:
    - `csvCommentChar`, the comment char used in your csv file
    - `csvDelimeter`, the values delimeter, that can be set to:
        - `0` for COMMA
        - `1` for TAB
        - `2` for SEMICOLON
        - `3` for SPACE
    - `csvQuoted`, the text qualifier, that can be set to:
        - `0` for none
        - `1` for SINGLE QUOTE
        - `2` for DOUBLE QUOTE


#### Exceptions
There are two type of exception:

- InvalidParamsException, thrown if the params you have passed to the import function are invalid - Eg. `$param['type'] = 'UPDATEADD'` and you haven't provided `$param['pk']`
- InvalidFileException, thrown if the file you have passed doesn't exist


## Examples

Include the `ZohoReports.php` file in your project, or if you prefer using an autoloader have a look at [test.php](https://github.com/freemancontingent/zohoreports-sdk/blob/master/tests/test.php)

### Simple import with default params, from csv file


````php
$zoho = new ZohoReports\ZohoReports('my-zoho-email', 'my-db', 'my-authtoken');

try {
  $res = $zoho->import('path/to/somefile.csv', 'My-Table');
} catch(Exception $e) {
  echo $e->getMessage();
  exit();
}
````

### Import type 'UPDATEADD'

````php
$zoho = new ZohoReports\ZohoReports('my-zoho-email', 'my-db', 'my-authtoken');

try {
  $res = $zoho->import('path/to/somefile.csv', 'My-Table', [
    'type' => 'UPDATEADD',
    'pk'   => 'id'
  ]);
} catch(Exception $e) {
  echo $e->getMessage();
  exit();
}
````

### Import multiple files

````php
$zoho = new ZohoReports\ZohoReports('my-zoho-email', 'my-db', 'my-authtoken');

$myFilesToImport = [
  'path/to/movies.csv' => 'movie_id',
  'path/to/actors.csv' => 'actor_id',
  'path/to/genres.csv' => 'genre_id'
];

foreach ($myFilesToImport as $path => $pk) {
  try {
    $res = $zoho->import($path, basename($path), [
      'type' => 'UPDATEADD',
      'pk'   => $pk
    ]);
  } catch(ZohoReport\InvalidFileException $e) {
    myLogger("File {$path} not imported. Exception: {$e->getMessage()}");
    continue;
  } catch(ZohoReport\InvalidParamsException $e) {
    myLogger("Wrong parameter for {$path}. {$e->getMessage()}");
    exit();
  }
}
````
