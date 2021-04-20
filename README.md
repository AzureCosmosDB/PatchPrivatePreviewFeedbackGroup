# PatchPrivatePreviewFeedbackGroup
This group will share the private preview documentation, issues for partial update feature. 
## Feature Overview
The Patch API feature is a top-level REST API to modify an item efficiently by sending only the modified properties/fields in a document from the client side as opposed to client performing a full document replace. 
## Main Benefits: 
* Reduced network call payload, avoiding whole document to be sent on the wire 
* Avoiding extra “READ “operation for OCC check by the client and hence saving on the extra read RU charges 
* Significant savings on end-to-end latency for a modifying the document 
* Avoid extra CPU cycles on client side to read doc, perform occ checks, locally patch document & then send it over the wire as replace API call 
* Multi Master conflict resolution to be transparent and automatic with path updates on discrete paths with the same document

## How to get started: 
Step 1: Whitelist your account : Email us cosmosdbpatchpreview@microsoft.com  with your “Cosmos DB” account name and subscription ID

Step2: Download the Nuget Package /  Maven package
-	.NET Nuget Package : NuGet Gallery | Microsoft.Azure.Cosmos 3.18.0-preview or attached nuget file in this repo
-	Java Maven Package: 

## Sample Code
### .NET : The full sample is found at xxxxxx 
~~~
//using PatchOperation command to Replace a property. The Patch is issued based on ID lookup for an id = 5, partition key value of pkey and setting TotalDues value to 0
Console.WriteLine("\n1.6 - Patching a item using its Id");

ItemResponse<SalesOrder> response = await container.PatchItemAsync<SalesOrder>(
                id: 5,
                partitionKey: new PartitionKey(“pkey”),
                patchOperations: new[] { PatchOperation.Replace("/TotalDue", 0) });


//using Conditional Patch Operation command to replace a property. The patch is issued based on a filter predicate condition; issued to id =5, partition key “pkey” and replaing the Shipped Date to current Datetime 
PatchItemRequestOptions patchItemRequestOptions = new PatchItemRequestOptions
            {
                FilterPredicate = "from c where (c.TotalDue = 0 OR NOT IS_DEFINED(c.TotalDue))"
            };
response = await container.PatchItemAsync<SalesOrder>(
               id: 5,
               partitionKey: new PartitionKey(“pkey”),
                patchOperations: new[] { PatchOperation.Replace("/ShippedDate",  DateTime.UtcNow) },
                patchItemRequestOptions);
  ~~~
### Java: The full sample is found at xxxxx

### JSON patch format is 
~~~
The JSON Patch API format is.  
PATCH /dbs/{db}/colls/{coll}/documents/{doc}
HTTP/1.1
Content-Type:application/json-patch+json
[
	{op=”set”,path=”/name/first”, value=”Bob”},
	{op=”set”, path=”/address/city”,value=”Jamaica”}
]
~~~

### Supported Operations:
- Add
- Remove
- Replace
- Set
- Increment as plus or minus “x”, i.e +/- X
- Conditional 

### Supported Modes:
- PatchOperation command
- Bulk
- Batch


## Frequently Asked Questions (FAQs)
#### Which APIs are supported ?
Current we support SQL API and Cassandra API

#### Is this an implementation of JSON Patch RFC 6905 ?
Cosmos DB Patch Operation is inspired by JSON Patch and follows it closely. Currently we do not implement all features of JSON patch such as (Copy, Move) and we do implement few other features (Conditional Patch) which is not in the specification

#### Is Patch compatible with Serverless , provided throughput and autoscale modes of billing ?

#### How is RU pricing calculated ?





