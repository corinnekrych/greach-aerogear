This is not meant to be an exact transcript of the talk but rather a sort of guide.

# Starters

## Interactivity

There is a small web server currently running for this talk. If 3G gods are with us, during the presentation you can ask questions and rank them so that we will discuss some at the end. You can also rate the content of each slide in real time. Please open the link in your browser and enjoy.

For the record this server runs using Node.js and the webapp uses the small library we made ourselves and will present during this talk.

## Slides built with

For credits, those slides are based on Flowtime.js framework.

# Introduction

## Web application

Let's take an assumption: we are building a web application. Of course it may run on mobiles and tablets, but also on wide screen desktops, small screen laptops. Actually any device which is able to connect to the web. What if the device on the left was a smart TV?

## Client server

And the application model is mainly the one of the client/server. With so many clients that the server generally cannot track all of them individually. You have one central point to which many (billions?) of clients connect.

We do not assume that there is one physical server as the diagram may suggest, but there is one central accessible place where data is stored and a lot of clients which access it.

Algorithms we will discuss would also work in a P2P architecture but would be less suitable.

# Mostly offline

There is another important aspect easily seen for mobile applications: they go offline very often (willingly or by constraint). How many times a day do you go through an area not covered by 3G with your phone? And sometimes they go offline for a long time. This one is easily seen for desktop web applications: for how long do you let your Gmail tab closed before reopening it?

When disconnected, any other client may modify the state of the server. And we generally expect applications to work while offline, maybe in a degraded mode, and to modify its state too. It means that when the client comes back online it needs to synchronize its state.

## Synchronization

What do we mean by synchronization? We generally have two actors in different states, separated by a network. They of course do not know in which state is the other. If I am A, did B moved to another state? Did I personally moved? Synchronization is the process of converging both to a common one. We can also speak of reconciliation.

The talk will be about the different constraints that you have here and possible solutions depending on your needs to solve that use case: as a client, when I come back online, how to synchronize with the server? We will be speaking about general aproaches and about one library that we are building to solve one use case.

# Common

We will discuss three different solutions but let's first see requirements, how to compare the different solutions and the general algorithm.

## No(t only) push!

The first idea of most people think "let's just push updates to clients!" and they will get them by the time they reconnect. So why aren't we taking this and BOOM, done. Because this is unsuitable.

First push notifications like Google Cloud Messaging are unreliable when the client is offline. So your client may not receive some of them and miss some updates, that will never get repaired.

Then we just said that there may be so many clients the server cannot track all of them. And some clients may never come again to your server, so what would you do for them? Queue all notifications they should receive forever?

We do not say that push is not great. It is a great addition to make applications more responsive. But it is not enough to get your clients synchronized with the server, which by the way can be combine with push notifications.

## Supported operations

We want to synchronize the result of applying all kind of operations. Both making the server aware of what the client did while offline and fetching changes made by all other clients.

Creating a new item on a client must be visible on others. An item being modified must be visible on all clients. And it is important to also mention that deleting an item must be propagated to other clients. Said like that it seems trivial, but it is usually the one aspect that people forget about in the initial design, and then it may be impossible to integrate naturally.

## Selective data model

Another important feature we usually want is a selective data model. The whole dataset usually must not be synchronized to all clients. First because of access rights, I should not see Facebook private messages from my neighbor. But also because of the volume of data involved, one would not want to try synchronizing every single tweet ever sent on a poor iPhone over 3G. So each user may wants to synchronize a personal subset of the dataset.

## Conflicts

We assumed that applications could go offline, so it must be partition tolerant. And with that usually comes conflicts, two users doing a different edit to the same entity. With the model taken here, the conflict is detected when the second client tries to push its local edits. To us the solution is to refuse this push and require the client to do the merge. That does not necessarily mean that the actual user will do the merge, but that the application running on the client must be designed to do it. It is specific to your business case, no global choice can be made by any library.

The good news is that with the chosen model, conflicts only happen when pushing local changes. When pulling, all changes from the server should be blindly taken.

## Initiated by the client

There is one thing common to all: synchronization is initiated by the client. As a first step the client must be setup to record changes happening while offline. If you want to stick to asynchronous UIs you can even record changes and push to the server asynchronously even when online. Then upon synchronization, the client starts by pushing all edits to the server, and only then pulls changes made by other clients.

The reasons for doing that are

