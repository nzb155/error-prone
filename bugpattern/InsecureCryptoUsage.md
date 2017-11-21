---
title: InsecureCryptoUsage
summary: A standard cryptographic operation is used in a mode that is prone to vulnerabilities
layout: bugpattern
tags: ''
severity: ERROR
providesFix: NO_FIX
---

<!--
*** AUTO-GENERATED, DO NOT MODIFY ***
To make changes, edit the @BugPattern annotation or the explanation in docs/bugpattern.
-->

_Alternate names: InsecureCipherMode_

## The problem
This checker looks for usages of standard cryptographic algorithms in
configurations that are prone to vulnerabilities. There are currently three
classes of problems that are covered by this checker:

*   Creating an instance of `javax.crypto.Cipher` using either the default
    settings or the notoriously insecure ECB mode. In particular, Java's default
    `Cipher.getInstance(AES)` returns a cipher object that operates in ECB mode.
    Dynamically constructed transformation strings are also flagged, as they may
    conceal an instance of ECB mode. The problem with ECB mode is that
    encrypting the same block of plaintext always yields the same block of
    ciphertext. Hence, repetitions in the plaintext translate into repetitions
    in the ciphertext, which can be readily used to conduct cryptanalysis. The
    use of IES-based cipher algorithms also raises an error, as all currently
    available implementations use ECB mode under the hood.

*   Using the Diffie-Hellmann protocol on prime fields. Most library
    implementations of Diffie-Hellman on prime fields have serious issues that
    can be exploited by an attacker. Any operation that may involve this
    protocol will be flagged by the checker. Implementations of the protocol
    based on elliptic curves (ECDH) are secure and should be used instead.

*   Using DSA for digital signatures. Some widely used crypto libraries accept
    invalid DSA signatures in specific configurations. The checker will flag all
    cryptographic operations that may involve DSA.


----------

### Positive examples
__InsecureCipherModePositiveCases.java__

{% highlight java %}
/*
 * Copyright 2015 Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.google.errorprone.bugpatterns.testdata;

import java.security.KeyFactory;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;
import javax.crypto.Cipher;
import javax.crypto.KeyAgreement;
import javax.crypto.NoSuchPaddingException;

/** @author avenet@google.com (Arnaud J. Venet) */
public class InsecureCipherModePositiveCases {
  static Cipher defaultAesCipher;

