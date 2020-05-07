---
layout: post
bigimg: /img/storagesystems.png
title: Storage Systems for Cheap but Capable Bastards
subtitle: Large files, small files, and scaling
date: 2020-05-01
tags: [infrastructure, storage, s3, sql, databases, file management]
---

I've been trying to find what storage to use in different use cases and to minimize how much I pay to other people while also getting all the features I can.  So without further ado - 
Reasons for using certain types of databases/storage engines/services assuming you're going to be loading HUGE amounts data:

**TLDR**: My recommendation is to use SeaweedFS for large files and Cassandra for small files.

### Big File Storage:
**S3**: You have gobs of money.  Loads.  Doesn't matter if your files are large or small, S3 will take them and serve them with ease.  But you are charged per download, delete and how much your store.  So if you have a large amount of small files and have to make a lot of calls to work with them the costs will eat into you if you don't pay attention.  But it doesn't matter because you have lots of money and don't care.

**Google Cloud Storage**: Again, you have loads of money.  Same exact reasoning as above.  Buckets and keys automatically scale though whereas S3 gets funny scaling issues at certain amount of calls per key or a certain amount of keys per bucket.  Brain dead except for setting up the keys, but that's actually pretty easy.

**Backblaze**:  Large file storage, works great.  Similar to s3 and they also have an s3 compatible api.  It has a small file issue, the cost to handle them, if you've gone over the free use quota is cheaper than s3 but will still add up.  Doesn't seem to have a hard rate limit though so go ahead and kill your budget with tiny cuts.

**Wasabi**: Large file storage, works GREAT.  Has a rate limit of 1000 gets and 100 puts/deletes per MINUTE.  MINUTE.  This is a large file or archival system foremost but they have free egress which is the bees knees.  Small files don't work because of the rate limit.  In October 2019 they had multi day downtime in one region.   It's not common but it's important to know.

**Minio**: S3 system but open source and works pretty darn well.  Has great speed for large files (GB/MBs) but it can't handle small files as fast as a database.  Works on SSD and HDD just fine.  Use for a video backend probably. Scales horizontally pretty easily - just make a new cluster and it handles itself.

**SeaweedFS**: This is an interesting one.  S3 and FUSE and other systems are all supported.  It handles large files just as quickly as Minio and small files a bit better though not even as fast as CockroachDB.  This is my pick for best large file handler especially over Minio because it allows adding volumes for buckets instead of just saying you need to use federation with etcd.

**Ceph**:  I don't know. A cursory search shows that it works best when you have full control over the hardware, as in bare metal control.  I haven't delved into it enough nor have I run ceph yet since I'm using the cloud (other people's computers) so I have less control over my machine choices. Seems to work just fine as an s3 server but I would use minio or seaweedfs for the ease of use instead.

### Lil' File Storage:

**Postgres**: Scales vertically but not horizontally, simple SQL though.  Works because it's postgres and does sql like sql should be done.  Has so many clients in many different languages.  As solid as you could ask for.

**Mysql**: Never used it. Doesn't scale horizontally and I don't want to deal with sharding.  Lots of people use it though so it no reason to not try.  I don't want to try though.

**FoundationDB**: Scales but **IT'S SSD ONLY**.  HDD performance is absolutely shite.  You might think it's fast in the beginning if you load test it but it quickly peters out, even if you do batching and multiple clients and split the transaction log and data onto different disks.  Because it's SSD only the costs/storage curves are steeper.  Much steeper.  Not to mention you have to organize your keys and prefixes correctly and there is no security or access control.  Security has to be handled before anyone reaches the db since everything is in the same namespace.

**Yugabyte**:  They put spin on their benchmarks.  Check with Jepsen directly, check with other db vendor rebuttals to their benchmarks.  It might work ok and they run on lms(-ish) which is the same thing rocksdb runs on and sqlite4 experimented with.  May be worth checking out but I'm not going to do it.

**Sqlite**: Fast.  Only scales vertically though.  Others are just as fast on HDDs. Use on local SSDs for the file system instead of the actual file system.  Make sure you batch your sql commands into transactions and you can have one sql store with multiple processes that call to it.

**rqlite**: Distributed Sqlite.  Replication is for redundancy and reads only. Reads scale horizontally.  Writes only scale vertically.  Otherwise it has the same benefits and limitations as sqlite.  It gets slightly slower due to extra network calls but that goes for everything else as well.

**ActorDB**: The topology and usage of actors is somewhat interesting but I have not tried it.  From an outsider's view it seems to be like CockroachDB but without the distributed processing engine.  It feels bad to say but I'm not motivated enough to give this one a try considering I would have to think about how to write my queries more so than using standard SQL.

**CockroachDB**: Scales vertically and horizontally, kinda slow but on HDD all you need are more disks for more cockroach procs for a local cluster.  Does sql just fine and uses plenty of postgres client libs so it's easy to connect.

**Cassandra**: This thing is fast.  If you don't need sql it's better than anything else and was more consistent under heavy load on HDD than ScyllaDB.  I don't even like java and I'm impressed. On a single disk It's possible to max out a 1Gbps connection.  I'll be using this over Cockroach for any remote KV stuff.  This is the winner for storing lots of small files.  It has replication and it's easy to add additional nodes.  Large files are a no-no but 10s of Kb are fine.  If you use HDDs then S3, Google Cloud Storage, Backblaze, Wasabi cannot win against this for storing small files in terms of cost.  Reliability and ease are one thing but I have more time than money so that's not happening.

**Keydb**: It's a young redis fork but it's turning into a redis/rocksdb by using lsm underneath for persistence.  Could be good but there's HSE, RocksDB, leveldb, etc.  So many choices here.

**Redis**: Memory only unless enterprise, single core.  Has some cluster capabilities.  No dice.

**Aerospike**: Similar to HSE and came to the market/community first.  It's SSD biased so that's a no go. 

Theres a lot more.  I can't get to them all, My choices are Cassandra for small files with a script to manage the scaling and SeaweedFS for the large files with another script to handle the scaling. Brain dead easy, cost effective, enough performance to handle large load of users when combined with edge vps servers.

