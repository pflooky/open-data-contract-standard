---
title: "Definition: Open Data Contract Standard (ODCS)"
description: "Details of the Open Data Contract Standard (ODCS). Includes fundamentals, datasets, schemas, data quality, pricing, stakeholders, roles, service-level agreements and other properties."
image: "https://raw.githubusercontent.com/bitol-io/artwork/main/horizontal/color/Bitol_Logo_color.svg"
---

# Open Data Contract Standard

## Executive Summary
This document describes the keys and values expected in a YAML data contract, per the **Open Data Contract Standard**. It is divided in multiple sections: [fundamentals (fka demographics)](#fundamentals), [schema](#schema), [data quality](#data-quality), [pricing](#pricing), [team](#team), [roles](#roles), [service-level agreement](#sla), and [other/custom properties](#custom-properties). Each section starts with at least an example followed by definition of each field/key.


## Table of content

1. [Fundamentals (fka demographics)](#fundamentals)
1. [Schema](#schema)
1. [Data quality](#data-quality)
1. [Support & communication channels](#support)
1. [Pricing](#pricing)
1. [Team](#team)
1. [Roles](#roles)
1. [Service-level agreement](#sla)
1. [Infrastructures & servers](#servers)
1. [Custom & other properties](#custom-properties)
1. [Examples](#full-example)


## Notes

* This contract is containing example values, we reviewed very carefully the consistency of those values, but we cannot guarantee that there are no errors. If you spot one, please raise an [issue](https://github.com/AIDAUserGroup/open-data-contract-standard/issues).
* Some fields have `null` value: even if it is equivalent to not having the field in the contract, we wanted to have the field for illustration purpose.
* This contract should be **platform agnostic**. If you think it is not the case, please raise an [issue](https://github.com/AIDAUserGroup/open-data-contract-standard/issues).


## Fundamentals
This section contains general information about the contract.

### Example

```YAML
apiVersion: 3.0.0 # Standard version
kind: DataContract

id: 53581432-6c55-4ba2-a65f-72344a91553a
name: seller_payments_v1
version: 1.1.0 # Data Contract Version 
status: production
domain: seller
dataProduct: payments
tenant: ClimateQuantumInc

description:
  purpose: Views built on top of the seller tables.
  limitations: null
  usage: null

# Data Quality
quality: null # See more information below

# Tags
tags: null
```

### Definitions

| Key                     | UX label         | Required | Description                                                                              |
|-------------------------|------------------|----------|------------------------------------------------------------------------------------------|
| apiVersion              | Standard version | Yes      | Version of the standard used to build data contract. Default value is `v3.0.0`.          |
| kind                    | Kind             | Yes      | The kind of file this is. Valid value is `DataContract`.                                 |
| id                      | ID               | Yes      | A unique identifier used to reduce the risk of dataset name collisions, such as a UUID.  |
| name                    | Name             | No       | Name of the data contract.                                                               |
| version                 | Version          | Yes      | Current version of the data contract.                                                    |
| status                  | Status           | Yes      | Current status of the data contract.                                                     |
| tenant                  | Tenant           | No       | Indicates the property the data is primarily associated with. Value is case insensitive. |
| domain                  | Domain           | No       | Name of the logical data domain.                                                         |
| dataProduct             | Data Product     | No       | Name of the data product.                                                                |
| description             | Description      | No       | Object containing the descriptions.                                                      |
| description.purpose     | Purpose          | No       | Intended purpose for the provided data.                                                  |
| description.limitations | Limitations      | No       | Technical, compliance, and legal limitations for data use.                               |
| description.usage       | Usage            | No       | Recommended usage of the data.                                                           |


## <a id="schema"/> Schema 
This section describes the schema of the data contract. It is the support for data quality, which is detailed in the next section. Schema supports both a business representation of your data and a physical implementation. It allows to tie them together.

In ODCS v3, the schema has evolved from the table and column representation, therefore the schema introduces a new terminology:

* **Objects** are a structure of data: a table in a RDBMS system, a document in a NoSQL database, and so on.
* **Properties** are attributes of an object: a column in a table, a field in a payload, and so on.
* **Elements** are either an object or a property.

Figure 1 illustrates those terms with a basic relational database.

<img src=img/elements-of-schema-odcs-v3.svg width=800/>

*Figure 1: elements of the schema in ODCS v3.*

### Examples

#### Complete schema

```YAML
schema:
  - name: tbl
    logicalType: object
    physicalType: table
    physicalName: tbl_1 
    description: Provides core payment metrics 
    authoritativeDefinitions: 
      - url: https://catalog.data.gov/dataset/air-quality 
        type: businessDefinition
      - url: https://youtu.be/jbY1BKFj9ec
        type: videoTutorial
    tags: null
    dataGranularityDescription: Aggregation on columns txn_ref_dt, pmt_txn_id
    properties:
      - name: txn_ref_dt
        businessName: transaction reference date
        logicalType: date
        physicalType: date
        description: null
        partitioned: true
        partitionKeyPosition: 1
        criticalDataElement: false
        tags: []
        classification: public
        transformSourceObjects:
          - table_name_1
          - table_name_2
          - table_name_3
        transformLogic: sel t1.txn_dt as txn_ref_dt from table_name_1 as t1, table_name_2 as t2, table_name_3 as t3 where t1.txn_dt=date-3
        transformDescription: Defines the logic in business terms. 
        examples:
          - 2022-10-03
          - 2020-01-28
      - name: rcvr_id
        primaryKey: true 
        primaryKeyPosition: 1
        businessName: receiver id
        logicalType: string
        physicalType: varchar(18)
        required: false
        description: A description for column rcvr_id.
        partitioned: false
        partitionKeyPosition: -1
        criticalDataElement: false
        tags: []
        classification: restricted
        encryptedName: enc_rcvr_id
      - name: rcvr_cntry_code
        primaryKey: false 
        primaryKeyPosition: -1
        businessName: receiver country code
        logicalType: string
        physicalType: varchar(2)
        required: false
        description: null
        partitioned: false
        partitionKeyPosition: -1
        criticalDataElement: false
        tags: []
        classification: public
        authoritativeDefinitions:
          - url: https://collibra.com/asset/742b358f-71a5-4ab1-bda4-dcdba9418c25
            type: businessDefinition
          - url: https://github.com/myorg/myrepo
            type: transformationImplementation
          - url: jdbc:postgresql://localhost:5432/adventureworks/tbl_1/rcvr_cntry_code
            type: implementation
        encryptedName: rcvr_cntry_code_encrypted
```

#### Simple Array

```yaml
schema:
  - name: AnObject
    logicalType: object
    properties:
      - name: street_lines
        logicalType: array 
        items:
          logicalType: string
```

#### Array of Objects

```yaml
schema:
  - name: AnotherObject
    logicalType: object
    properties:
      - name: x
        logicalType: array
        items:
          logicalType: object
          properties:
            - name: id
              logicalType: string 
              physicalType: VARCHAR(40)
            - name: zip
              logicalType: string
              physicalType: VARCHAR(15)
```

### Definitions

Note: the description needs to be updated.

#### Schema

| Key                                                    | UX label                     | Required | Description                                                                                                                                                                                                                                           |
|--------------------------------------------------------|------------------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| schema                                                 | schema                       | Yes      | Array. A list of elements within the schema to be cataloged.                                                                                                                                                                                          |

#### Applicable to Elements (either Objects or Properties)

| Key                      | UX label                     | Required | Description                                                                                                                                                                                                            |
|--------------------------|------------------------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name                     | Name                         | Yes      | Name of the element.                                                                                                                                                                                                   |
| physicalName             | Physical Name                | No       | Physical name.                                                                                                                                                                                                         |
| description              | Description                  | No       | Description of the element.                                                                                                                                                                                            |
| businessName             | Business Name                | No       | The business name of the element.                                                                                                                                                                                      |
| authoritativeDefinitions | Authoritative Definitions    | No       | List of links to sources that provide more details on the table; examples would be a link to an external definition, a training video, a GitHub repo, Collibra, or another tool. See `authoritativeDefinitions` below. |
| tags                     | Tags                         | No       | A list of tags that may be assigned to the elements (object or property); the tags keyword may appear at any level.                                                                                                    |

#### Applicable to Objects

| Key                                                    | UX label                     | Required | Description                                                                          |
|--------------------------------------------------------|------------------------------|----------|--------------------------------------------------------------------------------------|
| dataGranularityDescription                             | Data Granularity             | No       | Granular level of the data in the object. Example would be "Aggregation by country." |

#### Applicable to Properties

Some keys are more applicable when the described property is a column. 

| Key                      | UX label                     | Required | Description                                                                                                                                                                                                                           |
|--------------------------|------------------------------|----------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| primaryKey               | Primary Key                  | No       | Boolean value specifying whether the field is primary or not. Default is false.                                                                                                                                                       |
| primaryKeyPosition       | Primary Key Position         | No       | If field is a primary key, the position of the primary key element. Starts from 1. Example of `account_id, name` being primary key columns, `account_id` has primaryKeyPosition 1 and `name` primaryKeyPosition 2. Default to -1.     |
| logicalType              | Logical Type                 | Yes      | The logical field datatype. One of `string`, `date`, `number`, `integer`, `object`, `array` or `boolean`.                                                                                                                             |
| logicalTypeOptions       | Logical Type Options         | No       | Additional optional metadata to describe the logical type. See [here](#logical-type-options) for more details about supported options for each `logicalType`.                                                                         |
| physicalType             | Physical Type                | Yes      | The physical element data type in the data source. For example, VARCHAR(2), DOUBLE, INT.                                                                                                                                              |
| description              | Description                  | No       | Description of the element.                                                                                                                                                                                                           |
| required                 | Required                     | No       | Indicates if the element may contain Null values; possible values are true and false. Default is false.                                                                                                                               |
| unique                   | Unique                       | No       | Indicates if the element contains unique values; possible values are true and false. Default is false.                                                                                                                                |
| partitioned              | Partitioned                  | No       | Indicates if the element is partitioned; possible values are true and false.                                                                                                                                                          |
| partitionKeyPosition     | Partition Key Position       | No       | If element is used for partitioning, the position of the partition element. Starts from 1. Example of `country, year` being partition columns, `country` has partitionKeyPosition 1 and `year` partitionKeyPosition 2. Default to -1. |
| classification           | Classification               | No       | Can be anything, like confidential, restricted, and public to more advanced categorization. Some companies like PayPal, use data classification indicating the class of data in the column; expected values are 1, 2, 3, 4, or 5.     |
| authoritativeDefinitions | Authoritative Definitions    | No       | List of links to sources that provide more detail on element logic or values; examples would be URL to a GitHub repo, Collibra, on another tool.                                                                                      |
| encryptedName            | Encrypted Name               | No       | The element name within the dataset that contains the encrypted element value. For example, unencrypted element `email_address` might have an encryptedName of `email_address_encrypt`.                                               |
| transformSourceObjects   | Transform Sources            | No       | List of objects in the data source used in the transformation.                                                                                                                                                                        |
| transformLogic           | Transform Logic              | No       | Logic used in the column transformation.                                                                                                                                                                                              |
| transformDescription     | Transform Description        | No       | Describes the transform logic in very simple terms.                                                                                                                                                                                   |
| examples                 | Example Values               | No       | List of sample element values.                                                                                                                                                                                                        |
| criticalDataElement      | Critical Data Element Status | No       | True or false indicator; If element is considered a critical data element (CDE) then true else false.                                                                                                                                 |
| items                    | Items                        | No       | List of items in an array (only applicable when `logicalType: array`)                                                                                                                                                                 |

### Logical Type Options

Additional metadata options to more accurately define the data type.

| Data Type      | Key              | UX Label           | Required | Description                                                                                                                                                                                           |
|----------------|------------------|--------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| array          | maxItems         | Maximum Items      | No       | Maximum number of items.                                                                                                                                                                              |
| array          | minItems         | Minimum Items      | No       | Minimum number of items.                                                                                                                                                                              |
| array          | uniqueItems      | Unique Items       | No       | If set to true, all items in the array are unique.                                                                                                                                                    |
| date           | format           | Format             | No       | Format of the date. Follows the format as prescribed by [JDK DateTimeFormatter](https://docs.oracle.com/javase/8/docs/api/java/time/format/DateTimeFormatter.html). For example, format 'yyyy-MM-dd'. |
| date           | exclusiveMaximum | Exclusive Maximum  | No       | If set to true, all values are strictly less than the maximum value (values < maximum). Otherwise, less than or equal to the maximum value (values <= maximum).                                       |
| date           | exclusiveMinimum | Exclusive Minimum  | No       | If set to true, all values are strictly greater than the minimum value (values > minimum). Otherwise, greater than or equal to the minimum value (values >= minimum).                                 |
| date           | maximum          | Maximum            | No       | All date values are less than or equal to this value (values <= maximum).                                                                                                                             |
| date           | minimum          | Minimum            | No       | All date values are greater than or equal to this value (values >= minimum).                                                                                                                          |
| integer/number | exclusiveMaximum | Exclusive Maximum  | No       | If set to true, all values are strictly less than the maximum value (values < maximum). Otherwise, less than or equal to the maximum value (values <= maximum).                                       |
| integer/number | exclusiveMinimum | Exclusive Minimum  | No       | If set to true, all values are strictly greater than the minimum value (values > minimum). Otherwise, greater than or equal to the minimum value (values >= minimum).                                 |
| integer/number | format           | Format             | No       | Format of the value in terms of how many bits of space it can use and whether it is signed or unsigned (follows the Rust integer types).                                                              |
| integer/number | maximum          | Maximum            | No       | All values are less than or equal to this value (values <= maximum).                                                                                                                                  |
| integer/number | minimum          | Minimum            | No       | All values are greater than or equal to this value (values >= minimum).                                                                                                                               |
| integer/number | multipleOf       | Multiple Of        | No       | Values must be multiples of this number. For example, multiple of 5 has valid values 0, 5, 10, -5.                                                                                                    |
| object         | maxProperties    | Maximum Properties | No       | Maximum number of properties.                                                                                                                                                                         |
| object         | minProperties    | Minimum Properties | No       | Minimum number of properties.                                                                                                                                                                         |
| object         | required         | Required           | No       | Property names that are required to exist in the object.                                                                                                                                              |
| string         | format           | Format             | No       | Provides extra context about what format the string follows. For example, password, byte, binary, email, uuid, uri, hostname, ipv4, ipv6.                                                             |
| string         | maxLength        | Maximum Length     | No       | Maximum length of the string.                                                                                                                                                                         |
| string         | minLength        | Minimum Length     | No       | Minimum length of the string.                                                                                                                                                                         |
| string         | pattern          | Pattern            | No       | Regular expression pattern to define valid value. Follows regular expression syntax from ECMA-262 (https://262.ecma-international.org/5.1/#sec-15.10.1).                                              |

### Authoritative definitions

Updated in ODCS (Open Data Contract Standard) v2.2.1.

| Key  | UX label          | Required | Description                                                                                                                                                             |
|------|-------------------|----------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type | Definition type   | Yes      | Type of definition for authority: v2.2.1 adds standard values: `businessDefinition`, `transformationImplementation`, `videoTutorial`, `tutorial`, and `implementation`. |
| url  | URL to definition | Yes      | URL to the authority.                                                                                                                                                   |


## Data quality 
This section describes data quality rules & parameters. They are tightly linked to the schema described in the previous section.

### Example of data quality at the object level

Note:
* This example relies on a data quality tool called Elevate. It should be easily transformed to any other tool. If you have questions, please raise an [issue](https://github.com/AIDAUserGroup/open-data-contract-standard/issues).
* The Open Data Contract Standard has a provision for supporting multiple data quality tools.

```YAML
schema:
  - name: tab1
    logicalType: object
# ...
    quality:
      - code: countCheck # Required, name of the rule
        templateName: CountCheck # NEW in v2.1.0 Required
        description: Ensure row count is within expected volume range       # Optional
        toolName: Elevate # Required
        toolRuleName: DQ.rw.tab1.CountCheck # NEW in v2.1.0 Optional (Available only to the users who can change in source code edition)
        dimension: completeness                                             # Optional
        type: reconciliation                                                # Optional NEW in v2.1.0 default value for column level check - dataQuality and for table level reconciliation
        severity: error                                                     # Optional NEW in v2.1.0, default value is error
        businessImpact: operational                                         # Optional NEW in v2.1.0
        scheduleCronExpression: 0 20 * * *                                  # Optional NEW in v2.1.0 default schedule - every day 10 a.m. UTC
      - code: distinctCheck    
        description: enforce distinct values
        toolName: Elevate
        templateName: DistinctCheck
        dimension: completeness
        toolRuleName: DQ.rw.tab1.DistinctCheck
        columns:
          - txn_ref_dt
          - rcvr_id
        column: null
        type: reconciliation
        severity: error
        businessImpact: operational
        scheduleCronExpression: 0 20 * * *
        customProperties:                     
          - property: FIELD_NAME
            value: rcvr_id
          - property: FILTER_CONDITIONS
            value: 1>0
      - code: piiCheck
        description: null
        toolName: Elevate
        templateName: PIICheck
        toolRuleName: DQ.rw.tab1_2_0_0.piiCheck
        type: dataQuality
        severity: error
        businessImpact: operational
        scheduleCronExpression: 0 20 * * *
        customProperties:                     
          - property: FIELD_NAME
            value:
          - property: FILTER_CONDITIONS
            value:
```

### Example of data quality at the element level

```YAML
schema:
  - name: tab1
    logicalType: object
      - name: rcvr_id
        primaryKey: true # NEW in v2.1.0, Optional, default value is false, indicates whether the column is primary key in the table.
        businessName: receiver id
# ...
        quality:
          - code: nullCheck
            templateName: NullCheck
            description: column should not contain null values
            toolName: Elevate
            toolRuleName: DQ.rw.tab1_2_0_0.rcvr_cntry_code.NullCheck
            dimension: completeness # dropdown 7 values
            type: dataQuality
            severity: error
            businessImpact: operational
            scheduleCronExpression: 0 20 * * *
            customProperties:
              - property: FIELD_NAME
                value:
              - property: COMPARE_TO
                value:
              - property: COMPARISON_TYPE
                value: Greater than
```

### Definitions

| Key                            | UX label            | Required | Description                                                                                                                                                                                                                                                                                                                                                                                                                              |
|--------------------------------|---------------------|----------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| quality                        | Quality             | No       | Quality tag with all the relevant information for rule setup and execution.                                                                                                                                                                                                                                                                                                                                                              |
| quality.code                   | Quality Code        | No       | The Rosewall data quality code(s) indicating which quality checks need to be performed at the dataset, table, or column level. The quality keyword may appear at any level. Some quality checks require parameters, so the check can be completed (e.g., a list of fields used to identify a distinct row). Therefore, some quality checks may be followed by a single value or an array. See the appendix for a link to quality checks. |
| quality.templateName           | Template Name       | Yes      | The template name that indicates what is the equivalent template from the tool.                                                                                                                                                                                                                                                                                                                                                          |
| quality.description            | Description         | No       | Describe the quality check to be completed.                                                                                                                                                                                                                                                                                                                                                                                              |
| quality.toolName               | Tool Name           | Yes      | Name of the tool used to complete the quality check; Most will be Elevate initially.                                                                                                                                                                                                                                                                                                                                                     |
| quality.toolRuleName           | Tool Rule Name      | No       | Name of the quality tool's rule created to complete the quality check.                                                                                                                                                                                                                                                                                                                                                                   |
| quality.dimension              | Dimension           | No       | The key performance indicator (KPI) or dimension for data quality.                                                                                                                                                                                                                                                                                                                                                                       |
| quality.columns                | Columns             | No       | List of columns to be used in the quality check.                                                                                                                                                                                                                                                                                                                                                                                         |
| quality.column                 | Column              | No       | To be used in lieu of quality.columns when only a single column is required for the quality check.                                                                                                                                                                                                                                                                                                                                       |
| quality.type                   | Type                | No       | The type of quality check.                                                                                                                                                                                                                                                                                                                                                                                                               |
| quality.severity               | Severity            | No       | The severity of the quality rule.                                                                                                                                                                                                                                                                                                                                                                                                        |
| quality.businessImpact         | Business Impact     | No       | Consequences of the rule failure.                                                                                                                                                                                                                                                                                                                                                                                                        |
| quality.scheduleCronExpression | Schedule Expression | No       | Rule execution schedule details.                                                                                                                                                                                                                                                                                                                                                                                                         |
| quality.customProperties       | Custom Properties   | No       | Additional properties required for rule execution.                                                                                                                                                                                                                                                                                                                                                                                       |


## <a id="support"/> Support & communication channels
Support and communication channels help consumers find help regarding their use of the data contract. In version 3, ODCS opens the 

### Examples

```yaml
support:
  - channel: channel-name-or-identifier # Simple Slack communication channel
    url: https://aidaug.slack.com/archives/C05UZRSBKLY
  - channel: channel-name-or-identifier # Simple distribution list
    url: mailto:datacontract-ann@bitol.io
```

```yaml
support:
  - channel: channel-name-or-identifier
    tool: teams
    scope: interactive
    url: https://bitol.io/teams/channel/my-data-contract-interactive
  - channel: channel-name-or-identifier
    tool: teams
    scope: announcements
    url: https://bitol.io/teams/channel/my-data-contract-announcements
    invitationUrl: https://bitol.io/teams/channel/my-data-contract-announcements-invit
  - channel: channel-name-or-identifier-for-all-announcement
    description: All announcement for all data contracts
    tool: teams
    scope: announcements
    url: https://bitol.io/teams/channel/all-announcements
  - channel: channel-name-or-identifier
    tool: email
    scope: announcements
    url: mailto:datacontract-ann@bitol.io
  - channel: channel-name-or-identifier
    tool: ticket
    url: https://bitol.io/ticket/my-product
```

### Definitions

| Key                   | UX label       | Required | Description                                                                                                                       |
|-----------------------|----------------|----------|-----------------------------------------------------------------------------------------------------------------------------------|
| support               | Support        | No       | Top level for support channels.                                                                                                   |
| support.channel       | Channel        | Yes      | Channel name or identifier.                                                                                                       |
| support.url           | Channel URL    | Yes      | Access URL using normal [URL scheme](https://en.wikipedia.org/wiki/URL#Syntax) (https, mailto, etc.).                             |
| support.description   | Description    | No       | Description of the channel, free text.                                                                                            |
| support.tool          | Tool           | No       | Name of the tool, value can be `email`, `slack`, `teams`, `discord`, `ticket`, or `other`.                                        |
| support.scope         | Scope          | No       | Scope can be: `interactive`, `announcements`, `issues`.                                                                           |
| support.invitationUrl | Invitation URL | No       | Some tools uses invitation URL for requesting or subscribing. Follows the [URL scheme](https://en.wikipedia.org/wiki/URL#Syntax). |


## Pricing
This section covers pricing when you bill your customer for using this data product. Pricing is experimental in v2.1.1 of the data contract.

### Example

```YAML
price:
  priceAmount: 9.95
  priceCurrency: USD
  priceUnit: megabyte
```

### Definitions

| Key                 | UX label           | Required | Description                                                            |
|---------------------|--------------------|----------|------------------------------------------------------------------------|
| price               | Price              | No       | Object                                                                 |
| price.priceAmount   | Price Amount       | No       | Subscription price per unit of measure in `priceUnit`.                 |
| price.priceCurrency | Price Currency     | No       | Currency of the subscription price in `price.priceAmount`.             |
| price.priceUnit     | Price Unit         | No       | The unit of measure for calculating cost. Examples megabyte, gigabyte. |


## Team
This section lists team members and the history of their relation with this data contract. In v2.x, this section was called stakeholders.

### Example
```YAML
team:
  - username: ceastwood
    role: Data Scientist
    dateIn: 2022-08-02
    dateOut: 2022-10-01
    replacedByUsername: mhopper
  - username: mhopper
    role: Data Scientist
    dateIn: 2022-10-01
  - username: daustin
    role: Owner
    comment: Keeper of the grail
    name: David Austin
    dateIn: 2022-10-01
```

### Definitions
The UX label is the label used in the UI and other user experiences. It is not limited to BlueRacket.

| Key                     | UX label           | Required | Description                                                                                       |
|-------------------------|--------------------|----------|---------------------------------------------------------------------------------------------------|
| team                    | Stakeholders       | No       | Array.                                                                                            |
| team.username           | Username           | No       | The stakeholder's username or email.                                                              |
| team.role               | Role               | No       | The stakeholder's job role; Examples might be owner, data steward. There is no limit on the role. |
| team.dateIn             | Date In            | No       | The date when the user became a stakeholder.                                                      |
| team.dateOut            | Date Out           | No       | The date when the user ceased to be a stakeholder                                                 |
| team.name               | Full Name          | No       | Full name.                                                                                        |
| team.email              | Email              | No       | Explicit email.                                                                                   |
| team.comment            | Comment            | No       | Generic comment.                                                                                   |
| team.replacedByUsername | Replaced By        | No       | The username of the user who replaced the stakeholder                                             |


## Roles
This section lists the roles that a consumer may need to access the dataset depending on the type of access they require.

### Example

```YAML
roles:
  - role: microstrategy_user_opr
    access: read
    firstLevelApprovers: Reporting Manager
    secondLevelApprovers: 'mandolorian'
  - role: bq_queryman_user_opr
    access: read
    firstLevelApprovers: Reporting Manager
    secondLevelApprovers: na
  - role: risk_data_access_opr
    access: read
    firstLevelApprovers: Reporting Manager
    secondLevelApprovers: 'dathvador'
  - role: bq_unica_user_opr
    access: write
    firstLevelApprovers: Reporting Manager
    secondLevelApprovers: 'mickey'
```

### Definitions

| Key                        | UX label            | Required | Description                                                          |
|----------------------------|---------------------|----------|----------------------------------------------------------------------|
| roles                      | Roles               | No       | Array. A list of roles that will provide user access to the dataset. |
| roles.role                 | Role                | Yes      | Name of the IAM role that provides access to the dataset.            |
| roles.description          | Description         | No       | Description of the IAM role and its permissions.                     |
| roles.access               | Access              | No       | The type of access provided by the IAM role.                         |
| roles.firstLevelApprovers  | 1st Level Approvers | No       | The name(s) of the first-level approver(s) of the role.              |
| roles.secondLevelApprovers | 2nd Level Approvers | No       | The name(s) of the second-level approver(s) of the role.             |


## <a id="sla"/> Service-Level Agreement (SLA)
This section describes the service-level agreements (SLA). 

* Use the `Table.Column` to indicate the number to do the checks on, as in `SELECT txn_ref_dt FROM tab1`.
* Separate multiple table.columns by a comma, as in `table1.col1`, `table2.col1`, `table1.col2`.
* If there is only one table in the contract, the table name is not required.

### Example

```YAML
slaDefaultElement: tab1.txn_ref_dt # Optional, default value is partitionColumn.
slaProperties:
  - property: latency # Property, see list of values in DP QoS
    value: 4
    unit: d # d, day, days for days; y, yr, years for years
    element: tab1.txn_ref_dt # This would not be needed as it is the same table.column as the default one
  - property: generalAvailability
    value: 2022-05-12T09:30:10-08:00
  - property: endOfSupport
    value: 2032-05-12T09:30:10-08:00
  - property: endOfLife
    value: 2042-05-12T09:30:10-08:00
  - property: retention
    value: 3
    unit: y
    element: tab1.txn_ref_dt
  - property: frequency
    value: 1
    valueExt: 1
    unit: d
    element: tab1.txn_ref_dt
  - property: timeOfAvailability
    value: 09:00-08:00
    element: tab1.txn_ref_dt
    driver: regulatory # Describes the importance of the SLA: [regulatory|analytics|operational|...]
  - property: timeOfAvailability
    value: 08:00-08:00
    element: tab1.txn_ref_dt
    driver: analytics
```

### Definitions

| Key                    | UX label               | Required                       | Description                                                                                         |
|------------------------|------------------------|--------------------------------|-----------------------------------------------------------------------------------------------------|
| slaDefaultElement      | Default SLA element(s) | No                             | Element (using the element path notation) to do the checks on.                                      |
| slaProperties          | SLA                    | No                             | A list of key/value pairs for SLA specific properties. There is no limit on the type of properties. |
| slaProperties.property | Property               | Yes                            | Specific property in SLA, check the Data QoS periodic table. May requires units.                    |
| slaProperties.value    | Value                  | Yes                            | Agreement value. The label will change based on the property itself.                                |
| slaProperties.valueExt | Extended value         | No - unless needed by property | Extended agreement value. The label will change based on the property itself.                       |
| slaProperties.unit     | Unit                   | No - unless needed by property | **d**, day, days for days; **y**, yr, years for years, etc. Units use the ISO standard.             |
| slaProperties.element  | Element(s)             | No                             | Element(s) to check on. Multiple elements should be extremely rare and, if so, separated by commas. |
| slaProperties.driver   | Driver                 | No                             | Describes the importance of the SLA from the list of: `regulatory`, `analytics`, or `operational`.  |

## Infrastructure & servers
TBD

## <a id="custom-properties"/> Custom Properties
This section covers custom properties you may find in a data contract.

### Example

```YAML
customProperties:
  - property: refRulesetName
    value: gcsc.ruleset.name
  - property: somePropertyName
    value: property.value
  - property: dataprocClusterName # Used for specific applications like Elevate
    value: [cluster name]
```

### Definitions

| Key                       | UX label             | Required | Description                                                                                                       |
|---------------------------|----------------------|----------|-------------------------------------------------------------------------------------------------------------------|
| customProperties          | Custom Properties    | No       | A list of key/value pairs for custom properties. Initially created to support the REF ruleset property.           |
| customProperties.property | Property             | No       | The name of the key. Names should be in camel case–the same as if they were permanent properties in the contract. |
| customProperties.value    | Value                | No       | The value of the key.                                                                                             |


## Other Properties
This section covers other properties you may find in a data contract.

### Example

```YAML
contractCreatedTs: 2022-11-15 02:59:43
```


### Other properties definition

| Key                       | UX label             | Required | Description                                                  |
|---------------------------|----------------------|----------|--------------------------------------------------------------|
| contractCreatedTs         | Contract Created UTC | No       | Timestamp in UTC of when the data contract was created.      |

## Full example

[Check full example here.](examples/all/full-example.yaml)
