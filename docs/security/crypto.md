# __:material-security:{.lg .top} Cryptography__

This covers basics of encoding and cryptography. These concepts are used in authentication systems like SAML or OAuth, Client Server authentication, etc...

Key Concepts include

- [URL Encoding](#url-encoding)
- [Base64 Encoding](#base64-encoding)
- [Hashing](#hashing)
- [Symmetric Encryption](#symmetric-encryption)
- [Asymmetric Encryption](#asymmetric-encryption)
- [Hybrid Encryption](#hybrid-encryption)
- [Digital Signature & Verification](#digital-signature-and-verification)

!!! note
    For the purpose of illustration we will make use of Alice and Bob in our examples.

    Alice: The source/originator of data
    Bob: The recipent of data

## URL Encoding

This concept is used to encode special characters in an HTTP URL and vice-versa.

```{ .bash }
# Help
urlencode --help

# Url Encode  
urlencode -m "http://mudra?user=test name"

# Url Decode
urlencode -d http%3A//mudra%3Fuser=test+name
```

!!! warning
    This is not encryption

## Base64 Encoding

This concept is typically used to convert binary data to textual representation and vice-versa.

```{ .bash }
# Help 
base64 --help

# base64 Encoding. watch out for EOL (End Of Line) Char
# -n will not append a EOL
echo -n "This is a line" | base64

# base64 Decoding
echo -n VGhpcyBpcyBhIGxpbmU= | base64 -d

# base64 Encoding of file
base64 -i dataimage.png
base64 -i dataimage.png -o dataimage-base64.txt 

# Decoding 
base64 -d -i dataimage-base64.txt -o dataimage-decoded.png
```

!!! warning
    - This is not encryption
    - `base64` output may or may not end with new line. It depends on the input


## Hashing

This concept is used to convert data of any kind(text or binary) and length into a textual representation of fixed length.

```{ .bash }
# Help
openssl dgst -help # (1)!

# For File data
openssl dgst dataimage.png
openssl dgst -sha512 dataimage.png
openssl dgst -sha512 -binary dataimage.png # (2)!
openssl dgst -sha512 -binary -out dataimage-dgst dataimage.png  

# For Text data. watch out for EOL
echo -n xxx | openssl dgst -sha512

# Show again. Will hash to the same value. Danger
# Use of Hash in password

# Password.Default salt (show this twice)
openssl passwd -6 # (3)!

# own salt 
openssl passwd -6 -salt ThisIsASalt password
```

1.  `openssl dgst -list`: List digests algorithms.
    - `-sha256 (default)`
    - `-sha512`
    - `-md5`

2.  `openssl dgst` command outputs the digest in various formats. 
    - `-hex (default)` option changes the output format to hexadecimal. This is a human-readable format, but it's not very efficient in terms of storage or transmission.
    - `-binary` option changes the output format to binary, which is more efficient for storage and transmission, but not human-readable.

3.  These are the various hashing algorithms:
    - `-1`: MD5-based
    - `-5`: SHA256-based
    - `-6`: SHA512-based

!!! note
    - This fixed size representation is called __hash__ or __digest__
    - It's unique to the corresponding data and hence no collisions with other data
    - Data can not be recovered from the __hash/digest__
    - The various algorithms include `SHA-512 (creates a 512 bit hash)`, `SHA-256 (creates a 256 bit hash)`, `SHA-1`, `MD5`
    - It's used to store password securly and to verify integrity of downloaded files

!!! warning
    - This is not encryption
    - `openssl dgst` output ends with a new line
    - `openssl passwd` output ends with a new line


## Symmetric Encryption

This concept is used to encrypt data using a well known algorithm with secret key.

```{ .bash }
# Help 
openssl enc -help 

# List of Ciphers
openssl enc -list # (1)!

# AES encryption with secret(Salt is added automatically) hashed with SHA1. 
# Encrypting text
echo -n "this is junk" | openssl enc -aes-256-cbc -k secret -md sha1 -pbkdf2 -base64 # (2)!

# Decrypting text
echo "U2FsdGVkX18LzAz2kqRqo4B0dOAYUfLaxyN6b8ZZkNY=" | openssl enc -aes-256-cbc -d -k secret -md sha1 -pbkdf2 -base64


# Encrypting a file
openssl enc -aes-256-cbc -k secret -md sha1 -base64 -pbkdf2 -in dataimage.png -out dataimage-enc

# Decrypting an encrypted data
openssl enc -d -aes-256-cbc -k secret -md sha1 -base64 -pbkdf2 -in dataimage-enc -out dataimage-original.png
```

1.  Lists various encryption algorithms. ex: `-aes-256-cbc`, `-aes-128-ecb`, `-blowfish`, etc...
2.  - Various options to pass passphrase for secret key derivation

        - `-k password`
        - `-kfile file_path`
        - `-pass arg`: This option is more flexible and secure. The argument can take several forms:
            - `pass:password` - directly specifies the password
            - `env:var` - gets the password from an environment variable
            - `file:pathname` - gets the password from the first line of the file
            - `fd:number` - gets the password from a file descriptor
            - `stdin` - gets the password from standard input

    - `-md`: Digest algorithm used for key derivation. Use `openssl list -digest-commands` to list available digest algorithms.
    - `-base64`: Convert output to base64 format
    - `-pbkdf2`: used for key generation
    - `-iv`: initial vector. passed explicitly with `-K`. hexadecimal digits
    - `-K`: hexadecimal key. must be used with `-k` to generate `-iv`. {--Although does't make sense to pass secret and the Keu--}


??? example
    Alice encrypts the data using a `AES secret key` to produce CipherData and transmits it to Bob. Bob decrypts the CipherData using the same `AES secret key`

!!! note
    - The encrypted data is called __CipherData__
    - Data can be recovered using the __secret key__
    - The various algorithms include `AES-512 (512 bit key size)`, `AES-256 (256 bit key size)`, `AES-128`, `BlowFish`, `DES`

!!! warning
    - The secret key must be secure
    - Secure key distribution is one of it's short commings
    - `openssl enc` output ends with a new line


## Asymmetric Encryption

This concept is used to encrypt data using a key pair, i.e private key and public key. public key is distributed to users.

```{.bash}
# openssl genrsa
# generates a RSA private key 
# Help
openssl genrsa -help

# Create a RSA private key of size 2048
openssl genrsa -out private.pem 2048

# Create an encrypted RSA private key of size 2048
openssl genrsa -aes256 -out private_enc.pem 2048 # (1)! // encrypt using AES256 

# Create public key
# will ask for secret if encrypted
openssl rsa -in private.pem -pubout -out public.pem

# The pkeyutl command can be used to perform low-level public key operations using any 
# supported algorithm.
# encrypted using rsa
echo "Run for your life Now!" > important.txt

# Alice wants to send to bob. So encrypt using bob's public key 
openssl pkeyutl -encrypt -pubin -inkey public.pem -in important.txt -out important.enc

# if important.txt is > key size we will have issue

# Bob will decrypt it using his private key
openssl pkeyutl -decrypt -inkey private.pem -in important.enc

# Create a certificate request 
# Create a private key and self signed certificate
# -nodes - dont encrypt the private key
# -newkey rsa:2048 
# Why sha256 ? 
openssl req -x509 -nodes -sha256 -days 3650 -newkey rsa:2048 -keyout private.key -out certificate.crt 
# (2)!

# Creates a self-signed certificate using existing private key
openssl req -new -x509 -key private.key -out certificate.crt -days 365 -sha256
```

1.  `-aes256`: To encrypt the private key using a passphrase for additional security. The same passphrase must be used along with public key to decrypt data
2.  Creates private key and self-signed `x509` certificate
    - `rsa:2048`: key size
    - `-days 3650`: validity of 3650 days / 10 years
    - `-nodes`: don't encrypt private key. i.e. don't use a pass phrase
    - `-sha256`: digest/hashing algorithm used for signing. [refer](#digital-signature-and-verification)

??? example
    Alice encrypts the data using Bob's `RSA public key` to produce CipherData and transmits it to Bob. Bob decrypts the CipherData using his `RSA private key`

!!! note
    - The encrypted data is called __CipherData__
    - The various algorithms include `RSA(1024, 2048, 4096)`, `Ed25519`
    - Encrypt with public key, Decrypt with private key
    - Sign with private key, verify with public key.
    - Public key needs to be certified to establish legitamacy of entity. This is done by Certificate Authority(CA), which validates the identity of an entity. The public key certificate is aka x.509 certificate


!!! warning
    - Not efficent for encryption of large data
    - `openssl pkeyutl` doesn't end with a new line


## Hybrid Encryption

This concept is used for encrypting large data, which can not be handled by asymmetric encryption. 

```{.bash}
# Encrypt data using AES256 with a passphrase
cipher_data=$(echo -n "this is junk" | openssl enc -aes-256-cbc -k secret -md sha1 -pbkdf2 -base64)
echo $cipher_data

# Encrypt passphase using RSA public key
# Base64 encode binary to string format
cipher_key=$(echo -n "secret" | openssl pkeyutl -encrypt -pubin -inkey public.pem -in - | base64)
echo $cipher_key

# Base64 decode string to binary format
# Decrypt encrypted passphase using RSA private key
key=$(echo -n $cipher_key | base64 -d | openssl pkeyutl -decrypt -inkey private.pem -in -)
echo $key

echo $cipher_data | openssl enc -d -aes-256-cbc -k $key -md sha1 -pbkdf2 -base64
```

??? example
    Alice ecrypts the data(large) using a `AES secret key` to produce CipherData. Alice, also encrypts aforementioned `AES secret key` to produce a CipherKey, using Bob's `RSA public key`. Alice then transmits them both to Bob. Who, first decrypts the CipherKey using his `RSA private key` to recover the `AES secret key` and subsequently decrypts CipherData using the rcovered `AES secret key`

!!! note
    - The encrypted data is called __CipherData__
    - The encrypted public key is called __CipherKey__


## Digital Signature and Verification

This concept is used to verify the legitamacy of the data transmitted. Signing is done with asymmetric key pair, thus limited by the size of data.

{++Solution, hash(fixed length regardless of input) the data followed by encryption of hash with the private key++}. 

{++Verify the signature by comparing the hash of data recieved against the pre-generated hash(retrived by decrypting the encrypted hash using the public key).++}

```{.bash}
# Hashing data and Signing hash with private key
# Base64 encode binary to string format
signed_data_b64=$(echo -n "This is my data" | openssl dgst -sha512 -sign private.pem | base64)
echo $signed_data_b64

# Base64 decode string to binary format
echo -n $signed_data_b64 | base64 -d > signed_data
echo -n "This is my data" | openssl dgst -sha512 -verify public.pem -signature signed_data
```

??? example
    Alice hashes the data, signs the hash using her `RSA private key` and then trasnmits the data, signed hash, hashing algorithm to Bob. Bob, verifies the signed hash using Alice's `RSA public key`, which yields the original hash. Bobs then compares the retrieved hash with his own hash(by hashing the data), to validate legitemacy of the sender

!!! note
    - Data to be singed can't be large. Hence data is hashed first

!!! warning
    - `openssl dgst -sign` ends with a new line

## Miscellaneous

### Encodings

cryptographic objects like keys, certificates, and CSRs can be represented in various formats depending on the use case, security requirements, and compatability with cryptographic systems.

| Encoding | Type	| Humand-readable format? | File Extension	| Use Case |
| --- | ---	| --- | ---	| --- |
| HEX [^1] | Encoding | ✅ | `.hex` | Debugging, cryptographic hashes(MD5, SHA-256) |
| Base64 [^2] | Encoding | ✅ | `.b64` | Encoding binary data into text, URL encoding |
| DER [^3] | Binary | ❌ | `.der`, `.cer` | Certificates (efficient) |

| Format | Humand-readable format? | File Extension	| Use Case |
| --- | --- | ---	| --- |
| PEM [^4] | ✅ | `.pem`, `.crt`, `.cer`, `.key`, `.csr` | Suitable format for Keys, certificates, CSRs |
| JWK [^5] | ✅ (JSON) | `.jwk` | Web-based private key format |
| JWKS [^6] | ✅ (JSON) | `.jwk` | Public keys distribution in web-auth systems |

[^1]:  __HEX(Hexadecimal) Encoding__
    
    - Encoding format that represents binary data using base-16(0-9, A-F).
    - Each byte (8 bits) is represented by two hexadecimal characters.
    - No `=` padding needed
    - The output is exactly twice the size of the original binary data. i.e. `2x(1 byte - 2 hex chars)`
    ```
    01001101 01100001 01101110 -> 4D 61 6E
    ```

[^2]:  __Base64 Encoding__
    
    - Base64 represents binary data using a base-64 numbering system (A-Z, a-z, 0-9, +, /).
    - Every 3 bytes (24 bits) are split into 4 Base64 characters.
    - Uses `=` padding (if needed)
    - The output is exactly twice the size of the original binary data. i.e. `1.33x(3 bytes - 4 base64 chars)`
    ```
    01001101 01100001 01101110 -> TWFu
    ```

[^3]:  __DER (Distinguished Encoding Rules)__
    
    - DER is a binary format used for certificates and keys.
    - More compact than PEM
    ```
    30 82 01 0A 02 82 01 01 00 A0 11...
    ```

[^4]:  __PEM (Privacy-Enhanced Mail)__
    
    - Used for cryptographic objects like keys, certificates, and CSRs. i.e. Encapsulates cryptographic structures (PKCS#1, PKCS#8, X.509, etc.)
    - Base64-encoded DER (binary) data.
    - Always contains headers and footers.
    ```
    -----BEGIN RSA PRIVATE KEY-----
    MIIEpAIBAAKCAQEA7GZog5s3O0rCZmVw2 ...
    -----END RSA PRIVATE KEY-----
    ```

[^5]:  __JWK (Json Web Key)__
    
    - Represents a single cryptographic key(public/private) in a structured JSON format.
    - Used in JWT(Json Web Token) signing, encryption, and verification.
    ``` json
    {
        "kty": "RSA",
        "kid": "1234",
        "alg": "RS256",
        "use": "sig",
        "n": "0vx7agoebGcQCw...",
        "e": "AQAB"
    }
    ```

[^6]:  __JWKS (Json Web Key Set)__
    
    - A collection of multiple JWKs stored in a JSON array.
    - OAuth2 and OpenID Connect use JWKS URLs instead of X.509 certificates for key distribution.
    - Typically served from a well-known endpoint (e.g., https://auth.example.com/.well-known/jwks.json).
    - Enables automatic key rotation without breaking authentication.
    - ==JWKS is the JSON Web equivalent of a certificate chain.==
    ``` json
    {
        "keys": [
            {
                "kty": "RSA",
                "kid": "1234",
                "alg": "RS256",
                "use": "sig",
                "n": "0vx7agoebGcQCw...",
                "e": "AQAB"
            },
            {
                "kty": "EC",
                "kid": "5678",
                "alg": "ES256",
                "use": "sig",
                "crv": "P-256",
                "x": "abc123...",
                "y": "xyz456..."
            }
        ]
    }
    ```

### PKCS (Public Key Cryptography Standards)

- PKCS defines structures for cryptographic objects (keys, certificates, etc.).
- PKCS data can be stored in DER (binary) or PEM (Base64-encoded) format.
- Different PKCS standards apply to different types of cryptographic objects.

| Structures | Type | Human-readable PEM format? | File Extension	| Use Case |
| --- | ---	| --- | ---	| --- |
| PKCS#1 | Key Format | ✅ (PEM) | `.pem`, `.key` | Legacy RSA private/public keys |
| PKCS#8 | Key Format | ✅ (PEM)	| `.pem`, `.key` |	Modern private key storage (all algorithms) |
| PKCS#7 | Certificate Chain |	✅ (PEM) | `.p7b`, `.p7c` | Certificate chains & signatures |
| PKCS#10 | Certificate Signing Request(CSR) | ✅ (PEM) |	`.csr` | CSR for certificate requests |
| PKCS#12 (PFX) | Private key + certificate bundle | ❌ No | `.pfx`, `.p12` | Secure key + certificate storage |


### OpenSSL

| Category | Key Commands |
| :--- | :--- |
| Public Key Infrastructure (PKI) | `openssl verify`, `openssl crl`, `openssl ocsp` |
| Secure Communication (TLS/SSL) | `openssl s_client`, `openssl s_server` |
| Key & File Format Conversion | `openssl pkcs8`, `openssl pkcs7`, `openssl base64` |
| Random Number Generation | `openssl rand` |
| Debugging & Info Retrieval | `openssl version`, `openssl errstr`, `openssl list` |

=== "Message Digests"

    The `openssl dgst` command is used to compute message digests (hashes) of files or input data. In addition, we can use the `-hmac` option to add authentication with a secret key, this is called __Hashed Message Authentication Codes (HMACs)__.

    ``` .bash hl_lines="13"
    openssl dgst --help # (1)!

    # Generate an SHA256 hash for a file
    openssl dgst -sha256 -hmac "secret" file.txt
    # Output: Hex-encoded hash

    # Generate an HMAC-SHA256 hash for a file using a secret key
    openssl dgst -sha256 -hmac "mysecretkey" file.txt
    # Output: Hex-encoded hash
    ```

    1.  __Common Options__:
        - `-sha256`: Uses SHA-256 hashing algorithm. To see all options `openssl list -digest-algorithms`
        - `-sha512`: Uses SHA-512 hashing algorithm
        - `-md5`: Uses MD5 hashing algorithm
        - `-r`: Outputs hash in raw format
        - `-hmac <key>`: Computes HMAC with the given key
        - `-sign <private_key>`: Signs the hash with a private key
        - `-verify <public_key>`: Verifies a signature with a public key

=== "Signing and Verification"

    Signing ensures the integrity and authenticity of data, while verification confirms whether a signature is valid.

    __Using `openssl dgst` for Signing and Verification__

    - First hashes the input file using SHA-512 (or other hash functions like SHA-256).
    - Then signs the hash instead of the original message.
    - ==Suitable for standard digital signatures, following best practices for authentication.==

    ``` .bash hl_lines="5-6"
    # Sign a file using a private key (SHA-512)
    openssl dgst -sha512 -sign private.pem -out signature.bin input.txt
    # Output: Binary signature file (signature.bin)

    openssl dgst -sha512 -sign private.pem input.txt | base64 > signature.pem
    # Output: PEM format (signature.pem)

    # Verify the signature using the corresponding public key
    openssl dgst -sha512 -verify public.pem -signature signature.bin input.txt
    # Output: "Verified OK" or "Verification Failure"
    ```

    __Using `openssl pkeyutl` for Signing and Verification__

    - Directly signs or verifies raw data using asymmetric key operations.
    - Does not hash the input before signing.
    - Suitable when you need low-level cryptographic operations (e.g., working with PKCS#1, OAEP, or PSS padding).
    
    ``` .bash hl_lines="5-6"
    # Sign a file using a private key with RSA (PKCS#1 v1.5 padding)
    openssl pkeyutl -sign -inkey private.pem -in input.txt -out signature_pkey.bin
    # Output: Binary signature file (signature_pkey.bin)

    openssl pkeyutl -sign -inkey private.pem -in input.txt | base64 > signature_pkey.pem
    # Output: PEM format (signature_pkey.pem)

    # Verify the signature using the corresponding public key
    openssl pkeyutl -verify -inkey public.pem -in input.txt -sigfile signature_pkey.bin
    # Output: "Signature Verified Successfully" or an error message
    ```

=== "Encryption & Decryption"

    OpenSSL provides multiple commands for encryption and decryption:

    - `pkeyutl`: Used for asymmetric encryption with RSA or EC keys.
    - `enc`: Used for symmetric encryption with ciphers like AES.

    __Asymmetric Encryption & Decryption__

    ``` .bash
    openssl pkeyutl --help # (1)!

    # Encrypt a file using a public key (RSA)
    openssl pkeyutl -encrypt -in plaintext.txt -out encrypted.bin -pubin -inkey public.pem
    # Output: Binary encrypted file (raw RSA encryption)

    # Decrypt the file using a private key
    openssl pkeyutl -decrypt -in encrypted.bin -out decrypted.txt -inkey private.pem
    # Output: Decrypted plaintext file
    ```

    1.  __Common Options__:
        - `-encrypt`: Encrypts input using a public key
        - `-decrypt`: Decrypts input using a private key
        - `-pubin`: Specifies that the input key is a public key
        - `-inkey <file>`: Specifies the key file (private or public)
        - `-pkeyopt rsa_padding_mode:oaep`: Uses OAEP padding for RSA
        - `-pkeyopt rsa_padding_mode:pkcs1	`: Uses PKCS#1 v1.5 padding for RSA
            

    __Symmetric Encryption & Decryption__



    ``` .bash
    openssl enc --help # (1)!

    # Encrypt a file using AES-256 in CBC mode with a password
    openssl enc -aes-256-cbc -salt -pbkdf2 -in plaintext.txt -out encrypted.enc -pass pass:MySecretPassword
    # Output: Encrypted binary file (Base64 encoding optional)

    # Decrypt the AES-256 encrypted file
    openssl enc -d -aes-256-cbc -pbkdf2 -in encrypted.enc -out decrypted.txt -pass pass:MySecretPassword
    # Output: Decrypted plaintext file
    ```

    2.  __Common Options__:
        - `-e`: Encrypts the input(optional, as encryption is the default mode)
        - `-d`: Decrypts the encrypted file
        - `-aes-256-cbc`: Uses AES-256 encryption in CBC mode. Supports other ciphers, see full list `openssl enc --ciphers`
        - `-pbkdf2`: Enables PBKDF2 key derivation for stronger security
        - `-pass pass:<password>`: Specifies the password for encryption
        - `-salt`: Adds a salt to enhance security
        - `-base64`: Encodes output in Base64 instead of binary


=== "Key & Certificate Management"

    __Generating Private Keys__

    - `genpkey` is the modern command and supports multiple key types: RSA, EC, Ed25519, DH, etc.
    - `genrsa` is the legacy command and supports only RSA key type.

    ``` .bash
    openssl genpkey --help # (1)!

    # Generate a 2048-bit RSA private key
    openssl genpkey -algorithm RSA -out private.pem -pkeyopt rsa_keygen_bits:2048
    # Output: Private key file (PEM format)

    # Generate an EC private key using the prime256v1 curve
    openssl genpkey -algorithm EC -pkeyopt ec_paramgen_curve:P-256 -out ec_private.pem
    # Output: EC private key file (PEM format)

    # Generate a password-protected RSA private key (with AES-256 encryption)
    openssl genpkey -algorithm RSA -out encrypted_private.pem -aes-256-cbc -pass pass:MySecretPassword
    # Output: Encrypted private key file
    ```

    1.  __Common Options__:
        - `-algorithm <name>`: Specifies the key algorithm (e.g., `RSA`, `EC`, `ED25519`, `ED448`, `DH`, `DSA`legacy)
            - `-pkeyopt rsa_keygen_bits:<size>`: Sets the RSA key size (e.g., 1024, 2048, 3072, 4096 bits)
            - `-pkeyopt rsa_padding_mode:<mode>`: Sets RSA padding mode (e.g., `oaep`, `pkcs1`)
            - `-pkeyopt ec_paramgen_curve:<curve>`: Specifies the EC curve (e.g., P-256, P-384, P-521, secp256k1, etc.)
            - `ED25519`, `ED448`: does not require a key size or curve
            - `-pkeyopt dh_paramgen_prime_len:2048`: Specifies the key size (e.g., 1024, 2048, 4096 bits)
            - `-pkeyopt dsa_paramgen_bits:2048`: Specifies the key size (e.g., 1024, 2048, 3072 bits)
        - `-aes-256-cbc`: Uses AES-256 encryption in CBC mode. Supports other ciphers, see full list `openssl enc --ciphers`
        - `-pass pass:<password>`: Specifies a password for encryption
        - ` -outform PEM|DER`: Output format (DER or PEM). defaults to PEM
        - `-outpubkey <file>`: Output public key file
        - `-out <file>`: Output (private key) file

    
    __Generating Public Keys from Private Keys__

    ``` .bash
    openssl pkey --help # (1)!

    # Extract the public key from a private key
    openssl pkey -in private.pem -pubout -out public.pem
    # Output: Public key file (PEM format)

    # View private key details
    openssl pkey -in private.pem -noout -text
    # Output: human-readable format
    ```

    1.  __Common Options__:
        - `-in <file>`: Specifies the input key file (private or public)
        - `-pubout`: Extracts the public key from a private key
        - `-aes-256-cbc`: Encrypts the private key with AES-256 encryption. Supports other ciphers, see full list `openssl enc --ciphers`
        - `-pass pass:<password>`: Specifies a password for encryption
        - ` -outform PEM|DER`: Output format (DER or PEM). defaults to PEM
        - `-out <file>`: Output (private key) file


    __Generating Self-Signed Certificate & Certificate Signing Request(CSR)__

    ``` .bash
    openssl req --help # (1)!

    # Generate a self-signed certificate (valid for 365 days)
    openssl req -x509 -sha512 -new -key private.pem -keyform pem \
    -out certificate.pem -days 365 -subj "/CN=example.com"
    # Output: Self-signed certificate file (PEM format)

    # Display details of certificate
    openssl x509 -in certificate.pem -noout -text
    # Output: human-readable format

    # Generate a CSR and a new private key
    openssl req -new -newkey rsa:2048 -nodes -keyout private.pem -out request.csr
    # Generate a CSR using an existing private key
    openssl req -new -key private.pem -out request.csr \
    -subj "/C=US/ST=California/L=San Francisco/O=Call For Help/OU=IT/CN=example.com" \
    -addext "basicConstraints = critical, CA:false" \
    -addext "keyUsage = digitalSignature, keyEncipherment" \
    -addext "subjectAltName = DNS:server.example.com, DNS:www.example.com"
    # Output: Certificate Signing Request file (PEM format)

    # Display details of a CSR
    openssl req -text -noout -verify -in request.csr
    # Output: human-readable format
    ```

    1.  __Common Options__:
        - `-config`: This allows an alternative configuration file to be specified
        - `-new`: Creates a new certificate signing request (CSR)
        - `-key <file>`: Uses an existing private key for the CSR
        - `-newkey <alg>:<bits>`: Generates a new private key and CSR (e.g., `rsa:2048`)
        - `-nodes / -noenc`: Generates a private key without passphrase protection
        - `-keyout`: Specify the output file for the newly generated private key
        - `-x509`: Generates a self-signed certificate instead of a CSR
        - `-sha512`: Uses the SHA-512 digest algorithm. To see all options `openssl list -digest-algorithms`
        - `-days <n>`: Sets the validity period of a self-signed certificate (e.g., -days 365)
        - `-out <file>`: Specifies the output file (CSR or certificate)
        - `-verify`: Verifies the CSR content
        - `-subj <DN>`: Bypasses interactive prompts and provides a Distinguished Name (DN) in-line
        - `-addext key=value`: Add a specific extension to the certificate (if -x509 is in use) or certificate request. [refer](#extensions)


    ??? note "__Entity__"
        
        Entities refer to the parties(subjects) that are involved in the certificate issuance and usage. These are included as metadata in a certificate. Typical certificate entities include:

        - __Certificate Authority (CA)__: The entity that issues the certificate.
        - __Subject (End Entity)__: The entity (e.g., a server, user, or organization) that the certificate is issued for.
        - __Issuer__: The entity that signs and verifies the certificate (usually a CA).
        - __Public Key Owner__: The entity associated with the certificate’s private key (usually Subject). 


    ??? note "__Distinguished Name(DN)__"

        This is a structured identifier used to uniquely identify an entity within a Public Key Infrastructure (PKI). It appears in a digital certificate as a part of both the Subject and Issuer fields. [refer](https://docs.openssl.org/3.4/man1/openssl-req/#distinguished-name-and-attribute-section-format)

        ==A DN is formatted using a hierarchical structure with Attribute-Value Pairs==
        ``` .ini title="Example"
        Subject DN: CN=www.example.com, O=Example Inc., L=New York, ST=NY, C=US
        Issuer DN: CN=Example Root CA, O=Example Certificate Authority, C=US
        ```

        Common DN Attributes

        | Field | Description | Example Value |
        | --- | --- | --- |
        | CN (Commong Name) | The primary identifier (e.g., domain name or user) | `/CN=www.example.com` |
        | O (Organization) | The organization to which the entity belongs | `/O=MyCompany` |
        | OU (Organizational Unit) | A sub-division within an organization | `/OU=IT` |
        | L (Locality) | City or locality of the entity | `/L=San Francisco` |
        | ST (State/Province) | The state or province where the entity is located | `/ST=California` |
        | C (Country) | The country code (ISO 3166-1 alpha-2) | `/C=US` |
        | name	| full name | `name=John Doe` |
        | surname | | `SN=Doe` |
        | givenName | |	`GN=John` |
        | initials | | `initials=JD` |
        | dnQualifier | | `dnQualifier=123456` |


    <div id="extensions"></div>
    ??? note "__Extensions__"

        Extensions are additional fields in an X.509 certificate that define its usage, constraints, and metadata. These fields extend the certificate's purpose beyond just identifying the subject.

        [Standard extensions](https://docs.openssl.org/1.1.1/man5/x509v3_config/#standard-extensions). Some common extensions:

        - __subjectAltName (SAN)__: Adds alternative names (e.g., www.example.com, mail.example.com).
        
            This is a multi-valued extension that supports several types of name identifier, including __email__ (an email address), __URI__ (a uniform resource indicator), __DNS__ (a DNS domain name), __RID__ (a registered ID: OBJECT IDENTIFIER), __IP__ (an IP address), __dirName__ (a distinguished name), and __otherName__.

            ```
            subjectAltName = email:copy, email:my@example.com, URI:http://my.example.com/

            subjectAltName = IP:192.168.7.1

            subjectAltName = email:my@example.com, RID:1.2.3.4
            ```

        - __keyUsage__: Defines what the certificate can be used for (e.g., Digital Signature, Key Encipherment).
        - __basicConstraints__: Determines if the certificate is for a CA or an end entity.
        - __authorityKeyIdentifier__: Links the certificate to its issuer

    __Generating a certificate__

    `openssl ca` command often refers to a configuration file for ease 

    ``` .bash
    openssl ca --help # (1)!

    # Generating a certificate for a CSR
    openssl ca -config openssl.cnf -policy signing_policy \
    -extensions signing_extensions \
    -in request.csr -out server.crt -keyfile ca.key -cert ca.crt

    # Display details of certificate
    openssl x509 -in server.crt -noout -text
    # Output: human-readable format

    # Verify certificates against trusted CA certificates.
    # Checks if server.crt is properly signed by CA
    openssl verify -CAfile ca.crt server.crt
    ```

    1.  __Common Options__:
        - `-config <file>`: Specifies an alternative configuration file for CA parameters.
        - `-in <file>`: Specifies the certificate signing request (CSR) to sign.
        - `-out <file>`: Specifies the file to write the signed certificate.
        - `-keyfile <file>`: Specifies the CA's private key file used for signing.
        - `-cert <file>`: Specifies the CA's public certificate file.
        - `-days <n>`: Sets validity duration (in days) for the issued certificate.
        - `-extensions <section>`: Specifies the extensions section from the config file.
        - `-policy <policy>`: Sets the name-policy (e.g., policy_match, policy_loose) used to validate the CSR’s DN.
        - `-md <algorithm>`: Specifies the digest algorithm (e.g., sha256, sha512) for signing.
        - `-subj`: Supersedes subject name given in the request. Multi-valued RDNs can be formed by placing a + character instead of a / between the AttributeValueAssertions (AVAs) that specify the members of the set. Example: `/DC=org/DC=OpenSSL/DC=users/UID=123456+CN=John Doe`

    ``` .ini title="openssl.cnf"
    --8<-- "docs/security/openssl.cnf:ca"

    # (1)!
    --8<-- "docs/security/openssl.cnf:req"

    # (2)!
    --8<-- "docs/security/openssl.cnf:DN"

    --8<-- "docs/security/openssl.cnf:ca_extensions"

    # (3)!
    --8<-- "docs/security/openssl.cnf:cert_extensions"

    # (4)!
    --8<-- "docs/security/openssl.cnf:cert_policy"
    ```

    1.  
        Default config for `openssl req`, used while creating CSR. [options](https://docs.openssl.org/1.1.1/man1/req/#configuration-file-format)

        `x509_extensions`: extensions used for self-signed certificates

        `req_extensions`: extensions used for certificate request

    2. All available distinguished name [attributes](https://docs.openssl.org/3.4/man1/openssl-req/#distinguished-name-and-attribute-section-format)
    
    3. Extensions used for signing certificates. e.g. `openssl ca -extensions cert_extensions`

    4. Policy used for signing certificates. e.g. `openssl ca -policy cert_policy`


=== "Miscellaneous"

    ``` .bash
    # Debug/test SSL/TLS connections to servers.
    # Connects to a remote server, shows TLS handshake and certificates.
    openssl s_client -connect example.com:443

    # Create a temporary TLS/SSL server for testing clients.
    openssl s_server -accept 4433 -cert server.crt -key server.key

    # Convert PEM key to PKCS#8:
    # Convert private keys to a standardized, interoperable PKCS#8 format.
    openssl pkcs8 -topk8 -inform PEM -in private.key -out key-pkcs8.pem -nocrypt

    # Extract certificates from PKCS#7 file:
    # Convert and manage PKCS#7 bundles commonly used for distributing multiple certificates.
    openssl pkcs7 -inform PEM -in certs.p7b -print_certs -out certs.pem

    # Create a PKCS#12 file from key & certificate
    openssl pkcs12 -export -out certificate.p12 -inkey private.key -in certificate.crt -certfile ca_bundle.crt

    # Extract private key from PKCS#12 file
    openssl pkcs12 -in certificate.p12 -nocerts -nodes -out private.key
    # Extract certificate from PKCS#12 file
    openssl pkcs12 -in certificate.p12 -clcerts -nokeys -out certificate.crt


    # Base64
    openssl base64 -in binary.bin -out encoded.txt
    openssl base64 -d -in encoded.txt -out decoded.bin

    # Random Number Generation
    openssl rand -hex 32
    openssl rand 64 -out random.bin

    # List supported algorithms, cipher suites, or digests.
    openssl list -digest-algorithms
    openssl list -cipher-algorithms
    openssl list -public-key-algorithms
    ```