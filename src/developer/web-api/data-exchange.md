# Data exchange

## Aggregate data exchange

This section describes the aggregate data exchange service and API.

### Introduction

The aggregate data exchange service offers the ability to exchange data between instances of DHIS 2, and possibly other software which supports the DHIS 2 data value set JSON format. It also allows for data exchange within a single instance of DHIS 2, for instance for aggregation of tracker data and saving the result as aggregate data. 

The aggregate data exchange service is suitable for use-cases such as:

* Data exchange between an HMIS instance to a data portal or data warehouse instance of DHIS 2.
* Data exchange between a DHIS 2 tracker instance with individual data to an aggregate HMIS instance.
* Pre-computation of tracker data with program indicators saved as aggregate data values.
* Data reporting from a national HMIS to a global donor.

### Overview

The aggregate data exchange service allows for data exchange between a *source* instance of DHIS 2 and a *target* instance of DHIS 2. A data exchange can be *external*, for which the target instance is different/external to the source instance. A data exchange can also be *internal*, for which the target instance is the same as the source instance. The aggregate data exchange source can contain multiple source requests, where a source request roughly corresponds to an analytics API request.

The data value will be retrieved and transformed into the *data value set* format, and then pushed to the target instance of DHIS 2. The aggregate data exchange service supports *identifier schemes* to allow for flexibility in mapping metadata between instances.

Data will be retrieved and aggregated from the source instance using the analytics engine. This implies that data elements, aggregate indicators, data set reporting rates and program indicators can be referenced in the request to the source instance. A source request also contains periods, where both fixed and relative periods are supported, and organisation units. Any number of *filters* can be applied to a source request.

A data exchange can be run as a scheduled job, where the data exchange can be set to run at a specific interval. A data exchange can also be run on demand through the API.

To create and manipulate aggregate data exchanges, the `F_AGGREGATE_DATA_EXCHANGE_PUBLIC_ADD` / `F_AGGREGATE_DATA_EXCHANGE_PRIVATE_ADD` and `F_AGGREGATE_DATA_EXCHANGE_DELETE` authorities are required.

The aggregate data exchange definitions are regular metadata in DHIS 2, meaning that the definitions can be imported and exported between instances of DHIS 2. The exception is credentials (usernames and access tokens) which will not be exposed in metadata exports. Credentials are encrypted in storage to provide an additional layer of security.

The aggregate data exchange service was introduced in version 2.39, which means that the source instance of DHIS 2 must be version 2.39 or later. The regular DHIS 2 data value set format is used when pushing data, which means that the target instance can be any DHIS 2 version.

### Authentication

For data exchanges of type external, the base URL and authentication credentials for the target DHIS 2 instance must be specified. For authentication, basic authentication and personal access tokens (PAT) are supported.

It is recommended to either specify basic authentication or PAT authentication. If both are specified, PAT authentication takes precedence.

Note that PAT support was introduced in version 2.38.1, which means that in order to use PAT authentication, the target DHIS 2 instance must be version 2.38.1 or later.

 ### API

The aggregate data exchange API is covered in the following section.

#### Create aggregate data exchange

```
POST /api/aggregateDataExchanges
```

```
Content-Type: application/json
```

Example internal data exchange payload, where event data is computed with program indicators and saved as aggregate data values: 

```json
{
  "name": "Internal data exchange",
  "source": {
    "params": {
      "periodTypes": [
        "MONTHLY",
        "QUARTERLY"
      ]
    },
    "requests": [
      {
        "name": "ANC",
        "visualization": null,
        "dx": [
          "fbfJHSPpUQD",
          "cYeuwXTCPkU",
          "Jtf34kNZhzP"
        ],
        "pe": [
          "LAST_12_MONTHS",
          "202201"
        ],
        "ou": [
          "ImspTQPwCqd"
        ],
        "filters": [
          {
            "dimension": "Bpx0589u8y0",
            "items": [
              "oRVt7g429ZO", 
              "MAs88nJc9nL"
            ]
          }
        ],
        "inputIdScheme": "UID",
        "outputDataElementIdScheme": "UID",
        "outputOrgUnitIdScheme": "UID",
        "outputIdScheme": "UID"
      }
    ]
  },
  "target": {
    "type": "INTERNAL",
    "request": {
      "dataElementIdScheme": "UID",
      "orgUnitIdScheme": "UID",
      "categoryOptionComboIdScheme": "UID",
      "idScheme": "UID"
    }
  }
}
```

