# Jaspser Protocol

Jasper is a simple JSON-based format for interacting with IoT platforms.

## Endpoints

The following HTTP endpoints are supported:

     /jasper/v1/about  - Get information about this system
     /jasper/v1/points - Get the point information from this system
     /jasper/v1/values - Get the current value data from this system

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

## Points

The `/points` endpoint provides meta-data information about the available
points in this system.

    {
      "points": [
        {
          "addr": "av.1b6b",
          "name": "SetpointTemp",
          "unit": "°F"
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/SetpointTemp",
        },
        {
          "addr": "bv.1b75",
          "name": "FanStatus",
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/FanStatus"
        },
        {
          "addr": "av.1b6d",
          "name": "HeatingCoil",
          "unit": "%"
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/HeatingCoil",
        },
        {
          "addr": "eo.1b83",
          "name": "OccStatus",
          "enum": "occupied,unoccupied"
          "path": "/PxHome/Graphics/Campus/Building/Floor1/OccStatus",
        }
      ]
    }

Where each point object has:

  * `addr`: the unique addr for this point in the system (see [Point Addr](#point-addr) below)
  * `name`: the human readable name of this point
  * `path`: the optional path for where this point exists in the system
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

### Max Points Limit

Results can be restreted to a limit using `max_points` request argument:

    POST /jasper/v1/points

    max_points=2

    {
      "points": [
        {
          "addr": "av.1b6b",
          "name": "SetpointTemp",
          "unit": "°F"
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/SetpointTemp",
        },
        {
          "addr": "bv.1b75",
          "name": "FanStatus",
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/FanStatus"
        },
      ]
    }

### Path Prefix Filter

Results can be restricted to a system sub-path using the `path_prefix` request
argument:

    POST /jasper/v1/points

    path_prefix=/PxHome/Graphics/Campus/Building/Floor1/VavZoneC

    {
      "points": [
        {
          "addr": "av.1b6b",
          "name": "SetpointTemp",
          "unit": "°F"
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/SetpointTemp",
        },
        {
          "addr": "bv.1b75",
          "name": "FanStatus",
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/FanStatus"
        },
        {
          "addr": "av.1b6d",
          "name": "HeatingCoil",
          "unit": "%"
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/HeatingCoil",
        },
      ]
    }

## Values

The `/values` endpoint provides current values for points in this system.

    {
      "values": [
        { "addr":"av.1b6b", "val":72.3 },
        { "addr":"bv.1b75", "val":1 },
        { "addr":"av.1b6d", "val":25 },
        { "addr":"eo.1b83", "val":1 }
      ]
    }

Where `val` is one of:

  * If the current value is available and can be read, a `Number` instance.
    For boolean values `false` is `0` and `true` is `1`.

  * If the point exists but the value was unable to be read, then the string
    value `"na"`.

  * If the point does not exist or there is no current value supported, then
    the `null` value.

### Path Prefix Filter

Results can be restricted to a system sub-path using the `path_prefix` request
argument:

    POST /jasper/v1/values

    path_prefix=/PxHome/Graphics/Campus/Building/Floor1/VavZoneC

    {
      "values": [
        { "addr":"av.1b6b", "val":72.3 },
        { "addr":"bv.1b75", "val":1 },
        { "addr":"av.1b6d", "val":25 }
      ]
    }