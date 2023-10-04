# SmartPGP applet

SmartPGP is a free and open source implementation of the [OpenPGP card
3.4 specification](https://gnupg.org/ftp/specs/OpenPGP-smart-card-application-3.4.pdf) in JavaCard.


## Features

The following features are implemented at the applet level, but some
of them depend on underlying hardware support and available
(non-)volatile memory resources:

- RSA (>= 2048 bits modulus, 17 bits exponent) for signature, 
  encryption and authentication;

- On-board key generation and external private key import;

- PIN codes (user, admin and resetting code) up to 127 characters;

- Certificate up to 1 kB (DER encoded) for each key;

- Login, URL, and private DOs up to 256 bytes;

- Command and response chaining;

- AES 128/256 bits deciphering primitive;


## Default values

The SmartPGP applet is configured with the following default values:

- Admin PIN is 12345678;

- User PIN is 123456;

- No PUK (a.k.a. resetting code) is defined;

- RSA 2048 bits for PGP keys;

These values can be changed by modifying default values in the code
(see the [Constants](src/fr/anssi/smartpgp/Constants.java)
class).

When the applet is installed, one can use the `smartpgp-cli` utility
given in the `bin` directory to change these values. Keep in mind that
when you change the algorithm attributes of a PGP key, the key and the 
corresponding certificate are
erased. Also note that hard coded default values will be restored upon
a factory reset.


# Application support

Tokens following the OpenPGP card 3.4 specification are not yet fully
supported by most PGP applications.

## GnuPG

OpenPGP card 3.x is supported by [GnuPG](https://www.gnupg.org/)
starting from version 2.1.16.

## OpenKeychain

OpenPGP card 3.x is supported by [OpenKeychain](https://www.openkeychain.org/)
starting from version 4.2.


# Content of the repository

The repository contains several directories:

- `bin` contains a Python library and command line tool called
  `smartpgp-cli` to interact with an OpenPGP card 3.x;
  
- `src` contains the JavaCard source code of the SmartPGP applet;

- `videos` contains sample videos demonstrating smartcard interactions
  with OpenKeychain and K9 mail on Android Nexus 5.

## Build and installation instructions

### Prerequisites

- A Java compiler (No higher than OpenJDK 11 or equivalent)
- A device compliant with JavaCard 3.0.1 (or above) with enough available resources
  - Applet: ~23 KiB of non-volatile (eeprom/flash) memory
  - Persistant Data: ~10 KiB of non-volatile (eeprom/flash) memory
  - Transient Data: ~2 KiB of volatile (RAM) memory

<!-- - The [pyscard](https://pypi.org/project/pyscard/) and [pyasn1](https://pypi.org/project/pyasn1/)
  Python libraries for `smartcard-cli`. -->

### Importing RSA keys above 2048 bits

The default internal buffer that stores keys is configured with a default
value that is only large enough for RSA 2048 bit keys.

if your card is able to handle larger RSA key bit-lengths (3072 or 4096),
and you want to import those keys, you will need to increase the buffer size.

This can be accomplished by modifying `Constants.INTERNAL_BUFFER_MAX_LENGTH` in [Constants.java](src/dev/c0de/smartpgp/Constants.java)

#### RSA 2048 bit keys

When produced by OpenPGP, these keys are 949 Bytes in length.  
`Constants.INTERNAL_BUFFER_MAX_LENGTH` may not be smaller than `(short)0x3b6` (decimal: 950)

#### RSA 3072 bit keys

When produced by OpenPGP, these keys are 1294 Bytes in length.  
`Constants.INTERNAL_BUFFER_MAX_LENGTH` may not be smaller than `(short)0x50f` (decimal: 1295)

#### RSA 4096 bit keys

When produced by OpenPGP, these keys are 1644 Bytes in length.  
`Constants.INTERNAL_BUFFER_MAX_LENGTH` may not be smaller than `(short)0x66d` (decimal: 1645)

## Reducing flash and/or RAM consumption

The applet allocates all its data structures to their maximal size
at installation to avoid as much as possible runtime errors caused by
memory allocation failure. If your device does not have enough flash
and/or RAM available, or if you plan not to use some features
(e.g. stored certificates), you can adjust the applet to reduce its
resource consumption by tweaking the following variables:

- `Constants.INTERNAL_BUFFER_MAX_LENGTH`: the size in bytes of the
  internal RAM buffer used for input/output chaining. Chaining is
  especially used in case of long commands and responses such as those
  involved in private key import and certificate import/export;
  
- `Constants.EXTENDED_CAPABILITIES`, bytes 5 and 6: the maximal size
  in bytes of a certificate associated to a key. Following the OpenPGP
  card specification, a certificate can be stored for each of the
  three keys.


## Building the CAP file

- (Optional) Edit the `build.xml` file and replace the `APPLET_AID`
  with your a unique value. Alternatively, set the
  right AID instance bytes during applet installation.
  Generate the AID using [this tool](https://c0de.dev/c0de/SmartPGP-AID-Generator)

- Execute `ant` with no parameter to build `SmartPGPApplet.cap`

## Installing the CAP file

The CAP file installation depends on your device, so you have to refer
to the instructions given by your device manufacturer. Most open cards
relying on Global Platform with default keys are supported by
[GlobalPlatformPro](https://github.com/martinpaljak/GlobalPlatformPro).

Be careful to use a valid AID according to the OpenPGP card
specification (see section 4.2.1) for each card (`-create <AID>` with
GlobalPlatformPro)

Example Installation commands: 

- `gp --install SmartPGPApplet.cap --default`
- `gp --install SmartPGPApplet.cap --create <AID>`
