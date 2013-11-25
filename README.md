NeoNews Protocol
================

All protocol requests and responses are UTF-8.

Requests
--------

Requests begin with a

    NeoNews/<version> <hostname>

line.

The following line contains the verb and any required parameters.

Any following line, to a blank, may contain additional connection information.

### Verbs ###
* AUTH1 <username> <ephemeral>
* AUTH2 <username> <proof-of-session-key>
* CREATEUSER <username> <salt> <verifier>
* POSTMSG
* LIST <what> [SINCE <date>]
 * what: 'groups' or 'messages'
 * date is in ISO 8601 date
* GETMSG <message id>
* X-<verb> <options> ....
 * Implementation dependent

### Additional Headers ###

* AUTHPROOF <username> <ephemeral from auth> <encrypted server-ephemeral from auth> <iv> <session key> <otp from server> <iv>- Replay's the user's authentication cookie

Authentication
--------------

Authentication is done using [SRP(Secure Remote Password)](http://en.wikipedia.org/wiki/Secure_Remote_Password_protocol) ([RFC 2945](http://tools.ietf.org/html/rfc2945)) ([Official SRP Homepage](http://srp.stanford.edu/)).

The safe prime shall be:

The hash function shall be [SHA-256](http://tools.ietf.org/html/rfc4634).

The generator shall be 2.

The k paremeter shall be k = H(N, g).

Additionally, [TOTP (Time-based One-Time Passwords)](http://en.wikipedia.org/wiki/Time-based_One-time_Password_Algorithm) ([RFC 6238](http://tools.ietf.org/html/rfc6238)) will be used for session verification.

### Anonymous Access ###

Unregistered clients may use anon for both the username and password.

### Registration ###

To register a new user, a client may send CREATEUSER verb along with the username, salt, and verifier as defined by the protocol.

### Authentication ###

Authentication proceeds as per the SRP protocol.


When sending the client a proof of the session key, the server will also send a TOTP computed with a secret known to the servers and the time-step set to 60 min (an authenticated session can last at a minimum 1 hour and at most 2).

The TOTP shall be padded to a multiple of 128-bits via [PKCS #7](http://tools.ietf.org/html/rfc2315) padding. The padded block shall be different for each session generated by the server. The IV shall also be unique per session.

The padded block shall then be encrypted via AES with a key of the server's B value and the IV generated for this block.

The encrypted data as well as the IV are sent to the server.

The server's B value is padded and encrypted in the same manner. The Encrypted-B is also sent to the client.

The TOTP is used to limit the amount of time a session can last. It is encrypted to prevernt reply attacks of another users TOTP.

The B value is encrypted and sent in order to prevent servers from having to share session data.

When an AUTHPROOF header is sent, the A, B, username, salt, and verifier can be used to verify that the session key was generated via an authentication session with the server.  The TOTP (and must be decryptable) will time-out the session.  Both the TOTP and the session key must be correct in order for the request to continue.

