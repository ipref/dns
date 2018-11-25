# DNS for IPREF
IPREF uses a combination of an IP _address_ and a _reference_ to refer to hosts. Address and reference are separated by a '+' which may be surrounded by white spaces.

    IP '+' REF

For example:

    10.247.1.1 + 12345
    gw.example.com + 6-789a

# IPREF Address Format

References are opaque unsigned integers. Different representations may be used.

Default representation is a decimal string.

    10.247.1.1 + 5123457

Hex representation uses dashes to separate fields in reference strings. Dashes are ignored.

    10.247.1.1 + 51-23457
    10.247.1.1 + ab-d259e-24171
    10.247.1.1 + 235a-2156-bcd1                        -- 16 bit fields
    10.247.1.1 + 123e4567-e89b-12d3-a456-426655440000  -- UUID style

Decimal representation may use commas. Commas are optional and are ignored.

    10.245.2.2 + 85                     -- standard decimal 85
    10.247.1.1 + 0,85                   -- decimal 85
    10.247.1.1 + 12,345,136,118         -- classic thousands separator

Dotted decimal representation resembles IPv4 notation but it's not fixed to four positions.

    10.247.1.1 + 0.85                   -- single byte reference
    10.247.1.1 + 10.236.228.4.18        -- five byte reference
    10.247.1.1 + 28.48.236.172          -- four byte reference (not an IP address)

Commas, dashes, and periods must be surrounded by digits on both sides.

Definition of IPREF address in Augmented BNF:

    ipref = ip *WSP "+" *WSP ref

    ip =  dotted-decimal-string
    ip =/ dns-name

    dns-name = 1*( ALPHA / DIGIT ) *( "-" 1*(ALPHA / DIGIT) )

    ref =  decimal-string
    ref =/ hex-string
    ref =/ dotted-decimal-string

    decimal-string =        1*DIGIT *(*1(",") DIGIT)    ; comma optional, no spaces
    hex-string =            1*HEXDIG 1*("-" 1*HEXDIG)   ; at least one dash, no spaces
    dotted-decimal-string = 1*DIGIT 1*("." 1*DIGIT)     ; at least one dot, no spaces

# DNS  Resource Records

IPREF needs a dedicated record type. The plan is to register type AA with IANA. The syntax of an AA record would be similar to the syntax of an A record but it would provide for the reference portion. A sample entry would look like this:

    host1.example.com.  IN  AA   gw.example.com + 25b7-2345
    host2               IN  AA   gw.example.com + 2,347,275,120
    host3               IN  AA   10.247.1.1 + c184-234f-980a

As a workaround for an unavailable record type, IPREF uses TXT records to embed the contents of AA records. For example the above entries as TXT records look like this:

    host1.example.com.  IN  TXT  "AA gw.example.com + 25b7-2345"
    host2               IN  TXT  "AA gw.example.com + 2,347,275,120"
    host3               IN  TXT  "AA 10.247.1.1 + c184-234f-980a"

An IPREF resolver must issues queries for A, AAAA, and AA records, then prefer AA over A/AAAA. Typically, A/AAAA would not return if AA is found but resolver cannot rely on that behavior.

# Static IPREF address mapping

IPREF can use /etc/hosts to lookup static address mappings in lieu of, or in addition to, DNS queries. The syntax of /etc/hosts is extended to provide IPREF mappings. The extension is backward compatible. IPREF mappings are placed in comments and are marked by a '#' immediately followed by '='. A comment is still allowed at end of lines.

    IP [HOSTNAME]... [#= IPREF_MAPPING]

For example:

    192.168.1.174   host1.example.com
    192.168.2.87    host21              #= public + 2-bf47   # implicit gateway
    10.247.1.228    ext-host32          #= external gw.example.org + 2b78-2e89

A mapping entry may designate a host as local, i.e. not visible externally, or it may designate a host as public, in which case the host will be accessible externally via an allocated IPREF address.  The allocation may be implicit where the ip portion and the reference portion of the IPREF address is left to the system to assign. It may also be explicit where the reference, or both ip address and the reference, is listed directly in the entry. An entry may also describe an external IPREF address with its associated local encoded address by which local host may reach it.

The different mapping options are indicated by keywords such as _public_, _local_, or _external_. Keywords may be shortened to their first three letters.

The _local_ entries describe hosts that are local, inaccessible externally. The use of _local_ keyword, instead of omitting mapping, is sometimes convenient when switching host visibility from  public to local.

    192.168.1.174   host11 host1.example.com
    192.168.1.175   host12                      #= local
    192.168.1.177   host14                      #= loc

The _public_ entries describe hosts that are public, accessible externally via their IPREF addresses. The mapping describes how the hosts can be accessed externally. The gateway may be omitted if there is only one in the local network. These entries are not published via DNS. They're used for testing or for simple setups where DNS is not available or not used.

    192.168.2.88    host22                      #= public + 23a8-435cd
    192.168.2.89    host23 host23.example.com   #= pub gw.example.com + 76ca-b861

These _external_ entries describe hosts residing on other local networks which are accessible via IPREF addresses. These entries are used if DNS is not available or not used. The listed ip addresses refer to encoded addresses by which the external hosts can be reached from local hosts.

    10.247.1.228                                #= external gw.example.org + 2b78-2e89
    10.251.19.181   ext-host1                   #= ext      gw.example.org + 46c8-a104

Syntax for static mapping entries in Augmented BNF:

    mapping = host-part [ map-part ]

    host-part = ip *( 1*WSP hostname )

    ip = dotted-decimal
    hostname = dns-name

    map-part = *WSP "#=" *WSP [ ( local-map / public-map / external-map ) ]

    local-map =   *1( "local" / "loc" )

    public-map = ( "public" / "pub" ) 1*WSP [ ip ] *WSP "+" *WSP ref

    external-map = ( "external" / "ext" ) 1*WSP ip *WSP "+" *WSP ref
