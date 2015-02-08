---
layout: default
title: OpenPGP Public Key Association (PKA)
permalink: /guides/public-key-association/
description: Public Key Association (PKA) allows you to publish your OpenPGP key to your DNS record
tags: pka, Public Key Association, pgp, gpg, GnuPG, DNS
redirect_from:
  - /public-key-association/
  - /pka/
---

# Publishing A Public PGP Key via DNS

A keyserver is not the only way to publish your public pgp key.

One method of publishing public pgp keys, other then on a keyserver, is to use a <abbr title="Domain Name Server">DNS</abbr> record with a <abbr title="Public Key Association">PKA</abbr> (Public Key Association) entry. PKA is a simple way of storing the look up infromation for your public key in a DNS `TXT` record. PKA allows a user to send encrypted data to a email address by query the DNS server, caching the key&#039;s fingerprint, downloading the public key from the provided URL and validaing the key using the cached fingerprint.

## Creating the DNS Record

Start out by exporting your public key as an **ascii armored** (by using the `-a` flag) public key, to a file on your webserver.

<pre>$ gpg2 -a --export 0xB35B27E07F99ABEC > /var/www/html/keys/0xB35B27E07F99ABEC.asc</pre>

This key/file must be accessible via a public URL on the domain the email address is associated with. In this example, the file `/var/www/html/keys/0xB35B27E07F99ABEC.asc` is able to be downloaded by going to `http://key.bbs4.us/keys/0xB35B27E07F99ABEC.asc`.

Next you need to get your key&#039;s fingerprint, to do so, list your public key with the `--fingerprint` flag.

<pre>$ gpg2 --fingerprint --list-keys 0xB35B27E07F99ABEC

pub   nistp521/0xB35B27E07F99ABEC 2014-12-28 [expires: 2017-12-27]
      Key fingerprint = <strong>EEB4 454F D52B D587 A4AA  0D1F B35B 27E0 7F99 ABEC</strong>
uid                  Jonathan Zhang <jonathan@bbs4.us></pre>

Using your `Key fingerprint`, remove the spaces and add the content to the below line, with the URL for the above key.

<pre>v=pka1;fpr=EEB4454FD52BD587A4AA0D1FB35B27E07F99ABEC;uri=http://key.bbs4.us/keys/0xB35B27E07F99ABEC.asc</pre>

And add this to your DNS server as a `TXT` field.  The pointer of the record should be the user section of your email address followed by `_pka` (ie: `jonathan._pka.bbs4.us`). 

The full BIND DNS entry would be

    jonathan._pka.bbs4.us. 86400   IN      TXT     "v=pka1\;fpr=EEB4454FD52BD587A4AA0D1FB35B27E07F99ABEC\;uri=http://key.bbs4.us/keys/0xB35B27E07F99ABEC.asc"

## Testing the PKA Record

There are two main parts to a PKA setup, the first being the DNS record, and the second is the actual key to be downloaded.

### Testing the DNS Record

To test the DNS record, using `dig`, run the following command after changing **jonathan** and **bbs4.us** in the below example to your own **user** and **domain** part of your email address.

<pre>$ dig +short txt <strong>jonathan</strong>._pka.<strong>bbs4.us</strong></pre>

Should produce the same value as was entered into the DNS TXT record from above.

<pre>"v=pka1\;fpr=EEB4454FD52BD587A4AA0D1FB35B27E07F99ABEC\;uri=http://key.bbs4.us/keys/0xB35B27E07F99ABEC.asc"</pre>

### Testing the URL

Testing the url is simple, you can browse it it via your web client or you can download it via `curl`.

<pre>$ curl http://key.bbs4.us/keys/0xB35B27E07F99ABEC.asc</pre>

Should produce the public key in the `0xB35B27E07F99ABEC.asc` file you created above.

<pre>
-----BEGIN PGP PUBLIC KEY BLOCK-----
Version: GnuPG v2

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
Zv38fQ==
=+Dbo
-----END PGP PUBLIC KEY BLOCK-----
</pre>

## Importing a key via PKA

You may run the following command to import your key into your key ring. Just change **jonathan@bbs4.us** to the email address you wish to import.

<pre>$ echo "Test message" | gpg --auto-key-locate pka -ear <strong>jonathan@bbs4.us</strong></pre>

The command should produce the following output, note the line "automatically retrieved 'jonathan@bbs4.us' via **PKA**".

<pre>gpg: requesting key 7F99ABEC from http server bbs4.us
gpg: key 7F99ABEC: public key "Jonathan Zhang (ECC) <jonathan@bbs4.us>" imported
gpg: Total number processed: 1
gpg:               imported: 1  (RSA: 1)
gpg: <strong>automatically retrieved 'jonathan@bbs4.us' via PKA</strong>
gpg: 7F99ABEC: There is no assurance this key belongs to the named user

pub  nistp521/7F99ABEC 2014-06-21 jonathan Rude <jonathan@bbs4.us>
 Primary key fingerprint: EEB4 454F D52B D587 A4AA  0D1F B35B 27E0 7F99 ABEC

It is NOT certain that the key belongs to the person named
in the user ID.  If you *really* know what you are doing,
you may answer the next question with yes.

Use this key anyway? (y/N) <strong>y</strong></pre>

## Other PKA Resources

* More information may be found at [gushi.org](http://www.gushi.org/make-dns-cert/HOWTO.html), [df7cb.de](https://www.df7cb.de/blog/2007/openpgp-dns.html), [initd.net](http://www.initd.net/2010/12/adding-gpg-public-keys-to-your-dns.html), and [grepular.com](https://grepular.com/Publishing_PGP_Keys_in_the_DNS)
