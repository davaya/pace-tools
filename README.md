# PACE, OSQuery, and JADN

The DoD-sponsored Posture Attribute Collection and Evaluation (PACE) prototype uses
OSQuery and nmap to collect device attributes which are then stored in Elasticsearch.
OSQuery defines a collection of tables that constitute "a relational data-model to describe a device".
Elasticsearch is a document-oriented database:
> "Elasticsearch does not require you to specify a schema upfront.
> Throw a JSON-document at it, and it will do some educated guessing to infer its type.
> 
> Most relational databases also let you specify constraints to define what is and isn't consistent.
> For example, referential integrity and uniqueness can be enforced.
> Document oriented databases tend not to do this, and Elasticsearch is no different."

Feeding JSON data to an Elasticsearch database results in a guessed schema that supports
searching that data.
If the data is provided by OSQuery applications it should represent the OSQuery schema,
but it does not necessarily provide complete coverage of the schema and
it doesn't preserve datatype constraints or relationships defined by OSQuery.
For example, the account_policy_data "uid" property would be recognized by Elasticsearch
as a JSON number, but the fact that that number references information about a specific
user is lost.

In addition, data ingested by Elastic that does not originate from OSQuery,
such as nmap-derived data, does not necessarily reflect any defined schema.
Elastic will ingest any correct or incorrect data as provided and derive a schema
that accepts that data as correct.

JADN is used to create an asset repository schema separately from the tools used in the
PACE prototype, validate JSON data used by those tools, and define PACE-related OpenC2
actuator profiles.

## Example
An OSQuery `.table` file looks like:
```
table_name("account_policy_data")
description("Additional OS X user account data from the AccountPolicy section of OpenDirectory.")
schema([
    Column("uid", BIGINT, "User ID"),
    Column("creation_time", DOUBLE, "When the account was first created"),
    Column("failed_login_count", BIGINT, "The number of failed login attempts using an incorrect password. Count resets after a correct password is entered."),
    Column("failed_login_timestamp", DOUBLE, "The time of the last failed login attempt. Resets after a correct password is entered"),
    Column("password_last_set_time", DOUBLE, "The time the password was last changed"),
    ForeignKey(column="uid", table="users"),
])
implementation("account_policy_data@genAccountPolicyData")
examples([
  "select * from users join account_policy_data using (uid)",
])
```
JADN type definitions can be created directly from `.table` files. These are all `Record`
types because OSQuery is based on tables:
```
AccountPolicyData = Record                // Additional OS X user account data from the AccountPolicy section of OpenDirectory.
   1 uid                     Link(Users)  // User ID
   2 creation_time           Number       // When the account was first created
   3 failed_login_count      Integer      // The number of failed login attempts using an incorrect password. Count resets after a correct password is entered.
   4 failed_login_timestamp  Number       // The time of the last failed login attempt. Resets after a correct password is entered
   5 password_last_set_time  Number       // The time the password was last changed
```
## Usage
1. Install JADN in a Python environment (version 3.8 or later): `pip install jadn`
2. Run `make-artifacts.py`
3. Examine the `Out` directory for files generated from the example schema:
    1. x.jidl - formatted version of input
    2. x.jadn - native JADN (JSON format)
    3. x.md - markdown tables used in actuator profiles
    4. x.puml - PlantUML diagram - view at http://www.plantuml.com/plantuml/uml/
    5. x.json - JSON Schema (since the example does not have a containing structure like an OpenC2 command, the schema will accept any of the defined JSON objects without distinguishing among them.)

## Next Steps
Translate additional OSQuery tables by hand, write script to translate them all,
write JADN types for nmap-discovered data