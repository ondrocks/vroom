<!-- This file is part of VROOM. -->

<!-- Copyright (c) 2015-2016, Julien Coupey. -->
<!-- All rights reserved (see LICENSE). -->

This file describes the API to use with `vroom` command-line as of
version 1.1.0.

Contents:
- [Input format](#input)
- [Output format](#output)
- [Examples](#examples)

**Note**: the expected order for all coordinates arrays is [lon,lat].

# Input

The problem description is read from standard input or from a file
(using `-i`) and should be valid `json` formatted as follow.

| Key         | Description |
|-----------|-----------|
| [`jobs`](#jobs) |  array of `job` objects describing the places to visit |
| [`vehicles`](#vehicles) |  array of `vehicle` objects describing the available vehicles |
| [[`matrix`](#matrix)] | optional two-dimensional array describing a custom matrix |

**Warning**: only problems with one vehicle are supported in v1.1.0 so
at the moment, `vehicles` should have length 1.

## Jobs

A `job` object has the following properties:

| Key         | Description |
| ----------- | ----------- |
| `id` | an integer used as unique identifier |
| [`location`] | coordinates array |
| [`location_index`] | index of relevant row and column in custom matrix |

If a custom matrix is provided:

- `location_index` is mandatory
- `location` is optional but can be set to retrieve coordinates in the
  response

If no custom matrix is provided:

- a `table` query will be sent to OSRM
- `location` is mandatory
- `location_index` is irrelevant

## Vehicles

A `vehicle` object has the following properties:

| Key         | Description |
| ----------- | ----------- |
| `id` | an integer used as unique identifier |
| [`start`] | coordinates array |
| [`start_index`] | index of relevant row and column in custom matrix |
| [`end`] | coordinates array |
| [`end_index`] | index of relevant row and column in custom matrix |

### Notes on `vehicle` locations

- key `start` and `end` are optional for a `vehicle`, as long as at
  least one of them is present
- if `end` is omitted, the resulting route will stop at the last
  visited job, whose choice is determined by the optimization process
- if `start` is omitted, the resulting route will start at the first
  visited job, whose choice is determined by the optimization process
- to request a round trip, just specify both `start` and `end` with
  the same coordinates
- depending on if a custom matrix is provided, required fields follow
  the same logic than for `job` keys `location` and `location_index`

## Matrix

A `matrix` object is an array of arrays of unsigned integers
describing the rows of a custom cost matrix as an alternative to the
travel-time matrix computed by OSRM. Therefore, if a custom matrix is
provided, the `location`, `start` and `end` properties become
optional. Instead of the coordinates, row and column indications
provided with the `*_index` keys are used during optimization.

# Output

The computed solution is written as `json` on standard output or a file
(using `-o`), formatted as follow.

| Key         | Description |
| ----------- | ----------- |
| `code` | return code, `0` if no error was raised |
| `error` | error message (present iff `code` is different from `0`) |
| [`summary`](#summary) | object summarizing solution indicators |
| [`routes`](#routes) | array of `route` objects |

## Routes

A `route` object has the following properties:

| Key         | Description |
| ----------- | ----------- |
| `vehicle` | id of the vehicle assigned to this route |
| [`steps`](#steps) | array of `step` objects |
| `cost` | cost for this route |
| `geometry`* | polyline encoded route geometry |
| `duration`* | total route duration in seconds |
| `distance`* | total route distance in meters |

*: provided when using the `-g` flag with `OSRM`.

### Steps

A `step` object has the following properties:

| Key         | Description |
| ----------- | ----------- |
| `type` | a string that is either `start`, `job` or `end` |
| `location` | coordinates array for this step |
| `job` | id of the job performed at this step, provided if `type` value is `job` |

## Summary

The `summary` object has the following properties:

| Key         | Description |
| ----------- | ----------- |
| `cost` | total cost for all routes |
| `duration`* | total duration in seconds for all routes |
| `distance`* | total distance in meters for all routes |
| [`computing_times`](#computing-times) | details for run-time information |

*: provided when using the `-g` flag with `OSRM`.

### Computing times

The `computing_times` object is used to report execution times in
milliseconds.

| Key | Description |
| ----------- | ----------- |
| `loading` | time required to parse the problem, compute and load the cost matrix |
| `solving` | time required to apply the solving strategy |
| `routing`* | time required to retrieve the detailed route geometries |

*: provided when using the `-g` flag with `OSRM`.

# Examples

Using the following input describes a (very) small problem where
matrix computing rely on OSRM:

```javascript
{
  "vehicles": [
    {
      "id": 0,
      "start": [2.3526, 48.8604],
      "end": [2.3526, 48.8604]
    }
  ],
  "jobs": [
    {
      "id": 0,
      "location": [2.3691, 48.8532]
    },
    {
      "id": 1,
      "location": [2.2911, 48.8566]
    }
  ]
}
```

producing a solution that looks like:

```javascript
{
  "code": 0,
  "routes": [
    {
      "vehicle": 0,
      "cost": 1009,
      "steps": [
        {
          "location": [2.3526, 48.8604],
          "type": "start"
        },
        {
          "job": 1,
          "location": [2.2911, 48.8566],
          "type": "job"
        },
        {
          "job": 0,
          "location": [2.3691, 48.8532],
          "type": "job"
        },
        {
          "location": [2.3526, 48.8604],
          "type": "end"
        }
      ],
      "geometry": "o`fiHaqjMBSPHh@T^PHDTLLHLFJFLFlBz@r@VXL\\NJDHDlCjAJDHDCPCNCNl\n@dGBVFp@Oz@bBt@RHPHtDhBHDJD@@bD~AnB~@RJLFFDbAd@PJTH`Ad@LFCHGZSpAO~@kApGOn@K^?BSf@Yv@k@zAw@nBGPwBnDOTmBrCa\n@l@W\\{@fBUd@}BvE{@hBW|ASvAe@jDAP?DJtC@\\Ab@qArIG\\UhAWlA_@fBoBnIEREP]zAMh@Kb@e@vBo@vCS|@oBvJADI\\GZEPy\n@dEGXKf@kFtWI`@Ml@GXI^?BOr@Mr@CN[zA?f@KzADbNBx@D`EBt@@h@?v@?RAFAr@@jDFr@\\fDFh@@`@@f@@J?p@B|CPpVFnL?L\n?`A?|AGb@?F?\\BbAF`A?D?HDtAFfBLjBV|BD\\L~@X|A`@nB~@|C`AfCfAhBLT`@n@fAzBtCtEhBvCtCxEt@|@bAfAvAgCwAfCIm\n@{@_BcEyGW_@S[QNK\\a@~@aErHCBGN_AoAIMyDoFqBmCm@w@a@wAw@oDo@aDGYGSCK?ECGO_AcB{MW}B[yEw@{JQiBEc@Ca@Ai@\n?kACsDCaFKgACoFIiKCkE?g@C_EAg@CuD?g@CiDIgMC_CAwBBsANaAXkA^gAn@sCXqAJk@hAoEzCcN^cBj@eCLg@`@kBViAhAcFlBqIzCyLhE_\n[|@oKt@uJPyAL_@BMJe@|@kINkAlAkGxAaI`@wC^kBLq@Ji@lAuGTmAx@qCv@kDZsALa@b@u@p@}BRkAp@yBd@oA~@eCJY\\aAPi\n@`@sAp@_DZ_BF[x@iDZy@?_@n@sCDc@US|@iDd@eBDOLa@L]v@sAj@_@ZIpAEb@APAJ@HAe@gCG_@[uAEWy@{DgBsIEOKe@Ow@Q{\n@WqAKi@YsA]}AKm@COGe@Cg@Am@HWFY@Q?SAQGYIWOQSKSAMBKDEBILEBKVCNAHAR?P@RFVBJDJNP@?PHIpAEj@IbAIfAATQ~Be@\n`D?F[bBKp@CLERERu@fDCNGRCHg@rBENMd@[`A_@jAQd@G`@Il@Mz@q@rEQnA[tBEVG\\c@vCCPCRc@tCStACPET_@fCQjACNQjAu\n@bFGMGGECuAq@IGERmAtEELKGs@c@CCeCwAIEIEqA}@GEcAs@OMIImAoAEHWl@Sv@U`BCVQx@CL`Br@",
      "distance": 14615,
      "duration": 1009
    }
  ],
  "solution": {
    "cost": 1009,
    "duration": 1009,
    "distance": 14615,
    "computing_times": {
      "routing": 7,
      "solving": 0,
      "loading": 13
    }
  }
}
```


The following input makes use of the option to provide a custom matrix:

```javascript
{
  "vehicles": [
    {
      "id":0,
      "start_index":0,
      "end_index":3
    }
  ],
  "jobs": [
    {
      "id":1414,
      "location_index":1
    },
    {
      "id":1515,
      "location_index":2
    }
  ],
  "matrix": [
    [0,2104,197,1299],
    [2103,0,2255,3152],
    [197,2256,0,1102],
    [1299,3153,1102,0]
  ]
}
```

producing a solution that looks like:

```javascript
{
  "routes": [
    {
      "steps": [
        {
          "type": "start"
        },
        {
          "job": 1414,
          "type": "job"
        },
        {
          "job": 1515,
          "type": "job"
        },
        {
          "type": "end"
        }
      ],
      "cost": 5461,
      "vehicle": 0
    }
  ],
  "summary": {
    "computing_times": {
      "solving": 1,
      "loading": 0
    },
    "cost": 5461
  },
  "code": 0
}
```
