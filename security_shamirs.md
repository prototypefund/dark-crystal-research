
## Security considerations for Shamir's secret sharing

In general, Shamir's scheme is considered information-theoretically secure.  That is, individual shares contain absolutely no semantic information about the secret, and it can be said to be 'post quantum' cryptography.

An interesting anecdote, the root key for ICANN DNS security, effectively the key which secures the naming system of the internet, is held by seven parties, based in Britain, the U.S., Burkina Faso, Trinidad and Tobago, Canada, China, and the Czech Republic. Cryptographer Bruce Schneier has alleged that they are holders of Shamir's secret shares, which indicates the scheme is taken quite seriously.

However, it is not without its problems:

### The need for verification of individual shares

Harn and Lin consider the situation in which 'cheaters' claiming to be holders of shares introduce 'fake' shares, causing the incorrect secret to be recovered.  Of course without having the other shares they have no control over the content of the 'incorrect' secret. 

This is not so much of a concern for dark-crystal as we generally already have a way to validate who is a custodian, as we keep track of who shares were sent out to.  However we also considered the possibility that genuine holders of shares might have a motivation for not wanting the secret to be recovered, and could maliciously modify their share.  Furthermore, the shares might be modified by some accidental or external cause, and it is important to be able to determine which share is causing the problem.

It might be very easy to determine that we have recovered the wrong secret.  Either because we have some idea of how we expect it to look, or as we have implemented in dark crystal, by using a message authentication code (MAC).

However, the problem here is that although we might know for sure that we have not successfully restored our secret, we have no way of telling which share(s) have caused the problem, meaning we do not know who is responsible. 

The solution is to introduce some verification of shares, and a number of different methods of doing this have been proposed.  Typically, they rely on publicly publishing some information which allows verification of a given share.

Here are some possible solutions:

#### Feldman's scheme 

Paul Feldman proposed a scheme in 1987 which allows custodians to verify their own shares, using homomorphic encryption (an encryption scheme where computation can be done on encrypted data which when decrypted gives the same result as doing that computation on the original data) on top of Shamir's original scheme.

#### Pederson's scheme

Pederson's scheme, published in 1992, has the advantage that it can be used with secrets which are not uniformly random. For example, a password, or message in a human language. 

#### Schoenmakers scheme

More recently Berry Schoenmaker proposed a scheme which is designed to be publicly verifiable (originally introduced by Stadler, 1996).  That is, not only custodians, but anybody is able to verify that the correct shares were given.  The scheme is described in the context of an electronic voting application and focusses on validating the behaviour of the 'dealer' (the author of the secret).  But it can just as well be used to verify that returned shares have not been modified. 

#### Publicly publishing the encrypted shares  

