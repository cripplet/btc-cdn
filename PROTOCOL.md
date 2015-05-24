# BTC-CDN Protocol

Goals
----

* utilize the 40 bytes alloted to each BTC transaction using the OP_RETURN script
* allow storage of arbitrarily large data

Version 0.2
----

The 40 bytes of the OP_RETURN metadata is split accordingly:

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

VERSION
----

* reserve `VERSION == \b111` for future protocol revisions, where the data length fields will be different
* current `VERSION == \b000`

COMMAND
----

`\b0xxxx` is reserved for SEND-related commands, and `\b1xxxx` is reserved for auxiliary commands.

```
----------------------------------------------------
| COMMAND    | bytecode | description              |
|--------------------------------------------------|
| FILE       | \b00000  | file data                |
| FILESTART  | \b00001  | new file                 |
| FILETERM   | \b00010  | end file                 |
| SETADMIN   | \b10000  | set the admin address    |
| REDIR      | \b10100  | redirect data            |
| REDIRSTART | \b10001  | redirect data start      |
| REDIRTERM  | \b10110  | redirect data end        |
| TERMACCT   | \b11000  | this account has expired |
----------------------------------------------------
```

* `ADMIN_ADDRESS` is originally set to `''`, which means `ADMIN`-level commands may be sent from any address. Once `SETADMIN` is called, all future `ADMIN`-level transactions must originate from the corresponding address.
* Note that we can signify a short message by `OR`ing `FILE | FILESTART | FILETERM`, saving an transaction and associated confirmation fees.
* Similarly, we may `OR` `REDIR | REDIRSTART` or `REDIR | REDIRTERM` (but not all three simultaneously, due to data length).

DATA
----

```
------------------------------------------
| COMMAND    | DATA FORMAT (bytes)       |
|----------------------------------------|
| FILE       | COUNTER (4) | STRING (35) |
| FILESTART  | COUNTER (4) | STRING (35) |
| FILETERM   | COUNTER (4) | STRING (35) |
| SETADMIN   | ACCOUNT (36) or NULL      |
| REDIR      | COUNTER (4) | STRING (35) |
| REDIRSTART | COUNTER (4) | STRING (35) |
| REDIRTERM  | COUNTER (4) | STRING (35) |
| TERMACCT   | ACCOUNT (36) or NULL      |
------------------------------------------
```

* `SETADMIN` for each new address is originally set to `''`. Upon the first call to `SETADMIN`, an `ACCOUNT` must be provided to set an `ADMIN`. From then on, all `ADMIN`-level commands must be sent from the same `ACCOUNT`. To unset the `ACCOUNT`, `ACCOUNT` must send `SETADMIN` with a `NULL` payload.
* The payload for `REDIR` command is two transaction IDs separated by a single space: `FROM_TX TO_TX`. Each transaction ID must link to valid BTC-CDN files. Upon scanning from the original transaction ID, if a REDIR is reached, the decoder must discard the old data and look up the new file.
* `COUNTER` is a 0-indexed log which keeps track of the last-known file data message. `COUNTER` initializes to 0, should not skip any integer, and must not repeat. `COUNTER` is tracked by the uploader.
* In the case that `COUNTER == \Uffffffff`, we can call `TERMACCT` on a new BTC address and continue serving a file to the new address, thereby allowing arbitrarily large storage.

Example
----

* Sending "Hello World!" using `VERSION == \b000` in the first message of the account (i.e. `COMMAND == \b10011`, `COUNTER == \U00000000`):

```
0x130000000048656c6c6f20576f726c6421
```
