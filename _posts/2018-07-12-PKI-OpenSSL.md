---
layout:     post
title:      PKI OpenSSL
subtitle:   three level private key
date:       2018-07-12
author:     Richie
header-img: img/post-PKI-linux-app.jpg
catalog: true
tags:
    - Linux
    - PKI
    - open source
---
1.	Set up the root CA：
1.1 Create the rootCA directories and files：
richie@richie-VirtualBox:~/X582/rootca$ mkdir newcerts private conf
richie@richie-VirtualBox:~/X582/rootca$ chmod g-rwx,o-rwx private
richie@richie-VirtualBox:~/X582/rootca$ echo "01" > serial
richie@richie-VirtualBox:~/X582/rootca$ touch index.txt
1.2 Create the issuer's general information profile:
richie@richie-VirtualBox:~/X582/rootca$ vi "$HOME/X582/rootca/conf/gentestca.conf"
gentestca.conf:
####################################
[ req ]
default_keyfile = $ENV::HOME/rootca/private/rootcakey.pem
default_md = sha1
prompt = no
distinguished_name = ca_distinguished_name
x509_extensions = ca_extensions
 
[ ca_distinguished_name ]
organizationName = TestOrg
organizationalUnitName  = TestDepartment
commonName = TestCA
emailAddress = yuancanqin@hotmail.com
 
[ ca_extensions ]
basicConstraints = CA:true
########################################
1.3 Create a profile：
richie@richie-VirtualBox:~/X582/rootca$ vi "$HOME/X582/rootca/conf/rootca.conf"
rootca.conf:
####################################
[ ca ]
default_ca      = rootca                 # The default ca section
 
[ rootca ]
dir            = $ENV::HOME/rootca       # top dir
database       = $dir/index.txt          # index file.
new_certs_dir  = $dir/newcerts           # new certs dir
 
certificate    = $dir/rootcacert.pem         # The CA cert
serial         = $dir/serial             # serial no file
private_key    = $dir/private/rootcakey.pem  # CA private key
RANDFILE       = $dir/private/.rand      # random number file
 
default_days   = 365                     # how long to certify for
default_crl_days= 30                     # how long before next CRL
default_md     = sha1                    # message digest method to use
unique_subject = no                      # Set to 'no' to allow creation of
                                         # several ctificates with same subject.
policy         = policy_any              # default policy
 
