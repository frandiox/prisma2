# Error reference

## Error codes

Error codes make identification/classification of error easier. Moreover, we can have internal range for different system components

| Tool (Binary)        | Range | Description                                                                                                                                                                      |
| -------------------- | ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Common               | 1000  | Common errors across all binaries. Common by itself is not a binary.                                                                                                             |
| Query Engine         | 2000  | Query engine binary is responsible for getting data from data sources via connectors (This powers Photon). The errors in this range would usually be data constraint errors      |
| Migration Engine     | 3000  | Migration engine binary is responsible for performing database migrations (This powers lift). The errors in this range would usually be schema migration/data destruction errors |
| Introspection Engine | 4000  | Introspection engine binary is responsible for printing Prisma schema from an existing database. The errors in this range would usually be errors with schema inferring          |
| Schema Parser        | 5000  | Schema parser is responsible for parsing the Prisma schema file. The errors in this range are usually syntactic or semantic errors.                                              |
| Prisma Format        | 6000  | Prisma format powers the Prisma VSCode extension to pretty print the Prisma schema. The errors in this range are usually syntactic or semantic errors.                           |


## Error templates

### Common

#### P1000: Incorrect database credentials

##### Info

This error occurs when any tool in the Prisma Framework is not able to access your database due to incorrect database credentials.

##### Error message

Authentication failed against database server at `${database_host}`, the provided database credentials for `${database_user}` are not valid. <br /> <br /> Please make sure to provide valid database credentials for the database server at `${database_host}`.

##### How to fix

Ensure that the database credentials you've provide are correct. The database credentials are provided in the `datasource` definition in your [Prisma schema]() (for most database connectors, they're part of the connection string that's provided as the `url` field).

#### P1001: Database not reachable

##### Info

This error occurs when any tool in the Prisma Framework is not able to access your database for unknown reasons.

##### Error message

Can't reach database server at `${database_host}`:`${database_port}` <br /> <br /> Please make sure your database server is running at `${host}:${port}`.

##### How to fix

Ensure that your database server is running and that you can reach it via the connection string that has been provided as the `url` argument on the `datasource` definition in your [Prisma schema](). 

#### P1002: Database timeout

##### Info

This error occurs when any tool in the Prisma Framework sends a request to your database but doesn't receive a response within the timeout period defined by your database.

##### Error message

 The database server at `${database_host}`:`${database_port}` was reached but timed out. <br /> <br /> Please try again. <br /> <br /> Please make sure your database server is running at `${host}:${port}`.

##### How to fix

Retry to see if this was only a temporary issue. If the problem persists, you can either increase the timeout period of your database or try to figure out why the request can't be processed within the given time period.

#### P1003: Database does not exist

##### Info

This error occurs when any tool in the Prisma Framework connects successfully to the database server but can not connect to the _database_ that was specified in the connection string. The connection string is provided via the `url` field of the `datasource` definition in your [Prisma schema](). The database is is the last component in the path of the URL, e.g. in the following connection string for a PostgreSQL database, the database is called `mydb`:

```
postgresql://janedoe:mypassword@localhost:5432/mydb?schema=public
```

> **Note**: Different consumers of the SDK might handle that differently. For example, Lift shows an interactive dialog when running `lift save`, so the user can potentially create the missing database.

##### Error message

_MySQL & PostgreSQL:_

Database `${database_name}` does not exist on the database server at `${database_host}`. <br /> <br /> 

_SQLite:_

Database `${database_file_name}` does not exist on the database server at `${database_file_path}`.

##### How to fix

Ensure that the database exists at the specified location. If it doesn't exist yet, you can create a new one with the specified name to solve the issue.

#### P1004: Incompatible binary

##### Info

This error occurs when the query or migration engine binary is not compatible with the platform it is being used on.

##### Error message

Failed to spawn the binary `${binary_path}` process for platform `${platform}`.

##### How to fix

