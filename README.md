# Estuary Alternatives
# State of Estuary
By now, you should be aware that Estuary is no longer given any new invite codes. We are continuing serving our current customers data but Estuary itself is being sunsetted.

With that being said, we do have alternatives platforms that we can leverage inside our ecosystem. Will go over the most popular ones that you might be interested in looking into.

This document will focus on some technical aspects of how to on board your data into these platforms.

# Web3.Storage
## Overview
At the core of web3.storage is a storage service to safely secure and make your data available - giving developers the power of decentralized storage and content addressing via simple client libraries or an HTTP API.

Content uploaded to web3.storage is stored on instances of Elastic IPFS, a cloud-native implementation of IPFS to stay reliable and performant with the scale of uploads we were seeing. Elastic IPFS ensures your data is continuously and quickly available over the IPFS network once your data is uploaded.

Generally, before data is uploaded, it is locally converted into a CAR file, which contains your data transformed into a format that IPFS can consume (a directed acyclic graph, or DAG). This is very powerful - in this process, a content address is locally generated for your data, giving you a guarantee of your data’s fingerprint. Further, you can construct the DAG in useful ways, like making individual files within your folders or key-value pairs in your JSON have their own content addresses.

Once the CAR file is uploaded, a queue of geographically distributed Filecoin storage providers - selected for performance and availability - bid for the right to store these deals.
The Filecoin storage network provides a global network of storage providers who bid against each other to store data. web3.storage makes a minimum of 5 deals with the various storage providers.

These storage providers generate cryptographic proofs that they are continuously storing your data. The additional redundancy provided by these storage providers is very powerful. You can use the Filecoin blockchain to directly validate that your data is actually physically being stored - without having to take web3.storage’s word for it!

## Pricing
web3.storage offers a free tier that will allow you to store 5GiB of data for free. If you need additional storage, take a look at their pricing model @ https://web3.storage/pricing/

## API Token
Before being able to upload any files you will need to log into web3.storage and generate an API key. This can be done through their UI at web3.storage. Simply log in using your github account or sign up with your email. Once logged in, go under Account -> Create an API Token. This page should be familiar if you are used to generate API Keys in Estuary. Once you create a new key, we should be able to interact with their API now.

## Uploading Files
Store files using web3.storage. You can upload either a single file or multiple files.

Send the POST request with one of the following:

- a single file, with a single blob/file object as the body
- multiple files, as FormData with Content-Disposition headers for each part to specify filenames and the request header Content-Type: multipart/form-data

> Requests to this endpoint have a maximum payload size of 100MB. To upload larger files, see the documentation for the /car endpoint.

```shell
$ curl -X POST --data-binary @file.txt -H 'Authorization: Bearer YOUR_API_KEY' https://api.web3.storage/upload  -s | jq
{
  "cid":"bafkreid65ervf7fmfnbhyr2uqiqipufowox4tgkrw4n5cxgeyls4mha3ma"
}
```

## Retrieving Files
Now that we have uploaded a file, we should be able to fetch its cid directly in our browser with their `w3s.link` gateway.

`https://bafkreid65ervf7fmfnbhyr2uqiqipufowox4tgkrw4n5cxgeyls4mha3ma.ipfs.w3s.link/`

## Status
If you want to look at your file status, you can do so by calling the status endpoint with your cid.

```shell
$ curl 'https://api.web3.storage/status/bafybeidwfngv7n5y7ydbzotrwl3gohgr2lv2g7vn6xggwcjzrf5emknrki' -s | jq
{
    "cid": "bafkreid65ervf7fmfnbhyr2uqiqipufowox4tgkrw4n5cxgeyls4mha3ma",
    "dagSize": 519,
    "created": "2021-08-11T08:25:21.905806+00:00",
    "pins": [
        {
            "status": "Pinned",
            "updated": "2021-08-11T08:35:42.726691+00:00",
            "peerId": "12D3KooWSnniGsyAF663gvHdqhyfJMCjWJv54cGSzcPiEMAfanvU",
            "peerName": "web3-storage-dc13",
            "region": null
        },
        ...
    ],
    "deals": [
        {
            "dealId": 38314728,
            "storageProvider": "f02095132",
            "status": "Active",
            "pieceCid": "baga6ea4seaqesuag2dgpsenmmtyeitgi73b2eha7dj6ssmjqeihbrsxmuoumsba",
            "dataCid": "bafybeidtjfi5dqvdyadbadjukrzm2fuljw7wid35rn3bacdoopcdnug4ae",
            "dataModelSelector": "Links/146/Hash/Links/35/Hash/Links/0/Hash",
            "activation": "2023-05-29T02:15:00+00:00",
            "expiration": "2024-11-11T02:15:00+00:00",
            "created": "2023-08-31T13:25:03.463083+00:00",
            "updated": "2023-08-31T13:25:03.463083+00:00"
        },
        ...
    ]
}
```

Here is an overview of the fields returned by the status field and their description:

