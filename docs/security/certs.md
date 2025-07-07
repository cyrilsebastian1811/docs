# __:material-certificate-outline:{.lg .top} Certificates__

Trusting a Certificate: This involves explicitly marking a specific certificate as trusted in your system or application. This is typically done for self-signed certificates or certificates not issued by a trusted CA. When you trust a certificate, you're saying that you trust the identity that the certificate represents, and you trust the entity that issued the certificate. However, this trust does not extend to any other certificates that might be issued by the same entity.

Adding a CA to the Trust Store: This involves adding the root certificate of a CA to your system's or application's trust store. The trust store is a database of trusted CAs. When a CA is in the trust store, it means you trust that CA to issue certificates. Therefore, any certificate signed by that CA will be automatically trusted. This is a broader level of trust because it applies to any certificates issued by that CA, not just a single certificate.


## Certificate Authority (CA)

A certificate authority is a company or organization that acts to validate the identities of entities (such as websites, email addresses, companies, or individual persons) and bind them to cryptographic keys through the issuance of electronic documents known as digital certificates.

These certificates allow secure, encrypted communication between two parties through public key cryptography. The CA verifies the certificate applicant’s identity and issues a certificate containing their public key. The CA will then digitally sign the issued certificate with their own private key which establishes trust in the certificate’s validity.

### How Does a CA Validate and Issue Digital Certificates?

1. Applicant first generates a public and private key pair
2. The applicant then sends a certificate signing request (CSR) containing their public key and other identifying details to the CA through an online form.
3. CA validates the applicant’s identity and the right to claim credentials such as domain names for server certificates or email addresses for email certificates in the CSR.
4. If validation is successful, the CA issues the certificate containing the details and public key from the CSR, and also digitally signs the issued certificate with their own private key to confirm they verified the identity.


??? info

    __Certificates may contain information such as:__

    - Domain names

    - Email addresses

    - Business or individual identity

    - The public key used to enable encryption

    - Issuing CA details

    - Validity period

    - Certificate serial number

    - Signature to prevent tampering

    By issuing a certificate, the CA states that the public key contained within belongs to the listed identity.

<!-- ### Certificate Authority (CA) hierarchy -->

### How Do CAs Help Establish Trust?

For an issued certificate to be trusted, the issuing CA must be trusted. CAs establish trust through certificate chains, aka __CA hierarchy__.

A certificate chain links your end-entity certificate back to a trusted root CA certificate through intermediate issuing CAs:

Here's a basic overview of a typical CA hierarchy:

1. __Trusted Root CA(trust anchor)__: At the top of the hierarchy is the Root CA. This is the most trusted CA in the hierarchy, and it issues certificates to the Intermediate CAs. The Root CA's certificate is self-signed, meaning it's validated by the Root CA itself. The Root CA's certificate is typically installed in the trusted root store of servers, clients, and applications.

2. __Intermediate CA__: Below the Root CA are one or more Intermediate CAs. These CAs are issued certificates by the Root CA, and they issue certificates to either other Intermediate CAs or to End Entities. Using Intermediate CAs helps to protect the Root CA by allowing it to be kept offline and used only to sign Intermediate CA certificates.

3. __End Entities__: At the bottom of the hierarchy are the End Entities. These are the servers or clients that use the certificates to secure their communication. End Entities are issued certificates by the Intermediate CAs.

Certificate chains allow trust to be extended in a scalable, secure way. If a certificate is presented, it can be checked against the issuer's certificate, and this process can be followed up the chain until it reaches a trusted Root CA. If the chain of trust is valid, then the presented certificate is considered valid.

!!! danger
    The Root CA is a critical security component, and it should be protected accordingly. If the Root CA's private key is compromised, the entire trust model can be broken.


## Certificate Authority (CA) store / Trust store

A Certificate Authority (CA) store is a collection of trusted root and intermediate certificates that a system or application uses to validate or authenticate other certificates.

When a system or application needs to validate a certificate (for example, when establishing a secure HTTPS connection), it checks the certificate against the certificates in the CA store. If the certificate was issued by a CA that is included in the CA store, the certificate is considered valid.

The CA store is typically managed by the operating system, but some applications (like web browsers or Java) may use their own CA stores.

In the context of the script you provided, the CA store refers to the collection of trusted certificates that the system and potentially the Java runtime (if JAVA_HOME is set) use to validate certificates. The script installs specified certificates into this store.


### Trusting a Certificate

This involves explicitly marking a specific certificate as trusted in your system or application. This is typically done for self-signed certificates or certificates not issued by a trusted CA. When you trust a certificate, you're saying that you trust the identity that the certificate represents, and you trust the entity that issued the certificate. However, this trust does not extend to any other certificates that might be issued by the same entity.


### Adding a CA to the Trust Store

