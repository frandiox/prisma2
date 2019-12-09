# Using only Photon.js (without Lift)

You can use Photon.js as a database client in your application without using Lift for database migrations. This is useful for _existing applications_ when you want to run database migrations manually, when there already is a working migration system or when you don't have the rights inside your organization to perform database migrations yourself.

When using Photon.js without Lift, you obtain your data model definition by [_introspecting_](../introspection.md) your database schema and generating the Prisma [data model](../data-modeling.md) from it. The generated data model then serves as foundation for Photon.js's generated data access [API](./api.md). Whenever a schema migration is performed on the database later in the lifetime of your project, you need to re-introspect your database (which updates your data model) and re-generate your Photon.js API.

## Getting started with Photon.js

### 1. Set up project using `prisma2 init`

Run the following command to initialize a new project:

```
npx prisma2 init hello-world
```

Then follow the interactive prompt:

1. Select **Blank project**
1. Select your database type
    - **SQLite**
    - **MySQL**
    - **PostgreSQL**
    - MongoDB (coming soon)
1. Provide your database credentials ([more info](#database-credentials))
1. Select the database (MySQL) or schema (PostgreSQL) to introspect
1. Select **Only Photon** (i.e. uncheck Lift using <kbd>SPACE</kbd>)
1. Select your programming language
    - **JavaScript**
    - **TypeScript**
    - Go (coming soon)
1. Select a [**Demo script**](#demo-scripts) or start with **Just the Prisma schema**

Once you're done with the interactive prompt, the CLI sets out for 3 major tasks:

1. Introspecting your database schema
1. Generating a Prisma schema for your database based on the introspection
1. Generating the Photon.js API

### 2. Integrate Photon.js in your application

To start using Photon.js in your application, you first need to install it as an npm dependecy:

```
npm install @prisma/photon
```

It is recommended to also install the `prisma2` CLI as a development dependency:

```
npm install prisma2 --save-dev
```

Now you can import it from `node_modules/@prisma/photon` and start calling your database via the [generated Photon API](./api.md).

### 3. Customize your generated Photon.js API

One benefit of having the data model as an intermediate representation of your database schema is that lets you to _decouple_ the database schema from your data access API. For example, you can map cryptic table names to friendlier model names to be used in your API.

For example, when the following model was generated for you through the introspection:

```groovy
model _customers {
  id Int @id
  number_of_orders Int
}
```

By default, the generated API is based the model and fields names, e.g.:

```ts
await photon._customers.findMany({
  where: { number_of_orders: 5 }
})
```

You might prefer using camel casing rather than the snake case convention used in the database. You can therefore customize the mapping of a table/field name to a specific model/field name in the data model using the `map` attribute:

```groovy
model Customer @@map(name: "_customers") {
  id Int @id
  orderCount Int @map(name: "number_of_orders")
}
```

After running another `prisma2 generate`, your Photon API now looks as follows:

```ts
await photon.customers.findMany({
  where: { orderCount: 5 }
})
```

### 4. Evolve your application

Whenever the database schema changes throughout the lifetime of your application, you need to re-generate your Photon.js API to ensure it still matches the underlying database structures. The workflow for that typically involves two steps:

1. Re-introspecting your database schema to update the data model
1. Re-generate your Photon API

In CLI commands, this looks as follows:

```
npx prisma2 introspect
npx prisma2 generate
```

## Database credentials

<Details><Summary>Database credentials for <strong>SQLite</strong></Summary>
<br />
When using SQLite, you need to provide the _file path_ to your existing SQLite database file.

</Details>

<Details><Summary>Database credentials for <strong>MySQL</strong></Summary>
<br />
When using MySQL, you need to provide the following information to connect your existing MySQL database server:

- **Host**: The IP address/domain of your database server, e.g. `localhost`.
- **Post**: The port on which your database server listens, e.g. `5432` (PostgreSQL) or `3306` (MySQL).
- **User**: The database user, e.g. `admin`.
- **Password**: The password for the database user.
- **SSL**: Whether or not your database server uses SSL.

Once provided, the CLI will prompt you to select one of the existing **databases** on your MySQL server for introspection.

</Details>

<Details><Summary>Database credentials for <strong>PostgreSQL</strong></Summary>
<br />
When using PostgreSQL, you need to provide the following information to connect your existing PostgreSQL database server:

- **Host**: The IP address/domain of your database server, e.g. `localhost`.
- **Post**: The port on which your database server listens, e.g. `5432` (PostgreSQL) or `3306` (MySQL).
- **Database**: The name of the database which contains the schema to introspect. 
- **User**: The database user, e.g. `admin`.
- **Password**: The password for the database user.
- **SSL**: Whether or not your database server uses SSL.

Once provided, the CLI will prompt you to select one of the existing **schemas** on your PostgreSQL server for introspection.

</Details>


### Demo scripts

When you're selecting the **Demo script** option, the Prisma Framework CLI will provide a runnable Node.js/TypeScript script that showcases usage of the Photon API and gives you a foundation for further exploration.