- `cid` contains the same CID that was passed into the status method, so you don't have to keep track of which response goes with which request.
- `created` contains an ISO-8601 datetime string indicating when the content was first uploaded to web3.storage.
- `dagSize` contains the size in bytes of the Directed Acyclic Graph (DAG) that contains all the uploaded content. This is the size of the data that is transferred over the network to web3.storage during upload, and is slightly larger than the total size of the files on disk.
- `pins` contains an array of objects describing the IPFS nodes that have pinned the data, making it available for fast retrieval using the IPFS network.
- `deals` contains an array of objects describing the Filecoin storage providers that have made storage deals. These storage providers have committed to storing the data for an agreed period of time. Note that it may take up to 48 hours after uploading data to web3.storage before Filecoin deals are activated.

## Pinning
web3.storage provides a pinning service that complies with the IPFS Pinning Service API specification. **web3.storage's Pinning Service API is not to be used for ongoing production traffic, but rather for one-time migrations.**

**You do not need to request access if you are storing data with web3.storage directly. Data stored with web3.storage is persisted indefinitely by default. This API is only useful if you are looking to store data with web3.storage that is already available on the IPFS network. Even in these situations, if you are able to, we recommend you generate a CAR file from a IPFS node hosting the content and directly upload that to web3.storage (e.g., run ipfs dag export from your local node) rather than use the Pinning API.**

For a full list and documentation of all the available pinning service endpoints, visit the IPFS Pinning Service API endpoint documentation.

### Requesting access
To request access to the pinning service for your web3.storage account, you will need to navigate to your token management page and click the button labeled "Request API Pinning Access". Fill out the form and then, once approved, you will be able to access the pinning service API endpoints using your API token.
### Using the HTTP API
The web3.storage pinning service endpoint for all requests is https://api.web3.storage/pins.

> Web3.storage Pinning APIs only support raw, dag-pb, dag-cbor and dag-json IPLD codecs. The API doesn't support pinning content by providing IPNS records pointing to it.

### Add a pin
```shell
curl -X POST 'https://api.web3.storage/pins' \
  --header 'Accept: */*' \
  --header 'Authorization: Bearer <YOUR_AUTH_KEY_JWT>' \
  --header 'Content-Type: application/json' \
  -d '{
  "cid": "<CID_TO_BE_PINNED>",
  "name": "PreciousData.pdf"
}'
```

### List pins
```shell
curl -X GET 'https://api.web3.storage/pins' \
  --header 'Accept: */*' \
  --header 'Authorization: Bearer <YOUR_AUTH_KEY_JWT>'
```

## Additional Resources
You can find additional resources @ https://web3.storage/docs

