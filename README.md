pg_bktree hamming
=================

Extension Was Fixed To Build With Postgresql 13

HOT TO BUILD

apt-get update
apt-get install -y git make build-essential postgresql-server-dev-13
git clone git@github.com:evirma/pg_bktree.git /usr/local/src/bktree
cd /usr/local/src/bktree
make USE_PGXS=1 && make USE_PGXS=1 install
apt-get remove -y git make build-essential postgresql-server-dev-9.6
apt-get purge
apt-get clean

psql -c "CREATE EXTENSION bktree" -d your_database

This repository contains two SP-GiST index extensions.

Principally, they detail my progress while implementing a [BK-Tree][1] as a native
C PostgreSQL indexing extension. This is extremely useful for certain types of 
searches, primarily related to fuzzy-image searching.

The subdirectories are as follows:

 It implements a BK-tree index across a 64-bit
 p-hash data type. Basically, it results in a 65-ary tree, where the child
 nodes are distributed by the edit-distance from the intermediate node.

 This is (basically) a re-implementation of my pure-C++ BK-tree implementation
 that is located [here][2]. There are minor changes due to the requirements 
 of the SP-GiST system (normally, this would be a 64-ary tree, but because
 SP-GiST inner tuples cannot store data, you need a additional branch for 
 the matching case (e.g. edit distance of 0), whereas my C++ implementation 
 can store the matching data directly in the inner tuple, so the matching 
 case does not need an additional branch).

 As a side-benefit of having a robust and well-tested C++ implementation 
 of a BK-tree, I can make stronger garantees about the correctness of the 
 SP-GiST index implementation, simply by porting the tests for the C++ 
 version to use the PG version. That was done, and the resulting tests
 are [here][3] (see all the `Test_db_BKTree*.py` files).

Anyways, in benchmarking, the PostgreSQL implementation of a BK-tree is approximately 
33% - 50% as fast as the C++ implementation, presumably due to the additional 
overhead of the PG tuple, SP-GiST and GiST mechanisms. While this is annoying, it's 
not a significant-enough performance loss to motivate me to continue using 
the C++ version, due to the significant implementation complexity of maintaining 
an out-of-database additional index. Adding more RAM to the PostgreSQL host may also help
here. My test system has 32 GB of ram, and the C++ BK-Tree implementation alone requires 
~18 GB to contain the entire dataset.

[1] : http://blog.notdot.net/2007/4/Damn-Cool-Algorithms-Part-1-BK-Trees  
[2] : https://github.com/fake-name/IntraArchiveDeduplicator/blob/92da07a75928b803a23d0e2940c40013da8ea115/deduplicator/bktree.hpp  
[3] : https://github.com/fake-name/IntraArchiveDeduplicator/tree/master/Tests  