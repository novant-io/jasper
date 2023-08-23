# Jasper

Jasper is a simple JSON-based format for interacting with IoT platforms. It
supports broadcast discovery, system meta-data, and live value reading.

## Spec

  - [Protocol](jasper-spec-protocol.md)
  - [Discovery](jasper-spec-discovery.md)

## Example

    $ curl host/jasper/v1/about -XPOST -u username:password

    {
      "name": "Device 123",
      "vendor": "ACME",
      "model": "MX-100",
      "version": "1.2"
    }