Example external data exchange payload with basic authentication and ID scheme *code*, where data is pushed to an external DHIS 2 instance:

```json
{
  "name": "External data exchange with basic authentication",
  "source": {
    "requests": [
      {
        "name": "ANC",
        "visualization": null,
        "dx": [
          "fbfJHSPpUQD",
          "cYeuwXTCPkU",
          "Jtf34kNZhzP"
        ],
        "pe": [
          "LAST_12_MONTHS",
          "202201"
        ],
        "ou": [
          "ImspTQPwCqd"
        ],
        "inputIdScheme": "UID",
        "outputIdScheme": "CODE"
      }
    ]
  },
  "target": {
    "type": "EXTERNAL",
    "api": {
        "url": "https://play.dhis2.org/2.38.1.1",
        "username": "admin",
        "password": "district"
    },
    "request": {
      "idScheme": "CODE"
    }
  }
}
```

Example external data exchange payload with PAT authentication and ID scheme *code*, where data is pushed to an external DHIS 2 instance:

```json
{
  "name": "External data exchange with PAT authentication",
  "source": {
    "requests": [
      {
        "name": "ANC",
        "dx": [
          "fbfJHSPpUQD",
          "cYeuwXTCPkU",
          "Jtf34kNZhzP"
        ],
        "pe": [
          "LAST_12_MONTHS",
          "202201"
        ],
        "ou": [
          "ImspTQPwCqd"
        ],
        "inputIdScheme": "UID",
        "outputIdScheme": "CODE"
      }
    ]
  },
  "target": {
    "type": "EXTERNAL",
    "api": {
        "url": "https://play.dhis2.org/2.38.1.1",
        "accessToken": "d2pat_XIrqgAGjW935LLPuSP2hXSZwpTxTW2pg3580716988"
    },
    "request": {
      "idScheme": "CODE"
    }
  }
}
```

Response

```
201 Created
```

```json
{
  "httpStatus": "Created",
  "httpStatusCode": 201,
  "status": "OK",
  "response": {
    "responseType": "ObjectReport",
    "uid": "pG4bBTMiCqO",
    "klass": "org.hisp.dhis.dataexchange.aggregate.AggregateDataExchange",
    "errorReports": []
  }
}
```

#### Update aggregate data exchange

```
PUT /api/aggregateDataExchanges/{id}
```

```
Content-Type: application/json
```

The request payload is identical to the create operation.

Response

```
200 OK
```

```json
{
  "httpStatus": "OK",
  "httpStatusCode": 200,
  "status": "OK",
  "response": {
    "responseType": "ObjectReport",
    "uid": "pG4bBTMiCqO",
    "klass": "org.hisp.dhis.dataexchange.aggregate.AggregateDataExchange",
    "errorReports": []
  }
}
```

#### Get aggregate data exchange

```
GET /api/aggregateDataExchanges/{id}
```

``` 
Accept: application/json
```

The retrieval endpoints follow the regular metadata endpoint field filtering and object filtering semantics. JSON is the only supported response format.

Response

```
200 OK
```

#### Delete aggregate data exchange

```
DELETE /api/aggregateDataExchanges/{id}
```

Response

```
204 No Content
```

```json
{
  "httpStatus": "OK",
  "httpStatusCode": 200,
  "status": "OK",
  "response": {
    "responseType": "ObjectReport",
    "uid": "pG4bBTMiCqO",
    "klass": "org.hisp.dhis.dataexchange.aggregate.AggregateDataExchange",
    "errorReports": []
  }
}
```

#### Run aggregate data exchange

An aggregate data exchange can be run directly with a POST request to the following endpoint:

```
POST /api/aggregateDataExchanges/{id}/exchange
```

Response

```
200 OK
```

