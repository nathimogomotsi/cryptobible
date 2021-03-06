# Key Exchanges

## X25519 and X448

TKTK

## Elliptic Curve Diffie-Hellman (ECDH)

TKTK

## (Finite Field) Diffie-Hellman (FFDH or DH)

Diffie-Hellman is the first ever public-key crypto algorithm. It acts on
finite fields (**FFDH**) but has also been augmented to work in
elliptic curves (**ECDH**). While the two algorithms are very similar,
they are also subject to different types of attacks. In this page, we
refer to FFDH only. See the <a href="ECDH.mediawiki">ECDH</a> page for
the elliptic curve variant.

### What parameters and groups are secure?

Many implementations of Diffie-Hellman will often come with
pre-generated parameters. For example
<a href="https://tools.ietf.org/html/rfc8446#section-4.2.7">TLS 1.3
specifies exactly what DH groups can be negotiated</a>. While these
groups taken directly from
<a href="https://tools.ietf.org/html/rfc7919">RFC 7919</a> are secure,
this is not always the case and the slightest change can introduce
issues. For example:

* <a href="https://eprint.iacr.org/2016/644">How to Backdoor Diffie-Hellman</a> tells the tale of the Socat tool which was updated with a suspicious and non-secure DH modulus.
* <a href="http://blog.intothesymmetry.com/2016/10/the-rfc-5114-saga.html">RFC 5114</a> specifying DH groups was found to be backdoored with broken groups.

For these reasons, it is important to test the validity of the public
parameters used. <a href="https://github.com/mimoo/test_DHparams">Tools
exist for this</a>.

### What keysize should be used?

First, when we talk about keysize what we're talking about is the
bit-size of the public **modulus**.

As with every keysize concerns, www.keylength.com is often the answer
(spoiler alert: 2048-bit).

In addition, the
<a href="https://wordpress.rose-hulman.edu/holden/the-mathematics-of-secrets/cryptography-by-the-numbers/">cryptography
by the numbers</a> page gives some idea of what academic attacks have
been able to do (spoiler alert: a 768-bit modulus has been broken).
Although <a href="https://weakdh.org/">academic estimates</a> that state
adversary should be able to break 1024-bit modulus.

## RSA

RSA is both a signature and an asymmetric encryption scheme. It is
important to know which algorithm and which version of it you are using
(as not all of them are secure); they are mostly defined in PKCS\#1
documents (<a href="https://tools.ietf.org/html/rfc2313">v1.5</a>,
<a href="https://tools.ietf.org/html/rfc2437">v2.0</a>,
<a href="https://tools.ietf.org/html/rfc3447">v2.1</a>,
<a href="https://tools.ietf.org/html/rfc8017#section-7.2">v2.2</a>).

### What RSA encryption algorithms are secure?

There are many encryption schemes that have been standardized,
unfortunately not all of them are that great and the great ones are
often not implemented by libraries.

