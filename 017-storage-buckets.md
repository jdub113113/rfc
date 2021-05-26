# Storage Buckets

- Implementation Owner: @torstendittmann
- Start Date: 26-05-2021
- Target Date: (expected date of completion, dd-mm-yyyy)
- Appwrite Issue: https://github.com/appwrite/appwrite/issues/363

## Summary

[summary]: #summary

<!-- Brief explanation of the proposed contribution. Write your answer below. -->
We are adding Buckets to our Storage service. These will allow users to create containers around multiple files. Buckets will allow users to enforce logic around Files. The mentioned logic involves custom permissions and settings.
## Problem Statement (Step 1)

[problem-statement]: #problem-statement

**What problem are you trying to solve?**

Developers want to organize the files and their storage and have dedicated destinations with different Policies for specific use-cases.

**What is the context or background in which this problem exists?**

Right now there is no way of configuring Policies or Permissions for multiple Files. This results into developers not being able to configure access to the Storage Service.

**Once the proposal is implemented, how will the system change?**

The Storage API is going to have additional routes - that will allow configuring Buckets for the Server Side and uploading files to specific Buckets on the Client Side.

## Design proposal (Step 2)

[design-proposal]: #design-proposal

<!--
This is the technical portion of the RFC. Explain the design in sufficient detail keeping in mind the following:

- Its interaction with other parts of the system is clear
- It is reasonably clear how the contribution would be implemented
- Dependencies on libraries, tools, projects or work that isn't yet complete
- New API routes that need to be created or modifications to the existing routes (if needed)
- Any breaking changes and ways in which we can ensure backward compatibility.
- Use Cases
- Goals
- Deliverables
- Changes to documentation
- Ways to scale the solution

Ensure that you include examples, code-snippets etc. to allow the community to understand the proposed solution. **It would be best if the examples use naming conventions that you intend to use during the actual implementation so that changes can be suggested early on during the development.**

Write your answer below.

-->
In the Storage Service, Buckets are going to act like a folder that will add logic to how the API responds. A Bucket has following configurations available:
- **Enabled**
  - This setting decides if a Bucket is enabled and is accessible.
- **Adapter**
  - This decides to what Storage Adapter the files will be uploaded to. Possible options are Local, S3, DigitaOcean Spaces, etc.
- **Maximum File Size Limit**
  - This setting decides the maximum single file size that can be uploaded.
- **Maximum Number of Files**
  - This setting decides how many files a Bucket can contain.
- **Accepted File Extensions**
  - This setting decides what file extensions can be uploaded.
- **Encryption**
  - This setting decides if files are encrypted. This will come into play, when encrypting large files causes too much load.
- **Virus Scan**
  - This setting decides if files are scanned by ClamAV after upload. Same reasoning ass **Encryption**.
- **TTL**
  - This setting decides how long a file is supposed to alive. Files are going to be deleted automatically.
- **Permission**
  - This setting decides what permission role is allowed to upload files to the bucket. Read and Write permissions will still exist for each file and be the source of permission check.

A Bucket will have following Model:
```typescript
{
  $id: string;
  $write: string[];
  name: string;
  enabled: boolean;
  adapter: 'local'|'s3'
  encrypted: boolean;
  antivirus: boolean;
  adapterConfig?: object;
  ttl?: number;
  maximumFiles?: number;
  maximumFileSize?: number;
  allowedFileExtensions?: string;
}
```

### New Endpoints

In this section I will explain all necessary endpoints for integratting Storge Buckets.

#### `GET: /v1/storage/bucket/`

This endpoint lists all Buckets. *Server*

Payload:
- **search = ''** Search term to filter your list results.
- **limit = 25** Maximum number of Buckets to return.
- **offset = 0** Offset value.

#### `GET: /v1/storage/bucket/{bucketId}`

This endpoint returns a single Bucket by its unique ID. *Server*

#### `GET: /v1/storage/bucket/{bucketId}/files`

This endpoint returns a list of all Files in a specific Bucket. *Server & Client*

Payload:
- **filters = []** Array of filter strings.
- **limit = 25** Maximum number of Buckets to return.
- **offset = 0** Offset value.
- **search = ''** Search query.

#### `POST: /v1/storage/bucket`

This endpoint creates a new Bucket. *Server*

Payload:
- **name** Bucket name.
- **write** An array of strings with write permissions.
- **encrypted = true** Enable Encryption.
- **antivirus = true** Enable Anti Virus.
- **adapter = 'local'** Adapter to use.
- **adapterConfig = null** Adapter Config.
- **ttl = null** TTL for files.
- **maximumFiles = null** Maximum amount of files.
- **maximumFileSize = null** Maximum file size.
- **allowedFileExtensions = null** Allowed File extensions.

#### `PUT: /v1/storage/bucket/{bucketId}`

This endpoint updates a Bucket by its unique ID. *Server*

Payload:
- **name = null** Bucket name.
- **write = null** An array of strings with write permissions.
- **encrypted = null** Enable Encryption.
- **antivirus = null** Enable Anti Virus.
- **adapter = null** Adapter to use.
- **adapterConfig = null** Adapter Config.
- **ttl = null** TTL for files.
- **maximumFiles = null** Maximum amount of files.
- **maximumFileSize = null** Maximum file size.
- **allowedFileExtensions = null** Allowed File extensions.

#### `DELETE: /v1/storage/bucket/{bucketId}`

This endpoint deletes a Bucket by its unique ID. *Server*

### Changes to existing Endpoints

Since the first release of Buckets will not introduce a breaking change - the Buckets will be implement aside the current Storage service. Meaning the Buckets will be an additional feature to the existing API.

#### `POST: /v1/storage/files`

This endpoint creates a new file. This endpoint will have a 4th optional argument added called `bucket`. 


### Prior art

[prior-art]: #prior-art

<!--

Discuss prior art, both the good and the bad, in relation to this proposal. A
few examples of what this can include are:

- Does this functionality exist in other software and what experience has their
  community had?
- For other teams: What lessons can we learn from what other communities have
  done here?
- Papers: Are there any published papers or great posts that discuss this? If
  you have some relevant papers to refer to, this can serve as a more detailed
  theoretical background.

This section is intended to encourage you as an author to think about the
lessons from other software, provide readers of your RFC with a fuller picture.
If there is no prior art, that is fine - your ideas are interesting to us
whether they are brand new or if it is an adaptation from other software.

Write your answer below.
-->

### Unresolved questions

[unresolved-questions]: #unresolved-questions

<!-- What parts of the design do you expect to resolve through the RFC process before this gets merged? -->

<!-- Write your answer below. -->

### Future possibilities

[future-possibilities]: #future-possibilities

<!-- This is also a good place to "dump ideas", if they are out of scope for the RFC you are writing but otherwise related. -->

<!-- Write your answer below. -->