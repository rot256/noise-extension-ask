---
title:      'Additional Symmetric Keys'
author:
 - Mathias hall-Andersen (mathias@hall-andersen.dk)
 - Second Author (second@email)
revision:   '1'
status:     'unofficial/unstable'
date:       '2018-07-09'
link-citations: 'true'
---

# 1. Introduction

This extension is intended for applications
wishing to use key material derived by Noise for auxiliary purposes or other extensions of Noise.
Examples include:

* Resumption PSKs - Deriving a "resumption PSK" from an initial Noise session.
  The resumption PSK can be used to perform a PSK Noise handshake later.
  ASK can also be used to derive associated data, such as labels to identify PSKs.

* Used to enable 0-RTT with patterns not normally permitting encryption
  in the first message (e.g. XX pattern), by combining ASK with a psk modifier.

* Deriving keys used to generate random padding or other
  length-hiding / traffic-hiding countermeasures,
  include the derivation of obfuscation keys used
  to discourage middle boxes from parsing fields inside messages.

* Generate application-layer keys in case the Noise handshake is
  being used as the handshake / key-establishment component within
  a larger secure transport layer protocol.

# 2. Overview

The Additional Symmetric Keys API is centered around labels and chains:
**labels** are arbitrary byte strings,
each label allows the construction of a **chain** based on the value of the label and
the current SymmetricState value.
Each chain can then be **invoked** any number of times to generate a arbitrary amount of key material,
the API exposed by this extension consists of 3 methods:

* **EnableASK()**: Enables ASK, by setting a boolean `ask_enable = true`,
  which causes ASK master keys to be derived.

* **InitializeASK(labels)**:
  If ASK is enabled and an non-empty ASK master key is available,
  the function makes a set of labels and initializes a chain for each label.
  This method can be called multiple times, both during and after the handshake
  and replaces any previous ASK chains when called.

* **GetASK(label)**:
  Assuming ASK is enabled and initialized, returns the next key from the
  appropriate chain, and advances the appropriate ASK chain key, deleting the
  previous chain key.

# 3. Security considerations

The Additional Symmetric Keys extension is design to meet the following security goals:

* The ASKs should be independent from the Noise chaining key.
  Recovering knowledge about the Noise chaining key from any number of ASK
  outputs must be infeasible.

* The ASKs should be mutual independent;
  deriving any of the ASK outputs from any other should be infeasible.

* ASKs should be capable of serving as collision-resistant hashes of the session transcript
  at the security level expected by the employed hash function (256/512-bit).

# 4. Design

In addition to the security goals stated above, the extension seeks to meet the following design goals:

* The mechanism should not be restricted to using the output key-material
  for particular purposes (e.g. the use cases listed above).

* When not enabled, the ASK mechanism should incur no additional computational cost.
  This allows libraries to implement the extension without introducing significant
  overhead on applications not using the ASK API.

# 5. Implementation

# 6. Rationales

Rather than truncating to 32 bytes (size of a symmetric key),
full output of HKDF is returned (rathe

# 7. IPR

This document is hereby placed in the public domain.

# 8. Acknowledgements

This extension is based on the "Resumption PSKs" discussion between:

- str4d (str4d@i2pmail.org)
- Trevor Perrin (trevp@trevp.net)
- Christopher Wood (christopherwood07@gmail.com)
- David Wong (davidwong.crypto@gmail.com)

And in particular the [ASK proposal outlined by Trevor Perrin](https://moderncrypto.org/mail-archive/noise/2018/001713.html).
From which both terminology and implementation details has been lifted into this document.

# 9.  References
