# **LineBasedTime Protocol, version 1**
This file describes, how the first version of LineBasedTime protocol works. It is a simple open one-way line-based text protocol that can work over any reliable connection-based transport protocol (preferably TCP) and is designed to fetch time information from a server by remote clients over a network.

Its open-source reference server implementation can be found [here](https://github.com/ethernetlord/linebasedtime-server) and its client implementation [here](https://github.com/ethernetlord/linebasedtime-client).

---

## **Description**
The client initiates a TCP connection to the server. After the connection has been established, the server speaks with an ASCII message in the following format:
```
VER: (uint8_t as char[3])\n
ERR: (char)\n
UNX: (uint64_t as char[20])\n
UTC: (char[24])\n
LOC: (char[24])\n
```
The ```\n``` are single-character newline escape sequences, not literal *\n*. The char arrays' lengths DO NOT include space for a terminating null character.

The ```UNX```, ```UTC``` and ```LOC``` fields are NOT MANDATORY, if the server encounters an error and sets the ```ERR``` field to **Y**.


* ```VER```: The protocol version, encoded as a variable-length numeric string (between 1 and 3 characters) with values between 0 and 255 *(unsigned 8 bit integer)*. The current and only version is **1**.

* ```ERR```: A single character., that indicates, whether the server encountered an error while processing the client's request. **Y** when an error occurred, **N** when not.

* ```UNX```: The current Unix time (UTC) on the server's side (the number of elapsed seconds since Jan 1, 1970 00:00:00 UTC), encoded as a variable-length numeric string (between 1 and 20 characters) with values between 0 and 18446744073709551615 *(unsigned 64 bit integer)*.

* ```UTC```: A fixed-length 24-character date and time string formatted as **Www Mmm dd hh:mm:ss yyyy**, as produced by the C standard library's ```asctime()``` function *(excluding the trailing newline character)*, describing the current UTC time on the server's side in a human-readable form.

* ```LOC```: Same as ```UTC```, except it describes the current local time in the server's timezone, not the Coordinated Universal Time.

After the message has been sent by the server, it can end end the TCP connection on its side. Client can do the same, after it receives the message. The client NEVER speaks to the server.

The message length can be between **14 and 102 bytes** (```char```s; excluding a null terminating character, which isn't transmitted by the protocol itself, but can possibly be appended on the  client's side).

---

## **Examples**
### Example 1
*Situation:* The server can successfully process the client's request, with the server being in the UTC+1 timezone with the DST currently active:
```
C: <opens TCP connection to server>
S: <confirms incoming TCP connection>
S: VER: 1
S: ERR: N
S: UNX: 1592769880
S: UTC: Sun Jun 21 20:04:40 2020
S: LOC: Sun Jun 21 22:04:40 2020
S: <closes TCP connection>
C: <closes TCP connection>
```

### Example 2
*Situation:* The server couldn't process the request without an error on its side:
```
C: <opens TCP connection to server>
S: <confirms incoming TCP connection>
S: VER: 1
S: ERR: Y
S: <closes TCP connection>
C: <closes TCP connection>
```

---

## Author
Made by **EthernetLord**.
* Website: https://ethernetlord.eu/
* GitHub: @ethernetlord (https://github.com/ethernetlord/)
* Twitter: @ethernetlordcz (https://twitter.com/ethernetlordcz)
