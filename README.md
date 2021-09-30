# Partial Document Update Preview -  Docs and Feedback

This documentation is provided as a resource for participants in the private preview of Azure Cosmos DB partial document update.

Note: the partial document update preview is applicable to Core (SQL) API only. Users looking for partial updates in MongoDB API are advised to use the partial update operators (e.g. `$set`, `$inc`, etc) in `db.collection.update()` and `db.collection.findAndModify()`.

Please do provide us your feedback, questions or comments via the "issues" section on this repo (second tab on the top left)

## Feature Overview
Azure Cosmos DB partial document update is a top-level REST API to modify an item efficiently by sending only the modified properties/fields in a document from the client side as opposed to requiring the client to perform a full document replace. 

## Main Benefits: 
* Reduced network call payload, avoiding whole document to be sent on the wire 
* Avoiding extra “READ “operation for OCC check by the client and hence saving on the extra read RU charges
* Significant savings on end-to-end latency for a modifying the document
* Avoid extra CPU cycles on client side to read doc, perform occ checks, locally patch document & then send it over the wire as replace API call
* Multi-region write (formerly "multi-master") conflict resolution to be transparent and automatic with path updates on discrete paths with the same document

## Supported Operations, Modes, APIs & SDKs :
#### Operations
- Add
- Remove
- Replace
- Set
- Increment as plus or minus “x”, i.e +/- X

#### Modes
- Conditional patch based on a SQL-like filter predicate
- Single item patch operation command
- Multiple items patch operation in using bulk APIs
- Transactional batch patch operation

#### SDKs
- .NET
- Java
- NodeJS

## How to get started: 
Step 1: Whitelist your account by completing the nomination form : https://aka.ms/cosmos-partial-doc-update or emailing us cosmosdbpatchpreview@microsoft.com for specific clarifications.

