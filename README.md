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

# DNS  Resource Records

IPREF needs a dedicated record type. The plan is to register type AA with IANA. The syntax of an AA record would be similar to the syntax of an A record but it would provide for the reference portion. A sample entry would look like this:

    host1.example.com.  1800  AA   gw.example.com + 25b7-2345
    host2               1800  AA   gw.example.com + 2,347,275,120
    host3               1800  AA   10.247.1.1 + c184-234f-980a

As a workaround for an unavailable record type, IPREF uses TXT records to embed the contents of AA records. For example the above entries as TXT records look like this:

    host1.example.com.  1800  TXT  "AA gw.example.com + 25b7-2345"
    host2               1800  TXT  "AA gw.example.com + 2,347,275,120"
    host3               1800  TXT  "AA 10.247.1.1 + c184-234f-980a"

An IPREF resolver must issues queries for A, AAAA, and AA/TXT records, then prefer AA/TXT over A/AAAA. Typically, A/AAAA would not return if AA/TXT is found but resolver cannot rely on that behavior.
