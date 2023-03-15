CS594 \
Internet Draft \
Intended Status: Chess Class Project Specification \
Expires: April 2023

<h1 style="text-align: center;">Networked Chess Class Project</h1>

## Summary
[summary]: #summary

This memo describes the communication protocol for a networked chess client/server
system for the Internetworking Protocols class at Portland State University.

## Table of Contents
1. [Introduction][intro]
2. [Basic Information][basic-info]
3. [Message Infrastructure][msg-infra] \
   3.1. [Message Format][msg-format] \
   3.2. [Field Definitions][field-def] \
   3.3. [Operation Codes][op-codes] \
   3.4. [Error Messages][err-msgs] \
   3.5 [Keepalive Messages][keepalive-msgs]



## Introduction
[intro]: #introduction
---

This specification describes a simple Networked Chess protocol by which clients
can play chess games against one another. This system employs a central server which relays
the board position and moves played to other connected clients.

Clients can join chess games, which are a pair of clients subscribed to the same game move stream. Any move played on that game will be forwarded to the opponent of that game.

## Basic Information
[basic-info]: #basic-info
---

All communication descibed in this protocol takes place over TCP/IP, with the server
listening for connections on port 8088. Clients connect to this port and maintain a persistent
connection to the server. The client can send requests to the server over this open channel,
and the server can reply over the same channel. This protocol is asynchronous, as the client
is able to send messages to the server at any time, and the server may reply back.

The server has a user limitation and will only allow a finite amount of users and games at this
time. The server will notify users with an error code if they attempt to connect when the server is at its capacity. The server and client may choose to terminate the connection at any time for any reason.

## Message Infrastructure
[msg-infra]: #message-infrastructure
---

### Message Format
[msg-format]: #message-format
```
struct Packet {
    opcode: u8,
    length: u8,
    payload: [u8; self.length]
}
```

### Field Definitions
[field-def]: #field-definitions

- `opcode`: specifies what kind of message is contained in the payload
- `length`: specifies how many bytes are contained in the payload
- `payload`: variable length payload

### Operation Codes (opcodes)
[op-codes]: #operation-codes

| Op Code | Byte Value |
|---------|------------|
| CHESS_OPCODE_ERR | 0x00 |
| CHESS_OPCODE_KEEPALIVE | 0x01 |
| CHESS_OPCODE_JOIN | 0x02 |
| CHESS_OPCODE_LIST_GAMES | 0x03 |
| CHESS_OPCODE_LIST_GAMES_RESP | 0x04 |
| CHESS_OPCODE_CREATE_GAME | 0x05 |
| CHESS_OPCODE_JOIN_GAME | 0x06 |
| CHESS_OPCODE_LEAVE_GAME | 0x07 |
| CHESS_OPCODE_SEND_MOVE | 0x08 |
| CHESS_OPCODE_RECV_MOVE | 0x09 |
| CHESS_OPCODE_SEND_MSG | 0x0A |

---
### Error Messages
[err-msgs]: #error-messages
```
Packet {
    opcode: 0x00,
    length: 1,
    payload: [error_code]
}
```
### Usage
[err-usage]: #error-usage
Error message may be sent by either the client of the server prior to closing the TCP connection. If the client or the server receives this message, the connection should
be considiered terminated.


### Error Codes
| Op Code | Byte Value |
|---------|------------|
| CHESS_ERR_UNK | 0x00 |
| CHESS_ERR_ILLEGAL_OPCODE | 0x01 |
| CHESS_ERR_ILLEGAL_LEN | 0x02 |
| CHESS_ERR_NAME_EXISTS | 0x03 |
| CHESS_ERR_ILLEGAL_MOVE| 0x04 |
| CHESS_ERR_TOO_MANY_USERS | 0x05 |
| CHESS_ERR_TOO_MANY_GAMES | 0x06 |

---
### Keep Alive Messages
[keepalive-msgs]: #keepalive-messages
```
Packet {
    opcode: 0x01,
    length: 1,
    payload: [0xFF]
}
```
### Usage
[keepalive-usage]: #keepalive-usage
Provides an application-layer notification of a disconnected peer. 
Must be sent at least once every 5 seconds by both client and server 
to notify the other party that they are still connected.

Connection will be terminated upon missing 5 keep alive check-ins.

## Authors
Kyle Esquerra [<img style="width: 20px" alt="Github URL" src="https://raw.githubusercontent.com/gilbarbara/logos/1f372be75689d73cae89b6de808149b606b879e1/logos/github-icon.svg">](https://github.com/kesquerra)