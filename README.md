This is just a placeholder repository to keep my notes in for now.

The plan is to modify the valuestore/valuelocmap to support groups with named members. Easiest way to explain that is as a directory listing; the directory is the group, the items in that directory are the members, and those items have names unique to that directory.

Basic requirements:

* Add named member to group.
* Remove named member from group.
* Retrieve all members of group.
* Retrieve member of group by name.
* Retrieve member of group by keyhashes only (unsure if this is required really).

Starting idea is to copy valuestore/valuelocmap and make the modifications to support these groups. I considered some "go generate" templating ideas too, but it seemed harder to work with than it'd be to have copied code. I don't know for certain yet though; maybe the valuestore/groupstore will be generated code and the valuelocmap/grouplocmap be copied code.

The groupstore would differ from the valuestore in the methods provided (see above basic requirements) instead of the generic keyhash/value methods, and that'd use the grouplocmap instead of the valuelocmap. Most things should not change drastically, just slightly to store/retrieve/replicate/etc. the additional information.

The grouplocmap would differ from the valuelocmap in that each item would have an additional keyhash used to identify its uniqueness. In other words, instead of just keyhash, there would be groupKeyhash+memberKeyhash. To allow lookups by member name, there'd have to be a established convention where the leading bytes of an item's value would be the name (probably uint16:length+nbytes:name). Also, to allow for faster named lookups, the value length uint32 would be shortened to a uint16 and the other uint16 would become a checksum of the name.

For the groupKeyhash+memberName lookup, the 16bit checksum would be used to distribute within the larger memory groupLocBlock (using bits from memberKeyHash as needed to reach the bucket count bitrange) meaning that it just has to search bitrange-16bits worth of buckets of that hashtable.

In other words, imagine it was a 1bit checksum and 1bitrange hashtable (2 buckets); it only has to search the one bucket. Imagine the checksum was 1, so it'd check the 1 bucket. Now imagine it was a 2bitrange hashtable (4 buckets); so it'd just check the 10 and 11 buckets. 3bitrange, 100 101 110 and 111. Etc.

Of course it means the hashtables have to have power-of-two number of buckets, but that's not a big deal because it's actually already a requirement based on the existing valuelocmap code.

Not sure how to support an add-only-if-not-already-existing type method, but it'd be nice if we could. Pretty sure this sort of operation would have to be supported externally with a distributed lock or a single point where the operation can occur.

MemberName overwrites have an additional issue that, if they are the only references to external resources, those resources could be "orphaned". I'm pretty sure this will have to be handled externally as well; either by bring the concurrency for a given operation down to one, or having a reference table that can be walked later to detect orphans, etc.