[ policy_any ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
 
########################################
1.4 Generating CA private key and self-signed certificate (root certificate):
During the execution process, we need to enter the pass phrase to encrypt requester's key. Suggesting enter the password: 888888
richie@richie-VirtualBox:~/X582/rootca$ openssl req -x509 -newkey rsa:2048 -out rootcacert.pem -outform PEM -days 2190 -config "$HOME/X582/rootca/conf/gentestca.conf"
Generating a 2048 bit RSA private key
...........................................................+++
.....................................................................................................................+++
writing new private key to '/home/richie/X582/rootca/private/rootcakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
richie@richie-VirtualBox:~/X582/rootca$ ls
conf  index.txt  newcerts  private  rootcacert.pem  serial
req    # Generate a certificate request file
-x509 # Dedicated to CA to generate self-signed certificate
rsa:2048  # The encryption length is 2048
-new  # Generating new certificate signing request
-key  # Generate the private key file used for the request
-days # Validity period of the certificate
2.	Set up the subCA：
2.1 Create the subCA directories and files：
richie@richie-VirtualBox:~/X582/subca$ mkdir newcerts private conf
richie@richie-VirtualBox:~/X582/subca$ chmod g-rwx,o-rwx private
richie@richie-VirtualBox:~/X582/subca$ echo "01" > serial
richie@richie-VirtualBox:~/X582/subca$ touch index.txt
2.2 Create the file of sub issuer's information:
richie@richie-VirtualBox:~/X582/subca$ vi "$HOME/X582/subca/conf/gentestca.conf" 
gentestca.conf:
####################################
[ req ]
default_keyfile = $ENV::HOME/subca/private/subcakey.pem
default_md = sha1
prompt = no
distinguished_name = ca_distinguished_name
x509_extensions = ca_extensions
 
[ ca_distinguished_name ]
organizationName = TestOrg
organizationalUnitName  = TestDepartment
commonName = TestCA
emailAddress = yuancanqin@hotmail.com
 
[ ca_extensions ]
basicConstraints = CA:true
########################################
2.3 Create a profile：
richie@richie-VirtualBox:~/X582/subca$ vi "$HOME/X582/subca/conf/subca.conf"
subca.conf:
####################################
[ ca ]
default_ca      = subca                  # The default ca section
 
[ subca ]
dir            = $ENV::HOME/subca        # top dir
database       = $dir/index.txt          # index file.
new_certs_dir  = $dir/newcerts           # new certs dir
 
certificate    = $dir/subcacert.pem      # The CA cert
serial         = $dir/serial             # serial no file
private_key    = $dir/private/subcakey.pem  # CA private key
RANDFILE       = $dir/private/.rand      # random number file
 
default_days   = 365                     # how long to certify for
default_crl_days= 30                     # how long before next CRL
default_md     = sha1                    # message digest method to use
unique_subject = no                      # Set to 'no' to allow creation of
                                         # several ctificates with same subject.
policy         = policy_any              # default policy
 
[ policy_any ]
countryName             = optional
stateOrProvinceName     = optional
localityName            = optional
organizationName        = optional
organizationalUnitName  = optional
commonName              = supplied
emailAddress            = optional
 
########################################
 2.4 Create a private key and CSR (a public key is included in the certificate request):
richie@richie-VirtualBox:~/X582/subca$ openssl req -newkey rsa:1024 -keyout subcakey.pem -keyform PEM -out subcareq.pem -outform PEM -subj "/O=SubcaCom/OU=SubcaOU/CN=subca" 
Generating a 1024 bit RSA private key
.............++++++
.................++++++
writing new private key to 'subcakey.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
richie@richie-VirtualBox:~/X582/subca$ ls
conf  index.txt  newcerts  private  serial  subcakey.pem  subcareq.pem
req    # Generate a certificate request file
rsa:2048  # The encryption length is 2048
-new  # Generating new certificate signing request
-key  # Generate the private key file used for the request
-days # Validity period of the certificate

2.5 Issue the certificate for subCA with rootCA:
richie@richie-VirtualBox:~/X582/subca$ openssl ca -in subcareq.pem -out subcacert.pem -config "$HOME/X582/rootca/conf/rootca.conf"
Using configuration from /home/richie/X582/rootca/conf/rootca.conf
Enter pass phrase for /home/richie/X582/rootca/private/rootcakey.pem:
Can't open /home/richie/X582/rootca/index.txt.attr for reading, No such file or directory
140525588971968:error:02001002:system library:fopen:No such file or directory:../crypto/bio/bss_file.c:74:fopen('/home/richie/X582/rootca/index.txt.attr','r')
140525588971968:error:2006D080:BIO routines:BIO_new_file:no such file:../crypto/bio/bss_file.c:81:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
organizationName      :ASN.1 12:'SubcaCom'
organizationalUnitName:ASN.1 12:'SubcaOU'
commonName            :ASN.1 12:'subca'
Certificate is to be certified until Jun 15 04:26:21 2019 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
richie@richie-VirtualBox:~/X582/subca$ ls
conf       newcerts  serial         subcakey.pem
index.txt  private   subcacert.pem  subcareq.pem
ca     # Sign the certificate
-in    # Enter the certificate file

3.	Issue Ecu1 CA
3.1 Create a private key and CSR (a public key is included in the certificate request):
richie@richie-VirtualBox:~/X582/ecu1$ openssl req -newkey rsa:1024 -keyout ecu1key.pem -keyform PEM -out ecu1req.pem -outform PEM -subj "/O=Ecu1Com/OU=Ecu1OU/CN=ecu1" 
Generating a 1024 bit RSA private key
................................................++++++
...............................++++++
writing new private key to 'ecu1key.pem'
Enter PEM pass phrase:
Verifying - Enter PEM pass phrase:
-----
richie@richie-VirtualBox:~/X582/ecu1$ ls
ecu1key.pem  ecu1req.pem
req    # Generate a certificate request file
rsa:1024  # The encryption length is 2048
-new  # Generating new certificate signing request
-key  # Generate the private key file used for the request

3.2 Issue the certificate for ecu1 with subCA:
richie@richie-VirtualBox:~/X582/ecu1$ openssl ca -in ecu1req.pem -out ecu1cert.pem -config "$HOME/X582/subca/conf/subca.conf"
Using configuration from /home/richie/X582/subca/conf/subca.conf
Enter pass phrase for /home/richie/X582/subca/private/subcakey.pem:
Can't open /home/richie/X582/subca/index.txt.attr for reading, No such file or directory
140523910693312:error:02001002:system library:fopen:No such file or directory:../crypto/bio/bss_file.c:74:fopen('/home/richie/X582/subca/index.txt.attr','r')
140523910693312:error:2006D080:BIO routines:BIO_new_file:no such file:../crypto/bio/bss_file.c:81:
Check that the request matches the signature
Signature ok
The Subject's Distinguished Name is as follows
organizationName      :ASN.1 12:'Ecu1Com'
organizationalUnitName:ASN.1 12:'Ecu1OU'
commonName            :ASN.1 12:'ecu1'
Certificate is to be certified until Jun 15 04:30:15 2019 GMT (365 days)
Sign the certificate? [y/n]:y


1 out of 1 certificate requests certified, commit? [y/n]y
Write out database with 1 new entries
Data Base Updated
richie@richie-VirtualBox:~/X582/ecu1$ ls
ecu1cert.pem  ecu1key.pem  ecu1req.pem
ca     # Sign the certificate
-in    # Enter the certificate file




@end

```
