---
title: 'Experiment with GIN index of PostgreSQL DB'
date: 2019-04-04 14:49:16
lastmod: 2019-04-06 10:45:31
draft: false
categories:
  - 'PostgreSQL'
aliases:
  - '/experiment-with-gin-index-of-postgresql-db/'
---

PostgreSQL provides the index methods B-tree, hash, GiST, SP-GiST, GIN, and BRIN. Users can also define their own index methods, but that is fairly complicated. In this article, I will discuss the GIN index. GIN index is a very useful index method for a query, count, etc.

<strong>Use cases GIN index:</strong>
<ul>
 	<li>Full-Text search(tsvector)</li>
 	<li>Array Data Type</li>
 	<li>JSONB</li>
</ul>
<strong>What is the Gin index?</strong>
From the <a href="https://www.postgresql.org/docs/9.6/gin-intro.html">Docs</a>
<blockquote><em>"GIN stands for Generalized Inverted Index. GIN is designed for handling cases where the items to be indexed are composite values, and the queries to be handled by the index need to search for element values that appear within the composite items. For example, the items could be documents, and the queries could be searches for documents containing specific words."</em></blockquote>
![GIN index diagram](/images/GIN-index-1024x597.jpg)

Image credit: <a href="https://www.cybertec-postgresql.com">Cybertec Postgresql</a>
<strong>Create test data</strong>

The example given is done with PostgreSQL9.6 version (psql client 11.2 ) and Ubuntu 18.04. Machine hardware is RAM 8 GB and Core i5.
<strong>
</strong>Create a table with random data by following SQL command over psql client.
<pre class="lang:pgsql decode:true ">CREATE TABLE test_table as select s, md5(random()::text) from generate_series(1, 10000000) s;</pre>
By this created two columns. first one is <code>s</code> and another one is <code>md5</code>. Now let's search a text from our generate data.
<pre class="lang:pgsql decode:true">select * from new_test where md5 ilike '%b9e2747b6a98a277c2449f2a8969bd1e%';</pre>
I searched the last row data from the table. It quite slow brings to our result. The query time is 10132.924 ms or almost 10 seconds without GIN index.

Let's create GIN an index of the md5 column. By following SQL command.
<pre class="lang:pgsql decode:true">CREATE EXTENSION IF NOT EXISTS pg_trgm;CREATE INDEX new_table_search_idx ON new_test USING gin(md5 gin_trgm_ops);
</pre>
It will take time for creating index 10000000 rows.
Now recall the query.
<pre class="lang:pgsql decode:true">select * from new_test where md5 ilike '%b9e2747b6a98a277c2449f2a8969bd1e%';</pre>
It will take only a few milliseconds. Second query time is 62.219 ms. Obviously, this is a major improvement in our query.

To be continued ...