# NFT.storage
## Overview
NFT.Storage is a long-term storage service designed for off-chain NFT data (like metadata, images, and other assets) for up to 31GiB in size per individual upload. Data is content addressed using IPFS, meaning the URI pointing to a piece of data (“ipfs://…”) is completely unique to that data (using a content identifier, or CID). IPFS URLs and CIDs can be used in NFTs and metadata to ensure the NFT forever actually refers to the intended data (eliminating things like rug pulls, and making it trustlessly verifiable what content an NFT is associated with).

NFT.Storage stores many copies of uploaded data on the public IPFS network in two primary ways: in dedicated IPFS servers managed by NFT.Storage, and decentralized on Filecoin. Since IPFS is a standard used by many different storage services, it's easy to redundantly store data uploaded to NFT.Storage on any other IPFS-compatible storage solution from pinning services, to your local IPFS node, to other storage networks like Arweave or Storj. And as time goes on, NFT.Storage will increasingly decentralize itself as a public good!

## Pricing
Because NFT.Storage participates in the Filecoin Plus program, all data uploaded through the service is eligible for this 10x reward multiplier. This is such a powerful incentive for Filecoin storage providers to store user data that they tend to be willing to offer free storage and retrieval services in order to get this block reward multiple. As a result, most storage providers offer free storage and retrieval on Filecoin today and will continue to do so as long as block rewards continue to be a powerful incentive. This should be true for a very long time. While there is some additional infrastructure cost associated with running the NFT.Storage service, Protocol Labs is committed to maintaining this infrastructure indefinitely as part of our mission to grow the decentralized storage ecosystem and preserve humanity's information for future generations.

## API Token
It only takes a few moments to get a free API token from NFT.Storage. This token enables you to interact with the NFT.Storage service without using the main website, enabling you to incorporate files stored using NFT.Storage directly into your applications and services.

- Click API Keys to go to your NFT.Storage account page.
- Click Create an API token.
- Enter a descriptive name for your API token and click Create.
- Make a note of the Token field somewhere secure where you know you won't lose it. You can click Copy to copy your new API token to your clipboard.

> Do not share your API token with anyone else. This key is specific to your account.

## Uploading Files
To upload a single file, send a POST request to /upload with the binary file data as the request body. The Content-Type header should be set to a type that's appropriate for the content, for example, image/jpeg.

```shell
$ curl --location 'https://api.nft.storage/upload' \
--header 'Authorization: Bearer YOUR_API_KEY' \
--header 'Content-Type: image/jpeg' \
--data '@/path/to/file.jpg' | jq
{
    "ok": true,
    "value": {
        "cid": "bafkreifqusopuy5k56dnhsknofeo2fd36frzddjbpi3fkn3vh2djdvtebm",
        "created": "2023-08-31T14:27:04.437+00:00",
        "type": "image/jpeg",
        "scope": "test",
        "files": [],
        "size": 104271,
        "name": "Upload at 2023-08-31T14:27:04.437Z",
        "pin": {
            "cid": "bafkreifqusopuy5k56dnhsknofeo2fd36frzddjbpi3fkn3vh2djdvtebm",
            "created": "2023-08-31T14:27:04.437+00:00",
            "size": 104271,
            "status": "pinned"
        },
        "deals": []
    }
}
```

> The /upload endpoint can accept up to 100 MiB in each HTTP request. If your files are larger than 100 MiB, see the section below on CAR files, which can be used to split uploads between several HTTP requests.

## Retrieving Files
Now that we have uploaded a file, we should be able to fetch its cid directly in our browser with their `nftstorage.link` gateway.

`https://bafybeigvgzoolc3drupxhlevdp2ugqcrbcsqfmcek2zxiw5wctk3xjpjwy.ipfs.nftstorage.link`

## Status
```shell
$ curl 'https://api.nft.storage/bafkreidr4m5ethn4bj25nnfnwb5ykcltivfceosuzlbfainfjom2nwkllm' \
--header 'Authorization: Bearer YOUR_API_KEY' | jq
{
  "ok": true,
  "value": {
    "cid": "bafkreidr4m5ethn4bj25nnfnwb5ykcltivfceosuzlbfainfjom2nwkllm",
    "created": "2022-12-15T13:30:47.155+00:00",
    "type": "application/car",
    "scope": "session",
    "files": [],
    "size": 68910,
    "name": "Upload at 2022-12-15T13:30:47.155Z",
    "pin": {
      "cid": "bafkreidr4m5ethn4bj25nnfnwb5ykcltivfceosuzlbfainfjom2nwkllm",
      "created": "2022-12-15T13:30:47.155+00:00",
      "size": 68910,
      "status": "pinned"
    },
    "deals": [
      {
        "status": "active",
        "lastChanged": "2023-08-31T14:15:03.709152+00:00",
        "chainDealID": 35317845,
        "datamodelSelector": "Links/134/Hash/Links/1/Hash/Links/0/Hash",
        "statusText": null,
        "dealActivation": "2023-05-08T16:43:00+00:00",
        "dealExpiration": "2024-10-21T16:43:00+00:00",
        "miner": "f020378",
        "pieceCid": "baga6ea4seaqmksbhhyjht4oxknuxm6ywkw5vr4tzge6s43y5f7cbiglalll22ny",
        "batchRootCid": "bafybeicb5ktvfmyittcvv33372sogtyzwssdeqfyo33esy3zbuljfdvlza"
      },
      ...
    ]
  }
}
```

## Pinning
You can ask NFT.Storage to archive data that is already on the IPFS distributed storage network with this API. This data will remain perpetually available over IPFS (backed by Filecoin decentralized storage).

NFT.Storage provides a pinning service that is modeled closely on the IPFS Pinning Service API specification. **NFT.Storage's Pinning Service API is not to be used for ongoing production traffic, but rather for one-time migrations.**

**You do not need to request access if you are storing data with NFT.Storage directly. Data stored with NFT.Storage is persisted indefinitely by default. This API is only useful if you are looking to store data with NFT.Storage that is already available on the IPFS network. Even in these situations, if you are able to, we recommend you generate a CAR file from a IPFS node hosting the content and directly upload that to NFT.Storage (e.g., run ipfs dag export from your local node) rather than use the Pinning API.**

For a full list and documentation of all the available pinning service endpoints, visit the IPFS Pinning Service API endpoint documentation.

### Requesting access
To request access to the pinning service for your NFT.Storage account, you will need to request access from your API Key account page. Once approved, you will be able to access the pinning service API endpoints using your API token.

### Using the HTTP API
The NFT.Storage pinning service endpoint for all requests is https://api.nft.storage/pins. For additional documentation, please see the IPFS Pinning Service API endpoint documentation.

### Add a pin
```shell
$ curl -X POST 'https://api.nft.storage/pins' \
--header 'Accept: */*' \
--header 'Authorization: Bearer <YOUR_AUTH_KEY_JWT>' \
--header 'Content-Type: application/json' \
-d '{
"cid": "QmCIDToBePinned",
"name": "PreciousData.pdf"
}'
```

### List successful pins
```shell
curl -X GET 'https://api.nft.storage/pins' \
--header 'Accept: */*' \
--header 'Authorization: Bearer <YOUR_AUTH_KEY_JWT>'
```

## Additional Resources
You can find additional resources @ https://nft.storage/docs