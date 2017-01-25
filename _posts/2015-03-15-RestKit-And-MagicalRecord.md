---
layout: post
title: RestKit and MagicalRecord
---

Recently, I worked on a project where the app had to download a lot of JSON data and store the results in Core Data.

I decided this was a good time to try out some frameworks, so I installed the [pods](https://cocoapods.org/) for [RestKit](https://github.com/RestKit/RestKit) and [MagicalRecord](https://github.com/magicalpanda/MagicalRecord).  

## Trying RestKit

With RestKit, you make a mapping, like:

```objectivec
@{@“job_name”:@“jobName”}
```

Then you pass that on to the framework, along with the URL and other info, and it automatically downloads, parses, and saves that data into Core Data.


## Trying MagicalRecord

MagicalRecord also has convenience methods for importing JSON into Core Data.  Just follow the instructions in their [documentation](https://github.com/magicalpanda/MagicalRecord/blob/master/Docs/Importing-Data.md).  You go into your data model, and populate the User Info (right hand side).  Let’s say you have a field called ```displayName``` and the JSON object has a ```display_name``` field. For the ```displayName``` field, you would add a key called ```mappedKeyName``` and the value would be ```display_name```.

Once you’ve finished adding keys to the entity, to import something, you’d call:
```objectivec
Person *importedPerson = [Person MR_importFromObject:contactInfo];  
```

So, just make a few edits and the framework does the rest.  Great, right?  Well, it depends.  If the JSON object is fairly simple, and you’re importing 10 at a time, sure.  If the JSON object is gnarly, with multiple subarrays and dictionaries, AND you have to import 50 at a time AND establish relationships among them, not so much.  I tried both frameworks, but  I felt like I had to do too much work to shoehorn the frameworks into my code.

## How I Actually Imported The Data

I ended using [AFNetworking](https://github.com/AFNetworking/AFNetworking) to download the JSON, then on completion, I manually looped through all the JSON dictionaries and loaded them into Core Data.  For this, I used MagicalRecord’s saveWithBlock method.  While I didn’t use its convenience method for importing JSON, MagicalRecord still has a lot of great utilities.

It sets up a decent NSManagedObjectContext stack for you.  The stack follows best practices, and uses a persistent store->private queue context (root)->main queue context (default).  When you save, it creates a throwaway private context, and makes it a child of the root (private) context.  Overall, MagicalRecord eliminates a lot of Core Data boilerplate.

One last thing, don’t forget to use Apple’s best practices for bulk imports.  Basically, if you’re importing 50 JSON objects, you want to avoid doing 50 cycles of fetch/check id->create/update->save.  Instead, loop through all the objects’ unique IDs, and do one fetch using an “id IN” predicate.  This gets the old records.  Then do another loop and if an ID is not an old record, you create a new one.  So that’s only one fetch and one save vs. 50 fetches + saves.


