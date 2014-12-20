contentful-management.java
==========================

[![Build Status](https://travis-ci.org/contentful/contentful-management.java.svg)](https://travis-ci.org/contentful/contentful-management.java/builds#) [![Coverage Status](https://img.shields.io/coveralls/contentful/contentful-management.java.svg)](https://coveralls.io/r/contentful/contentful-management.java?branch=master)

Java SDK for [Contentful's][1] Content Management API.

[Contentful][1] is a content management platform for web applications, mobile apps and connected devices. It allows you to create, edit & manage content in the cloud and publish it anywhere via powerful API. Contentful offers tools for managing editorial teams and enabling cooperation between organizations.

Setup
=====

Grab via Maven:
```xml
<dependency>
  <groupId>com.contentful.java</groupId>
  <artifactId>cma-sdk</artifactId>
  <version>0.9.4</version>
</dependency>
```
or Gradle:
```groovy
compile 'com.contentful.java:cma-sdk:0.9.4'
```

The SDK requires at minimum Java 6 or Android 2.3.

### Default Client

The SDK uses [Retrofit][2] under the hood as a REST client, which detects [OkHttp][3] in your classpath and uses it if it's available, otherwise falls back to the default `HttpURLConnection`.

The recommended approach would be to add [OkHttp][3] as a dependency to your project, but that is completely optional.

You can also specify a custom client to be used, refer to the [official documentation][4] for instructions.

### ProGuard

```
-keepattributes Signature
-dontwarn rx.**
-dontwarn retrofit.**
-keep class retrofit.** { *; }
-keep class com.contentful.java.cma.** { *; }
-keep class com.google.gson.** { *; }
-keep class sun.misc.Unsafe { *; }
```

Usage
=====

### Client

In the beginning the API client instance should be created:

```java
CMAClient client = new CMAClient.Builder()
    .setAccessToken("access-token-goes-here")
    .build();
```

The access token can easily be obtained through the [management API documentation](https://www.contentful.com/developers/documentation/content-management-api/#getting-started).

A client can perform various operations on different types of resources (Assets, Content Types, Entries and Spaces). Every type of resource is represented by a module in the `CMAClient` class, for example:

```java
client.assets() // returns the Assets module
client.contentTypes() // returns the Content Types module
client.entries() // returns the Entries module
client.spaces() // returns the Spaces module
```

Each module contains a set of methods which can be used to perform various operations on the specified resource type. Every method has a corresponding asynchronous extension which can be accessed through the `async()` method of the module, for example:

Retrieving all spaces (synchronously):

```java
try {
  CMAArray<CMASpace> result = client.spaces().fetchAll();
  // success
} catch (RetrofitError e) {
  // handle error
}
```

or asynchronously:

```java
client.spaces().async().fetchAll(new CMACallback<CMAArray<CMASpace>>() {
  @Override protected void onSuccess(CMAArray<CMASpace> result) {
    // success
  }

  @Override protected void onFailure(RetrofitError retrofitError) {
    // failure
  }
});
```

Please note that the default `CMACallback` has an empty `onFailure()` implementation, which makes it optional to override it. All examples below will be provided in the asynchronous version and without the `onFailure()` override for brevity.

When creating resources, it is possible to specify an ID for the resource. In case no ID is specified one will be automatically generated by the API.

### Assets

Retrieving all assets from a space:

```java
client.assets().async().fetchAll("space-id", new CMACallback<CMAArray<CMAAsset>>() {
  @Override protected void onSuccess(CMAArray<CMAAsset> result) {
    // ...
  }
});
```

Retrieving an asset by ID:

```java
client.assets().async().fetchOne("space-id", "asset-id", new CMACallback<CMAAsset>() {
  @Override protected void onSuccess(CMAAsset result) {
    // ...
  }
});
```

Creating an asset:

```java
client.assets().async().create("space-id",
    new CMAAsset().setField("title", "cool asset", "en-US"),
    new CMACallback<CMAAsset>() {
      @Override protected void onSuccess(CMAAsset result) {
        // ...
      }
    });
```

Updating an asset:

```java
asset.setField("title", "new title", "en-US");

client.assets().async().update(asset, new CMACallback<CMAAsset>() {
  @Override protected void onSuccess(CMAAsset result) {
   // ...
  }
});
```

Deleting an asset:

```java
client.assets().async().delete("space-id", "asset-id", new CMACallback<Response>() {
  @Override protected void onSuccess(Response result) {
    // ...
  }
});
```

Archiving or unarchiving an asset:

```java
client.assets().async().archive(asset, new CMACallback<CMAAsset>() {
  @Override protected void onSuccess(CMAAsset result) {
    // ...
  }
});

client.assets().async().unArchive(asset, new CMACallback<CMAAsset>() {
  @Override protected void onSuccess(CMAAsset result) {
    // ...
  }
});
```

Checking if an asset is archived:

```java
asset.isArchived();
```

Publishing or unpublishing an asset:

```java
client.assets().async().publish(asset, new CMACallback<CMAAsset>() {
  @Override protected void onSuccess(CMAAsset result) {
    // ...
  }
});

client.assets().async().unPublish(asset, new CMACallback<CMAAsset>() {
  @Override protected void onSuccess(CMAAsset result) {
    // ...
  }
});
```

Checking if an asset is published:

```java
asset.isPublished();
```

### Content Types

Retrieving all content types from a space:

```java
client.contentTypes().async().fetchAll("space-id", new CMACallback<CMAArray<CMAContentType>>() {
  @Override protected void onSuccess(CMAArray<CMAContentType> result) {
    // ...
  }
});
```

Retrieving one content type by ID from a space:

```java
client.contentTypes()
    .async()
    .fetchOne("space-id", "content-type-id", new CMACallback<CMAContentType>() {
          @Override protected void onSuccess(CMAContentType result) {
            // ...
          }
        });
```

Creating a field for a content type:

```java
contentType.addField(
    new CMAField().setId("field-id")
        .setName("field-name")
        .setType(CMAFieldType.Text));
```

Deleting a field from the content type:

```java
contentType.deleteField("field-id");
```

Creating a content type:

```java
client.contentTypes()
    .async()
    .create("space-id", 
        new CMAContentType().setName("content-type-name")
            .addField(new CMAField().setId("id1").setName("name1").setType(CMAFieldType.Text))
            .addField(new CMAField().setId("id2").setName("name2").setType(CMAFieldType.Number)),
        new CMACallback<CMAContentType>() {
          @Override protected void onSuccess(CMAContentType result) {
            // ...
          }
        });
```

Deleting a content type:

```java
client.contentTypes()
    .async()
    .delete("space-id", "content-type-id", new CMACallback<Response>() {
      @Override protected void onSuccess(Response result) {
        // ...
      }
    });
```

Activating or deactivating a content type:

```java
client.contentTypes()
    .async()
    .publish(contentType, new CMACallback<CMAContentType>() {
      @Override protected void onSuccess(CMAContentType result) {
        // ...
      }
    });

client.contentTypes()
    .async()
    .unPublish(contentType, new CMACallback<CMAContentType>() {
      @Override protected void onSuccess(CMAContentType result) {
        // ...
      }
    });
```

Checking if a content type is active:

```java
contentType.isPublished();
```

Updating a content type:

```java
contentType.setName("new-name");

client.contentTypes()
    .async()
    .update(contentType, new CMACallback<CMAContentType>() {
      @Override protected void onSuccess(CMAContentType result) {
        // ...
      }
    });
```

### Entries

Retrieving all entries from a space:

```java
client.entries().async().fetchAll("space-id", new CMACallback<CMAArray<CMAEntry>>() {
  @Override protected void onSuccess(CMAArray<CMAEntry> result) {
    // ...
  }
});
```

Retrieving an entry by ID:

```java
client.entries().async().fetchOne("space-id", "entry-id", new CMACallback<CMAEntry>() {
  @Override protected void onSuccess(CMAEntry result) {
    // ...
  }
});
```

Creating an entry:

```java
client.entries().async().create("space-id", "content-type-id", 
    new CMAEntry().setField("title", "Mr. President", "en-US"),
    new CMACallback<CMAEntry>() {
      @Override protected void onSuccess(CMAEntry result) {
        // ...
      }
    });
```

Updating an entry:

```java
client.entries().async().update(entry, new CMACallback<CMAEntry>() {
  @Override protected void onSuccess(CMAEntry result) {
    // ...
  }
});
```

Deleting an entry:

```java
client.entries().async().delete("space-id", "entry-id", new CMACallback<Response>() {
  @Override protected void onSuccess(Response result) {
    // ...
  }
});
```

Archiving or unarchiving the entry:

```java
client.entries().async().archive(entry, new CMACallback<CMAEntry>() {
  @Override protected void onSuccess(CMAEntry result) {
    // ...
  }
});

client.entries().async().unArchive(entry, new CMACallback<CMAEntry>() {
  @Override protected void onSuccess(CMAEntry result) {
    // ...
  }
});
```

Checking if the entry is archived:

```java
entry.isArchived();
```

Publishing or unpublishing the entry:

```java
client.entries().async().publish(entry, new CMACallback<CMAEntry>() {
  @Override protected void onSuccess(CMAEntry result) {
    // ...
  }
});

client.entries().async().unPublish(entry, new CMACallback<CMAEntry>() {
  @Override protected void onSuccess(CMAEntry result) {
    // ...
  }
});
```

Checking if the entry is published:

```java
entry.isPublished();
```

### Spaces

Retrieving all spaces:

```java
client.spaces().async().fetchAll(new CMACallback<CMAArray<CMASpace>>() {
  @Override protected void onSuccess(CMAArray<CMASpace> result) {
    // ...
  }
});
```

Retrieving one space by ID:

```java
client.spaces().async().fetchOne("space-id", new CMACallback<CMASpace>() {
  @Override protected void onSuccess(CMASpace result) {
    // ...
  }
});
```

Deleting a space:

```java
client.spaces().async().delete("space-id", new CMACallback<Response>() {
  @Override protected void onSuccess(Response result) {
    // ...
  }
});
```

Creating a space:

```java
client.spaces().async().create("space-name", new CMACallback<CMASpace>() {
  @Override protected void onSuccess(CMASpace result) {
    // ...
  }
});
```

or in the context of the organization (if you have multiple organizations within your account):

```java
client.spaces()
    .async()
    .create("space-name", "organization-id", new CMACallback<CMASpace>() {
      @Override protected void onSuccess(CMASpace result) {
        // ...
      }
    });
```

Updating a space:

```java
space.setName("new-name");

client.spaces().async().update(space, new CMACallback<CMASpace>() {
  @Override protected void onSuccess(CMASpace result) {
    // ...
  }
});
```


License
=======

Copyright (c) 2014 Contentful GmbH. See [LICENSE.txt][5] for further details.


 [1]: https://www.contentful.com
 [2]: https://square.github.io/retrofit
 [3]: https://square.github.io/okhttp
 [4]: https://contentful.github.io/contentful-management.java
 [5]: LICENSE.txt
