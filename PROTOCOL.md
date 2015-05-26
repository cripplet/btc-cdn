# BTC-CDN Protocol

Goals
----

* utilize the 40 bytes alloted to each BTC transaction using the `OP_RETURN` script
* allow storage of arbitrarily large data

VERSION 0.2
----

The 40 bytes of the `OP_RETURN` metadata is split accordingly:

```
---------------
|  1   |  39  | // byte length
|-------------|
| HEAD | DATA |
---------------
```

Where the one-byte `HEADER` is comprised of the `VERSION` and `COMMAND` fields:

```
-----------
| 3 |  5  | // bit length
|---------|
| V | CMD |
-----------
```

So that multiple accounts can share the same drop-off, the interpretation of the metadata (including commands) will be on an address-by-address basis. That is, we shall treat every `SOURCE:DESTINATION` address pairing as a unique account and interpret any data sent to `DESTINATION` from `SOURCE` as a continuation of previous `SOURCE:DESTINATION transactions.` For the sake of simplicity, this means that in any given transaction, there must only be *one* unique `SOURCE` address.

VERSION
----

* reserve `VERSION == \b111` for future protocol revisions, where the data length fields will be different
* current `VERSION == \b000`

COMMAND
----

`\b1xxxx` is reserved for SEND-related commands, and `\b0xxxx` is reserved for auxiliary commands.

```
---------------------------------------------------
| COMMAND   | bytecode | description              |
|-------------------------------------------------|
| MESSAGE   | \b10000  | file data                |
| FILESTART | \b10001  | new file                 |
| FILETERM  | \b10010  | end file                 |
| TERMACCT  | \b00001  | this account has expired |
---------------------------------------------------
```

* Note that we can signify a short message by ORing `MESSAGE | FILESTART | FILETERM`, saving an transaction and associated confirmation fees.

DATA
----

```
-----------------------------------------
| COMMAND   | DATA FORMAT (bytes)       |
|---------------------------------------|
| MESSAGE   | COUNTER (4) | STRING (35) |
| FILESTART | COUNTER (4) | STRING (35) |
| FILETERM  | COUNTER (4) | STRING (35) |
| TERMACCT  | ACCOUNT (36) or NULL      |
-----------------------------------------
```

* `COUNTER` is a 0-indexed log which keeps track of the last-known file data message. `COUNTER` initializes to 0, should not skip any integer, and must not repeat. `COUNTER` is tracked by the uploader.
* In the case that `COUNTER == \Uffffffff`, we can call `TERMACCT` on a new BTC address and continue serving a file to the new address, thereby allowing arbitrarily large storage.

Example
----

* Sending "Hello World!" using `VERSION == \b000` in the first message of the account (i.e. `COMMAND == \b10011`, `COUNTER == \U00000000`):

```
0x130000000048656c6c6f20576f726c6421
```
