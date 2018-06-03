# DNS for IPREF
IPREF uses a combination of an IP address and a reference to refer to hosts.

    IP '+' REF

For example:

    192.168.1.1 + 12345
    
# IPREF Address Format

References are opaque unsigned integer values. Different representations are used.

Default representation is a hexadecimal string.

    192.168.1.1 + 5123457       -- interpreted as HEX
    192.168.1.1 + abd259e

  Hex string may be separated into fields using dashes. Dashes are ignored.

    192.168.1.1 + 51-23457
    192.168.1.1 + ab-d259e-24171
    192.168.1.1 + 235a-2156-bcd1                        -- 16 bit fields
    192.168.1.1 + 123e4567-e89b-12d3-a456-426655440000  -- UUID style

Unsigned decimal references include mandatory commas. Commas are ignored.

    192.168.1.1 + 0,085                 -- decimal 85
    192.168.1.1 + 12,345,136,118        -- classic thousands separator

Dotted decimal resembles IPv4 notation but is not fixed to four positions.

    192.168.1.1 + 0.85                  -- single byte reference
    192.168.1.1 + 10.236.228.4.18       -- five byte reference
    192.168.1.1 + 28.48.236.172         -- four byte reference (not an IP address)
