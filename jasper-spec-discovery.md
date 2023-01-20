# Jasper Discovery

Jasper supports local subnet discovery using UDP broadcast messages.

The well known port number for discovery is `9410`. The max size of Jasper
UDP packets is `1476` bytes.

## Client Request

A Jasper `who-is` discovery request payload format:

      7   6   5   4   3   2   1   0
    |---|---|---|---|---|---|---|---|
    |       0       |  dx_version   |  High bits reserved and always 0
    |---|---|---|---|---|---|---|---|
    |            op_code            |
    |---|---|---|---|---|---|---|---|

Where:

  * `dx_version` is UDP protocol version and currently always `1`
  * `op_code` is `0x01` to indicate `who-is`


## Source Response

A Jasper `i-am` discovery response payload format:

      7   6   5   4   3   2   1   0
    |---|---|---|---|---|---|---|---|
    |       0       |   dx_version  |  High bits reserved and always 0
    |---|---|---|---|---|---|---|---|
    |            op_code            |
    |---|---|---|---|---|---|---|---|
    |tls|    auth   |    api_ver    |
    |---|---|---|---|---|---|---|---|
    |         api_port_high         |  High 8-bits of 16-bit API port number
    |---|---|---|---|---|---|---|---|
    |         api_port_low          |  Low 8-bits of 16-bit API port number
    |---|---|---|---|---|---|---|---|
    |           name_len            |  Max value = 0x7f (127)
    |---|---|---|---|---|---|---|---|
    |                               |
    |            name *             |
    |                               |
    |---|---|---|---|---|---|---|---|
    |           vendor_len          |  Max value = 0x7f (127)
    |---|---|---|---|---|---|---|---|
    |                               |
    |            vendor *           |
    |                               |
    |---|---|---|---|---|---|---|---|
    |           model_len           |  Max value = 0x7f (127)
    |---|---|---|---|---|---|---|---|
    |                               |
    |            model *            |
    |                               |
    |---|---|---|---|---|---|---|---|
    |          version_len          |  Max value = 0x7f (127)
    |---|---|---|---|---|---|---|---|
    |                               |
    |            version *          |
    |                               |
    |---|---|---|---|---|---|---|---|

Where:

  * `dx_version` is UDP protocol version and currently always `1`
  * `op_code`    is `0x02` to indicate `i-am`
  * `tls` is:
      - `0`: HTTP is allowed for this source
      - `1`: HTTPS is required for this source
  * `auth` is:
      - `0x00`: no authenticaion is required for this source
      - `0x01`: HTTP Basic authentican is required for this source
  * `api_ver`  is API version and currently always `1`
  * `api_port` is the TCP port # used for API requests
  * `name`     is human readable name of source
  * `vendor`   is vendor name of source
  * `model`    is model name of source
  * `version`  is software version of this source