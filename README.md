# MongoDbGenericRepository
An example of generic repository implementation using the MongoDB C# Sharp 2.0 driver (async)

Now available as a nuget package:
https://www.nuget.org/packages/MongoDbGenericRepository/

# Usage examples

This repository is meant to be inherited from. 

You are responsible for managing its lifetime, it is advised to setup this repository as a singleton.

Here is an example of repository usage, where the TestRepository is implementing 2 custom methods:

```
    public interface ITestRepository : IBaseMongoRepository
    {
        void DropTestCollection<TDocument>();
        void DropTestCollection<TDocument>(string partitionKey);
    }
    
    public class TestRepository : BaseMongoRepository, ITestRepository
    {
        public TestRepository(string connectionString, string databaseName) : base(connectionString, databaseName)
        {
        }

        public void DropTestCollection<TDocument>()
        {
            _mongoDbContext.DropCollection<TDocument>();
        }

        public void DropTestCollection<TDocument>(string partitionKey)
        {
            _mongoDbContext.DropCollection<TDocument>(partitionKey);
        }
    }
```

The repository can be instantiated like so:

```
ITestRepository testRepository = new TestRepository(connectionString, "MongoDbTests");
```

To add a document, its class must inherit from the `Document` class or implement the `IDocument` interface:

```
    public class MyDocument : Document
    {
        public ReadTestsDocument()
        {
            Version = 2; // you can bump the version of the document schema if you change it over time
        }
        public string SomeContent { get; set; }
    }
```

The `IDocument` interface can be seen below:

```
    /// <summary>
    /// This class represents a basic document that can be stored in MongoDb.
    /// Your document must implement this class in order for the MongoDbRepository to handle them.
    /// </summary>
    public interface IDocument
    {
        DateTime AddedAtUtc { get; set; }
        Guid Id { get; set; }
        int Version { get; set; }
    }
```
This repository also allows you to partition your document accross multiple collections, this can be useful if you are running a SaaS application and want to keep good performance.

To use partitioned collections, you must define your documents using the PartitionedDocument class, which implements the IPartitionedDocument interface:
```
    public class MyPartitionedDocument : PartitionedDocument
    {
        public CreateTestsPartitionedDocument(string myPartitionKey) : base(myPartitionKey)
        {
            Version = 1;
        }
        public string SomeContent { get; set; }
    }
```

This partitioned key will be used as a prefix to your collection name.
The collection name is derived from the name of the type of your document, is set to lower case, and is currently very naively pluralized (a "s" is added at the end of the type name).

```
var myDoc = new MyPartitionedDocument("myPartitionKey");
_testRepository.AddOne(myDoc);
```

The above code will generate a collection named `myPartitionKey-mypartitioneddocuments`.

Please refer to the IntegrationTests project for more usage examples.

## Copyright
Copyright © 2017

## License
mongodb-generic-repository is under MIT license - http://www.opensource.org/licenses/mit-license.php

## Author
**Alexandre Spieser**

## Donations
Feeling like my work is worth a coffee? 
Donations are welcome and will go towards further development of this project as well as other MongoDb related projects. Use the wallet address below to donate.
BTC Donations: 1Qc5ZpNA7g66KEEMcz7MXxwNyyoRyKJJZ

*Thank you for your support and generosity!*

