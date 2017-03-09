---
title: WCF service using Cassandra 
layout: post
published: true
category: programming
tags: [wcf, cassandra, iis, C#]
comments: true
---

This is a brief note on how to use Apache Cassandra's user-defined types with C# WCF rest service

### Steps

 * Create user-defined type(Udt)
 * Create a mapper object for Udt in C#
 * Register C# Udt object
 * Connect to Cassandra cluster and create session
 * Define WCF DataContract and OperationContract
 * Host in IIS

#### 1. Create User Defined Type(Udt)

Create a user-defined type for  <u>address, fullname & users</u> as mentioned in [datastax tutorial for udt](https://docs.datastax.com/en/cql/3.1/cql/cql_using/cqlUseUDT.html){:target="_blank"}

#### 2. Create a mapper object for Udt in 'C#'

Create a [cqlpoco](https://github.com/LukeTillman/cqlpoco){:target="_blank"} in our case that will also be a Data Contract.

*class for `address`*

``` csharp
    public class Address
    {
        public string Street { get; set; }
        public string City { get; set; }
        public int ZipCode { get; set; }
        public IEnumerable<string> Phones { get; set; }
    }
```

*class for `fullname`*

``` csharp
public class FullName
 {
     public string FirstName { get; set; }
     public string LastName { get; set; }
 }
```

*finally, a class which represent a row in cassandra `users` table*

``` csharp
    public class User
    {
        public Guid Id { get; set; }
        public IDictionary<string, Address> Addresses { get; set; }
        public IEnumerable<FullName> DirectReport { get; set; }
        public FullName Name { get; set; }
    }
```

for other data type mapping refer [CQL data types to C# types](http://datastax.github.io/csharp-driver/features/datatypes/){:target="_blank"}

#### 3. Register C# Udt object

Configure poco object to the Apache Cassandra driver using [Mapper Component](http://datastax.github.io/csharp-driver/features/components/mapper/#configuring-mappings){:target="_blank"}.

*Map user-defined type (FullName and Address)*

``` csharp
session.UserDefinedTypes.Define(
          UdtMap.For<FullName>(),
          UdtMap.For<Address>()
             .Map(a => a.Street, "street")
             .Map(a => a.City, "city")
             .Map(a => a.ZipCode, "zip_code")
             .Map(a => a.Phones, "phones")
      );
```

In most of the cases, mapper will do the mapping for simple type like `Fullname` but a better controlled mapping is also possible like `Address` where we map field by field.

*Map `User` class representing a row in a table*

``` csharp
   Cassandra.Mapping.MappingConfiguration.Global.Define(new Map<User>().TableName("users")
                .Column(c => c.Id, cm => cm.WithName("id"))
                .Column(c => c.Addresses, cm => cm.WithName("addresses").WithFrozenKey())
                .Column(c => c.DirectReport, cm => cm.WithName("direct_reports").WithFrozenValue()));
```

#### 4. Connect to Cassandra cluster and create session

We are done with the cassandra and C# setup, now connect to cluster and use.

``` csharp
          cluster = Cluster.Builder().AddContactPoint("127.0.0.1").Build();
          session = cluster.Connect("mykeyspace");

          var users = new Table<User>(session);
          var user = users.
              Where(u => u.Id == Guid.Parse(guid))
              .FirstOrDefault()
              .Execute();

```

#### 5. Setting up a WCF 

It is a trivial task, decorate poco we just created with the DataMember attribute, declare OperationContract and provide a definition. Register mapping (step 3) preferably with a memoization library. I leveraged service contructor to initialise the mapping.

``` csharp
    public class UserService : IUserService
    {
        static UserServiceLibrary.Data.DataService dataService;
        static UserService()
        {
            dataService = new UserServiceLibrary.Data.DataService();
        }
    }    
``` 

#### 6. Hosting WCF service

*WCF Rest service can be either self hosted using `WebServiceHost`*

``` csharp
	WebServiceHost webhost =  new WebServiceHost(typeof(UserService));
            try
            {
                webhost.Open();
                Console.ReadLine();
                webhost.Close();

            }
            catch (Exception e)
            {
                Console.WriteLine(e);
                webhost.Abort();
            }
        }	
```
*or host it in IIS to take the advantage of on-demand loading and application pool using*

	<%@ ServiceHost Service="WcfWebService.UserService" 
    Factory="System.ServiceModel.Activation.WebServiceHostFactory"%>

---

:point_right: Complete source code for the above demo can be found at [github](https://github.com/VimleshS/wcf_on_iis_with_cassandra){:target="_blank"}

---