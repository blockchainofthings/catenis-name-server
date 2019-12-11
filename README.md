# Catenis Name Server

This is an implementation of the Catenis Name Service, which is used by Catenis nodes to map their IPFS repository root
 CID to a hostname so other Catenis nodes can reference it.
 
## The Catenis Name Service 

Catenis Name Service serves as a workaround for IPFS' IPNS, which currently does not meet Catenis' performance needs;
 it takes too long to publish a new name.
 
## Building the application

To build the application, issue the following commands:

```shell
nvm use
npm i
npm run build
```

## Starting the application

To start the application, issue the following command:

```shell
env NODE_CONFIG_ENV='<environment>' CNS_INSTANCE_IDX=<cns_index> node build/main.js
```

> **Note 1**: the term `<environment>` should be replaced with the appropriate deployment environment; either
 'development' (the default, if not set), 'sandbox' or 'production'.

> **Note 2**: the term `<cns_index>` above should be replaced with the index (starting from 1) of this Catenis Name Server
 application instance. 

## How it works

Catenis Name Server provides a REST API though which Catenis nodes can reset their own IPFS repository root CID
 and query for the IPFS repository root CID of other Catenis nodes.

## REST API

### Authentication

The requests to the Catenis Name Server REST API should be authenticated using the [HTTP Signature](https://github.com/joyent/node-http-signature)
 scheme.
 
Each Catenis node should be assigned an SSH RSA key pair for signing the HTTP requests. Use the following command to
 generate such key pair.

```shell
ssh-keygen -b 2048 -t rsa -m PEM -f ctn-node0_key
```

Catenis Name Server expects to retrieve Catenis nodes' public keys from *catenis.io*'s DNS by looking up TXT records
 associated with the ***ctn-node*** subdomain. Such TXT records should contain a JSON with the following keys:
- idx: \[Integer\] The Catenis node index.
- pubKey: \[String\] The SSH RSA public key.

The following is an example of such TXT record:

```text
ctn-node TXT {"idx":0,"pubKey":"ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQC2q/4V67+BMRrVIZGWLWc27B80kMc/NlJGSXbs2KuN4bmK4iyPA0ycTxavrb1y9eFIXg/ZM613DpeEsK7uaU9wtRZxaI1hFY3oWiEx28gU8Jh4XnBHgSNl4LibAKZIiOtiHOAqCAYUpQuU+j8Kr4GNKzP+QuuwQbOkEpImwv01Mpq+lUpqyvhL0GqMfQNaYvSjV02OZtRcHqQ1CfIuWxfs+iAht1WrW6C4+KZsaRbmZG63ujmvLIwG7zdV3y5NW9NUdoJTM59EHHOQ5yFn985C8Q5QUwnk544TZsMJJ4szVELWSzlrzGFpZfCygkEJmAJVAq+KRBQe/4z6PDfX1qIR"}
```

> **Note**: the client ID —in the format *&lt;username>@&lt;host-name>*— at the end of the public key file generated by
 the `ssh-keygen` command should not be included when setting the value of the `pubKey` key.  

### Set IPFS repository root CID method

Request: `POST /ctn-node/:nodeIdx/ipfs-root`

URL parameters:
- nodeIdx: \[Number\] The index of the Catenis node for which the IPFS repository root CID should be set

Request body: a JSON object containing the following keys:
- `cid`: \[String\] New IPFS CID of Catenis node's IPFS repository root.
- `lastUpdatedDate`: \[String\] ISO-8601 formatted date and time when CID for this Catenis node's IPFS repository root has last been recorded.

> **Note**: the `lastUpdatedDate` key is only taken into account (and thus should only be sent) when another Catenis
 Name Server instance is calling this method.

Success response body: a JSON containing the following keys:
- `status`: \[String\] The value **'success'**.

### Set Multiple IPFS repository root CIDs method

Request: `POST /ctn-node/ipfs-root`

Request body: a JSON object containing one or more of the following keys:
- `<Catenis_node_index>.cid`: \[String\] New IPFS CID of Catenis node's IPFS repository root.
- `<Catenis_node_index>.lastUpdatedDate`: \[String\] ISO-8601 formatted date and time when CID for this Catenis node's IPFS repository root has last been recorded.

> **Note**: the term `<Catenis_node_index>` should be replaced with the index of the Catenis node the IPFS repository
 root CID of which should be set.
 
Success response body: a JSON containing the following keys:
- `status`: \[String\] The value **'success'**.

> **Note**: this method should only be called by another Catenis Name Server instance.

### Retrieve IPFS repository root CID method

Request: `GET /ctn-node/:nodeIdx/ipfs-root`

URL parameters:
- `nodeIdx`: \[Number\] The index of the Catenis node for which the IPFS repository root CID should be retrieved.

Success response body: a JSON containing the following keys:
- `status`: \[String\] The value **'success'**.
- `data.cid`: \[String\] The current IPFS CID of Catenis node's IPFS repository root.

### Retrieve All IPFS repository root CIDs method

Request: `GET /ctn-node/ipfs-root`

Query string (optional) parameters:
- `updatedSince`: \[String\] ISO-8601 formatted date and time used to filter Catenis IPFS repository root CIDs to be returned.

Success response body: a JSON containing the following keys:
- `status`: \[String\] The value **'success'**.
- `data.<Catenis_node_index>.cid`: \[String\] CID of IPFS repository root for that Catenis node.
- `data.<Catenis_node_index>.lastUpdatedDate`: \[String\] ISO-8601 formatted date and time when CID has been last updated.

> **Note**: the term `<Catenis_node_index>` is replaced with the index of the specific Catenis node.

## License

This project is for Blockchain of Things' internal use only.

Copyright © 2019, Blockchain of Things Inc.