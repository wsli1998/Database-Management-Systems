# Chapter 13 | Data Storage Structures

## 13.1 | Database Storage Architecture

We did not cover this section in lecture.

## 13.2 | File Organization

A databases is stored as a collection of **files**, with each file having a sequence of **records**, and each record being a sequence of fields.

Each file is partitioned into fixed-length storage units called **blocks**. We assume that a file cannot be larger than the block size and that a file will be contained in a single block.

There are two ways to store a record:

-   fixed-length
-   variable-length

### Fixed-Length Records

Let us look at a file of _instructor_ records for our university database:

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-64.png)

Assume that each character occupies 1 byte and that numeric (8,2) occupies 8
bytes. Suppose that instead of allocating a variable amount of bytes for the attributes
_ID_, _name_, and _dept_name_, we allocate the maximum number of bytes that each attribute
can hold. Then, the _instructor_ record is 53 bytes long. A simple approach is to use the
first 53 bytes for the first record, the next 53 bytes for the second record, and so on.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-65.png)

However, there are two problems with this implementation:

1. If the block size is not a multiple of 53, some records will cross into the next block. This will require two block accesses to read/write that specific record.
2. Deleting records is a hassle. The deleted record must be filled up so that memory is still contiguous, or a way of ignoring deleted files must be implemented.

To solve the first problem, we allocate only as many records as can fit into the block. We could do this by dividing the block size by the record size and discarding the fractional part of the answer.

The solve the second problem of record deletion, there are three possible solutions:

1. Move the record that comes after it into the space formerly occupied by the deleted record, and so on, until every record following
   the deleted record has been moved. This solution is shown in Figure 13.2.
2. Move the last record of the file into the space formerly occupied by the deleted record. This solution is shown in Figure 13.3.
3. The usage of a file header.

Since solutions 1 and 2 require multiple block accesses, solution 3 is the most appealing.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-66.png)

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-67.png)

Since insertions tend to be more
frequent than deletions, it is acceptable to leave open the space occupied by the deleted
record and to wait for a subsequent insertion before reusing the space. To keep track of open spaces, we use an additional structure called a file header, which is often allocated a certain number of bytes at the beginning of a file.

A **file header** contains a variety of information about the file, but the one we are interested in is the location of the first record whose contents are deleted. The first deleted record will then point to the location of the second deleted element, and so on. This forms a linked list of deleted locations, which is often referred to as a **free list**.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-68.png)

## Variable-Length Records

Variable-length records exists because of several reasons such as strings, repeating fields like arrays, and multiple record types within a file.

There are two problems when implementing variable-length records:

1. How to represent a single record in such a way that individual attributes can be extracted easily, even if they are of variable length.
2. How to store variable-length records within a block, such that records in a block can be extracted easily.

There are two parts in the representation of a record with variable-length attributes:

-   First, there is an initial part with fixed-length information
-   Second, there are the actual contents of the variable-length attributes.

Fixed-length attributes are allocated as many bytes as required. Variable-length attributes are represented in the initial part of the record by a pair (_offset, length_), where _offset_ is the location where the attribute starts, and _length_ is length of the attribute in bytes.

An example of such a record representation is shown in Figure 13.5. The figure
shows an _instructor_ record whose first three attributes _ID_, _name_, and _dept_name_ are
variable-length strings, and whose fourth attribute _salary_ is a fixed-sized number. We
assume that the _offset_ and _length_ values are stored in two bytes each, for a total of 4
bytes per attribute. The _salary_ attribute is assumed to be stored in 8 bytes, and each
string takes as many bytes as it has characters.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-69.png)

Figure 13.5 also shows a **null bitmap**, which indicates which attributes of the record have a null value. Since the _salary_ attribute is the fourth attribute stored in the first section of memory, a null bitmap of 0001 would set the _salary_ attribute to null. Since Figure 13.5 only shows four attributes, the null bitmap can be stored in one byte, but more memory will be necessary for records with more attributes.

Next, we address the structure of a block. The **slotted-page structure** is commonly used for organizing records within a block and is shown in Figure 13.6.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-70.png)

The block's header contains the following information:

-   The number of records entries
-   The location of the end of the free space in the block
-   An array whose entries contain the location and size of each record.