```json
{
  "responseType": "ImportSummaries",
  "status": "SUCCESS",
  "imported": 36,
  "updated": 0,
  "deleted": 0,
  "ignored": 0,
  "importSummaries": ["<import summaries here>"]
}
```

An import summary describing the outcome of the data exchange will be returned, including the number of data values which were imported, updated, deleted and ignored.

#### Get source data

The aggregate data for the source request of an aggregated data exchange can be retrieved with a GET request to the following endpoint:

```
GET /api/aggregateDataExchanges/{id}/sourceData
```

```
Accept: application/json
```

Response

```
200 OK
```

The response payload format is identical with the analytics API endpoint in data value set format. This endpoint is useful for debugging purposes. Consult the analytics API guide for additional details.

### Data model

The aggregate data exchange data model / payload is described in the following section.

| Field                                             | Data type      | Mandatory   | Description                                                  |
| ------------------------------------------------- | -------------- | ----------- | ------------------------------------------------------------ |
| name                                              | String         | Yes         | Name of aggregate data exchange.                             |
| source                                            | Object         | Yes         | Source for aggregate data exchange.                          |
| source.params                                     | Object         | No          | Parameters for source request.                               |
| source.params.periodTypes                         | Array/String   | No          | Allowed period types for overriding periods in source request. |
| source.requests                                   | Array/Object   | Yes         | Source requests.                                             |
| source.requests.name                              | String         | Yes         | Name of source request.                                      |
| source.requests.visualization                     | String         | No          | Identifier of associated visualization object.               |
| source.requests.dx                                | Array/String   | Yes         | Identifiers of data elements, indicators, data sets and program indicators for the source request. |
| source.requests.pe                                | Array/String   | Yes         | Identifiers of fixed and relative periods for the source request. |
| source.requests.ou                                | Array/String   | Yes         | Identifiers of organisation units for the source request.    |
| source.requests.filters                           | Array (Object) | No          | Filters for the source request.                              |
| source.requests.filters.dimension                 | String         | No          | Dimension identifier for the filter.                         |
| source.requests.filters.items                     | Array/String   | No          | Item identifiers for the filter.                             |
| source.requests.inputIdScheme                     | String         | No          | Input ID scheme, can be `UID`, `CODE`.                       |
| source.requests.outputDataElementIdScheme         | String         | No          | Output data element ID scheme, can be `UID`, `CODE`.         |
| source.requests.outputOrgUnitIdScheme             | String         | No          | Output organisation unit ID scheme, can be `UID`, `CODE`.    |
| source.requests.outputIdScheme                    | String         | No          | Output general ID scheme, can be `UID`, `CODE`.              |
| source.target                                     | Object         | Yes         | Target for  aggregate data exchange.                         |
| source.target.type                                | String         | Yes         | Type of target, can be `EXTERNAL`, `INTERNAL`.               |
| source.target.api                                 | Object         | Conditional | Target API information, only mandatory for type `EXTERNAL`.  |
| source.target.api.url                             | String         | Conditional | Base URL of target DHIS 2 instance, do not include the `/api` part. |
| source.target.api.accessToken                     | String         | Conditional | Access token (PAT) for target DHIS 2 instance, used for PAT authentication. |
| source.target.api.username                        | String         | Conditional | Username for target DHIS 2 instance, used for basic authentication. |
| source.target.api.password                        | String         | Conditional | Password for target DHIS 2 instance, used for basic authentication. |
| source.target.request                             | Object         | No          | Target request information.                                  |
| source.target.request.dataElementIdScheme         | String         | No          | Input data element ID scheme, can be `UID`, `CODE`.          |
| source.target.request.orgUnitIdScheme             | String         | No          | Input organisation unit ID scheme, can be `UID`, `CODE`.     |
| source.target.request.categoryOptionComboIdScheme | String         | No          | Input category option combo ID scheme, can be `UID`, `CODE`. |
| source.target.request.idScheme                    | String         | No          | Input general ID scheme, can be `UID`, `CODE`.               |

### Error handlng

TODO: Write section. Test, fix and document invalid references in source request, target server being unavailable, import errors in target instance