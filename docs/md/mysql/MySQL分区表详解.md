在我们日常处理海量数据的过程中，如何有效管理和优化数据库一直是一个既重要又具有挑战性的问题。

分区表技术就为此提供了一种解决方案，尤其是在使用MySQL这类关系型数据库时。该技术将大型表的数据切割成更易于管理和查询的小块，从而提高了整体数据库操作的性能。

本文将详细探讨MySQL分区表的概念、实现方式以及具体应用场景，帮助读者更好地理解并运用这一高效的数据库优化策略。

## 分区表介绍

MySQL 数据库中的数据是以文件的形势存在磁盘上的，默认放在 `/var/lib/mysql/` 目录下面，我们可以通过 `show variables like '%datadir%'` 命令来进行查看：

![](https://static001.geekbang.org/infoq/6f/6f0be709cf719968bedde7d4b58849c4.png)

我们进入到这个目录下，就可以看到我们定义的所有数据库了，一个数据库就是一个文件夹，一个库中，有其对应的表的信息，如下：

![](https://static001.geekbang.org/infoq/5d/5d14fb10798d3dbf045cbb9d9b71c5ee.png)

在 MySQL 中，如果存储引擎是 MyISAM，那么在 data 目录下会看到 3 类文件：`.frm`、`.myi`、`.myd`，文件含义如下：

- `*.frm`：这个是表定义，是描述表结构的文件。

- `*.myd`：这个是数据信息文件，是表的数据文件。

- `*.myi`：这个是索引信息文件。

  

如果存储引擎是 `InnoDB`, 那么在 data 目录下会看到两类文件：`.frm`、`.ibd`，文件含义如下：

- `*.frm`：表结构文件。
- `*.ibd`：表数据和索引的文件。

无论是哪种存储引擎，只要一张表的数据量过大，就会导致 `*.myd`、`*.myi` 以及 `*.ibd` 文件过大，从而数据的查找就会变的很慢。

**为了解决这个问题，我们可以利用 MySQL 的分区功能，在物理上将这一张表对应的文件，分割成许多小块，如此，当我们查找一条数据时，就不用在某一个文件中进行整个遍历了，我们只需要知道这条数据位于哪一个数据块，然后在那一个数据块上查找就行了。**

另一方面，如果一张表的数据量太大，可能一个磁盘放不下，这个时候，通过表分区我们就可以把数据分配到不同的磁盘里面去。

**通俗地讲表分区就是将一大表，根据条件分割成若干个小表。**

如：某用户表的记录超过了 600 万条，那么就可以根据入库日期将表分区，也可以根据所在地将表分区。当然也可根据其他的条件分区。



**MySQL 从 5.1 版本开始添加了对分区的支持，分区的过程是将一个表或索引分解为多个更小、更可管理的部分。**

对于开发者而言，分区后的表使用方式和不分区基本上还是一模一样，只不过在物理存储上，原本该表只有一个数据文件，现在变成了多个，每个分区都是独立的对象，可以独自处理，也可以作为一个更大对象的一部分进行处理。



需要注意的是，分区功能并不是在存储引擎层完成的，常见的存储引擎如 `InnoDB`、`MyISAM`、`NDB` 等都支持分区。

但并不是所有的存储引擎都支持，如 `CSV`**、**`FEDORATED`**、**`MERGE` **等就不支持分区，因此在使用此分区功能前，应该对选择的存储引擎对分区的支持有所了解。

### 表分区的优缺点和限制

MySQL 分区有优点也有一些缺点，罗列如下：

优点：

- **查询性能提升**：分区可以将大表划分为更小的部分，查询时只需扫描特定的分区，而不是整个表，从而提高查询性能。特别是在处理大量数据或高并发负载时，分区可以显著减少查询的响应时间。
- **管理和维护的简化**：使用分区可以更轻松地管理和维护数据。可以针对特定的分区执行维护操作，如备份、恢复、优化和数据清理，而不必处理整个表。这简化了维护任务并减少了操作的复杂性。
- **数据管理灵活性**：通过分区，可以根据业务需求轻松地添加或删除分区，而无需影响整个表。这使得数据的增长和变化更具弹性，可以根据需求进行动态调整。
- **改善数据安全性和可用性**：可以将不同分区的数据分布在不同的存储设备上，从而提高数据的安全性和可用性。例如，可以将热数据放在高速存储设备上，而将冷数据放在廉价存储设备上，以实现更高的性能和成本效益。



缺点：

- **复杂性增加**：分区引入了额外的复杂性，包括分区策略的选择、表结构的设计和维护、查询逻辑的调整等。正确地设置和管理分区需要一定的经验和专业知识。
- **索引效率下降**：对于某些查询，特别是涉及跨分区的查询，可能会导致索引效率下降。由于查询需要在多个分区之间进行扫描，可能无法充分利用索引优势，从而影响查询性能。
- **存储空间需求增加**：使用分区会导致一定程度的存储空间浪费。每个分区都需要占用一定的存储空间，包括分区元数据和一些额外的开销。因此，对于分区键的选择和分区粒度的设置需要权衡存储空间和性能之间的关系。
- **功能限制**：在某些情况下，分区可能会限制某些 MySQL 的功能和特性的使用。例如，某些类型的索引可能无法在分区表上使用，或者某些 DDL 操作可能需要更复杂的处理。



在考虑使用分区时，需要综合考虑业务需求、查询模式、数据规模和硬件资源等因素，并权衡分区带来的优势和缺点。对于特定的应用和数据场景，分区可能是一个有效的解决方案，但并不适用于所有情况。

同时分区表也存在一些限制，如下：



限制：

- 在 MySQL 5.6.7 之前的版本，一个表最多有 1024 个分区，从 5.6.7 开始，一个表最多可以有 8192 个分区。
- 分区表无法使用外键约束。
- NULL 值会使分区过滤无效。
- 所有分区必须使用相同的存储引擎。

## 分区适用场景

分区表在以下情况可以发挥其优势，适用于以下几种使用场景：

- **大型表处理**：当面对非常大的表时，分区表可以提高查询性能。通过将表分割为更小的分区，查询操作只需要处理特定的分区，从而减少扫描的数据量，提高查询效率。这在处理日志数据、历史数据或其他需要大量存储和高性能查询的场景中非常有用。
- **时间范围查询**：对于按时间排序的数据，分区表可以按照时间范围进行分区，每个分区包含特定时间段内的数据。这使得按时间范围进行查询变得更高效，例如在某个时间段内检索数据、生成报表或执行时间段的聚合操作。
- **数据归档和数据保留**：分区表可用于数据归档和数据保留的需求。旧数据可以归档到单独的分区中，并将其存储在低成本的存储介质上。同时，可以保留较新数据在高性能的存储介质上，以便快速查询和操作。
- **并行查询和负载均衡**：通过哈希分区或键分区，可以将数据均匀地分布在多个分区中，从而实现并行查询和负载均衡。查询可以同时在多个分区上进行，并在最终合并结果，提高查询性能和系统吞吐量。
- **数据删除和维护**：使用分区表，可以更轻松地删除或清理不再需要的数据。通过删除整个分区，可以更快速地删除大量数据，而不会影响整个表的操作。此外，可以针对特定分区执行维护任务，如重新构建索引、备份和优化，以减少对整个表的影响。



分区表并非适用于所有情况。在选择使用分区表时，需要综合考虑数据量、查询模式、存储资源和硬件能力等因素，并评估分区对性能和管理的影响。

## 分区方式

**分区有两种方式，水平切分和垂直切分，MySQL 数据库支持的分区类型为水平分区，它不支持垂直分区。**

此外，MySQL 数据库的分区是局部分区索引，一个分区中既存放了数据又存放了索引。而全局分区是指，数据存放在各个分区中，但是所有数据的索引放在一个对象中。**目前，MySQL 数据库还不支持全局分区**。

## 分区策略

### RANGE 分区

RANGE 分区是 MySQL 中的一种分区策略，根据某一列的范围值将数据分布到不同的分区。每个分区包含特定的范围。下面是 RANGE 分区的定义方式、特点以及代码示例。



定义方式：

- **指定分区键**：选择作为分区依据的列作为分区键，通常是日期、数值等具有范围特性的列。
- **分区函数**：通过`PARTITION BY RANGE`指定使用 RANGE 分区策略。
- **定义分区范围**：使用`VALUES LESS THAN`子句定义每个分区的范围。



RANGE 分区的特点：

- **范围划分**：根据指定列的范围进行分区，适用于需要按范围进行查询和管理的情况。
- **灵活的范围定义**：可以定义任意数量的分区，并且每个分区可以具有不同的范围。
- **高效查询**：根据查询条件的范围，MySQL 能够快速定位到特定的分区，提高查询效率。
- **动态管理**：可以根据业务需求轻松添加或删除分区，适应数据增长或变更的需求。



以下是一个使用 RANGE 分区的代码示例：

```sql
CREATE TABLE sales (
	id INT,
	sales_date DATE,
	amount DECIMAL(10, 2)
)
PARTITION BY RANGE (YEAR(sales_date)) (
	PARTITION p1 VALUES LESS THAN (2020),
	PARTITION p2 VALUES LESS THAN (2021),
	PARTITION p3 VALUES LESS THAN (2022),
	PARTITION p4 VALUES LESS THAN MAXVALUE
);
```

在上述示例中，我们创建了名为`sales`的表，使用 RANGE 分区策略。根据`sales_date`列的年份范围将数据分布到不同的分区：

- `PARTITION BY RANGE (YEAR(sales_date))`：指定使用 RANGE 分区，基于`sales_date`列的年份进行分区。
- `PARTITION p1 VALUES LESS THAN (2020)`：定义名为`p1`的分区，包含年份小于 2020 的数据。
- `PARTITION p2 VALUES LESS THAN (2021)`：定义名为`p2`的分区，包含年份小于 2021 的数据。
- `PARTITION p3 VALUES LESS THAN (2022)`：定义名为`p3`的分区，包含年份小于 2022 的数据。
- `PARTITION p4 VALUES LESS THAN MAXVALUE`：定义名为`p4`的分区，包含超出定义范围的数据。

RANGE 分区允许根据列值的范围将数据分散到不同的分区中，适用于按范围进行查询和管理的情况。它提供了更灵活的数据管理和查询效率的提升。

### LIST 分区

LIST 分区是根据某一列的离散值将数据分布到不同的分区。每个分区包含特定的列值列表。下面是 LIST 分区的定义方式、特点以及代码示例。

定义方式：

- **指定分区键**：选择作为分区依据的列作为分区键，通常是具有离散值的列，如地区、类别等。

- **分区函数**：通过`PARTITION BY LIST`指定使用 LIST 分区策略。

- **定义分区列表**：使用`VALUES IN`子句定义每个分区包含的列值列表。

  

LIST 分区的特点：

- **列值离散**：根据指定列的具体取值进行分区，适用于具有离散值的列。
- **灵活的分区定义**：可以定义任意数量的分区，并且每个分区可以具有不同的列值列表。
- **高效查询**：根据查询条件的列值直接定位到特定分区，提高查询效率。
- **动态管理**：可以根据业务需求轻松添加或删除分区，适应数据增长或变更的需求。




以下是一个使用 LIST 分区的代码示例：

```sql
CREATE TABLE users (
	id INT,
	username VARCHAR(50),
	region VARCHAR(50)
)
PARTITION BY LIST (region) (
	PARTITION p_east VALUES IN ('New York', 'Boston'), 
	PARTITION p_west VALUES IN ('Los Angeles', 'San Francisco'), 
	PARTITION p_other VALUES IN (DEFAULT)
);
```

在上述示例中，我们创建了名为`users`的表，使用 LIST 分区策略。根据`region`列的具体取值将数据分布到不同的分区：

- `PARTITION BY LIST (region)`：指定使用 LIST 分区，基于`region`列的值进行分区。
- `PARTITION p_east VALUES IN ('New York', 'Boston')`：定义名为`p_east`的分区，包含值为'New York'和'Boston'的`region`列的数据。
- `PARTITION p_west VALUES IN ('Los Angeles', 'San Francisco')`：定义名为`p_west`的分区，包含值为'Los Angeles'和'San Francisco'的`region`列的数据。
- `PARTITION p_other VALUES IN (DEFAULT)`：定义名为`p_other`的分区，包含其他`region`列值的数据。

### HASH 分区

HASH 分区是使用哈希算法将数据均匀地分布到多个分区中。下面是 HASH 分区的定义方式、特点以及代码示例。

定义方式：

- **指定分区键**：选择作为分区依据的列作为分区键。

- **分区函数**：通过`PARTITION BY HASH`指定使用 HASH 分区策略。

- **定义分区数量**：使用`PARTITIONS`关键字指定分区的数量。

  

HASH 分区的特点：

- **数据均匀分布**：HASH 分区使用哈希算法将数据均匀地分布到不同的分区中，确保数据在各个分区之间平衡。

- **并行查询性能**：通过将数据分散到多个分区，HASH 分区可以提高并行查询的性能，多个查询可以同时在不同分区上执行。

- **简化管理**：HASH 分区使得数据管理更加灵活，可以轻松地添加或删除分区，以适应数据增长或变更的需求。

  

以下是一个使用 HASH 分区的代码示例：

```sql
CREATE TABLE sensor_data (
	id INT,
	sensor_name VARCHAR(50),
	value INT
)
PARTITION BY HASH (id) PARTITIONS 4;
```



在上述示例中，我们创建了名为`sensor_data`的表，使用 HASH 分区策略。根据`id`列的哈希值将数据分布到 4 个分区中：

- `PARTITION BY HASH (id)`：指定使用 HASH 分区，基于`id`列的哈希值进行分区。
- `PARTITIONS 4`：指定创建 4 个分区。

### KEY 分区

KEY 分区是根据某一列的哈希值将数据分布到不同的分区。不同于 HASH 分区，KEY 分区使用的是列值的哈希值而不是哈希函数。下面是 KEY 分区的定义方式、特点以及代码示例。



定义方式：

- **指定分区键**：选择作为分区依据的列作为分区键。
- **分区函数**：通过`PARTITION BY KEY`指定使用 KEY 分区策略。
- **定义分区数量**：使用`PARTITIONS`关键字指定分区的数量。



KEY 分区的特点：

- **哈希分布**：KEY 分区使用列值的哈希值将数据分布到不同的分区中，与哈希函数不同，它使用的是列值的哈希值。
- **高度自定义**：KEY 分区允许根据业务需求自定义分区逻辑，可以灵活地选择分区键和分区数量。
- **并行查询性能**：通过将数据分散到多个分区，KEY 分区可以提高并行查询的性能，多个查询可以同时在不同分区上执行。
- **简化管理**：KEY 分区使得数据管理更加灵活，可以轻松地添加或删除分区，以适应数据增长或变更的需求。



以下是一个使用 KEY 分区的代码示例：

```sql
CREATE TABLE orders (
	order_id INT,
	customer_id INT,
	order_date DATE
)
PARTITION BY KEY (customer_id) PARTITIONS 5;
```

在上述示例中，我们创建了名为`orders`的表，使用 KEY 分区策略。根据`customer_id`列的哈希值将数据分布到 5 个分区中：

- `PARTITION BY KEY (customer_id)`：指定使用 KEY 分区，基于`customer_id`列的哈希值进行分区。
- `PARTITIONS 5`：指定创建 5 个分区。

### COLUMNS 分区

MySQL 在 5.5 版本引入了 COLUMNS 分区类型，其中包括 `RANGE COLUMNS` 分区和 `LIST COLUMNS` 分区。以下是对这两种 COLUMNS 分区的详细说明：

**RANGE COLUMNS 分区**： RANGE COLUMNS 分区是根据列的范围值将数据分布到不同的分区的分区策略。它类似于 RANGE 分区，但是根据多个列的范围值进行分区，而不是只根据一个列。这使得范围的定义更加灵活，可以基于多个列的组合来进行分区。

下面是一个 RANGE COLUMNS 分区的代码示例：

```sql
CREATE TABLE sales (
	id INT,
	sales_date DATE,
	region VARCHAR(50),
	amount DECIMAL(10, 2)
)
PARTITION BY RANGE COLUMNS (region, sales_date) (
	PARTITION p1 VALUES LESS THAN ('East', '2022-01-01'),
	PARTITION p2 VALUES LESS THAN ('West', '2022-01-01'),
	PARTITION p3 VALUES LESS THAN ('East', MAXVALUE),
	PARTITION p4 VALUES LESS THAN ('West', MAXVALUE)
);
```

在上述示例中，我们创建了一个名为 sales 的表，并使用 RANGE COLUMNS 分区策略。根据 `region` 和 `sales_date` 两列的范围将数据分布到不同的分区。每个分区根据这两列的范围值进行划分。



**LIST COLUMNS 分区**： LIST COLUMNS 分区是根据列的离散值将数据分布到不同的分区的分区策略。它类似于 LIST 分区，但是根据多个列的离散值进行分区，而不是只根据一个列。这使得离散值的定义更加灵活，可以基于多个列的组合来进行分区。

下面是一个 LIST COLUMNS 分区的代码示例：

```sql
CREATE TABLE users (
	id INT,
	username VARCHAR(50),
	region VARCHAR(50),
	category VARCHAR(50)
)
PARTITION BY LIST COLUMNS (region, category) (
	PARTITION p_east VALUES IN (('New York', 'A'), ('Boston', 'B')), 
	PARTITION p_west VALUES IN (('Los Angeles', 'C'), ('San Francisco', 'D')), 
	PARTITION p_other VALUES IN (DEFAULT)
);
```

在上述示例中，我们创建了一个名为 users 的表，并使用 LIST COLUMNS 分区策略。根据 `region` 和 `category` 两列的离散值将数据分布到不同的分区。每个分区根据这两列的离散值进行划分。

## 常见分区命令

### 是否支持分区

在 MySQL5.6.1 之前可以通过命令 `show variables like '%have_partitioning%'` 来查看 MySQL 是否支持分区。如果 `have_partitioning` 的值为 YES，则表示支持分区。

从 MySQL5.6.1 开始，`have_partitioning` 参数已经被去掉了，而是用 `SHOW PLUGINS` 来代替。若有 partition 行且 STATUS 列的值为 ACTIVE，则表示支持分区，如下所示：

![](https://static001.geekbang.org/infoq/ff/ffc3b224f422333542f0c1648d9ae01a.png)

### 创建分区表

```sql
CREATE TABLE sales (
	id INT,
	sales_date DATE,
	amount DECIMAL(10, 2)
)
PARTITION BY RANGE (YEAR(sales_date)) (
	PARTITION p1 VALUES LESS THAN (2020),
	PARTITION p2 VALUES LESS THAN (2021),
	PARTITION p3 VALUES LESS THAN (2022),
	PARTITION p4 VALUES LESS THAN MAXVALUE
);
```

### 向分区表添加新的分区

```sql
ALTER TABLE sales
	ADD PARTITION (PARTITION p5 VALUES LESS THAN (2023));
```

### 删除指定的分区

```sql
ALTER TABLE sales DROP PARTITION p3;
```

### 重新组织分区

```sql
ALTER TABLE sales
	REORGANIZE p1, p2, p5 INTO (PARTITION p1 VALUES LESS THAN (2020), PARTITION p2 VALUES LESS THAN (2022), PARTITION p3 VALUES LESS THAN MAXVALUE);
```

### 合并相邻的分区

```sql
ALTER TABLE sales COALESCE PARTITION p1, p2;
```

### 分析指定分区的统计信息

```sql
ALTER TABLE sales ANALYZE PARTITION p1;：
```



总的来说，MySQL分区表在数据管理和查询性能上提供了显著的优势。它可以帮助我们处理大规模数据，提高查询速度，并改善系统性能。

然而，合理地、有效地实施分区策略也需要对业务需求和数据特性有深刻理解。虽然分区表的使用在许多场景下都是有益的，但仍需要注意其适用性及可能存在的限制。无论如何，掌握和使用MySQL分区表无疑是每个数据库管理员和开发人员工具箱中的一个重要工具。