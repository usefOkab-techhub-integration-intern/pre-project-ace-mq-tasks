# How to Expose HTTPS APIs in ACE (Redhat ACE Server is Assumed)

## Creating APIs with ACE

- As an example, we made a getCustomerData api which takes the id and gets the whole row of information from the database in a single message flow.
- Just a simple HTTP Input Node and an HTTP Output Node with a Compute Node that is configured to contact the database running server.
- Setting the path as /
- Final touch is we check the HTTPS checkbox in the HTTP Input Node properties to state that it exposes HTTPS API.

## Need for Key Store and Trustk Store 

- First, let us talk about the need for both the key store and the trust store.
- In TLS protocol, there is a party called the Certificate Authority (CA). In fact, they are a number of parties who have assymetric-cryptography-based certificates, which contain their public keys and is self-signed by their private keys.
- The CA certificates are called root certificates, and they are agreed to be trusted by your browser, and your browser keeps a list of root certificates that should remain updated in case one was revoked, expired or new CA is added.
- That sounds good but how does a website use HTTPS? There are intermediate CAs whose certificates are signed by other intermediate CAs and so on until we reach a certificate signed by one of hte main CAs.
- Therefore, if a website's certificate is signed by one of the valid intermediate CA, your browser does a certificate validity check up to the root certificate to indicate that it is a secure and authenticated website.
- This is all to verify the authenticity of the server who exposes some API, which is the task at hand. 
- The key store is the list of certficates that the server owns and the trust store is the list of root certificates that the server trusts
- Suppose a server wants to prove his authenticity and verify that of incoming requests. This is why we should have key store and trust store.

## Create Key store, Trust store and Server Certificate

- For simplicity, we will assume our server is a CA and is going to have a self signed certificate in his key store and in his trust store wince assuming he trusts himself
- Run the following command to open the IBM Key Manager
```bash
ikeyman
```
### Create Key Store
- Click on the "create a new key database file" icon on the top left
- Enter the keystore path and file name {ksPath}
- Add a password for keystore
- Click Ok
### Create Certificate
- Click on the New Self-Signed blue button on the right
- Enter your certificate label {brkcert}, your machine IP and its common name (CN).
- (Optional) Change the key size, validity period and algorithm, and add additional information 
- Click Ok
### Create Trust Store and add certificate to it
- Create trust store same as [key store creation](#create-key-store) nad iwth identical password
- Click on import blue button on the right, then place the ksPath and choose brkCert, and click Ok
- Now server certificate is imported in both: key store and trust store
### Extract Certificate
- Click on the certificate, then the Extract Certificate blue button no the bottom right
- Choose a path {brkCertPath}
- Choose base64 encoded format
- Change file extension to .pem
- Click Ok

## Configuring ACE Server with key store and trust store

We have made a bash script that performs this mission and it goes as follows
```bash
#!/bin/bash

usage() {
  echo "Usage: $0 -i inodeName -s isvrName -k keyStorePath -t trustStorePath -p password"
  exit 1
}

while getopts "i:s:k:p:t:" opt; do
  case "${opt}" in
    i) inodeName=${OPTARG} ;;
    s) isvrName=${OPTARG} ;;
    k) keyStorePath=${OPTARG} ;;
    t) trustStorePath=${OPTARG} ;;
    p) password=${OPTARG} ;;
    *) usage ;;
  esac
done

if [[ -z "$inodeName" || -z "$isvrName" || -z "$keyStorePath" || -z "$trustStorePath"  || -z "$password" ]]; then
  usage
fi

echo "Associating Keystore for $inodeName and $isvrName..."
mqsichangeproperties "$inodeName" -e "$isvrName" -o ComIbmJVMManager -n keystoreFile -v "$keyStorePath"
mqsichangeproperties "$inodeName" -e "$isvrName" -o ComIbmJVMManager -n keystoreType -v JKS
mqsichangeproperties "$inodeName" -e "$isvrName" -o ComIbmJVMManager -n keystorePass -v "$password"

echo "Associating Truststore for $inodeName and $isvrName..."
mqsichangeproperties "$inodeName" -e "$isvrName" -o ComIbmJVMManager -n truststoreFile -v "$trustStorePath"
mqsichangeproperties "$inodeName" -e "$isvrName" -o ComIbmJVMManager -n truststoreType -v JKS
mqsichangeproperties "$inodeName" -e "$isvrName" -o ComIbmJVMManager -n truststorePass -v "$password"


echo "JVM Configuration complete."
```

What is happening here is we are setting some variable for the JVM Manager attached to the IBM server and responsible for handling the server connections

## Open HTTPS Port

Using the following command, we open the HTTPS post where default is 7083 for HTTP Nodes (Our case) and 7843 for SOAP Nodes
```bash
mqsichangeproperties "$inodeName" -e "$isvrName" -b HTTPSConnector -n explicitlySetPortNumber -v 7083
```
For sure now our server needs a refresh which is done using the following command
```bash
mqsireload "$inodeName" -e "$isvrName"
```

## Get Postman Ready (Host)

### Endpoint Preparation
- In youe ACE collection, add a new POST HTTP request
- Place https://hostname:7083/ as an endpoint
- Place the json id attribute as the raw body

### Adding brkcert as root CA

- Now, the interesting part ... once our application is deployed, we will have an https api exposed in an authenticated server
- We need to verify that server ceritifcate that will be sent to us in the TLS handshake before our request reaches the server
- But, we assumed that server is CA in [here](#create-key-store-trust-store-and-server-certificate). Therefore, the only way to verify the server self-signed certificate is by adding it to the root CAs in Postman's settings
- Click on the setting icon in the header then choose settings
- Navigate to certificates and enable the toggle button beside CA
- Now we need to get the brkcert.pem from the redhat VM to import it in the CA Certificates
- Move the certificate file to any path on your host and import it

## Testing Endpoint

- Deploy your application from the ACE toolkit on the server we configured
- Go to our endpoint's settings and enable SSL Verification
- Click send to test API, and you should get a safe and sound response, but should take a little while more than HTTP requests sunce we have a handshake now and a certificate verification

Happy HTTPSing :)