The records themselves are stored contiguously starting from the end of the block. There is free space between the final entry in the header array and the first record. When a record is inserted, memory is allocated for it at the end of the free space, and an entry containing its location is size is added to the header. If a record is deleted, the space that the record once occupied is freed, and the records in the block before the deleted record are moved so that the free space remains contiguous.

The slotted-page structure requires that there be no pointers that point directly to
records. Instead, pointers must point to the entry in the header that contains the actual
location of the record.

## 13.3 | Organization of Records in Files

-   Heap file organization: Any record can be placed anywhere in the file where there is free space. There is no ordering of records, typically having a single file or set of files for each relation.

-   Sequential file organization: Records are stored in sequential order according to the value of a "search key" of each record.

### Heap File Organization

In a **heap file organization**, a record may be stored anywhere in the file corresponding to a relation. New records are inserted at the end of a file. It is efficient in insertion because no sorting is involved, but it is slow when reading because of linear search.

### Sequential File Organization

A **sequential file** is designed for efficient processing of records in sorted order base on some search key.

A **search key** is any attribute or set of attributes; it does not need to be a primary key or even a super key.

Records are chained together in sequential order by pointers, but records are also physically stored in search-key order to minimize the number of block accesses.

Figure 13.7 is a sequential file of _instructor_ records using _ID_ as the search key.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-71.png)

Deletions are managed using pointer chains as discussed previously.

Insertions, on the other hand, are much trickier. There are two rules when inserting into a sequential file:

1. Locate the record in the file that comes before the record to be inserted in search key order.
2. If there is a free record within the same block as this record, insert the new record there. Otherwise, insert the new record in
   an overflow block. Then adjust the points so as to chain together the records in search-key order.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-72.png)

Over a period of time, the correspondence between search-key order and physical order may be lost. In such a case, a file should be **reorganized** so that the physical order is regained. Reorganizations are costly and take a lot of time.

### Multitable Clustering File Organization

Most relational databases store each relation in seperate files. Thus, each file, and as a result, each block, stores record of only one relation.

However, it can be useful to store records of more than one relation in a single block. Take, for example, the following SQL query:

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-92.png)

This query computes a join of the _department_ and _instructor_ relations. Thus, for each tuple of _department_, the system must locate the _instructor_ tuples with the same value for _dept_name_. These records must be transferred from disk to main memory. In the worst case, each record resides on a different block, which makes us do one block read per record required by the query.

As a concrete example, consider the _department_ and _instructor_ relations of Figure 13.9 and 13.10, respectively.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-93.png)

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-94.png)

Figure 13.11 shows a file structure designed for the efficient execution of queries involving the natural join of _department_ and _instructor_. All the _instructor_ tuples for a particular _dept_name_ are stored near the _department_ tuple for that _dept_name_. We say that the two relations are clustered on the key _dept_name_.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-95.png)

A **multitable clustering file organization** is a file organization, like the one depicted in Figure 13.11, that stores related records of two or more relation in each block.

The **cluster key** is the attributes that define which records are stored together. In Figure 13.11, the cluster key is _dept_name_.

## 13.4 | Data-Dictionary Storage

**Metadata** is "data about data".

Relational schemas and other metadata about relations are stored in a structure called the **data dictionary** or **system catalog**.
The information stored in these include:

-   Names of the relations
-   Names of the attributes of each relation
-   Domains and lengths of attributes
-   Names of views defined on the database, and definitions of those views.
-   Integrity constraints (e.g. key constraints)

Additionally, they keep the following data on users of the system:

-   Names of users, default schemas of users, and passwords or other information to authenticate users.
-   Information about authorizations for each user.

The data dictionary may also note the storage organization (heap, sequential, hash, etc.) of relations, and the location where each relation is stored:

-   If relations are stored in operating system files, the dictionary would note the names of the file(s) containing each relation.
-   If the databases stores all relations in a single file, the dictionary may note the blocks containing records of each relation in a data structure (e.g. a linked list).

Index information (from Chapter 14) are also stored:

-   Name of the index
-   Name of the relation being indexed
-   Attributes on which the index is defined
-   Type of index formed

Often times, metadata is stored in the database itself as a "miniature database". Figure 13.12 shows a schema diagram for a toy data dictionary.

![](https://github.com/wslisam/Database-Management-Systems/blob/master/Book/Screenshots/databases-96.png)

## The rest of Chapter 13 was not covered in lecture.