Ensure the binary is compatible with the platform it is currently used on. Learn more about choosing the right binary for your platform [here](./core/generators/photonjs.md#specifying-the-right-platform-for-photonjs).

#### P1005: Unable to start the query engine

##### Info

This error occurs when the query or migration engine binary can not be started.

##### Error message

Failed to spawn the binary `${binary_path}` process for platform `${platform}`.

##### How to fix

Ensure the binary is compatible with the platform it is currently used on. Learn more about choosing the right binary for your platform [here](./core/generators/photonjs.md#specifying-the-right-platform-for-photonjs).

#### P1006: Binary not found

##### Info

This error occurs when a query engine binary for the current platform/operating system isn't available to Photon.

##### Error message

Photon binary for current platform `${platform}` could not be found. Make sure to adjust the generator configuration in the `schema.prisma` file. <br /> <br />`${generator_config}` <br /> <br />Please run `prisma2 generate` for your changes to take effect.

> **Note**: Tools (like Prisma CLI) consuming `generator_config` might color it using ANSI characters for better reading experience.

##### How to fix

Learn more about choosing the right binary for your platform [here](./core/generators/photonjs.md#specifying-the-right-platform-for-photonjs). After you've made changes the generator definition in your Prisma schema file, be sure to run `prisma2 generate` to apply them.

#### P1007: Missing write access to download binary

##### Info

This error occurs when the Prisma Framework CLI tries to download the query engine binary (which happens e.g. when you run `prisma2 generate`) but doesn't have the right permissions on your operating system to actually do so. 

##### Error message

Please try installing Prisma 2 CLI again with the `--unsafe-perm` option. <br /> Example: `npm i -g --unsafe-perm prisma2`.

##### How to fix

You can fix this issue by ensuring the Prisma Framework CLI has the right permissions to download the query engine binary, e.g. by using the `--unsafe-perm` option from npm:

```
npm install -g --unsafe-perm prisma2
```

#### P1008: Database operation timeout

##### Info

This error occurs when any tool in the Prisma Framework sends a request to your database but doesn't receive a response within the timeout period defined by your database.

##### Error message

Operations timed out after `${time}`.

##### How to fix

Retry to see if this was only a temporary issue. If the problem persists, you can either increase the timeout period of your database or try to figure out why the request can't be processed within the given time period.

#### P1009: Database already exists

##### Info

This error occurs when you're trying to create a database with a name that already exists on the database server you're targeting.

##### Error message

Database `${database_name}` already exists on the database server at `${database_host}:${database_port}`

##### How to fix

You can either rename the database that already exists to something else or decide to name the database you're about to create differently.

#### P1010: Database access denied

##### Info

##### Error message

User `${database_user}` was denied access on the database `${database_name}`

##### How to fix


### Query Engine

Note: Errors with `*` in the title represent multiple types and are less defined.

#### P2000: Input value too long

##### Info

##### Error message

The value `${field_value}` for the field `${field_name}` on the is too long for the field's type

##### How to fix


#### P2001: Record not found

##### Info

##### Error message

 The record searched for in the where condition (`${model_name}.${argument_name} = ${argument_value}`) does not exist

##### How to fix

#### P2002: Unique key violation

##### Info

##### Error message

Unique constraint failed on the field: `${field_name}`

##### How to fix

#### P2003: Foreign key violation

##### Info

##### Error message

Foreign key constraint failed on the field: `${field_name}`

##### How to fix

#### P2004: Constraint violation

##### Info

##### Error message

A constraint failed on the database: `${database_error}`

##### How to fix


#### P2005: Stored value is invalid

##### Info

##### Error message

The value `${field_value}` stored in the database for the field `${field_name}` is invalid for the field's type

##### How to fix

#### P2006: `*`Type mismatch: invalid (ID/Date/Json/Enum)

##### Info

##### Error message

The provided value `${field_value}` for `${model_name}` field `${field_name}` is not valid

##### How to fix

#### P2007: `*`Type mismatch: invalid custom type

##### Info

##### Error message

 Data validation error `${database_error}`

##### How to fix

#### P2008: Query parsing failed

##### Info

##### Error message

Failed to parse the query `${query_parsing_error}` at `${query_position}`

##### How to fix

#### P2009: Query validation failed

##### Info

##### Error message

Failed to validate the query `${query_validation_error}` at `${query_position}`

##### How to fix

### Migration Engine

#### P3000: Database creation failed

##### Info

##### Error message

Failed to create database: `${database_error}`

##### How to fix

#### P3001: Destructive migration detected

##### Info

##### Error message

Migration possible with destructive changes and possible data loss: `${migration_engine_destructive_details}`

##### How to fix

#### P3002: Migration rollback

##### Info

##### Error message

 The attempted migration was rolled back: `${database_error}`

##### How to fix

### Introspection

#### P4000: Introspection failed

##### Info

##### Error message

Introspection operation failed to produce a schema file: `${introspection_error}`.

##### How to fix

### Schema Parser

#### P5000: Schema parsing failed

##### Info

##### Error message

Failed to parse schema file: `${schema_parsing_error}` at `${schema_position}`

##### How to fix

#### P5001: Schema relational ambiguity

##### Info

##### Error message

There is a relational ambiguity in the schema file between the models `${A}` and `${B}`.

##### How to fix

#### P5002: Schema string input validation errors

##### Info

##### Error message

Database URL provided in the schema failed to parse: `${schema_sub_parsing_error}` at `${schema_position}`

##### How to fix

## Photon.js

#### Photon runtime validation error

##### Info

##### Error message

Validation Error: `${photon_runtime_error}`

##### How to fix

#### Query engine connection error

##### Info

##### Error message

The query engine process died, please restart the application

##### How to fix
