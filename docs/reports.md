---
title: Manta Usage Reports
markdown2extras: tables, code-friendly
apisections:
---
<!--
    This Source Code Form is subject to the terms of the Mozilla Public
    License, v. 2.0. If a copy of the MPL was not distributed with this
    file, You can obtain one at http://mozilla.org/MPL/2.0/.
-->

<!--
    Copyright (c) 2014, Joyent, Inc.
-->

There are several reports available that are delivered to /:login/reports.
* Access logs, delivered hourly to /:login/reports/access-logs
* Storage usage, delivered hourly to /:login/reports/usage/storage
* Requests and bandwidth usage, delivered hourly to
  /:login/reports/usage/request
* Compute usage, delivered hourly to /:login/reports/usage/compute
* Summarized billable usage, delivered daily to /:login/reports/usage/summary

**NOTE:** All numbers in usage reports (storage, request, compute, summaries)
are represented as strings. Some counters may exceed the limit of a
representable or precise integer depending on implementation and your usage.

A link to the latest report generated is available at
/:login/reports/usage/:category/latest or /:login/reports/access-logs/latest

All reports are delivered in newline-separated JSON format. The samples here
have been formatted for easy reading.

# Access logs

    {
        "billable_operation": "PUT",
        "remoteAddress": "::ffff:a03:5bec",
        "req": {
            "method": "PUT",
            "request-uri": "/fredkuo/stor/test",
            "headers": {
                "accept": "application/json",
                "content-type": "application/json; type=directory",
                "date": "Sun, 10 Mar 2013 10:00:02 GMT",
                "x-request-id": "175cc9ce-6342-44be-877d-9a1eaaa25e6e",
                "user-agent": "restify/2.1.1 (ia32-sunos; " +
                        "v8/3.11.10.25; OpenSSL/0.9.8w) node/0.8.14",
                "accept-version": "~1.0",
                "host": "manta.beta.joyent.us",
                "connection": "keep-alive",
                "transfer-encoding": "chunked"
            },
            "httpVersion": "1.1",
            "caller": {
                "login": "poseidon"
            },
        },
        "res": {
            "statusCode": 204,
            "headers": {
                "last-modified": "Wed, 13 Feb 2013 18:00:02 GMT",
                "date": "Sun, 10 Mar 2013 10:00:02 GMT",
                "server": "Manta",
                "x-request-id": "175cc9ce-6342-44be-877d-9a1eaaa25e6e",
                "x-response-time": 18,
                "x-server-name": "218e7193-45c8-41e1-b4a4-7a3e6972bea6"
            }
        }
    }

| Field | Notes |
| ----- | ----- |
| remoteAddress | always ipv6 |
| billable_operation | may differ from HTTP method |
| req.caller | if available, information about the user that made the request |

# Storage Usage

    {
        "storage": {
            "stor": {
                "directories": "31213",
                "keys": "141642",
                "objects": "141620",
                "bytes": "2290568504594"
            },
            "public": {
                "directories": "1",
                "keys": "1",
                "objects": "1",
                "bytes": "16092490"
            },
            "reports": {
                "directories": "1323",
                "keys": "1235",
                "objects": "1230",
                "bytes": "386149302"
            },
            "jobs": {
                "directories": "125588",
                "keys": "67627",
                "objects": "67627",
                "bytes": "10013853195"
            }
        },
        "date": "2013-06-24T17:00:00.000Z"
    }


Snaplinks existing in one namespace that point to an object in another
namespace are counted under the first namespace in this order: `stor` `public`
`jobs` `reports`. Objects smaller than 4096 bytes are rounded up to 4096 bytes.

| Field | Notes |
| ----- | ----- |
| date | hour that this report is for |
| stor, public, reports, jobs | usage under each namespace e.g. /:login/jobs |
| keys | total number of object keys, including snaplinks |
| objects | total unique objects |

# Request and Bandwidth Usage

    {
        "requests": {
            "type": {
                "DELETE": "2120",
                "GET": "3534",
                "HEAD": "315",
                "LIST": "668",
                "OPTIONS": "0",
                "POST": "288",
                "PUT": "19057"
            },
            "bandwidth": {
                "in": "4810494334",
                "out": "4513223887",
                "headerIn": "13660794",
                "headerOut": "6012783"
            }
        },
        "date": "2013-06-24T17:00:00.000Z"
    }

| Field | Notes |
| ----- | ----- |
| date | hour that this report is for |
| type | billable operation type, not HTTP method |
| bandwidth | bandwidth in and out only include successful GETs and PUTs |


# Compute Usage

    {
        "jobs": {
            "b1adf3a9-c893-4b0c-8217-8281b6eecfbd": {
                "0": {
                    "memory": "1024",
                    "disk": "8",
                    "seconds": "1",
                    "ntasks": "1",
                    "bandwidth": {
                        "in": "601874",
                        "out": "322"
                    }
                }
            },
            "c7c1782e-3929-4417-a24a-ab7ee8815fc1": {
                "0": {
                    "memory": "1024",
                    "disk": "8",
                    "seconds": "99",
                    "ntasks": "10",
                    "bandwidth": {
                        "in": "6959648",
                        "out": "4140"
                    }
                },
                "1": {
                    "memory": "1024",
                    "disk": "8",
                    "seconds": "16",
                    "ntasks": "2",
                    "bandwidth": {
                        "in": "1353284",
                        "out": "828"
                    }
                },
                "2": {
                    "memory": "1024",
                    "disk": "8",
                    "seconds": "2",
                    "ntasks": "1",
                    "bandwidth": {
                        "in": "659048",
                        "out": "414"
                    }
                }
            }
        },
        "date": "2013-06-24T17:00:00.000Z"
    }


Broken down by job and phase.

| Field | Notes |
| ----- | ----- |
| date | hour that this report is for |
| memory | megabytes of memory requested for the phase |
| disk | gigabytes of disk requested for the phase |
| seconds | total wall time spent across all tasks for the phase that hour |
| ntasks | number of tasks seen running for that phase that hour |
| bandwidth | all bytes, including any overhead |

# Summary

    {
        "date": "2013-06-23T00:00:00.000Z",
        "storageGBHours": "48939",
        "bandwidthGB": {
            "in": "119",
            "out": "36"
        },
        "requests": {
            "DELETE": "5346",
            "GET": "16351",
            "HEAD": "5984",
            "LIST": "12794",
            "OPTIONS": "0",
            "POST": "2556",
            "PUT": "81817"
        },
        "computeBandwidthGB": {
            "in": "6",
            "out": "1"
        },
        "computeGBSeconds": "54680"
    }

| Field | Notes |
| ----- | ----- |
| date | calendar day that this report is for |
| computeGBSeconds | seconds of compute time per GB memory |
