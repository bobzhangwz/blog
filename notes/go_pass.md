# Gopass workshop

## Install gopass

```
rew install pinentry-mac gnupg2
brew install gopass gopass-ui

PINENTRY=$(which pinentry-mac)
echo "pinentry-program ${PINENTRY}" >>~/.gnupg/gpg-agent.conf
```



##  Key management

原理: 非对称加密(key pair)

- 公钥加密, 私钥解密 - gopass
- 私钥加密, 公钥解密

### Generate key pair

gopass depends on the `gpg` program for encryption and decryption. You **must** have a
suitable key pair. To list your current keys, you can do:

```bash
gpg --list-secret-keys
```

If there is no output, then you don't have any keys. To create a new key:

```bash
gpg --full-generate-key
```

You will be presented with several options:

* Key type: Choose either "RSA and RSA" or "DSA and ElGamal".
* Key size: Choose at least 2048.
* Validity: 5 to 10 years is a good default.
* Enter your real name and primary email address.
* A comment is not necessary.
* Pass phrase: Make sure to pick a very long pass phrase, not just a simple password. Remember this should be stronger than any of the secrets you store in the password store. You can configure the GPG Agent later to save you repetitive typing.

### Private key management

```
# List allkeys
gpg --list-secret-keys

# Export private keys
gpg --export-secret-keys <Your email> > private.key

# Import private key when
gpg --import private.key
```

### Public key management

```bash
gpg --list-public-keys
gpg -a --export willy@email.com > willy.pub.asc
```



## Init a store

```bash
gopass init --store my-secret
# Create an empty git repo in github
gopass git remote add --store my-secret origin git@gh.com/Woile/keys.git

cat <<EOF | gopass insert --force ss-staging/test/user
User_Name: aw_a_salessvc_test_string_0000
Key: Value
EOF

gopass show ss-staging/test/user

# !!! Copy password to clipboard
gopass show -c my-company/willy@email.com

gopass edit ss-staging/test/user
gopass create --store mysecret test/test2
gopass generate mysecrets/test/test2
```



## Team share

### Export public key

```bash
gpg -a --export willy@email.com > willy.pub.asc
```

### Check current recipients

```bash
gopass recipients
```

### Add public key into gopass

```bash
gpg --import < willy.pub.asc
gpg --list-keys
gopass recipients add willy@email.com
```

### Synchronize with remotes

```bash
gopass sync
```

### Synchronzing a single store

```bash
gopass sync --store my-company
```
