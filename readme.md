# Jasper

Jasper is a simple JSON-based format for interacting with IoT platforms.

## Endpoints

The following HTTP endpoints are supported. A prefix may precede any of these
URI's (for example: `/jasper/v1/points`).

     /v1/about  - Get information about this system
     /v1/points - Get the point information from this system
     /v1/values - Get the current value data from this system

## About

The `/about` endpoint provides meta-data information about the remote system.

    {
      "vendor": "ACME",
      "model": "MX-100"
    }

Where:

  * `vendor`: the vendor of this system
  * `model`:  the model of this system

Additional system specific information may be returned.

## Points

The `/points` endpoint provides meta-data information about the available
points in this system.

    {
      "size": 3,
      "points": [
        {
          "id":   "av.1b6b",
          "name": "SetpointTemp",
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/SetpointTemp",
          "unit": "°F"
        },
        {
          "id":   "bv.1b75",
          "name": "Occupied",
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/Occupied"
        },
        {
          "id":   "av.1b6d",
          "name": "HeatingCoil",
          "path": "/PxHome/Graphics/Campus/Building/Floor1/VavZoneC/HeatingCoil",
          "unit": "%"
        }
      ]
    }

Where each point object has:

  * `id`: the unique id for this point in the system (see [Point ID](#point-id) below)
  * `name`: the human readable name of this point
  * `path`: the optional path for where this point exists in the system
  * `unit`: the optional unit for this point

<a id="point-id"></a>
### Point ID

Each point must have a unique identifier, where the format is:

    <type>.<suffix>

Where `type` must be one of:

  * `ai`: read-only analog input
  * `ao`: write-only analog output
  * `av`: readable and writable analog value
  * `bi`: read-only binary input
  * `bo`: write-only binary output
  * `bv`: readable and writable binary value

And `suffix` is opaque and internal to the platform, as long as the fully
qualified id can be uniquely identified.  By convention any fragments should
be dot-separated (`.`) for consistency.

Some examples:

    av.101
    ai.1b75
    bo.foo.bar

## Values

The `/values` endpoint provides current values for points in this system.

    {
      "size": 3,
      "values": [
        { "id":"av.1b6b", "val":72.3 },
        { "id":"bv.1b75", "val":1 },
        { "id":"av.1b6d", "val":25 }
      ]
    }

Where `val` is one of:

  * If the current value is available and can be read, a `Number` instance.
    For boolean values `false` is `0` and `true` is `1`.

  * If the point exists but the value was unable to be read, then the string
    value `"NaN"`.

  * If the point does not exist or there is no current value supported, then
    the `null` value.
