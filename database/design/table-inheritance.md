# Table Inheritance Design Strategies

Since relational database do not model OO concepts such as inheritance, if this needs to be implemented some 
compromises will occur.

There are several strategies to implement table inheritance in a relational database.

Types of Inheritance                | Key Uniqueness | Single Subtype | Attribute Validation | No Dead Weight Attributes | Can Prevent Abstract Parent | One-Table Queries 
----------------------------------- | -------------- | -------------- | -------------------- | ------------------------- | --------------------------- | -----------------
1. Permissive Class Table Inheritance  | Yes            | --             | Yes                  | Yes                       | --                          | --                
2. Exclusive Class Table Inheritance   | Yes            | Yes            | Yes                  | Yes                       | --                          | --                
3. Single Table Inheritance (Fowler)   | Yes            | Yes            | Yes                  | --                        | Yes                         | Yes               
4. Semistructured Single Table Inheritance |  Yes       | Yes            | --                   | Yes                       | Yes                         | Yes               
5. Concrete Table Inheritance (Fowler) | --             | --             | Yes                  | Yes                       | Yes                         | --                
6. PostgreSQL Inheritance              | --             | --             | Yes                  | Yes                       | --                          | --                

Features:

- **Key Uniqueness**: Guarantees the document ID is unique and not used by any another document.
- **Single Subtype**: Guarantees a document can have a single subtype only.
- **Attribute Validation**: Validates attributes types and values.
- **No Dead Weight Attributes**: Rows/Objects only include their own attributes and not unused attributes of other subtypes.
- **Can Prevent Abstract Parent**: Can guarantee there's no parent without a specific subtype (and its sub-attributes).
- **One-Table Queries**: SQL queries are simpler and more performant since they always use a single table only.

## 1. Permissive Class Table Inheritance

Does not guarantee exclusivity between children tables.

```sql
create table document (
  id int primary key not null,
  title varchar(30)
);

create table invoice (
  id int primary key not null,
  amount decimal(12, 2) not null,
  foreign key (id) references document (id)
);

create table book (
  id int primary key not null,
  isbn varchar(13) not null,
  foreign key (id) references document (id)
);
```

## 2. Exclusive Class Table Inheritance

Guarantees exclusivity between children tables.

```sql
create table document (
  id int primary key not null,
  title varchar(30),
  type char(1) not null check (type in ('I', 'B')),
  constraint uq unique (id, type)
);

create table invoice (
  id int primary key not null,
  amount decimal(12, 2) not null,
  type char(1) not null check (type = 'I'),
  foreign key (id, type) references document (id, type)
);

create table book (
  id int primary key not null,
  isbn varchar(13) not null,
  type char(1) not null check (type = 'B'),
  foreign key (id, type) references document (id, type)
);
```

## 3. Single Table Inheritance (Fowler)

- Benefit: Uses a single table. Queries are simple.
- Disadvantage: Rows/Objects carry unrelated, dead weight attributes.

```sql
create table document (
  id int primary key not null,
  title varchar(30),
  amount decimal(12, 2) check (type = 'I' and amount is not null or type <> 'I' and amount is null),
  isbn varchar(13)      check (type = 'B' and isbn is not null or type <> 'B' and isbn is null) 
);
```

## 4. Semistructured Single Table Inheritance

- Benefit: Uses a single table. Queries are simple.
- Disadvantage: There's no attribute validation whatsoever.

```sql
create table document (
  id int primary key not null,
  title varchar(30),
  type char(1) not null check (type in ('I', 'B')),
  attributes json -- Can also use CLOB, XML, ARRAY, VARCHAR, etc.
);
```

## 5. Concrete Table Inheritance (Fowler)

- Disadvantage: There can be document number collisions (id).

```sql
create table invoice (
  id int primary key not null,
  title varchar(30),
  amount decimal(12, 2) not null
);

create table book (
  id int primary key not null,
  title varchar(30),
  isbn varchar(13) not null
);
```

## 6. PostgreSQL Inheritance

- Disadvantage: PostgreSQL implementation on inheritance is quite incomplete and does not enforce any rule between tables.

```sql
create table document (
  id int primary key not null,
  title varchar(30)
);

create table invoice (
  amount decimal(12, 2) not null
) inherits (document);

create table book (
  isbn varchar(13) not null
) inherits (document);
```












