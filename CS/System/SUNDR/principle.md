# Secure Untrusted Data Repository

Secure Untrusted Data Repository (SUNDR) lets clients detect any attempts at unauthorized file modification by malicious server operators or users. SUNDR's protocol achieves a property called *fork consistency*, which guarantees that clients can detect any integrity or consistency failures as long as they see each other's file modifications.

## Setting

SUNDR provides a file system interface to remote storage, like NFS and other network file systems. All checkouts and commits then take through SUNDR, ensuring users will detect any attempts by the hosting site to tamper with repository contents.

Below shows SUNDR's basic architecture. When applications accesses the file system, the client software internally translates their system calls into a series of *fetch* and *modify* operations, where fetch means retrieving a file's contents or validating a cached local copy, and modify means making new file system state visible to other users.

![Basic SUNDR architecture](https://s2.loli.net/2022/12/22/cjnMm71UBpTw8E4.png)

To set up a SUNDR server, one runs the server software on a networked machine with dedicated SUNDR disks or partitions. The server can can host one or more file systems. To create a file system, one generates a public/private *superuser* signature key pair and **gives the public key to the server**, while keeping the private key secret. The private key provides *exclusive* write access to the root directory of the file system. It also directly or indirectly allows access to any file below the root. **However, the privileges are confined to that one file system**. Thus, when a SUNDR server hosts multiple file systems with different superusers, no single person has write access to all files.

Each user of a SUNDR file system also has a signature key. When establishing an account, users exchange public keys with the superuser. The superuser manages accounts with two superuser-owned file in the root directory of the file system: `.sundr.users` list users's public keys an numeric IDs, while `.sundr.group` designates groups and their membership. To mount a file system, one must specify the superuser's public key as a command-line argument to the client, and must therefore give the client access to the private key.

SUNDR assumes a user is aware of the last operation he performed. In the implementation, the client remembers the last operation it has performed on behalf of each user. To move between clients, a user needs both his private key and the last operation performed on his behalf.

## The SUNDR Protocol

SUNDR's protocol let clients detect unauthorized attempts to modify files, even by attackers in control of the server. When the server behaves correctly, a fetch reflects exactly the authorized modifications that happened before it. We call this property *fetch-modify consistency*.

If the server is dishonest, clients force a slightly weaker property called *fork consistency*. If $A$'s client accepts some modification by $B$, then at least until $B$ performed that modification, both users had identical, fetch-modify-consistent views of the file system.

### A Straw-man File System

Some assumptions:

+ Avoid any concurrent operations.
+ Allow unreasonable amounts of bandwidth and computation.
+ Maintain a single untrusted global lock.
+ To fetch or modify a file, a user first acquires the lock, then performs the desired operation, then releases the lock.

The straw-man file server stores a complete, ordered list of every fetch or modify operation every performed. Each operation also contains a digital signature from the user who performed it. The signature covers not just the operation but also *the complete history of all operations that precede it*. For example, after five operations the history might appear as follows:

![Operation history example](https://s2.loli.net/2022/12/22/NGWjuTRMOi5Lnlc.png)

To fetch or modify a file, a client acquires the global lock, downloads the entire history of the file system, and validates each user's most recent signature. The client also checks that its own user's previous operation is in the download history.

The client then traverses the operation history to construct a local copy of the file system. For each modify encountered, the client additionally checks that the operation was actually permitted, using the user and group files to validate the signing user against the file's owner or group. If all checks succeed the client appends a new operation to the list, signs the new history, sends it to the server, and releases the lock.

### Serialized SUNDR

The straw-man file system is impractical for two reasons:

+ It must record and ship around complete file system operation histories, requiring enormous amounts of bandwidth and storage.
+ The serialization of operations through a global lock is impractical for a multi-user network file system.

Instead of signing operation histories, as in the straw-man file system, SUNDR effectively takes the approach of *signing file system snapshots*. Roughly speaking, users sign messages that tie together the complete state of all files with two mechanisms:

+ All files writable by a particular user or group are efficiently **aggregated** into a single hash value called the *i-handle* using *hash trees*.
+ Each *i-handle* is tied to the latest version of every other i-handle using *version vectors*.

#### Data Structures

SUNDR names all on-disk data structures by cryptographic handles. The block store indexes most persistent data structures by their 20-byte SHA-1 hashes, making the server a kind of large, high performance hash table.

SUNDR also stores messages signed by users. These are indexed by a hash of the public key and an index number (so as to distinguish multiple message signed by the same key).

Below shows the persistent data structure SUNDR stores and indexes by hash, as well as the algorithm for computing i-handles. Every file is identified by a `<principal,i-number>` pair, where principal is the user or group allowed to write the file, and i-number is a per-principal inode number. ^fb9c02

#### Protocol

i-handles are stored in digitally-signed messages known as *version structures*, shown below. Each version structure is signed by a particular user. The structure must always contain the user's i-handle. In addition, it can **optionally** contain one or more i-handles of groups to which the user belongs. Finally, the version structure contains a version vector consisting of a version number for every user and group in the system.

When user $u$ performs a file system operation, $u$'s client acquires the global lock and downloads the latest version structure for each user and group. We call this set of version structures the *version structure list*, or VSL. The client then computes a new version structure $z$ by potentially updating i-handles and by setting the version numbers in $z$ to reflect the current state of the file system.

More specifically, to set the i-handles in $z$, on a fetch, the client simply copies $u$'s previous i-handle into $z$. For a modify, the client computes and include new i-handles for $u$ and for any groups whose i-tables it is modifying.

The client the sets $z$'s version vector to reflect the version number of **each VSL entry**. For any version structure like $z$, and any principal $p$, let $z[p]$ denote $p$'s version number in $z$'s version vector. For each principal $p$, if $y_{p}$ is $p$'s entry in the VSL, set $z[p] \leftarrow y_{p}[p]$.

Finally, the client bumps version numbers to reflect the i-handles in $z$. It sets $z[u] \leftarrow z[u] + 1$ and for any group $g$ whose i-handle $z$ contains, sets $z[g] \leftarrow z[g] + 1$.

The client then checks the VSL **for consistency**. Given two version structures $x$ and $y$, we define $x \leq y$ if and only if $\forall p \ x[p] \leq y[p]$. To check consistency, the client verifies that the VSL contains $u$'s previous version structure, and that the set of all VSL entries combined with $z$ is totally ordered by $\leq$. If it is, the user signs the new version structure and sends it to the server. The server adds the new structure to the VSL and retires the old entries for updated i-handles.