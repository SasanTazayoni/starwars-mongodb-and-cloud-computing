# Amazon S3 (Simple Storage Service)

Amazon S3 is AWS's object storage service — designed to store and retrieve any amount of data from anywhere on the internet.

---

## Everything is an Object

Unlike a traditional file system, S3 stores data as **objects**. Each object consists of two parts:

- **Data** — the file itself (an image, a video, a JSON document, a backup, etc.)
- **Metadata** — information *about* the file, such as its content type, size, and any custom tags you add

Every object is uniquely identified by a **key** — its name within the bucket. The combination of bucket + key always points to exactly one object, making every object in S3 unique.

Objects live inside **buckets** — think of a bucket as a top-level container, similar to a root folder. Bucket names must be **globally unique across all of AWS** — not just within your account, but across every AWS account worldwide. If a name is taken, it is taken for everyone. This means names like `my-bucket` or `test` are almost certainly already gone, so use something specific to your project.

---

## How Objects are Stored

S3 uses a **flat storage structure** — there are no real folders or directories. Instead, every object is stored at the bucket level and identified purely by its key.

What looks like a folder path (e.g. `images/profile/avatar.png`) is just a key with slashes in the name. The AWS console displays these as folders for readability, but underneath it is a single flat namespace.

![S3 Folder Illusion](../images/s3-folder-illusion.png)

When you store an object, S3 automatically distributes it across **multiple Availability Zones** within the chosen region — separate physical data centres with independent power and networking. This is what gives S3 its durability guarantee; even if an entire facility goes down, your data is unaffected.

Each object can be up to **5 TB** in size. Objects larger than 100 MB should be uploaded using **multipart upload**, which splits the file into chunks, uploads them in parallel, and reassembles them — making large uploads faster and more resilient to network interruptions.

---

## Secure

S3 buckets are **private by default** — nothing is publicly accessible unless you explicitly allow it. Access is controlled through:

- **IAM policies** — define which AWS users or roles can read, write, or delete objects
- **Bucket policies** — resource-level rules attached directly to the bucket
- **Encryption** — data can be encrypted at rest and in transit

### Making Objects Public

To make an object publicly accessible — for example, serving images to a web app or hosting a static site — you need to do two things:

1. **Disable Block Public Access** on the bucket (AWS enables this by default as a safety net)
2. **Attach a bucket policy** that explicitly grants public read permission

Without both steps, the object remains private even if you intend it to be public.

---

## Local AWS Credentials Setup

To interact with S3 programmatically (e.g. via boto3), AWS needs to know who you are. Credentials are stored locally in two files inside a hidden `.aws` folder in your home directory.

### Setup Steps

```bash
mkdir .aws        # create a hidden folder called .aws in your home directory
cd .aws           # navigate into it
touch credentials # create an empty file called credentials
touch config      # create an empty file called config
```

Then open each file with a text editor (e.g. `notepad credentials`) and paste in the content below.

### `~/.aws/credentials`

This file holds your AWS access keys. You get these from the IAM console when you create an access key for your user.

```ini
[default]
aws_access_key_id = ACCESS-KEY-GOES-HERE
aws_secret_access_key = SECRET-KEY-GOES-HERE
```

> **Never commit this file to git or share your real key values.** Anyone with your access key can use your AWS account and run up charges.

### `~/.aws/config`

This file sets your default region and output format.

```ini
[default]
region = eu-west-1
output = json
```

Once both files are in place, boto3 and the AWS CLI will pick them up automatically — no need to hardcode credentials in your code.

### Testing Your Credentials

Run this in Python to confirm everything is working:

```python
import boto3
from botocore.exceptions import NoCredentialsError, ClientError

s3_client = boto3.client('s3')

try:
    bucket_list = s3_client.list_buckets()
    print("Credentials valid. Buckets:", [b['Name'] for b in bucket_list['Buckets']])
except NoCredentialsError:
    print("No credentials found — check your ~/.aws/credentials file.")
except ClientError as e:
    print("Credentials rejected by AWS:", e.response['Error']['Message'])
```

- **Success** — prints your bucket list, confirming boto3 can authenticate
- **`NoCredentialsError`** — boto3 could not find your `~/.aws/credentials` file
- **`ClientError`** — credentials were found but AWS rejected them (wrong key or expired)

---

## Durable

S3 is designed for **99.999999999% (11 nines) durability**. AWS automatically replicates your data across multiple physical locations within a region, so a single hardware failure has no impact on your files.

---

## Scalable

There is no storage limit. You can store a single file or billions of them — S3 scales automatically without any configuration or capacity planning on your part.

---

## Versioning

S3 can keep every version of an object automatically. When versioning is enabled on a bucket, uploading a file with the same key does not overwrite the existing object — it creates a new version alongside it. Every version gets its own unique ID and can be retrieved, restored, or deleted independently.

This protects against two common problems:

