# Starwars MongoDB and Cloud Computing

![MongoDB](./tech/mongodb.png) ![Python](./tech/python.png) ![VSCode](./tech/vscode.png) ![Claude](./tech/claude.png) ![Shell](./tech/shell.png) ![AWS](./tech/aws.png)

---

# MongoDB

MongoDB is a **document-oriented NoSQL database** that stores data as flexible, JSON-like documents. Unlike traditional relational databases (like MySQL or PostgreSQL) that organise data into rigid rows and tables, MongoDB uses **collections** and **documents** — making it a natural fit for modern applications that deal with varied or frequently changing data.

## Types of NoSQL Databases

NoSQL ("Not Only SQL") is a broad category of databases that move away from the fixed table-and-row model of relational databases. There are four main types:

| Type | How data is stored | Examples | Best for |
|---|---|---|---|
| **Document** | JSON/BSON documents in collections | MongoDB, CouchDB | Flexible records, APIs, content |
| **Key-Value** | Simple key → value pairs | Redis, DynamoDB | Caching, sessions, leaderboards |
| **Wide-Column** | Rows with dynamic columns across partitions | Cassandra, HBase | Time-series, IoT, write-heavy workloads |
| **Graph** | Nodes and edges representing relationships | Neo4j, Amazon Neptune | Social networks, fraud detection, recommendations |

MongoDB is an example of a **document database** — the most widely used NoSQL type.

---

## Installing MongoDB Locally

Download links for all popular operating systems can be found here:
https://www.mongodb.com/try/download/community

Make sure to get the **Community** edition rather than the Enterprise edition.

### Windows

1. Scroll down to the installer download section
2. Choose **8.3.2 (current)**
3. Choose **Windows x64**
4. Choose **msi**
5. Click **Download**
6. Open the installer
7. Accept the license agreement and choose **"Complete"** install
8. Make sure to select **"Install MongoDB as a Service"**
9. Make sure to check the **"Install MongoDB Compass"** checkbox
10. Click **"Install"** and wait for it to finish

Once installed, you can open Compass and connect to your local MongoDB instance using the default connection string `mongodb://localhost:27017`.

---

## How does it compare to SQL?

| SQL Term    | MongoDB Equivalent | Description                          |
| ----------- | ------------------ | ------------------------------------ |
| Database    | Database           | A container for collections          |
| Table       | Collection         | A group of related documents         |
| Row         | Document           | A single record, stored as JSON/BSON |
| Column      | Field              | A key-value pair within a document   |
| Primary Key | `_id`              | Auto-generated unique identifier     |

## What does a document look like?

```json
{
  "_id": "64a7f3c2e4b0d1a2c3f4e5b6",
  "name": "Alice Johnson",
  "course": "Data Engineering",
  "enrolled": true,
  "grades": [85, 92, 78]
}
```

Documents are self-contained and can hold nested objects and arrays — something not natively possible in a SQL row. MongoDB stores data in **BSON** (Binary JSON) format internally, which supports richer data types and faster reads.

## MongoDB Use Cases and advantages/disadvantages

**Advantages:**

- Document Oriented Storage
- Easy Scaling (Horizontal)
- Fast/Efficient
- "Open Source"
  - Publicly available
  - Source code can be edited/changed at will
  - Free to use, at scale — no license fees etc.
  - Code can be distributed at will
- Flexible

**Disadvantages:**

- High memory usage and data redundancy
- Can be inconsistent
- Unsupported transactions

**Use Cases:**

- Social media posts/data
- API data (JSON)
- Mobile App Backend
- Caching
  - E.g. shopping cart
- Product info
- Media files/data
- CMS systems
- CRM systems
- IoT and Sensor data
- Logs and monitoring
- Gaming data
- Chat systems/apps

## Connecting Locally with MongoDB Compass

**MongoDB Compass** is the official GUI for MongoDB. It lets you explore databases, run queries, and manage data without the command line.

### Prerequisites

