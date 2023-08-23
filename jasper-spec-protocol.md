# Jasper Protocol

Jasper is a simple JSON-based format for interacting with IoT platforms.

## Endpoints

The following HTTP endpoints are supported:

     /jasper/v1/about   - Get information about this system
     /jasper/v1/sources - Get point sources from this system
     /jasper/v1/points  - Get point information for a source
     /jasper/v1/values  - Get current value data from points

## About

The `/about` endpoint provides meta-data information about the remote system.

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

    {
      "sources": [
        {
          "id":   "54d",
          "name": "Chiller",
          "path": "/Drivers/NiagaraNetwork/JACE-02/points/Chiller",
        },
        {
          "id":   "620",
          "name": "VAV-1",
          "path": "/Drivers/NiagaraNetwork/JACE-05/points/VAV-1",
        },
        {
          "id":   "621",
          "name": "VAV-2",
          "path": "/Drivers/NiagaraNetwork/JACE-05/points/VAV-2",
        },
        {
          "id":   "622",
          "name": "VAV-3",
          "path": "/Drivers/NiagaraNetwork/JACE-05/points/VAV-3",
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
          "name": "Zone Temp"
          "unit": "°F"
        },
        {
          "addr": "ao.ReturnTemp",
          "name": "Return Temp"
          "unit": "°F"
        },
        {
          "addr": "ao.DischargeTemp",
          "name": "Discharge Temp"
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