- **Accidental overwrites** — the previous version is always recoverable
- **Accidental deletes** — deleting an object just adds a delete marker; the original versions remain intact

Versioning is disabled by default and must be explicitly enabled per bucket. Once enabled it cannot be fully disabled, only suspended.

---

## Storage Classes

Not all data is accessed equally. S3 offers different **storage classes** so you only pay for the level of availability and retrieval speed you actually need.

| Storage Class | Best For | Retrieval Speed |
| --- | --- | --- |
| **S3 Standard** | Frequently accessed data | Milliseconds |
| **S3 Intelligent-Tiering** | Data with unpredictable access patterns — AWS automatically moves objects between tiers based on usage | Milliseconds |
| **S3 Standard-IA** (Infrequent Access) | Data accessed occasionally but must be available immediately when needed | Milliseconds |
| **S3 One Zone-IA** | Infrequent access, but stored in a single AZ — cheaper, less resilient | Milliseconds |
| **S3 Glacier Instant Retrieval** | Archive data that still needs to be accessible quickly | Milliseconds |
| **S3 Glacier Flexible Retrieval** | Archive data where a wait is acceptable | Minutes to hours |
| **S3 Glacier Deep Archive** | Long-term cold storage — lowest cost option | Up to 12 hours |

The general rule: the less frequently you need your data, the cheaper it is to store — but the longer it takes to get it back.

---

## Common Use Cases

- **Static website hosting** — S3 can serve HTML, CSS, and JavaScript files directly to a browser, no server required
- **Media storage** — store and serve images, videos, and audio files for a web or mobile app
- **Backups and snapshots** — a cheap, durable destination for database backups, log archives, and disaster recovery files
- **Data pipeline storage** — act as the landing zone for raw data before it is processed and loaded into a database or warehouse
- **Software distribution** — host installers, build artefacts, or any file that needs to be downloaded at scale

---

## What Do You Pay For?

S3 pricing is broken down into a few dimensions — you are never charged a flat fee, only for what you actually use:

| Cost Dimension | What It Means |
| --- | --- |
| **Storage** | Charged per GB stored per month. The rate varies by storage class — Standard is most expensive, Glacier Deep Archive is cheapest |
| **Requests** | Every API call costs a small amount — PUT, COPY, POST, LIST requests cost more than GET requests |
| **Data transfer out** | Data leaving S3 to the internet is charged per GB. Data coming *in* to S3 is free. Data transferred to other AWS services in the same region is also free |
| **Replication** | If you replicate objects across regions, you pay for the storage in both regions plus data transfer between them |

> **In practice:** For most small projects, S3 costs are negligible — storing 10 GB of images and serving moderate traffic will cost pennies per month. Costs only become significant at scale or with heavy cross-region data transfer.

---

## What is an Endpoint?

An **endpoint** is a URL that points to a specific resource — something a browser, app, or API can send a request to and get data back from.

Every object stored in S3 automatically gets a unique public URL in this format:

```
https://<bucket-name>.s3.<region>.amazonaws.com/<key>
```

For example:

```
https://my-app-assets.s3.eu-west-1.amazonaws.com/images/avatar.png
```

That URL is the endpoint for that object. Anyone (or any application) with access can send a GET request to it and receive the file directly — no server, no backend code, no database query.

---

## Why S3 is Good for Endpoints

S3 is well suited to serving files via URLs for several reasons:

- **No server required** — S3 handles the HTTP layer itself. You do not need to run a web server just to serve a file
- **Globally scalable** — S3 can handle millions of simultaneous requests without any configuration. A viral image or a popular file download will not overwhelm it
- **Stable, permanent URLs** — as long as the object exists and permissions allow it, the URL never changes. You can store that URL in a database and it will keep working
- **Works with CDNs** — S3 integrates directly with **CloudFront** (AWS's CDN), which caches your objects at edge locations around the world for faster delivery
- **Presigned URLs** — for private files, S3 can generate a temporary URL that expires after a set time. This lets you share a private object securely without making it permanently public

### Example use cases for S3 endpoints

- Store user profile pictures and save the S3 URL in your database — your app just renders the URL in an `<img>` tag
- Upload a processed report or export file and send the user a presigned URL to download it
- Host all the static assets (CSS, JS, images) for a web app directly from S3, bypassing the need for a file server

---

## S3 is Not a Data Lake

S3 is often *used as* the foundation of a data lake, but it is not one by itself — it is just storage.

A **data lake** is a centralised repository for storing large volumes of raw data in any format — structured (CSVs, databases), semi-structured (JSON, logs), or unstructured (images, videos). The idea is to dump everything in first and figure out how to use it later.

S3 provides the storage layer, but a data lake also requires tools for cataloguing, querying, and governing that data. On AWS, services like **Athena** (query), **Glue** (catalogue and transform), and **Lake Formation** (governance) are what turn an S3 bucket into an actual data lake.

In short: S3 is a bucket of objects. A data lake is a system built on top of it.