  static {
    try {
      // BUG: Diagnostic contains: the mode and padding must be explicitly specified
      defaultAesCipher = Cipher.getInstance("AES");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher defaultRsaCipher;

  static {
    try {
      // BUG: Diagnostic contains: the mode and padding must be explicitly specified
      defaultRsaCipher = Cipher.getInstance("RSA");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static final String AES_STRING = "AES";
  static Cipher defaultAesCipherWithConstantString;

  static {
    try {
      // BUG: Diagnostic contains: the mode and padding must be explicitly specified
      defaultAesCipherWithConstantString = Cipher.getInstance(AES_STRING);
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher explicitAesCipher;

  static {
    try {
      // BUG: Diagnostic contains: ECB mode must not be used
      explicitAesCipher = Cipher.getInstance("AES/ECB/NoPadding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher explicitDesCipher;

  static {
    try {
      // BUG: Diagnostic contains: ECB mode must not be used
      explicitDesCipher = Cipher.getInstance("DES/ECB/NoPadding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher explicitDesCipherWithProvider;

  static {
    try {
      // BUG: Diagnostic contains: ECB mode must not be used
      explicitDesCipherWithProvider = Cipher.getInstance("DES/ECB/NoPadding", "My Provider");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchProviderException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static String transformation;

  static {
    try {
      transformation = "DES/CBC/NoPadding";
      // BUG: Diagnostic contains: the transformation is not a compile-time constant
      Cipher cipher = Cipher.getInstance(transformation);
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static void transformationAsParameter(String transformation) {
    try {
      // BUG: Diagnostic contains: the transformation is not a compile-time constant
      Cipher cipher = Cipher.getInstance(transformation);
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  class CipherWrapper {
    Cipher cipher;

    // Make sure that the checker is enabled inside constructors.
    public CipherWrapper() {
      try {
        // BUG: Diagnostic contains: the mode and padding must be explicitly specified
        cipher = Cipher.getInstance("AES");
      } catch (NoSuchAlgorithmException e) {
        // We don't handle any exception as this code is not meant to be executed.
      } catch (NoSuchPaddingException e) {
        // We don't handle any exception as this code is not meant to be executed.
      }
    }
  }

  static Cipher complexCipher1;

  static {
    try {
      String algorithm = "AES";
      // BUG: Diagnostic contains: the transformation is not a compile-time constant
      complexCipher1 = Cipher.getInstance(algorithm);
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher complexCipher2;

  static {
    try {
      String transformation = "AES";
      transformation += "/ECB";
      transformation += "/NoPadding";
      // BUG: Diagnostic contains: the transformation is not a compile-time constant
      complexCipher2 = Cipher.getInstance(transformation);
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher IesCipher;

  static {
    try {
      // BUG: Diagnostic contains: the mode and padding must be explicitly specified
      IesCipher = Cipher.getInstance("ECIES");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("ECIES/DHAES/NoPadding");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("ECIESWITHAES/NONE/PKCS5Padding");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("DHIESWITHAES/DHAES/PKCS7Padding");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("ECIESWITHDESEDE/NONE/NOPADDING");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("DHIESWITHDESEDE/DHAES/PKCS5PADDING");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("ECIESWITHAES/CBC/PKCS7PADDING");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("ECIESWITHAES-CBC/NONE/PKCS5PADDING");
      // BUG: Diagnostic contains: IES
      IesCipher = Cipher.getInstance("ECIESwithDESEDE-CBC/DHAES/NOPADDING");

    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  interface StringProvider {
    String get();
  }

  public void keyOperations(StringProvider provider) {
    KeyFactory keyFactory;
    KeyAgreement keyAgreement;
    KeyPairGenerator keyPairGenerator;
    final String dh = "DH";
    try {
      // BUG: Diagnostic contains: compile-time constant
      keyFactory = KeyFactory.getInstance(provider.get());
      // BUG: Diagnostic contains: Diffie-Hellman on prime fields
      keyFactory = KeyFactory.getInstance(dh);
      // BUG: Diagnostic contains: DSA
      keyAgreement = KeyAgreement.getInstance("DSA");
      // BUG: Diagnostic contains: compile-time constant
      keyAgreement = KeyAgreement.getInstance(provider.get());
      // BUG: Diagnostic contains: Diffie-Hellman on prime fields
      keyPairGenerator = KeyPairGenerator.getInstance(dh);
      // BUG: Diagnostic contains: compile-time constant
      keyPairGenerator = KeyPairGenerator.getInstance(provider.get());
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }
}
{% endhighlight %}

### Negative examples
__InsecureCipherModeNegativeCases.java__

{% highlight java %}
/*
 * Copyright 2015 Google Inc. All Rights Reserved.
 *
 * Licensed under the Apache License, Version 2.0 (the "License");
 * you may not use this file except in compliance with the License.
 * You may obtain a copy of the License at
 *
 *     http://www.apache.org/licenses/LICENSE-2.0
 *
 * Unless required by applicable law or agreed to in writing, software
 * distributed under the License is distributed on an "AS IS" BASIS,
 * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 * See the License for the specific language governing permissions and
 * limitations under the License.
 */

package com.google.errorprone.bugpatterns.testdata;


import java.security.KeyFactory;
import java.security.KeyPairGenerator;
import java.security.NoSuchAlgorithmException;
import java.security.NoSuchProviderException;

import javax.crypto.Cipher;
import javax.crypto.KeyAgreement;
import javax.crypto.NoSuchPaddingException;

/**
 * @author avenet@google.com (Arnaud J. Venet)
 */
public class InsecureCipherModeNegativeCases {
  static Cipher aesCipher;
  static {
    // We don't handle any exception as this code is not meant to be executed.
    try {
      aesCipher = Cipher.getInstance("AES/CBC/NoPadding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static final String AES_STRING = "AES/CBC/NoPadding";
  static Cipher aesCipherWithConstantString;
  static {
    try {
      aesCipherWithConstantString = Cipher.getInstance(AES_STRING);
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher aesCipherWithProvider;
  static {
    try {
      aesCipherWithProvider = Cipher.getInstance("AES/CBC/NoPadding", "My Provider");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchProviderException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher arc4CipherConscrypt;
  static {
    try {
      arc4CipherConscrypt = Cipher.getInstance("ARC4", "Conscrypt");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchProviderException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher rc4CipherJsch;
  static {
    try {
      rc4CipherJsch = Cipher.getInstance("RC4", "JSch");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchProviderException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher arcfourCipherSunJce;
  static {
    try {
      arcfourCipherSunJce = Cipher.getInstance("ARCFOUR/ECB/NoPadding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher desCipher;
  static {
    try {
      desCipher = Cipher.getInstance("DES/CBC/NoPadding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher rsaCipher;
  static {
    try {
      rsaCipher = Cipher.getInstance("RSA/ECB/PKCS1Padding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  static Cipher aesWrapCipher;
  static {
    try {
      aesWrapCipher = Cipher.getInstance("AESWrap/ECB/NoPadding");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    } catch (NoSuchPaddingException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

  public void ellipticCurveDiffieHellman() {
    KeyFactory keyFactory;
    KeyAgreement keyAgreement;
    KeyPairGenerator keyPairGenerator;
    final String ecdh = "ECDH";
    try {
      keyFactory = KeyFactory.getInstance(ecdh);
      keyAgreement = KeyAgreement.getInstance("ECDH");
      keyPairGenerator = KeyPairGenerator.getInstance("EC" + "DH");
    } catch (NoSuchAlgorithmException e) {
      // We don't handle any exception as this code is not meant to be executed.
    }
  }

}
{% endhighlight %}
