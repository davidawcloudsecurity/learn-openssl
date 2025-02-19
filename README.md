# learn-openssl
What is and how to use openssl
To verify a PEM file, specifically to check if it's a valid certificate, key, or CSR, you can use OpenSSL in various ways depending on what the PEM file contains. Here are some common scenarios:

### Verify a Certificate (.crt or .pem)

If your PEM file contains a certificate:

```bash
openssl x509 -in your_certificate.pem -text -noout
```

- **-in your_certificate.pem**: Specifies the input file.
- **-text**: Outputs the certificate in human-readable format.
- **-noout**: Prevents the output of the encoded certificate.

If you want to check the certificate's validity period:

```bash
openssl x509 -in your_certificate.pem -noout -dates
```

### Verify a Private Key

If your PEM file contains a private key:

```bash
openssl rsa -in your_private_key.pem -check
```

- **-in your_private_key.pem**: Specifies the private key file.
- **-check**: Performs a check on the key to verify its integrity.

### Verify a CSR

If your PEM file is a Certificate Signing Request:

```bash
openssl req -in your_csr.pem -noout -text
```

- **-in your_csr.pem**: Specifies the CSR file.
- **-noout**: Suppresses the output of the encoded request.
- **-text**: Displays the text information of the CSR.

To verify the signature of the CSR:

```bash
openssl req -in your_csr.pem -verify -noout
```

### Matching Key with Certificate

To ensure that a private key matches a certificate (to confirm they are a pair):

1. **Get the modulus of the certificate:**
   ```bash
   openssl x509 -in your_certificate.pem -noout -modulus | openssl md5
   ```

2. **Get the modulus of the private key:**
   ```bash
   openssl rsa -in your_private_key.pem -noout -modulus | openssl md5
   ```

   If the outputs from both commands are the same, the key and certificate match.

### General PEM Verification

If you're unsure what type of PEM file you have, you can try:

- **Check if it's a certificate:**
  ```bash
  openssl x509 -in your_file.pem -noout -text || echo "Not a valid certificate"
  ```

- **Check if it's a private key:**
  ```bash
  openssl rsa -in your_file.pem -check || echo "Not a valid RSA private key"
  ```

- **Check if it's a CSR:**
  ```bash
  openssl req -in your_file.pem -noout -text || echo "Not a valid CSR"
  ```

These commands will either display information about the file or indicate if the file is not of the expected type. Remember, OpenSSL's error messages can sometimes be cryptic, so if you get an error, it doesn't necessarily mean the file is invalid, just that it might not be of the type you're checking for.
