---
title: "Setup GPG for your Github account"
tags:
  - github
  - security
---

### What is GPG?

GPG stands for GNU Privacy Guard. It's widely used for signing files, allows you to send your files and messages securely over the Internet.

It contains two parts:
- Public key - used to encrypt message.
- Private key - used to decrypt encrypted message.

### How Github use it to verify your commit?

When creating a new GPG key, `gpg` CLI will ask you your email address. Next, we update the `git` config to make it use the generated GPG key to encrypt the commit. 
After pushing the commit to Github, Github verifies this commit by compraring the committer's email address and the email address attached to the GPG key, and verified email address in your Github account.

### How to generate it?

You can quickly generate a GPG key by running:

```bash
 $ gpg --full-generate-key
```
And then answer some questions.

You can list all GPG keys available inside your computer by:

```bash
$ gpg --list-secret-keys --keyid-format=long

/Users/hubot/.gnupg/secring.gpg
------------------------------------
sec   4096R/3AA5C34371567BD2 2016-03-10 [expires: 2017-03-10]
uid                          Hubot 
ssb   4096R/42B317FD4BA89E7A 2016-03-10
```

And you can export your GPG public key by:

```bash
$ gpg --armor --export 3AA5C34371567BD2

# Prints the GPG key ID, in ASCII armor format
```

And finally, you copy GPG key, beginning with `-----BEGIN PGP PUBLIC KEY BLOCK-----` and ending with `-----END PGP PUBLIC KEY BLOCK-----`.


### Adding GPG key to your Github account

Just access [Github config](https://github.com/settings/keys) and paste the key from above.

### Tell Git your singing key

We tell Git to use GPG key to encrypt our commit by:

```bash
$ git config --global user.signingkey 3AA5C34371567BD2
```

`3AA5C34371567BD2` is your GPG key ID.