- MongoDB installed and running locally
- [MongoDB Compass](https://www.mongodb.com/try/download/compass) downloaded and installed

### Steps

1. Open **MongoDB Compass**
2. In the connection string field on the home screen, enter:

```
mongodb://localhost:27017
```

3. Click **"Connect"** — Compass will connect and display your databases in the left panel

![New Connection dialog](images/compass-new-connection.png)

Once connected, you will see your databases listed in the left panel:

![Compass connected](images/compass-connected.png)

> **Tip:** If the connection fails, make sure the MongoDB service is running. On Windows, check Services or run `mongod` in a terminal.

---

## Creating a Database and Collection

### Using MongoDB Compass

1. In the left panel, click **"+ Create database"**
2. Enter a **Database Name** (e.g. `sparta`) and an initial **Collection Name** (e.g. `institute`)
3. Click **"Create Database"**

![Create Database dialog](images/compass-create-database.png)

> MongoDB requires at least one collection when creating a database.

### Using MongoDB Compass to add a Collection to an existing Database

1. Select your database from the left panel
2. Click **"+ Create collection"**, enter a name, and click **"Create Collection"**

---

## MongoShell

By default mongosh will open with `test`

We can create and switch to a new database with the `use` command:

```mongosh
use sparta
```

Once we run this command, the test>database indicator should change to sparta>. We can also use the db command to show the active database.

```mongosh
db.createCollection("institute")
```

### Adding Documents via MongoDB Compass

**Insert a single document:**

1. Open your collection in Compass
2. Click **"Add data"** → **"Insert document"**
3. Edit the JSON in the editor (the `_id` field is auto-generated)
4. Click **"Insert"**

![Insert Document editor](images/compass-insert-document.png)

**Insert multiple documents:**

1. Click **"Add data"** → **"Insert document"**
2. Replace the `{}` with an array of documents:

```json
[{ "course": "Data Engineering" }, { "course": "Data Analysis" }]
```

3. Click **"Insert"**

---

### Adding Documents via mongosh

To insert data into our collection:

```mongosh
db.institute.insertOne({name: "New document"})
```

Add multiple documents in one command:

```mongosh
db.institute.insertMany([{"course": "Data Engineering"}, {"course": "Data Analysis"}])
```

How to view documents in a collection in mongosh:

```mongosh
db.institute.find()
```

![Collection view in Compass](images/compass-collection-view.png)

---

## Validation

MongoDB allows you to enforce rules on documents in a collection using **JSON Schema validation**. This ensures that only documents matching your defined structure can be inserted.

### Creating a Collection with Validation

The example below creates a `students` collection that requires every document to have a `name` (string) and an `age` (integer):

```mongosh
db.createCollection("students", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["name", "age"],
      properties: {
        name: { bsonType: "string" },
        age: { bsonType: "int" }
      }
    }
  }
})
```

A successful output looks like:

![Validation collection created](images/validation-create-collection.png)

### Invalid Document

Attempting to insert a document that fails validation — here `age` is missing:

```mongosh
db.students.insertOne({ name: "Alice" })
```

MongoDB rejects the insert with a validation error:

![Invalid insert error](images/validation-invalid.png)

### Valid Document

Inserting a document that meets all the validation rules:

```mongosh
db.students.insertOne({ name: "Alice", age: 25 })
```

MongoDB confirms the insert was successful:

![Valid insert success](images/validation-valid.png)

---

## Working with a Films Collection

### Creating the Collection with Validation

The `films` collection requires every document to have a `title` (string), `year` (int), and `genre` (string):

```mongosh
db.createCollection("films", {
  validator: {
    $jsonSchema: {
      bsonType: "object",
      required: ["title", "year", "genre"],
      properties: {
        title: { bsonType: "string" },
        year: { bsonType: "int" },
        genre: { bsonType: "string" }
      }
    }
  }
})
```

![Films collection created](images/films-create-collection.png)

---

### Inserting Documents

**insertMany** — insert multiple documents in one command:

```mongosh
db.films.insertMany([
  { title: "Inception", year: 2010, genre: "Sci-Fi" },
  { title: "The Dark Knight", year: 2008, genre: "Action" },
  { title: "Interstellar", year: 2014, genre: "Sci-Fi" }
])
```

![insertMany output](images/films-insert-many.png)

**insertOne** — insert a single document:

```mongosh
db.films.insertOne({ title: "The Matrix", year: 1999, genre: "Sci-Fi" })
```

![insertOne output](images/films-insert-one.png)

**insert()** — the older deprecated method, still works but not recommended:

```mongosh
db.films.insert({ title: "Pulp Fiction", year: 1994, genre: "Crime" })
```

MongoDB will show a `DeprecationWarning` and recommend using `insertOne`, `insertMany`, or `bulkWrite` instead:

![insert() deprecation warning](images/films-insert-deprecated.png)

---

### Searching for Documents

**Find all documents** in a collection:

```mongosh
db.films.find()
```

![find() all documents](images/films-find-all.png)

**Filter by a field** — find all Sci-Fi films:

```mongosh
db.films.find({ genre: "Sci-Fi" })
```

Only documents matching the filter are returned:

![find() with filter](images/films-find-filter.png)

---

### Updating Documents

**updateOne** — update the first document that matches the filter. Here we correct the year for Inception:

```mongosh
db.films.updateOne({ title: "Inception" }, { $set: { year: 2011 } })
```

`matchedCount: 1` and `modifiedCount: 1` confirm one document was updated:

![updateOne output](images/films-update-one.png)

**updateMany** — update all documents that match the filter. Here we rename the genre for all Sci-Fi films:

```mongosh
db.films.updateMany({ genre: "Sci-Fi" }, { $set: { genre: "Science Fiction" } })
```

`matchedCount: 3` and `modifiedCount: 3` confirm all matching documents were updated:

![updateMany output](images/films-update-many.png)

---

### Deleting Documents

**deleteOne** — delete the first document that matches the filter:

```mongosh
db.films.deleteOne({ title: "Pulp Fiction" })
```

`deletedCount: 1` confirms the document was removed:

![deleteOne output](images/films-delete-one.png)

---

## Embedding vs Referencing

When designing a MongoDB database, one of the key decisions is how to handle **related data**. There are two main approaches: **embedding** and **referencing**.

---

### Embedding

Embedding means storing related data **directly inside a document** as a nested object or array.

```json
{
  "_id": "1",
  "name": "Alice Johnson",
  "address": {
    "street": "123 Main St",
    "city": "London",
    "postcode": "EC1A 1BB"
  },
  "courses": ["Data Engineering", "Data Analysis"]
}
```

**When to use embedding:**

- The nested data is always accessed together with the parent document
- The nested data doesn't change frequently
- The relationship is **one-to-few** (e.g. a user with a few addresses)
- You want fast reads — one query returns everything you need

**Advantages:**

- Single query to retrieve all related data
- No JOINs required
- Better read performance

**Disadvantages:**

- Documents can grow very large
- Duplicated data if the same nested data is shared across multiple documents
- Harder to update nested data across many documents

---

### Referencing

Referencing means storing related data in a **separate collection** and linking to it using an `_id` — similar to a foreign key in SQL.

```json
// users collection
{
  "_id": "1",
  "name": "Alice Johnson",
  "course_ids": ["101", "102"]
}

// courses collection
{
  "_id": "101",
  "title": "Data Engineering"
}
```

To retrieve the full data, you would need to query both collections.

**When to use referencing:**

- The related data is large or frequently updated
- The related data is shared across many documents
- The relationship is **one-to-many** or **many-to-many**
- You want to avoid data duplication

**Advantages:**

- Smaller, cleaner documents
- Easier to update shared data in one place
- Better for complex or large relationships

**Disadvantages:**

- Requires multiple queries to retrieve related data
- More complex application logic

---

### Summary

|                  | Embedding                | Referencing                  |
| ---------------- | ------------------------ | ---------------------------- |
| Data location    | Inside the document      | Separate collection          |
| Query complexity | Simple (one query)       | Complex (multiple queries)   |
| Best for         | One-to-few relationships | One-to-many / many-to-many   |
| Update ease      | Harder across documents  | Easier (update in one place) |
| Performance      | Faster reads             | Faster writes/updates        |

---

## Replica Sets

A **replica set** is a group of MongoDB servers that all hold copies of the same data. It is MongoDB's built-in mechanism for **high availability** and **data redundancy**.

### How it works

A replica set always has:

- **Primary** — the only node that accepts write operations. All changes flow through here.
- **Secondaries** — one or more nodes that continuously replicate data from the primary. They can serve read requests but cannot accept writes.
- **Arbiter** *(optional)* — a lightweight node that holds no data. Its sole purpose is to vote in elections.

```
         ┌─────────────┐
Writes → │   Primary   │
         └──────┬──────┘
                │ replication
       ┌────────┴────────┐
       ▼                 ▼
 ┌──────────┐     ┌──────────┐
 │Secondary │     │Secondary │
 └──────────┘     └──────────┘
```

### Automatic failover

If the primary becomes unavailable, the remaining nodes hold an **election** and promote one of the secondaries to primary — automatically, with no manual intervention. This is what makes MongoDB resilient to server failures.

> **In short:** Replica sets protect against data loss and keep your database available even if a server goes down.

---

## Sharding

**Sharding** is MongoDB's approach to **horizontal scaling** — distributing data across multiple machines so that no single server has to hold or process everything.

### The problem it solves

A single MongoDB server has limits on storage and throughput. When a dataset grows too large for one machine, sharding splits it across many servers called **shards**, each holding a subset of the data.

### How it works

A sharded cluster has three components:

| Component | Role |
|---|---|
| **Shards** | Each shard is a replica set that stores a portion of the data |
| **Config servers** | Store metadata about which shard holds which data |
| **mongos (query router)** | The entry point for client requests — routes queries to the correct shard(s) |

MongoDB uses a **shard key** — a field you choose — to decide how to distribute documents across shards. For example, if your shard key is `country`, all UK documents might go to Shard A and all US documents to Shard B.

```
Client → mongos (router)
              │
    ┌─────────┼─────────┐
    ▼         ▼         ▼
 Shard A   Shard B   Shard C
 (replica  (replica  (replica
  set)      set)      set)
```

> **In short:** Replica sets protect against failure. Sharding handles growth. In production, each shard in a sharded cluster is itself a replica set — so you get both.

---

## PyMongo — Interacting with MongoDB via Python

**PyMongo** is the official Python driver for MongoDB. It lets you connect to a MongoDB database and perform CRUD operations using Python instead of the Mongo shell.

### Setup

```bash
pip install pymongo requests
```

### Star Wars Database

The Python scripts in this project fetch Star Wars data from the [SWAPI API](https://swapi.info/) and populate a `starwars` database in MongoDB, using **referencing** between collections (pilots stored as `_id` references rather than embedded objects).

Run each script to populate the corresponding collection:

```bash
python mongodb/starships.py
python mongodb/vehicles.py
python mongodb/planets.py
python mongodb/species.py
python mongodb/films.py
```

> **Note:** `characters` must be pre-populated separately as it is referenced by all other collections.

### Querying with PyMongo

`queries.py` demonstrates common query patterns against the `characters` collection:

```python
import pymongo

client = pymongo.MongoClient()
db = client['starwars']

# Find one document
db.characters.find_one({"name": "Darth Vader"})

# Filter with projection
db.characters.find_one(
    {"name": "Darth Vader"},
    {"name": 1, "height": 1, "_id": 0}
)

# Iterate a cursor
for doc in db.characters.find({"eye_color": "yellow"}):
    print(doc["name"])

# Aggregation — average height of female characters
db.characters.aggregate([
    {"$match": {"gender": "female"}},
    {"$group": {"_id": "$gender", "avg_height": {"$avg": "$height"}}}
])
```

Run it with:

```bash
python mongodb/queries.py
```

---

# ☁️ Cloud Computing

> A comprehensive reference covering cloud fundamentals, deployment models, service types, major providers, market share, and certifications for data professionals.

---

## Table of Contents

1. [What is Cloud Computing?](#1-what-is-cloud-computing)
2. [What are Data Centres?](#2-what-are-data-centres)
3. [How Do You Know If Something is Running in the Cloud?](#3-how-do-you-know-if-something-is-running-in-the-cloud)
4. [Cloud Deployment Models — The Main Two](#4-cloud-deployment-models--the-main-two)
5. [Cloud Deployment Models — The Two More Complex](#5-cloud-deployment-models--the-two-more-complex)
6. [The Three Main Types of Cloud Services](#6-the-three-main-types-of-cloud-services)
7. [Advantages of Cloud Computing for a Business](#7-advantages-of-cloud-computing-for-a-business)
8. [Potential Disadvantages and Pitfalls](#8-potential-disadvantages-and-pitfalls)
9. [Cloud Market Share as of 2026](#9-cloud-market-share-as-of-2026)
10. [The Big 3 Cloud Providers — What They're Known For](#10-the-big-3-cloud-providers--what-theyre-known-for)
11. [What Do You Usually Pay For in the Cloud?](#11-what-do-you-usually-pay-for-in-the-cloud)
12. [Examples of Cloud Data Services](#12-examples-of-cloud-data-services)
13. [Cloud Certifications for Data Professionals](#13-cloud-certifications-for-data-professionals)

---

## 1. What is Cloud Computing?

Cloud computing is the on-demand delivery of computing resources — including servers, storage, databases, networking, software, and analytics — over the internet, typically on a pay-as-you-go basis.

Rather than owning and maintaining physical hardware and data centres, organisations and individuals can rent access to these resources from a cloud provider. Think of it like electricity from the national grid: you don't generate your own power, you simply draw from a shared resource and pay for what you use.

### Key Characteristics of Cloud Computing

| Characteristic | Description |
|---|---|
| **On-demand self-service** | Provision resources instantly without human interaction from the provider |
| **Broad network access** | Accessible from anywhere with an internet connection |
| **Resource pooling** | Shared infrastructure serves multiple users (multi-tenancy) |
| **Rapid elasticity** | Scale up or down quickly based on demand |
| **Measured service** | Usage is monitored, controlled, and billed transparently |

### A Brief History

Cloud computing didn't appear overnight. Mainframe time-sharing in the 1960s planted the conceptual seed. Amazon Web Services launched in 2006, widely considered the birth of modern cloud infrastructure. Since then, the market has grown from virtually nothing to an industry worth nearly **$1 trillion** globally by 2026.

> 💡 **In simple terms:** "The cloud" is not a mysterious digital sky. It is a network of physical servers sitting in warehouses around the world, accessed over the internet. When you save to Google Drive or use Netflix, you are using cloud computing.

---

## 2. What are Data Centres?

A **data centre** is a large physical facility that houses the computing infrastructure that powers cloud services. They contain thousands of servers, networking equipment, storage systems, and the power and cooling infrastructure to keep it all running 24/7/365.

### What's Inside a Data Centre?

- **Servers** — The physical machines that process and store data
- **Networking equipment** — Routers, switches, and cables connecting everything
- **Storage systems** — High-capacity hard drives and SSDs
- **Power systems** — Redundant power supplies, uninterruptible power supplies (UPS), and backup generators
- **Cooling systems** — Air conditioning, liquid cooling, and hot/cold aisle containment to prevent hardware from overheating
- **Security** — Physical security including biometric access, CCTV, and security personnel

### Scale of Hyperscale Data Centres

The world's largest cloud providers operate **hyperscale data centres** — facilities so large they can contain hundreds of thousands of servers. A single hyperscale data centre can consume as much electricity as a small town.

### 🎥 Real-World Data Centre Walkthrough

See inside a real professional data centre in this excellent guided tour:

**▶ [Data Center Tour & Technical Deep Dive — Power, Data and Cooling Infrastructure (YouTube)](https://www.youtube.com/watch?v=gsN_CJJDy_o)**

> Hosted by a guided tour of the Deft data centre, this video covers the power infrastructure, cooling systems, backup generators, and network redundancy that keep cloud services running around the clock.

---

## 3. How Do You Know If Something is Running in the Cloud?

This is a surprisingly nuanced question. There is no flashing banner that says "cloud". Instead, look for these indicators:

### Indicators That Something is Cloud-Based

**Accessibility**
- Accessible from any device with an internet connection
- No need to install software locally (or a thin client app is provided)
- Data persists and syncs across devices automatically

**Scalability signals**
- The service scales with usage (e.g. storage grows with no hardware purchases)
- You receive a bill based on consumption rather than a one-off hardware cost

**Infrastructure signals (technical)**
- The application runs on IP ranges or domains owned by AWS, Azure, or Google Cloud
- Tools like `nslookup` or `traceroute` resolve to cloud provider infrastructure
- URLs may include `amazonaws.com`, `azurewebsites.net`, `appspot.com`, etc.

**Operational signals**
- You are not responsible for patching the underlying OS or hardware
- The provider handles backups, availability, and disaster recovery
- Service Level Agreements (SLAs) promise uptime, not hardware ownership

### Common Examples in Daily Life

| Service | Cloud Provider |
|---|---|
| Gmail / Google Drive | Google Cloud |
| Microsoft 365 / OneDrive | Microsoft Azure |
| Netflix | AWS |
| Spotify | Google Cloud |
| Dropbox | AWS |
| Slack | AWS |

---

## 4. Cloud Deployment Models — The Main Two

A **cloud deployment model** describes *where* infrastructure lives, *who* owns it, and *who* can access it. There are four recognised models; the two most common are:

---

### 🌐 Public Cloud

**Definition:** Infrastructure owned and operated by a third-party provider and delivered over the internet to many customers who share the same underlying hardware.

**How it works:** You sign up with AWS, Azure, or Google Cloud, provision resources through a web console or API, and pay for what you use. The provider manages everything — the hardware, the data centres, the cooling, the networking.

**Characteristics:**
- Multi-tenant (resources shared between customers, but logically separated)
- No upfront capital expenditure
- Virtually unlimited scalability
- Provider responsible for hardware and physical security

**Best for:** Startups, web applications, development and testing environments, organisations without specialist IT teams.

**Examples:** AWS, Microsoft Azure, Google Cloud Platform

---

### 🏢 Private Cloud

**Definition:** Infrastructure dedicated exclusively to a single organisation, either hosted on-premises or by a third party, but not shared with anyone else.

**How it works:** The organisation builds and maintains its own cloud environment — usually using technologies like VMware, OpenStack, or a managed private cloud service — giving them full control over hardware, security, and configuration.

**Characteristics:**
- Single-tenant (resources dedicated to one organisation)
- Higher upfront and ongoing costs
- Maximum control and customisation
- Meets strict compliance and data sovereignty requirements

**Best for:** Banks, healthcare providers, government agencies, large enterprises with strict regulatory requirements.

**Examples:** On-premises VMware environments, Oracle Cloud at Customer, IBM Cloud Private

---

## 5. Cloud Deployment Models — The Two More Complex

---

### 🔀 Hybrid Cloud

**Definition:** A combination of public and private cloud environments, linked so that data and applications can move between them seamlessly.

**How it works:** An organisation runs sensitive workloads on a private cloud (or on-premises) while using public cloud for less sensitive tasks, burst capacity, or cost efficiency. The two environments are connected via secure links (VPN, dedicated connections like AWS Direct Connect or Azure ExpressRoute).

**Key use cases:**
- **Cloud bursting** — overflow to public cloud during peak demand
- **Gradual migration** — move workloads to the cloud incrementally
- **Data sovereignty** — keep regulated data on-premises while using cloud for analytics
- **Disaster recovery** — private cloud primary, public cloud as failover

**Challenges:** More complex to manage; requires consistent security policies across environments; data transfer latency.

**Best for:** Large enterprises undergoing cloud migration, e-commerce during seasonal spikes, financial services.

---

### 🌍 Multi-Cloud

**Definition:** The use of cloud services from two or more different cloud providers, often for different workloads or to avoid vendor lock-in.

**How it works:** An organisation might use AWS for core infrastructure, Google Cloud for BigQuery analytics, and Azure for Microsoft 365 integration — each provider selected for its particular strengths.

**Why organisations choose it:**
- **Avoid vendor lock-in** — not dependent on a single provider
- **Best of breed** — use each provider's strongest services
- **Resilience** — if one provider has an outage, workloads can shift
- **Cost optimisation** — leverage pricing competition between providers

**Challenges:** Increased operational complexity; requires skills across multiple platforms; data egress costs between clouds can be significant.

> 📌 **Note on terminology:** Hybrid and multi-cloud are often confused. *Hybrid* typically refers to mixing cloud with on-premises. *Multi-cloud* refers to using multiple public cloud providers.

---

## 6. The Three Main Types of Cloud Services

![IaaS PaaS SaaS Diagram](images/iaas-paas-saas.png)

Cloud services are categorised by how much the provider manages versus how much you manage. The three core models are:

---

### 🏗️ Infrastructure as a Service (IaaS)

**What you get:** Virtualised computing infrastructure — servers, storage, networking — delivered over the internet.

**What you manage:** Operating systems, middleware, runtime environments, applications, and data.

**What the provider manages:** Physical hardware, data centre, hypervisor.

**Analogy:** Renting land and building materials — you construct your own house.

**Examples:**
- **AWS EC2** — Virtual machines on demand
- **AWS S3** — Object storage
- **Azure Virtual Machines**
- **Google Compute Engine**

**Best for:** IT teams that need full control over their stack, organisations with unique security requirements, running custom or legacy software.

---

### 🛠️ Platform as a Service (PaaS)

**What you get:** A managed platform for building, testing, deploying, and running applications — without managing the underlying infrastructure.

**What you manage:** Applications and data.

**What the provider manages:** Everything below the application layer — OS, runtime, middleware, networking, servers.

**Analogy:** Renting a fully fitted kitchen — you cook the meal, you don't build the kitchen.

**Examples:**
- **AWS Elastic Beanstalk**
- **Azure App Service**
- **Google App Engine**
- **Heroku**

**Best for:** Development teams who want to focus on code, not infrastructure. Ideal for rapid application development.

---

### 💻 Software as a Service (SaaS)

**What you get:** A fully built, ready-to-use application delivered over the internet via a browser or thin client.

**What you manage:** Just your data and user settings.

**What the provider manages:** Everything — infrastructure, platform, application code, updates, security.

**Analogy:** Eating at a restaurant — you consume the meal, you don't cook or clean.

**Examples:**
- **Gmail / Google Workspace**
- **Microsoft 365**
- **Salesforce**
- **Slack**
- **Zoom**

**Best for:** End users and businesses that want to use software without managing it. The most widely used cloud model by volume.

---

### Responsibility Comparison

| Layer | IaaS | PaaS | SaaS |
|---|---|---|---|
| Applications | ✅ You | ✅ You | ☁️ Provider |
| Runtime | ✅ You | ☁️ Provider | ☁️ Provider |
| OS | ✅ You | ☁️ Provider | ☁️ Provider |
| Servers/Storage | ☁️ Provider | ☁️ Provider | ☁️ Provider |
| Data Centre | ☁️ Provider | ☁️ Provider | ☁️ Provider |

---

## 7. Advantages of Cloud Computing for a Business

### ✅ Cost Efficiency

Traditional IT requires significant capital expenditure (CapEx) — servers, hardware, data centre space, power, and the staff to maintain it. Cloud computing converts this to operational expenditure (OpEx), paying only for what you consume. There is no need to over-provision hardware "just in case".

### ✅ Scalability and Elasticity

Cloud infrastructure scales on demand. A retailer can handle Black Friday traffic spikes without owning hardware sized for peak usage year-round. Resources can scale up in minutes and scale back down to reduce cost.

### ✅ Speed and Agility

Development teams can provision servers, databases, and environments in minutes via a web console or API — a process that previously took weeks through physical procurement. This dramatically shortens time-to-market for products and features.

### ✅ Global Reach

Major cloud providers operate data centres across dozens of regions worldwide. A business can deploy applications closer to their users globally, reducing latency, without needing to physically build infrastructure in each country.

### ✅ Disaster Recovery and Business Continuity

Cloud providers offer built-in redundancy, geographic replication, and backup services. Achieving the same level of resilience with on-premises infrastructure is significantly more expensive and complex.

### ✅ Automatic Updates and Maintenance

The provider handles hardware maintenance, OS patching (for managed services), and security updates — freeing internal teams from routine maintenance work.

### ✅ Collaboration

Cloud-based tools (Google Workspace, Microsoft 365, GitHub) enable teams across the globe to collaborate on the same files, codebases, and projects in real time.

### ✅ Innovation at Lower Risk

The low barrier to entry allows businesses — particularly startups — to experiment with new technologies (AI, ML, IoT, big data) without large upfront investments.

---

## 8. Potential Disadvantages and Pitfalls

While cloud computing offers significant benefits, it is not without risk. Businesses should understand these pitfalls before committing.

### ⚠️ Security and Data Privacy

Storing data with a third-party provider introduces risks. Misconfigurations — such as an exposed S3 bucket — have been responsible for some of the largest data breaches in history. Businesses must understand their shared responsibility for securing cloud environments.

### ⚠️ Vendor Lock-In

Once an organisation builds heavily on one provider's proprietary services (AWS Lambda, Azure Cosmos DB, Google BigQuery), migrating to another provider becomes extremely costly and technically complex. This gives providers pricing power over time.

### ⚠️ Unpredictable Costs (Cloud Sprawl)

Pay-as-you-go pricing is a double-edged sword. Without governance and monitoring, cloud bills can spiral unexpectedly — a phenomenon known as *cloud sprawl*. Forgotten resources, over-provisioned instances, and data egress fees are common culprits.

### ⚠️ Downtime and Dependency on Internet Connectivity

Cloud services require internet access. If the provider experiences an outage (AWS, Azure, and Google Cloud have all had significant outages), services become unavailable. Organisations must plan for this dependency.

### ⚠️ Compliance and Data Sovereignty

Some industries (healthcare, finance, government) have strict regulatory requirements about where data is stored and processed. Navigating cloud compliance (GDPR, HIPAA, PCI DSS) requires careful planning and specialist expertise.

### ⚠️ Limited Control

In managed and SaaS services, the provider controls the underlying infrastructure. If they make changes to their platform, pricing, or service terms, businesses must adapt — sometimes with little notice.

### ⚠️ Skills Gap

Cloud platforms are complex. Moving to the cloud without the appropriate skills internally (or externally) leads to poor architecture, security vulnerabilities, and wasted spend.

---

## 9. Cloud Market Share as of 2026

![Cloud Market Share 2026](images/cloud-market-share.jpeg)

The cloud infrastructure market has grown at extraordinary pace. The global cloud market is valued at approximately **$917 billion in 2026**, with Synergy Research Group expecting it to cross **$1 trillion** before year-end.

### The Big Three Dominate

As of Q1 2026, three providers account for the overwhelming majority of the market:

| Provider | Market Share (Q1 2026) | YoY Revenue Growth |
|---|---|---|
| **Amazon Web Services (AWS)** | ~30% | ~19–20% |
| **Microsoft Azure** | ~21–25% | ~40% |
| **Google Cloud** | ~13–14% | ~28–63% |
| **Others (Alibaba, Oracle, IBM, etc.)** | ~32% | Varied |

> 📊 Source: Synergy Research Group Q4 2025 / Q1 2026 data. Note that exact figures vary between research firms depending on methodology and timing.

### Key 2026 Trends

- The **Big Three together command approximately 68%** of total enterprise cloud spending
- **AWS** remains the leader by revenue but is growing slowest — its market share is gradually eroding
- **Azure** is closing the gap rapidly, now serving **85% of Fortune 500 companies**, boosted by AI and Microsoft 365 integration
- **Google Cloud** is the fastest-growing in share terms, powered by AI workloads and BigQuery analytics
- **Multi-cloud adoption** has hit **89%** of enterprises — most large organisations now use more than one provider
- **AI-related cloud spending** accounts for **19% of total cloud spend** in 2026, up from just 8% in 2023

---

## 10. The Big 3 Cloud Providers — What They're Known For

### 🟠 Amazon Web Services (AWS)

**Parent company:** Amazon  
**Market share:** ~30% (Q1 2026)  
**Launched:** 2006

AWS is the pioneer of modern cloud computing and remains the market leader. It offers the broadest and deepest portfolio of cloud services — over **200 fully featured services** — across compute, storage, databases, machine learning, IoT, and more.

**Known for:**
- The largest ecosystem and marketplace of third-party integrations
- Best-in-class breadth of services — if it exists in cloud, AWS probably has it
- Strong startup and developer adoption (Netflix, Airbnb, Reddit all run on AWS)
- Global infrastructure spanning **38 geographic regions** and **120 availability zones**
- Deep expertise in serverless (AWS Lambda) and container workloads
- Strategic partnership with **Anthropic** for enterprise AI

**Key USPs:**
- First-mover advantage and 20 years of operational maturity
- Largest global network of availability zones — ideal for low-latency, globally distributed applications
- Most comprehensive AI/ML service catalogue (SageMaker, Bedrock, Trainium chips)
- AWS Marketplace: thousands of ready-to-deploy third-party software solutions

---

### 🔵 Microsoft Azure

**Parent company:** Microsoft  
**Market share:** ~21–25% (Q1 2026)  
**Launched:** 2010

Azure is the enterprise cloud platform of choice, deeply integrated with the Microsoft ecosystem that most large organisations already use. Its growth has been extraordinary — 40% year-over-year — driven by AI adoption and Microsoft 365 integration.

**Known for:**
- Best integration with Microsoft products (Windows Server, Active Directory, Office 365, Teams, Dynamics)
- Unparalleled enterprise credibility — preferred by regulated industries
- **Exclusive partnership with OpenAI**, making Azure the primary platform for GPT-model deployments
- Strong hybrid cloud story via **Azure Arc** and **Azure Stack**
- Dominant in industries with heavy Microsoft dependencies (financial services, government, healthcare)

**Key USPs:**
- Serving **85% of Fortune 500** companies
- Azure Active Directory (Entra ID) as the de-facto enterprise identity platform
- Best-in-class compliance portfolio — over 100 compliance certifications
- Native integration with GitHub, DevOps tooling, and Power Platform
- OpenAI models accessible natively across all Azure enterprise services

---

### 🔴 Google Cloud Platform (GCP)

**Parent company:** Alphabet (Google)  
**Market share:** ~13–14% (Q1 2026)  
**Launched:** 2011

Google Cloud is the data and AI specialist. While third in overall market share, it punches above its weight in analytics, machine learning, and Kubernetes — technologies Google invented or pioneered.

**Known for:**
- **BigQuery** — the industry-leading serverless data warehouse for analytics at petabyte scale
- The best AI/ML infrastructure (TensorFlow, Vertex AI, Tensor Processing Units)
- Invented **Kubernetes** — the world's dominant container orchestration platform
- Superior **networking infrastructure** built on Google's private fibre backbone
- Strong appeal to data engineers, data scientists, and ML practitioners
- Achieved **profitability for the first time in 2025** — a major strategic milestone

**Key USPs:**
- **Tensor Processing Units (TPUs)** — purpose-built AI chips that can significantly undercut GPU costs for large-scale AI training
- Best serverless analytics story with BigQuery (no infrastructure to manage)
- Google's global private network — one of the world's largest and lowest-latency
- Strong open-source commitment (Kubernetes, TensorFlow, Apache Beam)
- Most competitive pricing for AI workloads — generally 5–10% cheaper than AWS/Azure for comparable GPU/AI tasks

---

## 11. What Do You Usually Pay For in the Cloud?

Cloud pricing follows a **consumption-based model** — you pay for what you use, when you use it. The key billing dimensions are:

### Compute

- **Virtual machines / instances** — billed per hour or per second based on the size (vCPUs, RAM) and type (general purpose, compute-optimised, GPU)
- **Serverless functions** — billed per invocation and execution duration (e.g. AWS Lambda, Azure Functions, Google Cloud Functions)
- **Container services** — billed based on CPU and memory allocated to containers

### Storage

- **Object storage** — billed per GB stored per month (e.g. S3, Azure Blob, Google Cloud Storage)
- **Block storage** — persistent disks attached to VMs, billed per GB provisioned
- **Archive storage** — cheaper tiers for infrequently accessed data

### Data Transfer (Egress)

One of the most misunderstood costs. Data flowing **into** the cloud is typically free. Data flowing **out** (egress) is charged per GB and can be significant at scale. This is also a major driver of vendor lock-in.

### Databases

- Managed database services (RDS, Cloud SQL, Cosmos DB) are billed by compute size, storage, and I/O operations
- Data warehouse services (BigQuery, Redshift, Synapse) charge for storage and query compute separately

### Networking

- Load balancers, VPNs, dedicated connections (AWS Direct Connect, Azure ExpressRoute), and CDN usage all carry separate charges

### Managed Services and APIs

- AI/ML APIs (OpenAI via Azure, AWS Bedrock, Vertex AI) — billed per API call or per token processed
- Monitoring, logging, security services — billed by volume of data ingested or events processed

### 💡 Cost Management Best Practices

- **Right-size instances** — don't over-provision
- **Reserved instances / committed use discounts** — commit to 1–3 years for significant savings (40–70% off on-demand prices)
- **Spot / preemptible instances** — up to 90% cheaper for fault-tolerant workloads
- **Use cost management tools** — AWS Cost Explorer, Azure Cost Management, GCP Billing

---

## 12. Examples of Cloud Data Services

Data professionals interact with cloud platforms primarily through data services. Here are the most important categories and tools:

### 🗄️ Object / Data Lake Storage

The foundation for data lakes — highly scalable, cheap storage for raw and processed data in any format.

| Service | Provider |
|---|---|
| **Amazon S3** (Simple Storage Service) | AWS |
| **Azure Data Lake Storage Gen2** (ADLS) | Azure |
| **Google Cloud Storage** | GCP |

### 🏛️ Cloud Data Warehouses

Optimised for analytical queries on large structured datasets. The workhorses of modern BI and analytics.

| Service | Provider | Key Characteristic |
|---|---|---|
| **Amazon Redshift** | AWS | MPP cluster-based; deep AWS integration |
| **Azure Synapse Analytics** | Azure | Unified analytics + data warehouse platform |
| **Google BigQuery** | GCP | Fully serverless; pay-per-query; excellent for ad-hoc analysis |
| **Snowflake** | Multi-cloud | Separates compute and storage; runs on AWS, Azure, and GCP |
| **Databricks** | Multi-cloud | Lakehouse platform combining data lake + warehouse capabilities |

### 🔄 Data Integration and ETL/ELT

Services for moving and transforming data between systems.

| Service | Provider |
|---|---|
| **AWS Glue** | AWS |
| **Azure Data Factory** | Azure |
| **Google Cloud Dataflow** | GCP |
| **AWS Database Migration Service (DMS)** | AWS |

### 📡 Streaming and Real-Time Data

For processing data in motion — event streams, IoT telemetry, clickstreams.

| Service | Provider |
|---|---|
| **Amazon Kinesis** | AWS |
| **Azure Event Hubs** | Azure |
| **Google Cloud Pub/Sub** | GCP |
| **Apache Kafka (managed)** — Confluent Cloud | Multi-cloud |

### 🤖 AI/ML Platforms

For building, training, and deploying machine learning models.

| Service | Provider |
|---|---|
| **Amazon SageMaker** | AWS |
| **Azure Machine Learning** | Azure |
| **Google Vertex AI** | GCP |
| **Databricks MLflow** | Multi-cloud |

---

## 13. Cloud Certifications for Data Professionals

Cloud certifications validate your skills and are recognised globally by employers. As a data professional, targeting the right certifications can significantly boost career prospects and salary.

### 🟦 Foundational Certifications (Start Here)

If you are new to cloud, these entry-level certifications build core concepts and terminology. No hands-on experience required.

| Certification | Provider | Cost |
|---|---|---|
| **AWS Certified Cloud Practitioner (CLF-C02)** | AWS | ~$100 |
| **Microsoft Azure Fundamentals (AZ-900)** | Azure | ~$165 |
| **Google Cloud Digital Leader** | GCP | ~$200 |

---

### 🟨 Associate / Professional Level (Core Targets)

These are the most recognised and widely sought-after certifications in the industry.

| Certification | Provider | Focus |
|---|---|---|
| **AWS Certified Solutions Architect – Associate (SAA-C03)** | AWS | Designing distributed cloud systems |
| **AWS Certified Data Engineer – Associate (DEA-C01)** | AWS | Data pipelines, storage, transformation on AWS |
| **Azure Data Engineer Associate (DP-203)** | Azure | Building data solutions on Azure |
| **Azure Solutions Architect Expert (AZ-305)** | Azure | Designing enterprise Azure architectures |
| **Google Professional Data Engineer** | GCP | Designing data systems on Google Cloud |
| **Google Professional Cloud Architect** | GCP | Designing and managing GCP solutions |

---

### 🟥 Specialty Certifications (Advanced)

For professionals looking to specialise:

| Certification | Provider | Focus |
|---|---|---|
| **AWS Certified Machine Learning – Specialty** | AWS | ML workflows on AWS |
| **AWS Certified Data Analytics – Specialty** | AWS | Advanced analytics on AWS |
| **Azure AI Engineer Associate (AI-102)** | Azure | Building AI solutions on Azure |
| **Google Professional Machine Learning Engineer** | GCP | ML engineering on GCP |
| **SnowPro Core** | Snowflake | Snowflake data platform fundamentals |
| **Databricks Certified Associate Developer for Apache Spark** | Databricks | Spark and lakehouse development |

---

### 📋 Recommended Certification Path for Data Professionals

```
BEGINNER
    └── AWS Cloud Practitioner OR Azure AZ-900 OR Google Cloud Digital Leader
            ↓
INTERMEDIATE
    └── AWS Data Engineer Associate (DEA-C01)
        OR Azure Data Engineer Associate (DP-203)
        OR Google Professional Data Engineer
            ↓
ADVANCED
    └── AWS Machine Learning Specialty
        OR Google Professional ML Engineer
        + Snowflake SnowPro Core
        + Databricks Spark Developer
```

### 💡 Tips

- **AWS** certifications are recommended if you want the widest job market globally
- **Azure** certifications are ideal if your organisation or target employers are Microsoft-centric
- **GCP** certifications are particularly strong for data engineering and ML roles
- **Snowflake** and **Databricks** certifications are increasingly valuable as these platforms have become cloud-agnostic data standards
- Certifications typically require renewal every **2–3 years**
- Combine certification study with hands-on practice — all three providers offer free tiers

---

## Summary Reference Card

| Topic | Key Points |
|---|---|
| **Cloud definition** | On-demand computing over the internet, pay-as-you-go |
| **Data centres** | Physical facilities housing servers, storage, and networking |
| **Main deployment models** | Public (shared), Private (dedicated) |
| **Complex deployment models** | Hybrid (public + private), Multi-cloud (multiple providers) |
| **Service models** | IaaS (infrastructure), PaaS (platform), SaaS (software) |
| **Market leader** | AWS ~30%, Azure ~21–25%, GCP ~13–14% |
| **AWS known for** | Breadth of services, maturity, ecosystem |
| **Azure known for** | Enterprise integration, Microsoft ecosystem, OpenAI |
| **GCP known for** | Data/AI, BigQuery, Kubernetes, TPUs |
| **What you pay for** | Compute, storage, egress, databases, APIs |
| **Key data services** | S3, BigQuery, Redshift, Snowflake, Databricks, Synapse |
| **Entry cert** | Cloud Practitioner / AZ-900 / Digital Leader |
| **Data professional cert** | AWS DEA-C01, Azure DP-203, GCP Data Engineer |

---

*Last updated: June 2026 | Sources: Synergy Research Group, Gartner, IBM, AWS, Microsoft, Google Cloud official documentation*