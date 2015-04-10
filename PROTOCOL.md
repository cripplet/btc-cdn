# btc-cdn (btcdn?)

Goals
----

* we know 80 bytes per btc tx
* we want (potentially) arbitrary storage
* anonyminifjeklwam;fewa


# VERSION 0.1
80 bytes split into:

PROTOCOL
----

VERSION:CMD:DATA

VERSION: 3 bits (8 versions...?)
CMD: 5 bits (32 commands)
DATA: CMD-dependent

VERSION
----

* reserve VERSION == 111 for future protocol revisions -- CMD/DATA split may be different in future
* current VERSION == 000

CMD
----

Envision the most-used command will be to SEND data. We want to ensure that as little of the DATA field will be used up for aux data.
1xxxx is reserved for SEND-related commands, 0xxxx is reserved for aux commands.

* MSG = 10000 -- basic message
* FILESTART = 10001 -- this is the first message of a file
* FILETERM = 10002 -- this is the last message of a file
* TRANFERACCT = (0xxxx) -- this account is now expired

DATA
----

* MSG: DATA = CNT:STRING, where CNT is a 4 byte counter starting at 0. CNT is tracked on host-side. STRING is 75 bytes long.
* FILESTART: same as MSG. CNT DOES NOT RESET
* FILETERM: same as MSG, except STRING may be less than 75 bytes. CNT DOES NOT RESET
* TRANFERACCT: DATA = BTCACCT (36 bytes) OR NULL (account term)

Examples
----

The first message sent MUST be (in bits)

[VERSION]:10001:[0 * 32]:[DATA]
