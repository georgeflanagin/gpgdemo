# Original description of the methods in Wanna Cry

1. The malware creates a 2048 bit RSA key pair.
1. The private key is encrypted using a public key that is included with the malware.
1. For each file, a new random AES key is generated.
1. This random AES key is then encrypted using the public user key.
1. To decrypt the files, the user’s private key needs to be decrypted, which
requires the malware author's private key.

The above is a little confusing and poorly worded, and apparently
written by someone who did not understand public key cryptography
well enough to write clearly about it.  I had to read it twice to
grok the process. The method is, in fact, fairly standard stuff.

# Key points:

- *public* keys are used in the encryption process.
- *private* keys are used in the decryption process.
- If you have any _private key_, you also have the _public key_
in the sense that you can derive the public key from the
private key. Don't believe me? Try it with ssh keys.
- _Files_ are never encrypted with public key cryptography. Period.
Instead, AES keys (i.e., a symmetric key cypher) are used to
encrypt the files, and the AES keys are encrypted with public
key cryptography.

# Here is what the description of the method should have said

1. The malware creates a 2048 bit RSA key pair *for this user's
files.* The reason is that we don't want to have key sharing among
the thousands of "users" of our system. For maximum ROI, each
user's files should be encrypted with a key created specially for
that one user.
1. The private key *for this user's files* is encrypted using a
public key that is included with the malware. This is no big deal,
because public keys are ... well ... *public*. But *whose* public
key are we talking about? In the last step we learn that it is the
malware author's public key. So Evil Kelly's public key will be used
to encrypt Dumb User's private key. Now it makes sense.
1. For each file, a new random AES key is generated. This random AES key 
is then encrypted using Evil Kelly' public key.
1. To decrypt the files, the user’s private key needs to be decrypted,
which requires the malware author's private key.


# Step 0: Generate a key pair for Evil Kelly.

The command is `gpg --gen-key`. This will be Evil Kelly's key pair.

```bash
[master][m1(gflanagi):~/ishtar3]: gpg --gen-key
gpg (GnuPG/MacGPG2) 2.0.30; Copyright (C) 2015 Free Software Foundation, Inc.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA (default)
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
Your selection? 1
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 5
Key expires at Sun May 21 09:15:11 2017 EDT
Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Kelly Flanagin, Bf. D.
Email address: kflanagin@stanford.edu
Comment: Throwaway key for demonstrations
You selected this USER-ID:
    "Kelly Flanagin, Bf. D. (Throwaway key for demonstrations) <kflanagin@stanford.edu>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
You need a Passphrase to protect your secret key.

You don't want a passphrase - this is probably a *bad* idea!
I will do it anyway.  You can change your passphrase at any time,
using this program with the option "--edit-key".

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
gpg: key 99A1B5CF marked as ultimately trusted
public and secret key created and signed.

gpg: checking the trustdb
gpg: 3 marginal(s) needed, 1 complete(s) needed, PGP trust model
gpg: depth: 0  valid:   5  signed:  10  trust: 0-, 0q, 0n, 0m, 0f, 5u
gpg: depth: 1  valid:  10  signed:   3  trust: 10-, 0q, 0n, 0m, 0f, 0u
gpg: next trustdb check due at 2017-05-21
pub   2048R/99A1B5CF 2017-05-16 [expires: 2017-05-21]
      Key fingerprint = 6294 61A2 EA41 2BD6 EA2D  3A8F 3238 23DF 99A1 B5CF
uid       [ultimate] Kelly Flanagin, Bf. D. (Throwaway key for demonstrations) <kflanagin@stanford.edu>
sub   2048R/B8E4E532 2017-05-16 [expires: 2017-05-21]
```

# Step 1 Export the new keys so we can distribute them.

`gpg` assumes you want to export the public key only because exporting
the private key is such a compromise of security.

