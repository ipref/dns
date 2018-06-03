# DNS for IPREF
IPREF uses a combination of an IP _address_ and a _reference_ to refer to hosts. Address and reference are separated by a '+' which may be surrounded by white spaces.

    IP '+' REF

For example:

    10.247.1.1 + 12345
    gw.example.com + 6789a
    
# IPREF Address Format

References are opaque unsigned integer values. Different representations are used.

Default representation is a hexadecimal string.

    10.247.1.1 + 5123457        -- interpreted as HEX
    10.247.1.1 + abd259e

  Hex string may be separated into fields using dashes. Dashes are ignored.

    10.247.1.1 + 51-23457
    10.247.1.1 + ab-d259e-24171
    10.247.1.1 + 235a-2156-bcd1                        -- 16 bit fields
    10.247.1.1 + 123e4567-e89b-12d3-a456-426655440000  -- UUID style

Unsigned decimal references include mandatory commas. Commas are ignored.

    10.247.1.1 + 0,85                   -- decimal 85
    10.247.1.1 + 12,345,136,118         -- classic thousands separator

Dotted decimal resembles IPv4 notation but it's not fixed to four positions.

    10.247.1.1 + 0.85                   -- single byte reference
    10.247.1.1 + 10.236.228.4.18        -- five byte reference
    10.247.1.1 + 28.48.236.172          -- four byte reference (not an IP address)

Definition of IPREF address using Augmented BNF

    ipref = ip *WSP "+" *WSP ref
    
    ip =  dotted-decimal-string
    ip =/ dns-name
    
    dns-name = 1*(ALPHA / DIGIT) *( "-" 1*(ALPHA / DIGIT))
    
    ref =  hex-string
    ref =/ decimal-string
    ref =/ dotted-decimal-string

    hex-string =            1*HEXDIG *(*1("-") HEXDIG)  ; dash optional, no spaces
    decimal-string =        1*DIGIT 1*("," 1*DIGIT)     ; at least one comma, no spaces
    dotted-decimal-string = 1*DIGIT 1*("." 1*DIGIT)     ; at least one dot, no spaces