* the number of edits on the client will remain small and relatively easy to track, it concerns the action of only one user
* batch updates may be better handled by calling a specific endpoint on the server rather than sending individual modifications
* updates usually require some business rules to be validated or conflicts to be solved, that automatic synchronization could not handle

## Evaluation

How will we evaluate the different solutions we will talk about?

First and most obviously in terms of bandwidth. Of course since client and server are separated by a network, and not an in-datacenter one, this is an important criteria. If the data to synchronize weights 100Mb and that only one kb-sized items changed since the last synchronization one do not expect to need a big pack to be downloaded.

Second criteria is the number of roundtrips. Some algorithms will involve a complex set of requests to be made to the server and the bigger the number of roundtrips, the bigger synchronization duration will be.

We will also take into account the computational cost of synchronization. Does it need a lot of CPU on the client and/or on the server to work? For the client it is important depending on your target. For the server it may impact cost because of the number of CPUs you will need to provision. But the criteria also includes memory, manipulated data structures may consume a lot of RAM.

Recovery after computation errors and bugs is to be taken into account. Will a rogue-edit in your database be synchronized to clients? What happens in you had a bug in your client implementation and that he missed some data? Will the algorithm smoothly recover once you fix the bug or will you need to cleanup the client and restart from scratch? Those are important aspects, who never fought against an email desparately unread on your frontend while you marked it as read hundreds of times?

The last criteria is how easy is setup. Do you need to tie yourself with a complex data framework? Is it something which can be easily plugged into your existing architecture? How much should you tinker before getting something stable?

# Wholesale

## Algorithm

The algorithm is the simplest we could imagine: transfer the whole dataset to the client. Selective data model is also easy, just parameterize the database query on the endpoint and you have it.

That may seem like a horrible idea, but at least it works, no surprise! And I bet there are use cases for which it is perfectly valid. Do not optimize before you need to.

## Evaluation

Evaluation is pretty straigforward here. Bandwidth is wasted because non modified entities are nonetheless transferred. Only one roundtrip is needed. Errors, rogue edits, everything is resolved on the next synchronization. And the setup if of course very simple.

## Versioned entities

For many business cases one attaches a version identifier to entities, that is every time you modify something it gets a new version. All version are sometimes stored to keep track of what happens. That can be used to improve wholesale transfer by only sending all entity identities and latest versions. If one does not have an explicit attribute, consistent hashing of meaningful attributes can be used.

Notice that if you keep track of all versions it can be leveraged to let the application do three-way merges upon conflict.

## Rsync

Did you think this cannot work? It doesn't scale at all? Well this is the way Rsync works, so it may not be so dumb in the end.

In Rsync the client requests `MD5` hash of all files on the target, computes the same locally, and sends only different and missing files. Rsync makes other optimizations by taking chunks of files instead of whole files, and even being able to detect chunks for which offset changed.

## Versioned algorithm

So the same algorithm can be applied there. The only difference is that the client requests modified entities on the server rather than pushing them.

## Versioned evaluation

Bandwidth is a bit reduced compared to the other but one still needs to transmit a datastructure whose size is proportional to the number of items in the store, regardless the number of edits.

Roundtrips on the other hand increases a lot because the client must fetch modified entities one by one after having the list of changed ones. One could also use lazy loading in the client storage and actually request entity content only when needed.

Computational cost remains limited, error correction is preserved if one guaranties that there are no rogue edits to the database for which the version would not change. Setup remains very easy.

# Timestamps

## How to

The idea here is to record on the server the list of changes and associate them to edit timestamp in an ordered log. On the client on needs to store the last synchronization timestamp. Upon connection the client requests all modified entities since the last checkpoint and the last safe checkpoint.

## Version identifier

Even though we generally speak of numeric timestamps because it looks easy to deal with. What is important there is having:

* a version identifier which changes whenever anything in the dataset changes
* a way to retrieve operations between two versions

It may look strange but a version control system like Git and its JS implementation can be a solution for some (relatively rare) cases, namely if you need the full repository history to live on clients.

The main difference between timestamps and other solutions is that timestamps are ordered and if you index your database on them you can find all modifications within two instants in a very efficient manner. Git commits on the other hand require to look through parents of each commit until finding the last one the client knew about.

## Clocks

There is still an important thing to think about when using timestamps: they must come from a globally stricly increasing source.

