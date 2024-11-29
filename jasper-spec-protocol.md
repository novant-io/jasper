# Jasper Protocol

Jasper is a simple JSON-based format for interacting with IoT platforms.

## Endpoints

The following HTTP endpoints are supported:

     /jasper/v1/about   - Get information about this system
     /jasper/v1/sources - Get point sources from this system
     /jasper/v1/points  - Get point information for a source
     /jasper/v1/values  - Get current value data from points
     /jasper/v1/write   - Write back to a system point
     /jasper/v1/batch   - Invoke multiple operations in single request

## About

The `/about` endpoint provides meta-data information about the remote system.

Request:

    POST /jasper/v1/about HTTP/1.1

Response:

    {
      "name": "Device 123",
      "vendor": "ACME",
      "model": "MX-100",
      "version": "1.2"
    }

Where required fields are:

  * `name`:    the user defined display name of this system
  * `vendor`:  the vendor of this system
  * `model`:   the model of this system
  * `version`: the primary version of this system

Additional system specific information may be returned.

## Sources

A source is any grouped collection of points.  A Jasper endpoint must contain
at least one source. The `/sources` endpoint provides meta-data information
about the available sources in this system.

Request:

    POST /jasper/v1/sources HTTP/1.1

Response:

    {
      "sources": [
        {
          "id":   "54d",
          "name": "Chiller",
          "path": "/Drivers/NiagaraNetwork/JACE-02/points/Chiller"
        },
        {
          "id":   "620",
          "name": "VAV-1",
          "path": "/Drivers/NiagaraNetwork/JACE-05/points/VAV-1"
        },
        {
          "id":   "621",
          "name": "VAV-2",
          "path": "/Drivers/NiagaraNetwork/JACE-05/points/VAV-2"
        },
        {
          "id":   "622",
          "name": "VAV-3",
          "path": "/Drivers/NiagaraNetwork/JACE-05/points/VAV-3"
        }
      ]
    }

Where each source object has:

  * `id`: required unique id for this source in the system
  * `name`: required human readable name of this point
  * `path`: optional path where this source exists in the system

The `id` is a `String` value type, but the format is opaque and system
dependent.

## Points

The `/points` endpoint provides meta-data information about the available
points under a source.  It takes a required `source_id` argument.

Request:

    POST /jasper/v1/points HTTP/1.1
    Content-Type: application/x-www-form-urlencoded

    source_id=620

Response:

    {
      "points": [
        {
          "addr": "av.DamperPosition",
          "name": "Damper Position"
        },
        {
          "addr": "bv.FanStatus",
          "name": "Fan Status"
        },
        {
          "addr": "av.ZoneTemp",
          "name": "Zone Temp",
          "unit": "°F"
        },
        {
          "addr": "ao.ReturnTemp",
          "name": "Return Temp",
          "unit": "°F"
        },
        {
          "addr": "ao.DischargeTemp",
          "name": "Discharge Temp",
          "unit": "°F"
        }
      ]
    }

Where each point object has:

  * `addr`: required unique addr for this point under a source (see [Point Addr](#point-addr) below)
  * `name`: required human readable name of this point
  * `unit`: the optional unit for this point
  * `enum`: the optional comma-separated list of enum ordinal names

### Point Addr

Each point must have a unique address, where the format is:

    <type>.<suffix>

Where `type` must be one of:

  * `ai`: read-only analog input
  * `ao`: write-only analog output
  * `av`: readable and writable analog value
  * `bi`: read-only binary input
  * `bo`: write-only binary output
  * `bv`: readable and writable binary value
  * `ei`: read-only enumerated input
  * `eo`: write-only enumerated output
  * `ev`: readable and writable enumerated value

And `suffix` is opaque and internal to the platform, as long as the fully
qualified id can be uniquely identified. By convention any fragments should
be dot-separated (`.`) for consistency.

Some examples:

    av.101
    ai.1b75
    bo.foo.bar

## Values

The `/values` endpoint provides current values for points under a source.  It
takes a required `source_id` argument.

Request:

    POST /jasper/v1/values HTTP/1.1
    Content-Type: application/x-www-form-urlencoded

    source_id=620

Response:

    {
      "values": [
        { "addr":"av.DamperPosition", "val":72.0 },
        { "addr":"bv.FanStatus", "val":1 },
        { "addr":"av.ZoneTemp", "val":72.0 },
        { "addr":"ao.ReturnTemp", "val":73.142 },
        { "addr":"ao.DischargeTemp", "val":68.230 }
      ]
    }

Where `val` is one of:

  * If the current value is available and can be read, a `Number` instance.
    For boolean values `false` is `0` and `true` is `1`.

  * If the point exists but the value was unable to be read, then the string
    value `"na"`.

  * If the point does not exist or there is no current value supported, then
    the `null` value.

## Write

The `/write` endpoint allows writing back to a system point. The arguments are:

  * Required `source_id` of source
  * Required `point_addr` of point
  * Required `val` to write to given `point_addr`
  * Optional priority `level`

Where the provided `val` is a floating point number. For boolean values
`false` is `0` and `true` is `1`.

For systems that support priority levels, the `level` argument is used to
specify the write level. To clear a given `level` pass the string `null` as
the `val` argument.

Request:

    POST /jasper/v1/write HTTP/1.1
    Content-Type: application/x-www-form-urlencoded

    source_id=620&point_addr=av.DamperPosition&val=50.0

Response:

    {
      "status": "ok"
    }

## Batch

The `/batch` endpoint allows invoking multiple operations in a single request.
The batch endpoint differs in that it takes a JSON body describing the commands
to invoke:

Request:

    POST /jasper/v1/batch HTTP/1.1
    Content-Type: application/json

    {
      "ops": [
        { "op":"about" },
        { "op":"values", "source_id":"54d" },
        { "op":"values", "source_id":"621" }
      ]
    }

Response:

    {
      "results": [
        { "name": "Device 123", "vendor": "ACME", ... },
        { "values": [...] },
        { "values": [...] },
      ]
    }

Each item in the `"ops"` list models a "request", where the required `"op"`
key captures the endpoint, and the remaining keys map to the endpoint
arguments.  All argument values must be string types for consistency.