Step2: Download the Nuget Package /  Maven package
- .NET Nuget Package : NuGet Gallery | Microsoft.Azure.Cosmos 3.18.0-preview or attached nuget file in this repo
- Java Maven Package: Maven Repository: com.azure » azure-cosmos » 4.15.0-beta.1(https://mvnrepository.com/artifact/com.azure/azure-cosmos/4.15.0-beta.1)

Step3: Send us your feedback, comments, questions using the Issues tab on this repo. 

## Note : 
During **preview** the JSON Patch feature can take 5-8 minutes to be enabled after Collection is created. 
This will not happen after GA as it would be enabled by default.
So, for preview, ensure that you are not using newly created Collections. 
If you do, you may experience error 500 "An unknown error occurred while processing this request. 

## Sample Code
### .NET : 
The full sample is found at [.NET v3 samples](https://github.com/Azure/azure-cosmos-dotnet-v3/blob/3fa885fdd84e2f8852d2a1d5c75c56b642b5bba3/Microsoft.Azure.Cosmos.Samples/Usage/ItemManagement/Program.cs)

#### Patching an item with a single patch operation: 
~~~~
//using PatchOperation command to Replace a property. The Patch is issued based on ID lookup for an id = 5, partition key value of pkey and setting TotalDues value to 0
Console.WriteLine("\n1.6 - Patching a item using its Id");
ItemResponse<item> response = await container.PatchItemAsync<item>(
                id: 5,
                partitionKey: new PartitionKey(“pkey”),
                patchOperations: new[] { PatchOperation.Replace("/TotalDue", 0) });
~~~~
#### Patching a document with multiple patch operations
~~~~
//using PatchOperation command to do multiple patches on a document. The Patch is issued based on ID lookup for an id = 5, partition key value of pkey.
Console.WriteLine("\n1.6 - Patching a item using its Id");

List<PatchOperation> patchOperations = new List<PatchOperation>();
patchOperations.Add(PatchOperation.Add("/nonExistentParent/Child", "bar"));
patchOperations.Add(PatchOperation.Remove("/cost"));
patchOperations.Add(PatchOperation.Increment("/taskNum", 6));
patchOperations.Add(patchOperation.Set("/existingPath/newproperty",value));

container.PatchItemAsync<item>(
                id: 5,
                partitionKey: new PartitionKey(“pkey”),
                patchOperations: patchOperations );
~~~~

#### Conditional patch syntax based on filter predicate
~~~~
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
~~~~

#### Sample transactional patch for batch patch operation
~~~~
List<PatchOperation> patchOperationsUpdateTaskNum12 = new List<PatchOperation>()
            {
                PatchOperation.Add("/children/1/pk", "patched"),
                PatchOperation.Remove("/description"),
                PatchOperation.Add("/taskNum", 8)
		PatchOperation.Replace("/taskNum", 12)
            };

TransactionalBatchPatchItemRequestOptions requestOptionsFalse = new TransactionalBatchPatchItemRequestOptions()
            {
                FilterPredicate = "from c where c.taskNum = 3"
            };

TransactionalBatchInternal transactionalBatchInternalFalse = (TransactionalBatchInternal)containerInternal.CreateTransactionalBatch(new Cosmos.PartitionKey(testItem.pk));
transactionalBatchInternalFalse.PatchItem(id: testItem1.id, patchOperationsUpdateTaskNum12, requestOptionsFalse);
transactionalBatchInternalFalse.PatchItem(id: testItem2.id, patchOperationsUpdateTaskNum12, requestOptionsFalse);
transactionalBatchInternalFalse.ExecuteAsync());
~~~~

### Java: 
The Maven package is found at [Maven Repository: com.azure » azure-cosmos » 4.15.0-beta.1 (mvnrepository.com)](https://mvnrepository.com/artifact/com.azure/azure-cosmos/4.15.0-beta.1)

#### Patching a document with multiple patch operations
~~~~
//using PatchOperation command to do multiple patches on a document. The Patch is issued based on ID lookup for an testitem.id, partition key value of testitem.status.
CosmosPatchOperations cosmosPatchOperations = CosmosPatchOperations.create();
cosmosPatchOperations.add("/children/1/CamelCase", "patched");
cosmosPatchOperations.remove("/description");
cosmosPatchOperations.replace("/taskNum", newTaskNum);
cosmosPatchOperations.set("/valid", false);
CosmosPatchItemRequestOptions options = new CosmosPatchItemRequestOptions();
CosmosItemResponse<ToDoActivity> response = this.container.patchItem(
	            testItem.id,
	            new PartitionKey(testItem.status),
	            cosmosPatchOperations,
	            options,
	            ToDoActivity.class);
~~~~

#### Sample of a conditional patch syntax based on filter predicate
~~~~
//using Conditional Patch options. The patch is issued based on a filter predicate condition; issued to id = testitem.id, partition key “testitem.status” 
CosmosPatchOperations cosmosPatchOperations = CosmosPatchOperations.create();
cosmosPatchOperations.add("/children/1/CamelCase", "patched");
cosmosPatchOperations.remove("/description");
cosmosPatchOperations.replace("/taskNum", newTaskNum);
cosmosPatchOperations.set("/valid", false);

CosmosPatchItemRequestOptions options = new CosmosPatchItemRequestOptions();
options.setFilterPredicate("from root where root.taskNum = " + conditionvalue);
CosmosItemResponse<ToDoActivity> responseFail = this.container.patchItem(
                testItem.id,
                new PartitionKey(testItem.status),
                cosmosPatchOperations,
                options,
                ToDoActivity.class);
~~~~

#### Sample transactional patch for batch operations
~~~~
CosmosPatchOperations cosmosPatchOperations = CosmosPatchOperations.create();
cosmosPatchOperations.set("/cost", testDoc.getCost() + 12);

//for conditional filter predicate - optional
TransactionalBatchPatchItemRequestOptions transactionalBatchPatchItemRequestOptionsTrue = new TransactionalBatchPatchItemRequestOptions();
transactionalBatchPatchItemRequestOptionsTrue.setFilterPredicate("from root where root.cost = " + costValue);

TransactionalBatch batch = TransactionalBatch.createTransactionalBatch(this.getPartitionKey(this.partitionKey1));
batch.createItemOperation(testDoc);
batch.patchItemOperation(testDoc.getId(), cosmosPatchOperations);
TransactionalBatchResponse batchResponse = container.executeTransactionalBatch(batch).block();
~~~~

### NodeJS:

The npm package can be found at [NPM package: com.azure » @azure/cosmos » 3.14.1 (npmjs.com)](https://www.npmjs.com/package/@azure/cosmos/v/3.14.1)


#### Patching an item with a single patch operation

```
const replaceOperation: PatchOperation[] =
[{
op: "replace",
path: "/lastName",
value: "Martin"
}];
const { resource: testItem1 } = await container.item(testItem.lastName).patch(replaceOperation);
```

#### Patching a document with multiple patch operations

```
const multipleOperations: PatchOperation[] = [
{
op: "add",
path: "/aka",
value: "MeFamily"
},
{
op: "replace",
path: "/lastName",
value: "Jose"
},
{
op: "remove",
path: "/parents"
},
{
op: "set",
path: "/address/zip",
value: 90211
},
{
op: "incr",
path: "/address/zip",
value: 5
}
];
const { resource: testItem2 } = await container.item(testItem.id).patch(multipleOperations);
```

#### Conditional patch syntax based on filter predicate

```
const operations : PatchOperation[] = [
{
op: "add",
path: "/newImproved",
value: "it works"
}
];
const condition = "from c where NOT IS_DEFINED(c.newImproved)";
const { resource: testitem3 } = await container.item(itestItem.id).patch({ condition, operations });
```

#### Sample transactional patch for bulk patch operation

```
const operations = [    
    {
      operationType: BulkOperationType.Patch,
      partitionKey: {},
      id: patchItemId,
      resourceBody: {
        operations: [{ op: PatchOperationType.set, path: "/class", value: "2021" }]
      }
    }
  ];
const response = await container.items.bulk(operations);
```

## Frequently Asked Questions (FAQs)

#### Is this an implementation of JSON Patch RFC 6902?
Azure Cosmos DB partial document update is inspired by JSON patch and follows it closely. Currently we do not implement all features of JSON patch such as (Copy, Move) and we do implement few other features (Conditional Patch) which is not in the specification

#### Is partial document update compatible with serverless , provisioned throughput and autoscale modes of billing?
Partial document update will be available across serverless and provisioned billing models. 

#### How is RU pricing calculated ?
Partial document update is normalized into request unit billing in the same style as other database operations. Users should not expect a significant reduction in RU.

#### What is the difference between operator Set and Replace?
For a detailed answer, refer to [this issue](https://github.com/AzureCosmosDB/PatchPrivatePreviewFeedbackGroup/issues/2)

#### Is there a limit to the number of partial document updates operations that can be done within a single patch specification?
There is a limit of 10 patch operations that can be added in a single patch specification. If we need this number to be increased, please send us an [email](cosmosdbpatchpreview@microsoft.com)

#### Is Patch supported for system properties?
We do not support Patch API on system properties. However, if you want it for some usecase, you can send an [email](cosmosdbpatchpreview@microsoft.com) and we can enable this for your account.

#### Is partial document update supported on local Azure CosmosDB emulator?
Yes, please follow the below steps to enable it.
1) Download the latest Azure CosmosDB emulator from https://aka.ms/cosmosdb-emulator
2) Open PowerShell in admin mode
3) cd 'C:\Program Files\Azure Cosmos DB Emulator'
4) C:\Program Files\Azure Cosmos DB Emulator> **.\CosmosDB.Emulator.exe /EnablePreview**