```bash
[master][m1(gflanagi):~/ishtar3]: gpg -a --export 99A1B5CF
-----BEGIN PGP PUBLIC KEY BLOCK-----
Comment: GPGTools - https://gpgtools.org

mQENBFka+7QBCADB7DbtO2z7Dwm26DeY2m5N+lYDhgSIyR0x9IAqB49pZ+u0rumC
sNp3ymx/0fWuaHGEIj/EOnvlAVyW0aLKwYCw4nDhHywv1cm837CCK6qoYmYuVmGg
XBWyYgvsf5y9k9bBDRjXzT+A98aejWMs9Jli+xe3XA7AC1OSO2Wo4pEmqEmpGeLt
pPeH4T/hptI6ursdLTkaGzpJxSDLHjNsG/f9RCwf/aEuTYMvL+k4R0rfadJtJIR5
G/Jv1jjxFQtaW+VYIYLYiDIjNwUReX6CHlwRteKRKpwD+phtqLYG+0us4k3UyIcc
Y7lhKc3kGcvh1PTaKkv0TWTxdbxubX6IYzvzABEBAAG0UktlbGx5IEZsYW5hZ2lu
LCBCZi4gRC4gKFRocm93YXdheSBrZXkgZm9yIGRlbW9uc3RyYXRpb25zKSA8a2Zs
YW5hZ2luQHN0YW5mb3JkLmVkdT6JAT0EEwEKACcFAlka+7QCGwMFCQAGl4AFCwkI
BwMFFQoJCAsFFgIDAQACHgECF4AACgkQMjgj35mhtc8+pQgAir8WWJa5zZqBBQzc
v2Hn5Nb0JUm+Ou3CwdELAbW4BQ+Lalk92wh1Tdxsta8uUQS+T0M6YCEBKTwJnIjX
PdVF4+wPi9gP+8Y+1sDCX0hFehYKWEd6pLcdgQt6J6KBk4UYG0Ea3I2emx3ZG6R0
2yKKBIzBZx/A0JkPHtoT3TX0LZbUnSI+KEJWgo5q1QpVz/lBeruT5e+kWAkLL5Ef
6BYTnU7RTBEkXz9IDmEEM+qTIkVWysPKo/xqXn0Vv9eOLJZ4OlEOav9hTbmIEvW5
RYymTaZAitxIlJCGE2HmOQNNd0CaeQLzfahQxshSt9y0ZiIyJIslLlEDfFKPk18t
4pZbibkBDQRZGvu0AQgAxAjm3iXP7TpFht2tY9ZWoRlDT4FsZ/OcF8HtCz7rHqoQ
ceZ4ApMO62Xm5BaiaMCQtY6FVRwso5BtjekARvXwSbPFZmilJtZMUwwVBDAhKrT6
KbaaMJ9K/Mw+gz4yoPHLAqilA32SngczBC0lFsIY7pwQwcDLAgIht1m5KBFQhE2t
AyVkxSXrEhqEhDKGW71qIROb1hlgYWYsM0n1aHOCsHklVrqp4WBZ2/AUPT9Myq6b
/LQ/o0K/IlZdw85Ua4GiszVW9kzBW8zyr7RfIK/Err8zHIypCgtnpKU5v4wZRguY
deVjPK5IBfr9OV9aIgnEuN04/r52L8Jpx2oxYy3+KwARAQABiQElBBgBCgAPBQJZ
Gvu0AhsMBQkABpeAAAoJEDI4I9+ZobXPDKwIAKrZWpjc1bhKXaQyhMqvrf2MVdSf
QMJiBfNT+9XfBCf35cHY+Y8/CmYS2Oal4Kg/cJSfbZUeDi9zCFLoFZEpGhamuiKI
SpPTjwVItX05vUnHlTSAAgkxIhQS+4iokkHemBOlopn4pEaHfCrLi8P8V4ClI6AV
OQK5UdHAEWdzJekfFY9I59lCN5suInjxzORCx6EOj8fP2vTxHthZDZYLycOck8f1
lMNI81tkgP13rlywcUdEqjsPsxdChMQ910tX8B/dPNPqG8nncB2bKE4y+jzE0uar
xDTpwLRCPpWHL6fP7FRqiJ60r6v4VdHChN2Nsb6vCoOH6dwkA6xJId1VJqc=
=zibN
-----END PGP PUBLIC KEY BLOCK-----

[master][m1(gflanagi):~/ishtar3]: gpg -a --export 99A1B5CF >malware.key.pub
```

