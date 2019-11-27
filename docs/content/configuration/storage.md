---
title: Configuring Storage
description: Configuring Storage in Athens
---

## Storage

The Athens proxy supports many storage types:

1. [Memory](#memory)
1. [Disk](#disk)
1. [Mongo](#mongo)
1. [Google Cloud Storage](#google-cloud-storage)
1. [AWS S3](#aws-s3)
1. [Minio](#minio)
    1. [DigitalOcean Spaces](#digitalocean-spaces)
    1. [Alibaba OSS](#alibaba-oss)
    1. and other S3 / Minio compatible interfaces
1. [Azure Blob Storage](#azure-blob-storage)

All of them can be configured using `config.toml` file. You need to set a valid driver in `StorageType` value or you can set it in environment variable `ATHENS_STORAGE_TYPE` on your server.
Also for most of the drivers you need to provide additional configuration data which will be described below.

## Memory

This storage doesn't need any specific configuration and it's also used by default in the Athens project. It writes all of data into local disk into `tmp` dir.

**This storage type should only be used for development purposes!**

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "memory"

## Disk

Disk storage allows modules to be stored on a file system. The location on disk where modules will be stored can be configured.

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "disk"

    [Storage]
        [Storage.Disk]
            RootPath = "/path/on/disk"

where `/path/on/disk` is your desired location. Also it can be set using `ATHENS_DISK_STORAGE_ROOT` env

## Mongo

This driver uses a [Mongo](https://www.mongodb.com/) server as data storage. On start this driver will create an `athens` database and `module` collection on your Mongo server.

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "mongo"

    [Storage]
        [Storage.Mongo]
            # Full URL for mongo storage
            # Env override: ATHENS_MONGO_STORAGE_URL
            URL = "mongodb://127.0.0.1:27017"

            # Not required parameter
            # Path to certificate to use for the mongo connection
            # Env override: ATHENS_MONGO_CERT_PATH
            CertPath = "/path/to/cert/file"

            # Not required parameter
            # Allows for insecure SSL / http connections to mongo storage
            # Should be used for testing or development only
            # Env override: ATHENS_MONGO_INSECURE
            Insecure = false

            # Not required parameter
            # Allows for use of custom database 
            # Env override: ATHENS_MONGO_DEFAULT_DATABASE
            DefaultDBName = athens

            # Not required parameter
            # Allows for use of custom collection 
            # Env override: ATHENS_MONGO_DEFAULT_COLLECTION
            DefaultCollectionName = modules
## Google Cloud Storage

This driver uses [Google Cloud Storage](https://cloud.google.com/storage/) and assumes that you already have an `account` and `bucket` in it.
If you never used Google Cloud Storage there is [quick guide](https://cloud.google.com/storage/docs/creating-buckets#storage-create-bucket-console)
how to create `bucket` inside it.

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "gcp"

    [Storage]
        [Storage.GCP]
            # ProjectID to use for GCP Storage
            # Env overide: GOOGLE_CLOUD_PROJECT
            ProjectID = "YOUR_GCP_PROJECT_ID"

            # Bucket to use for GCP Storage
            # Env override: ATHENS_STORAGE_GCP_BUCKET
            Bucket = "YOUR_GCP_BUCKET"

## AWS S3

This driver is using the [AWS S3](https://aws.amazon.com/s3/) and assumes that you already have `account` and `bucket` created in it.
If you never used Amazon Web Services there is [quick guide](https://docs.aws.amazon.com/AmazonS3/latest/gsg/GetStartedWithS3.html) how to create `bucket` inside it.
After this you can pass your credentials inside `config.toml` file.  If the access key ID and secret access key are not specified in `config.toml`, the driver will attempt to load credentials for the default profile from the [AWS CLI configuration file](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-files.html) created during installation.

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "s3"
    
    [Storage]
            [Storage.S3]
            ### The authentication model is as below for S3 in the following order
            ### If AWS_CREDENTIALS_ENDPOINT is specified and it returns valid results, then it is used
            ### If config variables are specified and they are valid, then they return valid results, then it is used
            ### Otherwise, it will default to default configurations which is as follows
            # attempt to find credentials in the environment, in the shared
            # configuration (~/.aws/credentials) and from ec2 instance role
            # credentials. See
            # https://godoc.org/github.com/aws/aws-sdk-go#hdr-Configuring_Credentials
            # and
            # https://godoc.org/github.com/aws/aws-sdk-go/aws/session#hdr-Environment_Variables
            # for environment variables that will affect the aws configuration.
            # Setting UseDefaultConfiguration would only use default configuration. It will be deprecated in future releases 
            # and is recommended not to use it.

            # Region for S3 storage
            # Env override: AWS_REGION
            Region = "MY_AWS_REGION"

            # Access Key for S3 storage
            # Env override: AWS_ACCESS_KEY_ID
            Key = "MY_AWS_ACCESS_KEY_ID"

            # Secret Key for S3 storage
            # Env override: AWS_SECRET_ACCESS_KEY
            Secret = "MY_AWS_SECRET_ACCESS_KEY"
    
            # Session Token for S3 storage
            # Not required parameter
            # Env override: AWS_SESSION_TOKEN
            Token = ""

            # S3 Bucket to use for storage
            # Env override: ATHENS_S3_BUCKET_NAME
            Bucket = "MY_S3_BUCKET_NAME"

            # If true then the default aws configuration will be used. This will
            # attempt to find credentials in the environment, in the shared
            # configuration (~/.aws/credentials) and from ec2 instance role
            # credentials. See
            # https://godoc.org/github.com/aws/aws-sdk-go#hdr-Configuring_Credentials
            # and
            # https://godoc.org/github.com/aws/aws-sdk-go/aws/session#hdr-Environment_Variables
            # for environment variables that will affect the aws configuration.
            # Env override: AWS_USE_DEFAULT_CONFIGURATION
            UseDefaultConfiguration = false

            # https://docs.aws.amazon.com/sdk-for-go/api/aws/credentials/endpointcreds/
            # Note that this the URI should not end with / when AwsContainerCredentialsRelativeURI is set
            # Env override: AWS_CREDENTIALS_ENDPOINT
            CredentialsEndpoint = ""

            # Env override: AWS_CONTAINER_CREDENTIALS_RELATIVE_URI
            # If you are planning to use AWS Fargate, please use http://169.254.170.2 for CredentialsEndpoint
            # Ref: https://docs.aws.amazon.com/AmazonECS/latest/developerguide/task-metadata-endpoint-v2.html
            AwsContainerCredentialsRelativeURI = ""

## Minio

[Minio](https://www.minio.io/) is an open source object storage server that provides an interface for S3 compatible block storages. If you have never used minio, you can read this [quick start guide](https://docs.minio.io/).  Any S3 compatible object storage is supported by Athens through the minio interface. Below, you can find different configuration options we provide for Minio. Example configuration for Digital Ocean and Alibaba OSS block storages are provided below.

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "minio"

    [Storage]
        [Storage.Minio]
            # Endpoint for Minio storage
            # Env override: ATHENS_MINIO_ENDPOINT
            Endpoint = "127.0.0.1:9001"

            # Access Key for Minio storage
            # Env override: ATHENS_MINIO_ACCESS_KEY_ID
            Key = "YOUR_MINIO_SECRET_KEY"

            # Secret Key for Minio storage
            # Env override: ATHENS_MINIO_SECRET_ACCESS_KEY
            Secret = "YOUR_MINIO_SECRET_KEY"

            # Enable SSL for Minio connections
            # Defaults to true
            # Env override: ATHENS_MINIO_USE_SSL
            EnableSSL = false

            # Minio Bucket to use for storage
            # Env override: ATHENS_MINIO_BUCKET_NAME
            Bucket = "gomods"

#### DigitalOcean Spaces

For Athens to communicate with [DigitalOcean Spaces](https://www.digitalocean.com/products/spaces/), we are using Minio driver because DO Spaces tries to be [fully compatible with it](https://developers.digitalocean.com/documentation/spaces/).
Also configuration for this storage looks almost the same in our proxy as for [Minio](#minio).

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "minio"

    [Storage]
        [Storage.Minio]
            # Address of DO Spaces storage
            # Env override: ATHENS_MINIO_ENDPOINT
            Endpoint = "YOUR_ADDRESS.digitaloceanspaces.com"

            # Access Key for DO Spaces storage
            # Env override: ATHENS_MINIO_ACCESS_KEY_ID
            Key = "YOUR_DO_SPACE_KEY_ID"

            # Secret Key for DO Spaces storage
            # Env override: ATHENS_MINIO_SECRET_ACCESS_KEY
            Secret = "YOUR_DO_SPACE_SECRET_KEY"

            # Enable SSL
            # Env override: ATHENS_MINIO_USE_SSL
            EnableSSL = true

            # Space name in your DO Spaces storage
            # Env override: ATHENS_MINIO_BUCKET_NAME
            Bucket = "YOUR_DO_SPACE_NAME"

            # Region for DO Spaces storage
            # Env override: ATHENS_MINIO_REGION
            Region = "YOUR_DO_SPACE_REGION"

#### Alibaba OSS

For Athens to communicate with [Alibaba Cloud Object Storage Service](https://www.alibabacloud.com/product/oss), we are using Minio driver.
Also configuration for this storage looks almost the same in our proxy as for [Minio](#minio).

##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "minio"

    [Storage]
        [Storage.Minio]
            # Address of Alibaba OSS storage
            # Env override: ATHENS_MINIO_ENDPOINT
            Endpoint = "YOUR_ADDRESS.aliyuncs.com"

            # Access Key for Minio storage
            # Env override: ATHENS_MINIO_ACCESS_KEY_ID
            Key = "YOUR_OSS_KEY_ID"

            # Secret Key for Alibaba OSS storage
            # Env override: ATHENS_MINIO_SECRET_ACCESS_KEY
            Secret = "YOUR_OSS_SECRET_KEY"

            # Enable SSL
            # Env override: ATHENS_MINIO_USE_SSL
            EnableSSL = true

            # Parent folder in your Alibaba OSS storage
            # Env override: ATHENS_MINIO_BUCKET_NAME
            Bucket = "YOUR_OSS_FOLDER_PREFIX"

## Azure Blob Storage

This driver uses [Azure Blob Storage](https://azure.microsoft.com/services/storage/blobs/) 

> If you never used Azure Blog Storage, here is a [quickstart](https://aka.ms/azureblob-quickstart)

It assumes that you already have the following:

- [An Azure storage account](https://docs.microsoft.com/azure/storage/common/storage-account-overview?toc=%2fazure%2fstorage%2fblobs%2ftoc.json)
- [The credentials (storage account key)](https://docs.microsoft.com/rest/api/storageservices/authorize-with-shared-key)  
- A container (to store blobs)


##### Configuration:

    # StorageType sets the type of storage backend the proxy will use.
    # Env override: ATHENS_STORAGE_TYPE
    StorageType = "azureblob"

    [Storage]
        [Storage.AzureBlob]
            # Storage Account name for Azure Blob
            # Env override: ATHENS_AZURE_ACCOUNT_NAME
            AccountName = "MY_AZURE_BLOB_ACCOUNT_NAME"

            # Account Key to use with the storage account
            # Env override: ATHENS_AZURE_ACCOUNT_KEY
            AccountKey = "MY_AZURE_BLOB_ACCOUNT_KEY"

            # Name of container in the blob storage
            # Env override: ATHENS_AZURE_CONTAINER_NAME
            ContainerName = "MY_AZURE_BLOB_CONTAINER_NAME"