This involves adding the root certificate of a CA to your system's or application's trust store. The trust store is a database of trusted CAs. When a CA is in the trust store, it means you trust that CA to issue certificates. Therefore, any certificate signed by that CA will be automatically trusted. This is a broader level of trust because it applies to any certificates issued by that CA, not just a single certificate.

=== "Linux (Fedora / CentOS / RHEL)"

    1. Extract the CA certificate of a website into `.crt` file
    ```{ .bash .copy }
    echo | openssl s_client -servername hostname -connect hostname:port 2>/dev/null | openssl x509 -outform PEM -out path/to/certificate.crt
    ```

    2. Add the above certificate to trusted CA directory
    ``` { .bash .copy }
    cp /path/to/certificate.crt /etc/pki/ca-trust/source/anchors # (1)!
    ```

        1. This directory is used for trusted CA certificates that you add manually. Alternatively, copy the certificate to `/etc/pki/ca-trust/source/blacklist` to explicitly distrust the CA

    3. Update the CA Store. This command ensures that the new certificate is included in the bundle that applications use to validate certificates.
    ```{ .bash .copy }
    sudo update-ca-trust extract # (1)!
    ```

        1.  `update-ca-trust`: command ensures that the new certificate is included in the bundle that applications use to validate certificates.

            `extract`: This is a command that tells `update-ca-trust` to extract and regenerate the output files (the consolidated bundle of CA certificates).

    4. Validate the new CA has been added to the bundled.
        ```{ .bash .copy }
        openssl x509 -in certificate.crt -issuer -noout -subject # (1)!

        # output
        issuer= /C=US/ST=California/L=Sunnyvale/O=Fortinet/OU=Certificate Authority/CN=FG1K5D3I15802189/emailAddress=support@fortinet.com
        subject= /CN=*.docker.com
        ```

        1.  Use the output of this command as query parameters for the next command

        ```{ .bash .copy }
        trust list | grep *.docker.com # (1)!
        ```

        1. Here are some filters you can use with `trust list --filter`:

            - `ca-anchors`: This filter limits the output to CA certificates that are anchors. These are the root certificates that are used to validate other certificates.

            - `blacklist`: This filter limits the output to certificates that are explicitly distrusted.

            - `certificate <filename>`: This filter limits the output to the certificate specified by <filename>.

            - `pkcs11-id <id>`: This filter limits the output to the certificate with the specified PKCS#11 ID.

            - `pkcs11-label <label>`: This filter limits the output to the certificate with the specified PKCS#11 label.

            - `pkcs11-model <model>`: This filter limits the output to certificates from a PKCS#11 device with the specified model.

            - `pkcs11-manufacturer <manufacturer>`: This filter limits the output to certificates from a PKCS#11 device with the specified manufacturer.

            - `pkcs11-serial <serial>`: This filter limits the output to certificates from a PKCS#11 device with the specified serial number.

            - `pkcs11-token <token>`: This filter limits the output to certificates from a PKCS#11 device with the specified token.


        !!! info "Deleting a CA from trust store"

            1. Delete the certificate
            ```{.bash .copy}
            sudo rm /etc/pki/ca-trust/source/anchors/your_certificate.pem
            ```

            2. Update the CA trust database
            ```{.bash .copy}
            sudo update-ca-trust extract
            ```

=== "Mac OS"

    1. Extract the CA certificate of a website into `.pem` file
    ```{ .bash .copy }
    echo -n | openssl s_client -connect your.domain.com:443 | sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /path/to/certificate.pem

    echo | openssl s_client -servername your.domain.com -connect your.domain.com:port 2>/dev/null | openssl x509 -outform PEM -out path/to/certificate.pem
    ```

    3. Import and Trust the above certificate to the System keychain
    ``` { .bash .copy }
    sudo security add-trusted-cert -d -r trustRoot -k /Library/Keychains/System.keychain /path/to/certificate.pem    
    ```

    3. Validate the new CA has been added to trust store
    ```{.bash .copy}
    security find-certificate -c your.domain.com /Library/Keychains/System.keychain
    ```

        !!! info "Deleting a CA from trust store"

            1. Find the certificate's SHA-1 hash
            ```{.bash .copy}
            security find-certificate -c "your.domain.com" -a -Z /Library/Keychains/System.keychain | grep SHA-1
            ```

            2. Delete the certificat
            ```{.bash .copy}
            sudo security delete-certificate -Z SHA-1_HASH /Library/Keychains/System.keychain
            ```

!!! warning

    Please note that these methods only checks if the certificate is in the trust store. They do not validate the certificate itself. If you want to validate the certificate, you can use the openssl verify command.
    
    Verify the Certificate: Before adding the new CA certificate to the CA store, you should verify that it has been signed by a trusted Root CA. You can do this using the openssl command:
    ```{ .bash .copy }
    openssl verify -CAfile root_ca.crt /path/to/certificate.crt
    ```
