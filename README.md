# Fidelius
What does it mean?

Derived form [Fidelius Charm](https://harrypotter.fandom.com/wiki/Fidelius_Charm), a magic spell that can conceal secrets.

In the context of projectEka, this is a reference service that does the following 
- Generates key Parameters (Private key, Public key and nonce).
- For HIP encrypt given data based on the key parameters passed  and give public key in a format that can be shared with the HIU.
- For HIU decrypt the data based on the key parameters passed 


### How to run

#### Using Docker
- Build docker container `docker build -t fidelius .`
- Run docker container `docker container run -p 8090:8090 fidelius` _(optionally use `-d` flag to run the container in the background)_

#### Locally
- Clone repo `git clone git@github.com:sukreet/fidelius.git`
- build jar `./gradlew clean build `
- Run application on port `java -Dserver.port=8090 -jar build/libs/fidelius-0.0.1-SNAPSHOT.jar 
`


## Available APIs
### API to generate key material
Request :  
 
 `curl --location --request GET 'http://localhost:8090/keys/generate' \
                                --header 'Accept: application/json'`

Response : 

`{
     "privateKey": "CIcMqJzOE+CUpDbkp2WAwfatqppQqiP4q6yOYPYCeBo=",
     "publicKey": "BECzeFbSDRJTeedGinISJzPdSuC5ZTtniV+XNXAFIwtnNukNnQWcKVOKhJiiGtJc3bfKGXhWOqQZVyQlsSUv8WA=",
     "nonce": "tBZ0wkJEQJ0rRmkCGF3Utr/ASwydkoGMfUhhl8Nj9r4="
 }`
### API to Encrypt data (for HIP) 
Request : 

`curl --location --request POST 'http://localhost:8090/encrypt' \
--header 'Content-Type: application/json' \
--data-raw '{
    "receiverPublicKey": "Public key received as a part of data transfer request",
    "receiverNonce": "Nonce received as a part of data transfer request",
    "senderPrivateKey": "Private key generated by sender (HIP) using /keys/generate",
    "senderPublicKey": "Public key generated by sender (HIP) using /keys/generate",
    "senderNonce": "Nonce key generated by sender (HIP) using /keys/generate",
    "plainTextData": "valid fhir document"
}'`

Response : 

`{
     "encryptedData": "encrypted FHIR document",
     "keyToShare": "Public key that needs to be shared with HIU along with encrypeted data"
 }`

### API to decrypt data (for HIU)

Request :

`curl --location --request POST 'http://localhost:8090/decrypt' \
 --header 'Content-Type: application/json' \
 --data-raw '{
     "receiverPrivateKey": "Private key corresponding to the public that was shared whith HIP as part of data request",
     "receiverNonce": "Nonce that was shared with HIP as part of data request",
     "senderPublicKey": "public key received from HIP along encrypeted data",
     "senderNonce": "nonce received from HIP along encrypeted data",
     "encryptedData": "encrypted FHIR document"
 }'`
 
Response : 

`{
     "decryptedData": "FHIR document"
 }`
 
