# Double Blind

Double Blind is an app for you to semi-anonymously sign messages for a group of
people. Each message can be verified as sent from **someone** in the group, but the
exact member cannot be identified (this is sometimes called a [group
signature](http://en.wikipedia.org/wiki/Group_signature)). These can be use for
anonymous feedback, forums, or many other semi-anonymous applications.

Double Blind uses public SSH keys to identify people, so groups are just a list
of public SSH keys. You can view public SSH keys from various places; for
example, you can view someone's GitHub keys at <https://github.com/stevenhao.keys>.

Currently, only RSA keys are supported.

## Signing Messages

To sign a message, you first need to have an SSH RSA key, e.g. `~/.ssh/id_rsa`.
(Rest assured, your SSH private key never leaves your machine.) To generate a
new one, use
```
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa -N ""
```

Then, to prove you own the key, sign the message `E PLURIBUS UNUM; DO NOT SHARE`
and paste the output in the `Your Double Blind Key` box:
```
echo "E PLURIBUS UNUM; DO NOT SHARE" | ssh-keygen -Y sign -n double-blind.xyz -f ~/.ssh/id_rsa
```
DO NOT share this output with anyone; Double Blind uses it as your proof of
identity.

Then, fill in the message you'd like to sign, a group name to mark which group
you're signing for, and a list of SSH keys for the members of the group. You can
directly ask for SSH keys, or you can look them up on GitHub at
<https://github.com/ecnerwala.keys>. Only RSA keys are supported for now.

Now, sign the message using the `Sign` button. This process might take
a little bit. Then, you can use the `Share Link` button next to the group
signature to get a sendable link with the group signature filled in. You can
also use the `Share Link` button next to the message to get a prefilled message,
if you'd like to share it with others to sign as well.

## Verifying Signatures

To verify a group signature, simply paste the group signature into the box on
the right hand side and click the `Verify` button. The Message, Group Name, and
Group Public Keys will be populated from the signature, but they may not be
truthful if the signature verification fails.

## Advanced Feature - Signer IDs

Double Blind supports an additional mode which allows you to sign messages with
a randomly generated **masked signer identity**. (You can enable this with the
`Secret ID` toggle.) In the default mode, group signatures are completely
anonymous beyond group membership (though beware, two group signatures with
identical contents may be identical). With masked identities, you are still
anonymous, but two messages with the same masked identity must correspond to the
same public key.

When generating masked identities, you need to specify an **identity
namespace**. Multiple messages signed with the same namespace and same public
key will produce the same masked identity, so you namespaces should be unique,
long, random strings unless you're explicitly trying to link your messages.

Additionally, you can **reveal** your masked identity with an **identity
revealer**. This will link your masked identity to your true public key, so do
not share the identity revealer unless you'd like to deanonymize your messages.

Due to the nature of RSA signatures, in some cases, a malicious actor may
construct a tampered RSA private key which allows them to sign
two messages in the same identity namespace but with two **different** masked
identities. Thus, DO NOT rely on masked identity for determining unique
identities, e.g. for anonymous voting protocols. There is a planned protocol
extension which will allow users to prove they do not have a tampered public
key, which would make this safe.

## Underlying Concepts

### ZK Proofs

Double Blind is built using [Zero-Knowledge
Proofs](https://en.wikipedia.org/wiki/Zero-knowledge_proof) (ZK proofs). ZK
proofs are essentially signatures which require knowledge of a value satisfying
a specific function in order to generate correctly (so they prove knowledge of
the value); however, they do not reveal these values to any validator (so they
are zero-knowledge). Surprisingly, ZK proofs can be constructed for *any*
computable function.

For Double Blind, the function we care about is
```
RSA_verify(YOUR_PUBLIC_SSH_KEY, YOUR_DOUBLE_BLIND_KEY, "E PLURIBUS UNUM; DO NOT SHARE") && GROUP.contains(YOUR_PUBLIC_SSH_KEY)
```
A ZK proof of this statement shows that you own your public ssh key and are part
of the group, but does not reveal your public ssh key beyond that.

In addition, for any fixed function, we can actually devise a scheme that
produces a very short proof: it is the same size irrespective of the
size/complexity of the function. Verification time is also constant; this
requires a precomputed short "verification key" which cryptographically encodes
the particular function. These **succinct** proofs are called zkSNARKs (Succinct
Non-interactive ARguments of Knowledge). zkSNARKs can be verified very quickly,
but signing (proving) a message still requires time proportional to the size of
the function.

### ZK Proof Construction

ZK proof protocols are generally specified as "arithmetic circuits" which
enforce particular constraints on the inputs. These circuits allow you to
constrain that two hidden "signals" add or multiply to another; signals can
correspond to provided inputs or can be computed intermediates.

### Signer ID

## How it works

RSA: https://en.wikipedia.org/wiki/RSA_(cryptosystem)

Circuit Diagram: https://excalidraw.com/#json=prlRhzCaDe1HEmrabvXs4,LSAmJLJh5JDf38Xh2g3i4g

Circom: https://github.com/iden3/circom

SnarkJS: https://github.com/iden3/snarkjs

SSHSIG: https://github.com/openssh/openssh-portable/blob/master/PROTOCOL.sshsig, https://github.com/openssh/openssh-portable/blob/master/ssh-keygen.c

PKCS 1: https://datatracker.ietf.org/doc/html/rfc8017#section-9.2