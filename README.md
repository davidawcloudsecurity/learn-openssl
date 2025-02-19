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

When you have both root and intermediate certificates for your Tableau Server setup, here's how you can use OpenSSL to verify them, and how you should handle them in your server configuration:

### Verifying Certificates with OpenSSL

1. **Verify the Root Certificate:**

   ```bash
   openssl x509 -in root_certificate.pem -text -noout
   ```

   This command will display the details of the root certificate, including the issuer, subject, validity period, and public key details. 

2. **Verify the Intermediate Certificate:**

   ```bash
   openssl x509 -in intermediate_certificate.pem -text -noout
   ```

   Similarly, this will show you the details of the intermediate certificate. 

3. **Check the Certificate Chain:**

   You can verify if the intermediate certificate is indeed signed by the root certificate:

   ```bash
   openssl verify -CAfile root_certificate.pem intermediate_certificate.pem
   ```

   If the output is `intermediate_certificate.pem: OK`, it means the intermediate certificate is correctly signed by the root certificate.

### Using Root and Intermediate Certificates with Tableau Server

To use these certificates with Tableau Server:

- **Certificate Bundle:**
  - You'll typically need to create a bundle of certificates that includes your server's certificate, the intermediate certificate(s), and sometimes the root certificate (though the root isn't always included in the bundle for browsers to validate the chain). The order is important:

    - Your server's certificate (e.g., `server.crt`)
    - Intermediate certificate(s) (e.g., `intermediate.crt`)
    - Optionally, Root certificate (e.g., `root.crt`)

    Concatenate these into one file:

    ```bash
    cat server.crt intermediate.crt > fullchain.pem
    ```

    If you have multiple intermediates or if you're including the root:

    ```bash
    cat server.crt intermediate1.crt intermediate2.crt root.crt > fullchain.pem
    ```

- **Configure Tableau Server:**
  - Tableau Server needs to know where to find your SSL certificate and key. If you're using a load balancer for SSL termination, you might not need to install these on Tableau itself, but rather in the load balancer or reverse proxy configuration. However, if you're setting SSL directly on Tableau Server:

    - Place your private key (`server.key`) and certificate bundle (`fullchain.pem`) in a secure location.
    - Use `tabadmin` or `tsm` (depending on your version of Tableau Server) to update the SSL configuration:
      
      For older versions with `tabadmin`:
      
      ```bash
      tabadmin set ssl.cert.file "/path/to/fullchain.pem"
      tabadmin set ssl.key.file "/path/to/server.key"
      tabadmin configure
      tabadmin restart
      ```

      For newer versions with TSM:

      ```bash
      tsm security external-ssl enable --cert-file "/path/to/fullchain.pem" --key-file "/path/to/server.key"
      tsm pending-changes apply
      ```

- **Ensure Correct Permissions:**
  - The certificate and key files should have restricted permissions to enhance security.

- **Testing:**
  - After applying changes, test your SSL setup by accessing your Tableau site via HTTPS from a browser. Check for any SSL/TLS warnings. Tools like SSL Labs' SSL Test can provide a comprehensive analysis of your SSL configuration.

Remember, security practices might dictate that you don't include the root certificate in the chain you provide to clients, as most modern browsers already trust well-known root CAs. However, you can use it for verification with OpenSSL as shown.
