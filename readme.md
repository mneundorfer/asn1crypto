# asn1crypto

A fast, pure Python library for parsing and serializing ASN.1 structures. In
addition to an ASN.1 BER/DER decoder and DER serializer, the project includes
a bunch of ASN.1 structures for use with various common cryptography standards:

| Standard               | Module              | Source                                                                                                                 |
| X509                   | `asn1crypto.x509`   | [RFC5280](https://tools.ietf.org/html/rfc5280)                                                                         |
| CRL                    | `asn1crypto.crl`    | [RFC5280](https://tools.ietf.org/html/rfc5280)                                                                         |
| OCSP                   | `asn1crypto.ocsp`   | [RFC6960](https://tools.ietf.org/html/rfc6960)                                                                         |
| PKCS#12                | `asn1crypto.pkcs12` | [RFC7292](https://tools.ietf.org/html/rfc7292)                                                                         |
| PKCS#8                 | `asn1crypto.keys`   | [RFC5208](https://tools.ietf.org/html/rfc5208)                                                                         |
| PKCS#1 v2.1 (RSA keys) | `asn1crypto.keys`   | [RFC3447](https://tools.ietf.org/html/rfc3447)                                                                         |
| DSA keys               | `asn1crypto.keys`   | [RFC3279](https://tools.ietf.org/html/rfc3279)                                                                         |
| Elliptic curve keys    | `asn1crypto.keys`   | [SECG SEC1 V2](http://www.secg.org/sec1-v2.pdf)                                                                        |
| PKCS#5 v2.1            | `asn1crypto.pkcs5`  | [PKCS#5 v2.1](http://www.emc.com/collateral/white-papers/h11302-pkcs5v2-1-password-based-cryptography-standard-wp.pdf) |
| CMS (and PKCS#7)       | `asn1crypto.cms`    | [RFC5652](https://tools.ietf.org/html/rfc5652), [RFC2315](https://tools.ietf.org/html/rfc2315)                         |
| TSP                    | `asn1crypto.tsp`    | [RFC3161](https://tools.ietf.org/html/rfc3161)                                                                         |
| PDF signatures         | `asn1crypto.pdf`    | [PDF 1.7](http://wwwimages.adobe.com/content/dam/Adobe/en/devnet/pdf/pdfs/PDF32000_2008.pdf)                           |

## License

*asn1crypto* is licensed under the terms of the MIT license. See the
[LICENSE](LICENSE) file for the exact license text.

## Dependencies

Python 2.7, 3.3, 3.4 or pypy. *No third-party packages required.*

## Version

0.9.0 - [changelog](changelog.md)

## Installation

```bash
pip install asn1crypto
```

## Documentation

 - `asn1crypto.core`
 - `asn1crypto.algos`
 - `asn1crypto.keys`
 - `asn1crypto.x509`
 - `asn1crypto.crl`
 - `asn1crypto.ocsp`
 - `asn1crypto.pkcs5`
 - `asn1crypto.pkcs12`
 - `asn1crypto.cms`
 - `asn1crypto.tsp`
 - `asn1crypto.pdf`

## Why Another Python ASN.1 Library?

Python has long had the [pyasn1](https://pypi.python.org/pypi/pyasn1) and
[pyasn1_modules](https://pypi.python.org/pypi/pyasn1-modules) available for
parsing and serializing ASN.1 structures. While the project does include a
comprehensive set of tools for parsing and serializing, the performance of the
library can be very poor, especially when dealing with bit fields and parsing
large structures such as CRLs.

After spending extensive time using *pyasn1*, the following issues were
identified:

 1. Poor performance
 2. Verbose, non-pythonic API
 3. Out-dated and incomplete definitions in *pyasn1-modules*
 4. No simple way to map data to native Python data structures
 5. No mechanism for overriden universal ASN.1 types

The *pyasn1* API is largely method driven, and uses extensive configuration
objects and lowerCamelCase names. There were no consistent options for
converting types of native Python data structures. Since the project supports
out-dated versions of Python, many newer language features are unavailable
for use.

Time was spent trying to profile issues with the performance, however the
architecture made it hard to pin down the primary source of the poor
performance. Attempts were made to improve performance by utilizing unreleased
patches and delaying parsing using the `Any` type. Even with such changes, the
performance was still unacceptably slow.

Finally, a number of structures in the cryptographic space use universal data
types such as `BitString` and `OctetString`, but interpret the data as other
types. For instance, signatures are really byte strings, but are encoded as
`BitString`. Elliptic curve keys use both `BitString` and `OctetString` to
represent integers. Parsing these structures as the base universal types and
then re-interpreting them wastes computation.

*asn1crypto* uses the following techniques to improve performance, especially
when extracting one or two fields from large, complex structures:

 - Delayed parsing of byte string values
 - Persistence of original ASN.1 encoded data until a value is changed
 - Lazy loading of child fields
 - Utilization of high-level Python stdlib modules

While there is no extensive performance test suite, the
`CRLTests.test_parse_crl` test case was used to parse a 21MB CRL file on a
late 2013 rMBP. *asn1crypto* parsed the certificate serial numbers in just
under 8 seconds. With *pyasn1*, using definitions from *pyasn1-modules*, the
same parsing took over 4,100 seconds.

For smaller structures the performance difference can range from a few times
faster to an order of magnitude of more.

