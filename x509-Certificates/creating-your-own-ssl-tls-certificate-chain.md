# Creating Your Own SSL/TLS Certificate Chain

## Creating Root Certificate Authority (CA)
The root certificate authority is the certificate that issues, signs, and verifies other certificates for a given system. This CA root certificate is required to sign all new certificates, as well as to verify the authenticity of a provided certificate.

To create a root CA, there are three main steps:

1) Create a private key for the CA.
2) Create a self-signed certificate for the CA.
3) Install the root CA on various workstations.

Once this is done, every device that will utilize the certificate chain to establish a secure connection needs to have its own certificate, created by the following steps:

4) Create _certificate signing request_ (CSR) for the client.
5) Sign the newly created CSR with the root CA key, generating a new certificate in the process.
6) Install the newly created certificate on the original client machine.

### 1) Create a private key for the CA
Generating a new root CA key is simple and can be done with a simple command:

`openssl genrsa -des3 -out rootCA.key 2048`

- `openssl` generates the certificate using _openssl_, and industry standard open-source SSL/TLS platform.
- `genrsa` indicates to generate the key using the _RSA_ algorithm.
- `-des3` causes the generated key to be encrypted with a password. After running the command, you will be prompted to enter the password. Do not lose this password, as without it, your private key will be useless.
- `-out rootCA.key` defines the file that will be created that contains the private key.
- `2048` is the size of the private key that will be generated, in bits. This should always be _at least_ 2048. 
   
The resulting file will contain the root CA private key. This key must be kept <b><i>SECRET</b></i> under all circumstances, as anybody with this key could masquerade as you and sign any arbitrary CSR without your consent.

### 2) Create self-signed root CA certificate
Using the private key generated in (1), we can now generate a _self-signed certificate_ for our root CA:

`openssl req -x509 -new -nodes -key rootCA.key -sha256 -days 1024 -out rootCA.pem`
- `openssl req` generates and processes certificate requests in the *PKCS#10* format. 
- `-x509` causes the program to output a self signed certificate instead of a certificate request. Since we are generating a self-signed CA certificate, we need this option.
- `-new` generates a certificate signing request. This is required to create a self-signed certificate. Note that when `-x509` is specified, `openssl` usually won't create a CSR by default. By specifying `-new`, we tell it to override the default behavior and create one anyway. This CSR will not be output, and is only used while generating the certificate.
- `-nodes` tells `openssl` that if a new private key is generated during execution, the key _will not_ be encrypted. Since we have already created an encrypted private key, we don't need another one, so specify this so the command works correctly. 
- `-key rootCA.key` is the file containing the root CA private key we generated in (1)
- `-sha256` indicates that we want to use the _SHA-256_ algorithm to sign the temporary CSR that is created during this process. Note that the `man` pages indicate this option as `-[digest]`, if you want to do more research.
- `-days 1024` causes the generated certificate to expire in a set amount of time, in this case, 1024 _days_. After this expiry, the certificate will no longer be valid, and a new one must be issued. This should not affect any other certificates that have been signed with the rootCA.key, but will stop any client from using this certificate to validate a certificate until a new one is issued (i.e. after expiring, client certificates will not need to be regenerated, just the root CA certificate). Only valid when `-x509` is specified.
- `-out rootCA.pem` indicates the output file to store the public certificate, in _Privacy Enhanced Mail_ (.pem) format. `.pem` is the x509 format used by most software, and is simply a container format for other x509 certificates.

Running this command will first ask you to enter the password you used for encrypting the private key, then prompt you with the x509 fields such as CN, DN, etc... Fill these out as needed. The resulting certificate will then be output as `rootCA.pem`. 

This is the file you will need to install on any and all client machines utilizing a certificate generated from this root CA. You can move this file around in the public without much worry, although I recommend making sure you secure it's transmission using technologies such as `SFTP`, `HTTPS`, or something similar (`scp` program is the easiest implementation for SFTP and is available on almost all OS in production).

### 3) Install the root CA on client machines
Simply move the rootCA.pem file to a location on all client machines that will use this system. If the software that will use the Certificate Chain requires additional configuration to trust the root CA certificate, do this now.

Now, for each client machine, follow these steps:

### 4) Generate a Certificate Signing Request
Login to the machine that you want to create a certificate for. Navigate to the directory that will hold the certificates, and run the following command to generate a new private key:

``openssl genrsa -out client.key 2048``
- `openssl genrsa` creates a new RSA private key (No encrypted password. This is to facilitate easy usage by client software.)
- `-out client.key` will store the resulting key in the `client.key` file in the current directory.
- `2048` indicates the key will be created using 2048 bits.

Next, generate the CSR and sign it using the newly generated private key:

``openssl req -new -key client.key -out client.csr``
- `openssl req` creates a new CSR from the given parameters.
- `-new` indicates that a CSR should be generated. This is required by openssl.
- `-key client.key` is the key we generated in the previous step.
- `-out client.csr` is the file the CSR will be stored in.

Follow the prompts and enter the correct information. When you finish, you will now have the `.csr` file that the rootCA will sign and use to generate the final certificate.

### 5) Sign the CSR and Generate the Certificate
Securely move the .csr file generated in (4) to the signing machine. Once the .csr file is on the signing machine, follow these steps:

# TODO: FINISH THIS DOCUMENT