You can always get a better look at the above nonsense with the `--list-packets` command in `gpg`:

```bash
[master][m1(gflanagi):~/ishtar3]: gpg --list-packets malware.key.pub
:public key packet:
    version 4, algo 1, created 1494940596, expires 0
    pkey[0]: [2048 bits]
    pkey[1]: [17 bits]
    keyid: 323823DF99A1B5CF
:user ID packet: "Kelly Flanagin, Bf. D. (Throwaway key for demonstrations) <kflanagin@stanford.edu>"
:signature packet: algo 1, keyid 323823DF99A1B5CF
    version 4, created 1494940596, md5len 0, sigclass 0x13
    digest algo 10, begin of digest 3e a5
    hashed subpkt 2 len 4 (sig created 2017-05-16)
    hashed subpkt 27 len 1 (key flags: 03)
    hashed subpkt 9 len 4 (key expires after 5d0h0m)
    hashed subpkt 11 len 4 (pref-sym-algos: 9 8 7 3)
    hashed subpkt 21 len 4 (pref-hash-algos: 10 9 8 11)
    hashed subpkt 22 len 4 (pref-zip-algos: 2 3 1 0)
    hashed subpkt 30 len 1 (features: 01)
    hashed subpkt 23 len 1 (key server preferences: 80)
    subpkt 16 len 8 (issuer key ID 323823DF99A1B5CF)
    data: [2048 bits]
:public sub key packet:
    version 4, algo 1, created 1494940596, expires 0
    pkey[0]: [2048 bits]
    pkey[1]: [17 bits]
    keyid: 6E4C8A08B8E4E532
:signature packet: algo 1, keyid 323823DF99A1B5CF
    version 4, created 1494940596, md5len 0, sigclass 0x18
    digest algo 10, begin of digest 0c ac
    hashed subpkt 2 len 4 (sig created 2017-05-16)
    hashed subpkt 27 len 1 (key flags: 0C)
    hashed subpkt 9 len 4 (key expires after 5d0h0m)
    subpkt 16 len 8 (issuer key ID 323823DF99A1B5CF)
    data: [2048 bits]
```

If you should ever need to export a secret key, this is the command to do it:

```bash
[master][m1(gflanagi):~/ishtar3]: gpg -a --export-secret-key 99A1B5CF
-----BEGIN PGP PRIVATE KEY BLOCK-----
Comment: GPGTools - https://gpgtools.org

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
Jqc=
=may/
-----END PGP PRIVATE KEY BLOCK-----

[master][m1(gflanagi):~/ishtar3]: gpg -a --export-secret-key 99A1B5CF >malware.key
```

If you inspect an exported file, you will see something similar:

