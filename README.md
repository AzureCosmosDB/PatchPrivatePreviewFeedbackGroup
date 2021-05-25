# Partial Document Update Preview -  Docs and Feedback
This group will share the private preview documentation, issues for partial update feature for JSON patch for SQL API.

Please do provide us your feedback, questions or comments via "issues" section on this repo - 2nd tab on the top left here. 

## Feature Overview
The Partial Document update API feature is a top-level REST API to modify an item efficiently by sending only the modified properties/fields in a document from the client side as opposed to client performing a full document replace. 
## Main Benefits: 
* Reduced network call payload, avoiding whole document to be sent on the wire 
* Avoiding extra “READ “operation for OCC check by the client and hence saving on the extra read RU charges 
* Significant savings on end-to-end latency for a modifying the document 
* Avoid extra CPU cycles on client side to read doc, perform occ checks, locally patch document & then send it over the wire as replace API call 
* Multi Master conflict resolution to be transparent and automatic with path updates on discrete paths with the same document

## Supported Operations, Modes, APIs & SDKs :
#### Operations
- Add
- Remove
- Replace
- Set
- Increment as plus or minus “x”, i.e +/- X

#### Modes
- Conditional Patch based on a SQL like filter predicate
- Single item Patch Operation command
- Multiple items Patch Operation in using Bulk APIs
- Transactional Batch Patch Operation

#### SDKs
- .NET
- Java

## How to get started: 
Step 1: Whitelist your account : Fill the nomination form : https://aka.ms/cosmosdbpatch or email us cosmosdbpatchpreview@microsoft.com for specific clarifications.

Step2: Download the Nuget Package /  Maven package
- .NET Nuget Package : NuGet Gallery | Microsoft.Azure.Cosmos 3.18.0-preview or attached nuget file in this repo
- Java Maven Package: Maven Repository: com.azure » azure-cosmos » 4.15.0-beta.1(https://mvnrepository.com/artifact/com.azure/azure-cosmos/4.15.0-beta.1)

Step3: Send us your feedback, comments, questions using the Issues tab on this repo. 

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
#### Patching an document with multiple patch operations
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

#### Conditional Patch syntax based on filter predicate
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

#### Sample Transactional Patch for Batch Patch Operation
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

#### Patching an document with multiple patch operations
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

#### Sample of a conditional Patch syntax based on filter predicate
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

#### Sample Transactional Patch for Batch operations
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


## Frequently Asked Questions (FAQs)

#### Is this an implementation of JSON Patch RFC 6905 ?
Cosmos DB Patch Operation is inspired by JSON Patch and follows it closely. Currently we do not implement all features of JSON patch such as (Copy, Move) and we do implement few other features (Conditional Patch) which is not in the specification

#### Is Patch compatible with Serverless , provisioned throughput and autoscale modes of billing ?
Patch will be available across serverless and provisioned billing models. 

#### How is RU pricing calculated ?
There will not be significant reduction in RU. However there are other benefits as mentioned in the documents. We are also evaluating RU charging model as we plan for GA


#### Is there a limit to the number of Patch operations that can be done within a single Patch specification?
There is a limit of 10 patch operations that can be added in a single patch specification. If we need this number to be increased, please send us an email to cosmosdbpatchpreview@microsoft.com


#### Is Patch supported on local CosmosDB emulator?
Yes, please follow the below steps to enable it.
1) Download the latest CosmosDB emulator from https://aka.ms/cosmosdb-emulator
2) Open PowerShell in admin mode
3) cd 'C:\Program Files\Azure Cosmos DB Emulator'
4) C:\Program Files\Azure Cosmos DB Emulator> **.\CosmosDB.Emulator.exe /overrides='enableJsonPatch:true'**


