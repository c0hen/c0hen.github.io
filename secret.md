---
layout: default
title: Secrets
description: Password and secret management with GPG and pass
tags: password gpg tips otp secret signing encryption gnupg pass
---

* Table of contents
{:toc}

## Create password store {#create-password-store}

Using [pass](https://git.zx2c4.com/password-store).

```sh
export PASSWORD_STORE_DIR="$HOME/.password-store"
export backup_store_dir="/media/backup/.password-store"
pass init pgpKeyID
pass git init -b main
pass git config --bool --add pass.signcommits true
mkdir -p "$backup_store_dir"
cd "$backup_store_dir"
git init -b temp
cd "$PASSWORD_STORE_DIR"
pass git remote add backup "$backup_store_dir"
pass git push -u backup
cd "$backup_store_dir"
git switch main
git branch -d temp
```

## Add binary files as secrets

```sh
pass insert --multiline my/file < /my/file.bin
```

## OTP URI with pass-otp

Prompt for an otpauth URI.
```sh
pass otp insert library_access
```
Full URI with all parameters.
```
otpauth://totp/Falvey%20Library:john.doe@email.com?secret=HXDMVJECJJWSRB3HWIZR4IFUGFTMXBOZ&issuer=Falvey%20Library&algorithm=SHA1&digits=6&period=30
```
Only append the OTP secret key using defaults for other parameters.
```sh
pass otp append --secret --issuer FalveyLibrary --account john.doe@email.com library_access
```
Use your webcam to scan a QR code of the otpauth URI with [zbar](https://github.com/mchehab/zbar).
```sh
zbarcam -q --raw | pass otp insert totp-secret
```

## Pass show

Copy line to clipboard.
```sh
pass show my_account -c
pass show my_account -c2
pass otp my_account -c
```

## Operations and hardening in a distributed environment

Hardening hints in [durable configuration files](https://github.com/ioerror/duraconf).

Encrypt the disk to prevent traces of PGP key backups.

### Verify, sign, trust

[GNU Privacy Handbook](https://www.gnupg.org/gph/en/manual/x334.html).

Get other keys the password store is encrypted to from the keyserver.

```sh
gpg --recv-keys $(cat .gpg-id)
```

Verify, sign, and marginally trust at least three other keys.

```sh
gpg --with-fingerprint keyID # verify fingerprint out of band
gpg --sign-key keyID # sign
gpg --send-keys keyID # send signed key to keyserver
gpg --edit-key keyID # trust the key
```

```
trust # 3 , marginally
save
```

### Add new user to key store

1. Get public key `gpg --import pubkey.asc` or `gpg --receive-key keyID`
1. Verify `gpg --fingerprint keyID` out of band
1. `gpg --sign-key keyID`
1. `gpg --send-keys keyID`
1. Reencrypt the password store adding the new key `pass init $(cat .gpg-id) keyID`
1. Ensure three people complete this process.

## GnuPG

GnuPG implements the [OpenPGP standard](https://www.rfc-editor.org/rfc/rfc9580.html).
PGP key in modern usage means a key conforming to the OpenPGP standard.

### Configuration

```
# $HOME/.gnupg/gpg.conf
default-recipient-self
default-key keyID
keyid-format long
keyserver hkps://keyring.debian.org:443
```

Show current configuration for gpg components.

```sh
gpgconf
gpgconf -v --list-options gpg
gpgconf -v --list-options gpg-agent
gpgconf -v --list-options dirmngr
```

### Key info

Show keys from file.

```sh
gpg --show-keys debian-keyring.gpg
```

Show secret keys with subkey fingerprints.

```sh
gpg --list-secret-keys --with-subkey-fingerprint mainKeyID
```

Key info details in the GPG repository `git://git.gnupg.org/gnupg.git`, [GPG repository mirror](https://github.com/gpg/gnupg/blob/master/doc/DETAILS).

- `pub` - public primary key
- `sub` - public subkey
- `sec` - secret primary key
- `ssb` - secret subkey
- `[E]` - encrypt
- `[S]` - sign
- `[C]` - certify
- `[A]` - authentication
- `[D]` - disabled

Suffixing the key ID or fingerprint with `!`, a specific subkey or the primary key can be targeted. By default GnuPG uses the latest key with the required capability.

### Encrypt file with a passphrase

Encrypt using symmetric encryption and do not cache the password. Test with clear results.

```sh
gpg --no-symkey-cache --symmetric file.txt
gpg --decrypt file.txt.gpg > decrypted_file.txt
```

### Create and edit key

```sh
gpg --full-generate-key
gpg --list-secret-keys
gpg --edit-key keyID
```

Add subkey, change key usage. Signing keys can be changed to authentication keys.

```
help
addkey
list
key 0 # select main key
change-usage # change usage of the main key
key 2 # select subkey
change-usage # change selected subkey
save
```

Replace user identity to change info, add another.
```
adduid
uid 1
deluid
adduid
save
```

### Backup and restore key

It is good practice to create a key with subkeys for signing, encryption, authentication and remove signing capability from the main key, storing the main private key offline. This makes revocation easy.

Backup the secret keys. Optionally pipe to `paperkey` for a printable backup.

```sh
gpg --export-options backup --export-secret-keys mainKeyID > secretKeys.gpg
gpg --export-options backup --export-secret-keys mainKeyID! > mainSecretKey.gpg
gpg --export-options backup --export-secret-subkeys mainKeyID > secretSubKeys.gpg
```

Save backups on an offline device. Main secret certification key is rarely needed.

Optionally remove the main secret certification key from the system.

```sh
gpg --delete-secret-keys mainKeyID!
gpg --list-secret-keys mainKeyID
```

Missing key is marked '#'.

```
sec#  ed25519/keyID
```

Restore the key, keeping trust info.

```sh
gpg --import-options 'restore keep-ownertrust' --import secretKeys.gpg
```

Transfer trust info over ssh. Export is better than copying `trustdb.gpg`.

```sh
gpg --export-ownertrust | ssh user@othermachine gpg --import-ownertrust
```

Get the ASCII armored public key, convert it to PGP format and restore a secret key from a paperkey.

```sh
wget <url> --output-document backup.export.public.keys.asc
gpg --dearmor < backup.export.public.keys.asc > public.dearmored.gpg
paperkey --pubring public.dearmored.gpg \
--secrets backup.export.secret.key.paperkey |\
gpg --import-options 'restore keep-ownertrust' --import -
```

### Key compromised

```sh
gpg --edit-keys keyID
```

```
key compromisedSubKeyID
revkey
save
```

Upload the updated key to keyserver.

```sh
gpg --send-keys keyID
```

Export the public key as ASCII armored text, identifying the key with the user name. Easy to publish when keyserver usage is not an option.

```sh
gpg -a --export tester > tester.pub.asc
```

### Encrypt file to hidden recipient

Hide the key ID of this user's key. This option helps to hide the receiver of the message and is a limited countermeasure against traffic analysis.

```sh
gpg --encrypt --multifile --hidden-recipient key3ID file.txt file.png
gpg --decrypt file.txt.gpg > file.txt_decrypted.txt
```
```
gpg: encrypted with ECDH key, ID 0000000000000000
gpg: anonymous recipient; trying secret key key1ID ...
gpg: ecdh failed in gcry_cipher_decrypt: Checksum error
gpg: anonymous recipient; trying secret key key2ID ...
gpg: ecdh failed in gcry_cipher_decrypt: Checksum error
gpg: anonymous recipient; trying secret key key3ID ...
gpg: okay, we are the anonymous recipient.
```

### Configure GnuPG network components

GnuPG's network access daemon configuration, client.

```
# $HOME/.gnupg/dirmngr.conf
keyserver hkps://keyring.debian.org:443
```

Hosting a [Web Key Directory](https://wiki.gnupg.org/WKDHosting).

### [Gnupg FAQ on algorithms](https://gnupg.org/faq/gnupg-faq.html#advanced_topics)

The future is [elliptical-curve cryptography](https://wiki.gnupg.org/ECC) (ECC), which will bring a level of safety comparable to RSA-16384. Gnupg defaults to algorithms using it.

#### [Curve25519](https://cr.yp.to/ecdh.html) algorithms.

EdDSA - [Edwards-Curve Digital Signature Algorithm](https://www.rfc-editor.org/info/rfc8032/)
`ed25519`.

`cv25519` is an encryption algorithm.