```
[master][m1(gflanagi):~/ishtar3]: gpg --list-packets malware.key
:secret key packet:
    version 4, algo 1, created 1494940596, expires 0
    skey[0]: [2048 bits]
    skey[1]: [17 bits]
    skey[2]: [2047 bits]
    skey[3]: [1024 bits]
    skey[4]: [1024 bits]
    skey[5]: [1021 bits]
    checksum: 3cd3
    keyid: 323823DF99A1B5CF
:user ID packet: "Kelly Flanagin, Bf. D. (Throwaway key for demonstrations) <kflanagin@stanford.edu>"
:signature packet: algo 1, keyid 323823DF99A1B5CF
    version 4, created 1494940596, md5len 0, sigclass 0x13
    digest algo 10, begin of digest 3e a5
    hashed subpkt 2 len 4 (sig created 2017-05-16)
    hashed subpkt 27 len 1 (key flags: 03)
    hashed subpkt 9 len 4 (key expires after 5d0h0m)
    hashed subpkt 11 len 4 (pref-sym-algos: 9 8 7 3)
    hashed subpkt 21 len 4 (pref-hash-algos: 10 9 8 11)
    hashed subpkt 22 len 4 (pref-zip-algos: 2 3 1 0)
    hashed subpkt 30 len 1 (features: 01)
    hashed subpkt 23 len 1 (key server preferences: 80)
    subpkt 16 len 8 (issuer key ID 323823DF99A1B5CF)
    data: [2048 bits]
:secret sub key packet:
    version 4, algo 1, created 1494940596, expires 0
    skey[0]: [2048 bits]
    skey[1]: [17 bits]
    skey[2]: [2045 bits]
    skey[3]: [1024 bits]
    skey[4]: [1024 bits]
    skey[5]: [1024 bits]
    checksum: 41a0
    keyid: 6E4C8A08B8E4E532
:signature packet: algo 1, keyid 323823DF99A1B5CF
    version 4, created 1494940596, md5len 0, sigclass 0x18
    digest algo 10, begin of digest 0c ac
    hashed subpkt 2 len 4 (sig created 2017-05-16)
    hashed subpkt 27 len 1 (key flags: 0C)
    hashed subpkt 9 len 4 (key expires after 5d0h0m)
    subpkt 16 len 8 (issuer key ID 323823DF99A1B5CF)
    data: [2048 bits]
```

So we send our public key along in the malware package. The malware package
includes the necessary GPG code to do encryption and key generation. It arrives
on the user's desktop with a big almighty **PLOP**.

The malware now needs to generate a key pair for the user. We have already seen the
details, so here is the gist only. The user will not see these steps; it is all done
programmatically just like it is in Canøe.

```bash
[master][m1(gflanagi):~/ishtar3]: gpg --gen-key
...blah blah blah ...
What keysize do you want? (2048)
Requested keysize is 2048 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 30
Key expires at Thu Jun 15 10:02:53 2017 EDT
Is this correct? (y/N) y
```

OK ... key point here. If the user doesn't pay up in 30 days, the
key will expire, and the user will never be able to decrypt anything.
Back to our regularly scheduled programming.

```
GnuPG needs to construct a user ID to identify your key.

Real name: Dumb User
Email address: clickbait@chum.org
Comment: I click on anything.
You selected this USER-ID:
    "Dumb User (I click on anything.) <clickbait@chum.org>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? O
...blah blah blah...
gpg: key 036C2809 marked as ultimately trusted
public and secret key created and signed.
...blah blah blah...
pub   2048R/036C2809 2017-05-16 [expires: 2017-06-15]
      Key fingerprint = B50E EFB3 6E41 DA64 3224  4B8D DE37 9199 036C 2809
uid       [ultimate] Dumb User (I click on anything.) <clickbait@chum.org>
sub   2048R/A5FAE61D 2017-05-16 [expires: 2017-06-15]
```

# Step 2:

We need to encrypt Dumb User's private key with Evil Kelly's public key
that was included in the malware package. Effectively, we are doing this:

```bash
gpg -a --export-secret-key 036C2809 > dumb.user.secret.key
gpg -a -quiet --trust-model always --yes -e --recipient kflanagin@stanford.edu dumb.user.secret.key
gpg --delete-secret-keys 036C2809
```

# Step 3: For each file, a new random AES key is generated.

Remember what we said about public key cryptography not being used for encryption
of files. This is the standard way that `gpg` works. Let's encrypt a file and take a 
look at the result:

```bash
[master][m1(gflanagi):~/ishtar3]: gpg -a -quiet --trust-model always --yes -e --recipient kflanagin@stanford.edu -o ishtar.py.gpg ishtar.py
[master][m1(gflanagi):~/ishtar3]: ll ishtar.py*
-rwxr-xr-x  1 gflanagi  staff  17719 May  9 08:23 ishtar.py
-rw-r--r--  1 gflanagi  staff   9085 May 16 10:45 ishtar.py.gpg
```

# Step 4: This random AES key is then encrypted using the public user key.

Nothing special -- this is simply the way that `gpg` works. If you didn't do this, you would have no
way of knowing which key could be used to decrypt the AES key that has been used to encrypt 
the file.