The problem with this, is it only helps in this context if the encryption scheme used is deterministic.  That is to say encrypting the same message with the same key twice will reliably give the same output.  The problem here is that such encryption schemes are vulnerable to replay attacks.  Most modern symmetric schemes introduce some random nonce to evade this problem. The scheme we are using (libsodium's secret box) typically takes a 24 byte random nonce.  So this is not a good option.

#### Publicly publishing the hash of each share

This is also something we considered, but feel that it gives custodians more unnecessary extra information, which gives security implications, and less accountability compared to other methods.

#### Signing shares

In dark-crystal's key backup and recovery, we have the advantage that we have a notion of identity of the secret owner and the custodians, since we use cryptographic signing and asymmetric encryption to validate who sent a message and to control to whom messages are sent. Since the secret owner has an established public key, and all messages are signed, they can sign individual shards, and this signature stays with the shard data until the point of recovery.  This means that, provided we know the public key of the secret owner, we can be sure that a given share has not been tampered with, or accidentally modified.

#### Implementations of publicly verifiable secret sharing

If you are looking for a publicly verifiable secret sharing implementation:

- [songgeng87/PubliclyVerifiableSecretSharing](https://github.com/songgeng87/PubliclyVerifiableSecretSharing) - C implementation built on secp256k1 used by EOS
- [FabioTacke/PubliclyVerifiableSecretSharing](https://github.com/FabioTacke/PubliclyVerifiableSecretSharing) - A Swift implementation 
- [dfinity/vss](https://github.com/dfinity/vss) - Dfinity's NodeJS implementation built on BLS, and used for their distributed key generation

However, these do not give a drop-in replacement for the secrets library used in dark-crystal key backup. 

### Share size has a linear relationship with secret size

Anyone holding a share is able to determine the length of the secret.  Particular kind of cryptographic keys have a characteristic length.  So the scheme gives away more information to custodians than is necessary.  A solution to this is to add padding to the secret to increase share length to a standard amount.

### Revoking shares if trust in a custodian is lost

Suppose we loose trust in one person holding a share. This might be because they had their computer stolen. Or maybe we had a really bad argument with them. Or maybe we found out they weren't the person they were claiming to be.

In Shamir's original paper he states that one of the great advantages of the scheme is that it is possible to create as many distinct sets of shares as you like without needing to modify the secret. Each set of shares is incompatible with the other sets. Using [Glenn Rempe's implementation](https://github.com/grempe/secrets.js), if we run the share method several times with the same secret, we get each time a different set of shares. When generating a new set, an extra check could be done to rule out the extremely improbable case that an identical set had been generated.

This means in a conventional secret sharing scenario (imagine the shares are written on paper and given to the custodians), we could simply give new shares to the custodians we do still trust and ask them to destroy the old ones. This would make the share belonging to the untrusted person become useless.

However, depending on the transmissionHowever, depending on the transmission system used it might not be so simple to destroy the message. This is particularly an issue for peer-to-peer systems which use immutable logs. One solution to this is to use ephemeral keys which are used only a single time for a particular share and can be deleted when a new set of shares is issued. This gives greatly increased security, but the cost is more keys to manage and increased complexity of the model. 

### Trusted execution environments

Having a good system of encryption does not give us security if the host system is compromised. Many options exist for which provide a secure execution environment to address this issue, for example Dyne.org's [Zenroom](https://zenroom.dyne.org).

However, if we are to assume the host system is insecure, there are many more considerations one needs to make, especially if secret is initially stored on disk. This goes beyond the scope of assessing the security of Shamir's scheme. 

## Conclusion

There are a variety of good options for addressing the security issues posed by Shamir's scheme. However, we have focussed here mainly on technical limitations of the scheme.  There are many other social aspects which pose threats to our model, which we will explore in another article. 

## References

- Beimel, Amos (2011). "Secret-Sharing Schemes: A Survey" http://www.cs.bgu.ac.il/~beimel/Papers/Survey.pdf
- Blakley, G.R. (1979). "Safeguarding Cryptographic Keys". Managing Requirements Knowledge, International Workshop on (AFIPS). 48: 313–317. doi:10.1109-/AFIPS.1979.98.
- Feldman, Paul (1987) "A practical scheme for non-interactive Verifiable Secret Sharing" Proceedings of the 28th Annual Symposium on Foundations of Computer Science
- Harn, L. & Lin, C. Detection and identification of cheaters in (t, n) secret sharing scheme, Des. Codes Cryptogr. (2009) 52: 15. https://link.springer.com/article/10.1007/s10623-008-9265-8
- Pedersen T.P. (1992) [Non-Interactive and Information-Theoretic Secure Verifiable Secret Sharing.](https://link.springer.com/chapter/10.1007%2F3-540-46766-1_9) In: Feigenbaum J. (eds) Advances in Cryptology — CRYPTO ’91. CRYPTO 1991. Lecture Notes in Computer Science, vol 576. Springer, Berlin, Heidelberg 
- Schneier, Bruce (2010) - DNSSEC Root Key held by 7 parties worldwide https://www.schneier.com/blog/archives/2010/07/dnssec_root_key.html
- Schoenmakers, Berry (1999) "A Simple Publicly Verifiable Secret Sharing Scheme and its Application to Electronic Voting" Advances in Cryptology-CRYPTO'99, volume 1666 of Lecture Notes in Computer Science, pages 148-164, Berlin, 1999. Springer-Verlag. 
- Shamir, Adi (1979). "How to share a secret". Communications of the ACM. 22 (11): 612–613. doi:10.1145/359168.359176.
- Zenroom, a virtual machine for fast cryptographic operations on elliptic curves, https://zenroom.dyne.org/

## See Also...

- Our [list of applications and articles on Shamir's secret sharing](https://gitlab.com/dark-crystal/research/-/blob/master/shamirs_secret_sharing_applications.md)
- [Brainstorming Coconut-related scenarios](https://gitlab.com/dark-crystal/research/-/blob/master/coconut_brainstorm.md) ('Coconut death' refers to a role playing game we did as part of our research where we tried to recover the keys of members of the group who had been hit by coconuts)
- [Thoughts on verifying received shards in dark crystal](https://gitlab.com/dark-crystal/research/-/blob/master/verifying_recived_shards.md)