And this is almost impossible to achieve if you are relying only on computer clock because clocks on different nodes are not in sync. One way to achieve it is given in [Google's Spanner](http://research.google.com/archive/spanner.html) paper. Only leader nodes can assign write timestamps, and they are allowed to do so only for a period of time explicitly allowed by all slaves. If the leader fails, slaves must wait until the lease period expires before electing a new leader. And to definitely be sure that the lease period expired, they had to plug GPS sensors on their computers and house atomic clocks in their datacenters.

Even if you have only one node, its clock may go backwards! For example when syncing with an NTP server and discovering its physical clock goes too fast.

So you need a timestamp oracle which gives you strictly increasing timestamps. We did not find any so if you know one please email it to us.

## Filters

The way to implement selective data model in this paradigm is using filters. When a client fetches edits since a checkpoint, one checks a filter function and only includes entities that fullfills a condition.

It works pretty well but has a few drawbacks:

* one needs to iterate on the whole log since the checkpoint, even though only a few items are to be included
* only soft deletes are permitted because one needs entity content in order to determine if it belongs to the view and if deletion information must be propagated to this particular client
* for the same reason data that is used in filters must be immutable, if it were not clients could download entities which are included in the view at some point and then never be notified from the server that they changed because they are no longer considered included

This is the kind of working that [CouchDB](http://wiki.apache.org/couchdb/Replication#Filtered_Replication) uses.

## Evaluation

## Couch/PouchDB

ID: An identifier
Sequence: An ID provided by the changes feed
Revision: Could be the version
Document A document is JSON entity with a unique ID and revision.
Database A collection of documents with a unique URI
URI An uri is defined by the 2396 . It can be an URL as defined in 1738.
Source Database from where the Documents are replicated
Target Database where the Document are replicated
Checkpoint Last source sequence ID

As you can see, this combines version and logs and Couch DB is a protocol whic works over http 1.1
You can add easyly filter by passing a filter parameter to the change feed, a function which return true if the change needs to be added to the change feed.
The main issue is that it synchronize  or replicates changes from one DB to another instead of a real selective model (Filter might help for this issue)

## Couch Pouch DB The algorithm itself
1. Assign an unique identifier to the source Database..

2. Save this identifier in a special Document named _local/<uniqueid> on the Target database.
This document isn’t replicated. It will collect the last Source sequence ID, the Checkpoint, from the previous replication process.

3. Get the Source changes feed by passing it the Checkpoint using the since parameter by calling the /<source>/_changes URL.
Feed can be longpoll or feed=continuous parameters. Then the feed will continuously get the changes.

4. Collect a group of Document/Revisions ID pairs from the changes feed and send them to the target databases on the /<target>/_revs_diffs URL. The result will contain the list of revisions NOT in the Target database.

5. GET each revisions from the source Database by calling the URL /<source>/<docid>?revs=true&rev=<revision> .
This will get the document with the parent revisions.

6. Collect a group of revisions fetched at previous step and store them on the target database
using the Bulk Docs API with the new_edit: false JSON property to preserve their revisions ID.

7. After the group of revision is stored on the Target Database, save the new Checkpoint on the Source database.


# Mathematical

The last aproach to synchronization is less direct but gives interesting results.

## Primitive

For that to work, we will first describe the data structure we need. There are several ways to implement it, we will see one later on.

The primitive is thus: given an integer `n` the server can build a data structure of size `O(n)`, that allows the client to compute it's delta from the server if the number of differences in this delta is less than `2^n`. A difference is adding, removing or modifying an item.

`n` is called the level from now on because it represents level of details.

## Algorithm

Given that structure, the algorithm is pretty simple as executed from the client. Remember that the client already pushed its changes and that it can blindly take edits from the server.

It starts by setting a counter `i` to 0. Then it fetches the previously mentioned data structure from the server at level `i = 0` and tries to compute the difference of the local store to the server using that structure. This may succeed or fail depending on the actual number of differences. If it succeeds, the difference is applied to the local store. If it fails, it increments the counter and retries with a larger level of details until it succeeds.

## Performances

With this solution, bandwidth is used very efficiently because data transferred is proportional the the changeset size, regardless how many items are to be synchronized. To be more precise it is proportional to the number of entities modified times the size of an entity. So entity choice may impact performances.

The solution leads to a few roundtrips, logarithmic growth in term of changed entities number is not that much, but still greater than other aproaches that did it in one shot.

So in terms of those it is damn efficient. Let's discuss other criterion.

## Mathsync

First: setup. Building the data structure may look hard right? Resolving it on the client may seem hard too? So we tried developing a library which does it for you. We are doing it on our spare time and really starting to scratch the surface, there remains a lot to be done he and you are welcome contributing! It will serve to present the actual data structure implementation which we took from a paper. It is linked from our project homepage if you want to read it.

## Mathsync data structure

The data structure is not that hard to build and can be built in linear time.

In the end it is an array of `n` buckets, a bucket being an object with three properties:

* a counter of the number of items stored in the bucket
* a byte array set to the XOR of all items in the bucket
* a byte array set to the XOR of the hash of items in the bucket

Technically, we use TypedArrays to implement byte arrays and transmit them in base64.

One start with an array of empty buckets, then for each item we serialize it as a bytes.

Out of those bytes it generates the list of buckets to store the item in, usually 3. It internally uses consistent hashing to get those buckets. Each target bucket is then updated accordingly by increasing the number of items, xoring the hash and the content.

To deserialize on the other side, first remove all the items you have locally. Then search for a bucket with +/-1 item and consider it as added/removed. Add or remove it in the structure and restart searching for singleton buckets. If there are no any, then retry with a larger level of details. XOR ensures that operations commute and get reversed.

## Mathsync server setup

Server setup only consists in three steps

* defining a `serializer` that converts items to TypedArrays, common ones are planned
* creating a `summarizer` object bound to an array of items, other more flexible strategies than a fixed arrays are planned
* exposing the result of this summarizer over an http entry point

We need a custom serializer because one needs to have consistent convertion to bytes.

## Mathsync client setup

Client setup requires the serializer and `summarizer` from the server plus

* a `deserializer` doing the opposite to the `serializer`
* another `summarizer` fetching server content
* a resolver orchestrating the use of both `summarizer`s

## Ahead of time computation

You can guess the cost of computing the data structure is pretty heavy. If there are tens of thousands of entities to fetch from the database for each user query then your infrastructure will have a hard time.

The good news if you suffer from this is that you can store a cache them. The best is to use an iterative map/reduce job, like views in CouchDB.

## Errors

The magical part of that aproach is that errors get corrected at no cost. If there is a bug in the implementation or that your cache entered a wrong state, for example an incorrectly applied iterative map/reduce job, the next time synchronization occurs all problems are gone.

In the timestamp aproach an error basically means that one needs to invalidate all client stores and make them download again the whole data they need. With the mathematical one at worse you need to clean intermediate caches. But clients only download actually different data.

## Selective data model

Selective data model fits naturally in this model because the endpoint can filter content it extracts from the database before compressing it. Just select all items that should be present on the application and compress it altogether.

The data structure is actually very flexible. Because it uses only XOR operations one can efficiently combine several structures together if they represent disjoint set of entities. Or do a substraction.

Say you have many users that belongs to a few groups, and that entities they can access depend on groups they belong to. One can store one structure for each group, which is not that many, and merge them together in a single one at runtime depending on the registered user.

## Evaluation

We already saw that network performances are pretty good.

Computational cost is pretty high because with a lot of binary operations. Since the data structure is map/reduce friendly it can be computed ahead of time but then we can count memory used to cache it as a computation cost.

Error correction is very interesting here, because even rogue edits to your database are taken into account. Except if you have caches or use map/reduce on top of the database, but then you just need to clean those.

Remains setup. I don't know what you think about the library. I would say the library makes it a bit easier. If you feel interested, give it a try and submit your feedback, good and bad. We are eager to change if the way we do things do not fit a real need.

# Conclusion

In conclusion the first thing we want to say is: start simple! For you prototype I am very sure that wholesale transfer is enough. So do it, validate that your app is interesting to people. Scaling comes later on, you will have solutions to do that if your design is clean with synchronization done in an abstracted service aproach. Please do not tie your implementation with a given library or framework.

Then if you can obtain a reliable edit log our of your database, make sure you use only its time and that its clock is actually monotonic, then use the log aproach. It allows you to scale with a simple setup.

But if you can't prove that your log will be correct and not miss anything, because it is a hard assumption, then move to the mathematical aproach. Mathematics never lie. Another reason is if you need greater flexibility on what to synchronize. But keep in mind that the setup is a bit larger. If you decide to use this aproach and find our library convenient, use it and contribute to it! There is room for improvements, we are just scratching the surface.