```bash
[master][m1(gflanagi):~/ishtar3]: gpg --list-packets ishtar.py.gpg
:pubkey enc packet: version 3, algo 1, keyid 6E4C8A08B8E4E532
	data: [2047 bits]
:encrypted data packet:
	length: unknown
	mdc_method: 2
gpg: encrypted with 2048-bit RSA key, ID B8E4E532, created 2017-05-16
      "Kelly Flanagin, Bf. D. (Throwaway key for demonstrations) <kflanagin@stanford.edu>"
:compressed packet: algo=2
:literal data packet:
	mode b (62), created 1494945952, name="ishtar.py",
	raw data: 17719 bytes
```

A fact that is often overlooked. A file can be encrypted so that more than one party can decrypt it.
We do this in Canøe. We encrypt all the outbound files with the vendor's key, and we encrypt the file
with our key as well. It is just a matter of adding an additional recipient:

```bash
[master][m1(gflanagi):~/ishtar3]: gpg -a -quiet --trust-model always --yes -e --recipient kflanagin@stanford.edu --recipient me@georgeflanagin.com -o ishtar.py.gpg ishtar.py
```

How can this possibly work? It is only the AES key that we are really encrypting. Two encrypted copies of the AES
key are placed in the file along with the file's encrypted contents. Graphically, it looks like this:

```
ishtar.py.gpg
+-------------------------------------------------+
| AES key encrypted for me@georgeflanagin.com     |
+-------------------------------------------------+
| AES key encrypted for kflanagin@stanford.edu    |
+-------------------------------------------------+
|     the zipped data from ishtar.py encrypted    |
|     with the AES key.                           |
|                                                 |
+-------------------------------------------------+
```

When seen with the `--list-packets` option of `gpg`, it looks like this:

```
[master][m1(gflanagi):~/ishtar3]: gpg --list-packets ishtar.py.gpg
:pubkey enc packet: version 3, algo 1, keyid 6E4C8A08B8E4E532
	data: [2047 bits]
:pubkey enc packet: version 3, algo 1, keyid 44F41DCF6BFC3AFB
	data: [4094 bits]
:encrypted data packet:
	length: unknown
	mdc_method: 2
gpg: encrypted with 4096-bit RSA key, ID 6BFC3AFB, created 2016-07-02
      "George Flanagin (at home) <me@georgeflanagin.com>"
gpg: encrypted with 2048-bit RSA key, ID B8E4E532, created 2017-05-16
      "Kelly Flanagin, Bf. D. (Throwaway key for demonstrations) <kflanagin@stanford.edu>"
:compressed packet: algo=2
:literal data packet:
	mode b (62), created 1494946569, name="ishtar.py",
	raw data: 17719 bytes
```

# 5. To decrypt the files, the user’s private key needs to be decrypted, which requires the malware author's private key.

To review:
- We have a disc full of encrypted data files.
- We have the malware on our disc, too. 
- The malware has an encrypted copy of the private key we need to decrypt our stuff.
- The private key we need to use is encrypted with Evil Kelly's public key. 
- Evil Kelly's public key is of no use in decryption.

So we part with a BitCoin, or some fraction thereof. The block chain is updated, and this creates 
a receipt. 

- The malware presents the encrypted private key to Evil Kelly, along with proof of payment.
- Evil Kelly keeps his word, and decrypts the private key that has been stored on Dumb User's machine all along.
- The decrypted, read-to-use private key is put on the malware's keyring. 
- The malware then decrypts the files. 

`gpg` assumes that you want to decrypt a file; i.e., decryption is `gpg`'s default operation. Remember that it 
knows which key has been used to encrypt the file because that information is in the `.gpg` file's header. 
`gpg` will use any key listed in the header for which it has the private key; you don't have to tell it anything.

```bash
gpg ishtar.py.gpg
```

# 6. Scary fact

The above is very close to what Canøe does with the _ExLibris_ integration. Many parts of the above are
what Canøe does all the time. 