* <a href="https://tools.ietf.org/html/rfc2313">The PKCS\#1 v1.5 padding</a> was discovered to be broken in 1998. You should avoid this algorithm at all cost unless you have very good reasons and have implemented strong mitigations. See our section <a href="#is-rsa-pkcs1-v15-secure">Is RSA PKCS\#1 v1.5 secure?</a> for more information.
* <a href="https://tools.ietf.org/html/rfc8017#section-7.2">OAEP</a> was introduced in PKCS\#1 v2 and has an incorrect proof of security as well as issues close to PKCS\#1 v1.5 but that are easier to mitigate. See our section <a href="#is-rsa-oaep-secure">Is RSA-OAEP secure?</a> for more information.
* <a href="https://eprint.iacr.org/2000/060">OAEP+</a> corrects the proof of security of OAEP. Unfortunately did not get standardized.
* <a href="https://eprint.iacr.org/2002/034">OAEP++</a> is another attempt at correcting the proof of security of OAEP. Unfortunately did not get standardized.
* <a href="https://tools.ietf.org/html/rfc5990">RSA-KEM</a> (<a href="https://eprint.iacr.org/2001/112.pdf">from Shoup's work</a>) is the simplest way to encrypt with RSA, it also benefits from a tight security proof. Unfortunately it is not always implemented by libraries.

### Is RSA PKCS\#1 v1.5 secure?

**No**.
<a href="http://archiv.infsec.ethz.ch/education/fs08/secsem/bleichenbacher98.pdf">Bleichenbacher
found an attack on PKCS\#1 v1.5 in 1998</a> which allows to decrypt the
content of any encrypted payload, as long as the server advertises (or
leaks) the validity of a ciphertext's PKCS\#1 v1.5 padding. In spite of
this, most certificates on the internet (and thus most internet
protocols) are stil using this old and broken primitive. This has led to
difficult to implement workarounds and mitigations, that have
consequently been found not to work over and over:

* <a href="http://archiv.infsec.ethz.ch/education/fs08/secsem/bleichenbacher98.pdf">Bleichenbacher (CRYPTO 1998)</a> also called the 1 million message attack, BB98, padding oracle attack on PKCS\#1 v1.5, etc.
* <a href="https://eprint.iacr.org/2003/052">Klima et al. (CHES 2003)</a>
* <a href="https://www.nds.rub.de/media/nds/veroeffentlichungen/2012/12/19/XMLencBleichenbacher.pdf">Blei­chen­ba­cher’s At­tack Strikes Again: Brea­king Pkcs\#1 V1.5 In Xml En­cryp­ti­on (ESORICS 2012)</a>
* <a href="https://eprint.iacr.org/2011/615.pdf">Degabriele et al. (CT-RSA 2012)</a>
* <a href="https://eprint.iacr.org/2012/417">Bardou et al. (CRYPTO 2012)</a>
* <a href="https://www.cs.unc.edu/~reiter/papers/2014/CCS1.pdf">Cross-Tenant Side-Channel Attacks in PaaS Clouds (CCS 2014)</a>
* <a href="https://www.usenix.org/conference/usenixsecurity14/technical-sessions/presentation/meyer">Revisiting SSL/TLS Implementations: New Bleichenbacher Side Channels and Attacks (USENIX 2014)</a>
* <a href="https://www.nds.rub.de/media/nds/veroeffentlichungen/2015/08/21/Tls13QuicAttacks.pdf">On the Security of TLS 1.3 and QUIC Against Weaknesses in PKCS\#1 v1.5 Encryption (CCS 2015)</a>
* <a href="https://drownattack.com/">DROWN (USENIX 2016)</a>
* <a href="https://robotattack.org/">Return Of Bleichenbacher's Oracle Threat (USENIX 2018)</a>
* <a href="https://www.usenix.org/system/files/conference/usenixsecurity18/sec18-felsch.pdf">The Dangers of Key Reuse: Practical Attacks on IPsec IKE (USENIX 2018)</a>

### Is RSA-OAEP secure?

RSA OAEP standardized in PKCS\#1 v2.x is deemed "good enough" if you do
not have access to better standards like
<a href="https://tools.ietf.org/html/rfc5990">RSA-KEM</a>. Nonetheless,
implementations of OAEP must mitigate against
<a href="http://archiv.infsec.ethz.ch/education/fs08/secsem/Manger01.pdf">Manger's
attack</a> meaning that they must check in **constant time** the full
padding of the ciphertext, even if the first null byte is incorrect. For
example an error returning too early could give indication on the value
of the first padding byte.

### What keysize should be used?

First, when we talk about keysize what we're talking about is the
bit-size of the RSA public **modulus**.

As with every keysize concerns, www.keylength.com is often the answer
(spoiler alert: 2048-bit).

In addition, the
<a href="https://wordpress.rose-hulman.edu/holden/the-mathematics-of-secrets/cryptography-by-the-numbers/">cryptography
by the numbers</a> page gives some idea of what academic attacks have
been able to do (spoiler alert: a 768-bit modulus has been broken).
Although <a href="https://weakdh.org/">academic estimates</a> that state
adversary should be able to break 1024-bit modulus.

### Is it safe to use a non-standardized RSA algorithm?

Probably not, depending on how you use RSA there might exist more
attacks. See
<a href="https://crypto.stanford.edu/~dabo/papers/RSA-survey.pdf">Twenty
Years of Attacks on the RSA Cryptosystem</a>.

### Are there any known issues with RSA Key generation?

RSA's key generation requires good **randomness** and good algorithms
to generate **prime numbers**. Both these requirements have been
butchered in the past:

* <a href="https://factorable.net/">Mining Your Ps and Qs: Detection of Widespread Weak Keys in Network Devices</a>
* <a href="https://crocs.fi.muni.cz/public/papers/rsa_ccs17">ROCA (CCS 2017)</a>

To ensure that a key generation algorithm is safe, it needs to be
audited.

### Side Channel on RSA, a problem?

not really, unless your threat model involve attackers being able to
physically observe or tamper with your device OR you are running
adversarial code on the same computer. TKTK (fault attacks (rowhammer),
timing attacks, etc.)

