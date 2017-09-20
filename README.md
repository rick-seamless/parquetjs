# parquet.js

[![Build Status](https://travis-ci.org/ironSource/parquetjs.png?branch=master)](http://travis-ci.org/ironSource/parquetjs)
[![License: MIT](https://img.shields.io/badge/License-MIT-green.svg)](https://opensource.org/licenses/MIT)
[![npm version](https://badge.fury.io/js/parquetjs.svg)](https://badge.fury.io/js/parquetjs)

This package contains a fully asynchronous, pure JavaScript implementation of
the [Parquet](https://parquet.apache.org/) file format. The implementation conforms with the
[Parquet specification](https://github.com/apache/parquet-format) and is tested
for compatibility with Apache's Java [reference implementation](https://github.com/apache/parquet-mr).

**What is Parquet?**: Parquet is a column-oriented file format; it allows you to
write a large amount of structured data to a file, compress it and then read parts
of it back out efficiently. The Parquet format is based on [Google's Dremel paper](https://www.google.co.nz/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwj_tJelpv3UAhUCm5QKHfJODhUQFggsMAE&url=http%3A%2F%2Fwww.vldb.org%2Fpvldb%2Fvldb2010%2Fpapers%2FR29.pdf&usg=AFQjCNGyMk3_JltVZjMahP6LPmqMzYdCkw).

Installation
------------

To use parquet.js with node.js, add this to your `package.json` and run `npm install`:

    "dependencies": {
      "parquetjs": "^0.0.1"
    }

Usage
-----

Once you have installed the parquet.js library, you can import it as a single
module:

    var parquet = require('parquetjs');

Parquet files have a strict schema, similar to a table in an SQL database. So,
in order to produce a parquet file we first need to declare a new schema.

Note the Parquet schema supports nesting, so you can store arbitrarily complex and
nested records in a single row (more on that later) while still maintaining good
compression.


Here is the full example to write out our 'fruits.parquet' file:

``` js
var schema = new parquet.ParquetSchema({
  "name": { type: "STRING" },
  "quantity": { type: "INT64" },
  "price": { type: "DOUBLE" },
  "date": { type: "TIMESTAMP" },
  "in_stock": { type: "BOOLEAN" }
});

var writer = new parquet.ParquetFileWriter(schema, 'test.parquet');
writer.appendRow({name: 'apples', quantity: 10, price: 2.5, date: +new Date(), in_stock: true});
writer.appendRow({name: 'oranges', quantity: 10, price: 2.5, date: +new Date(), in_stock: true});
writer.end();
```

Nested Rows & Arrays
--------------------

Parquet supports nested schemas that allow you to store rows that have a more
complex structure than a simple tuple of scalar values. To declare a schema
with a nested field, omit type `type` key and add a `fields` list instead:

Consider this example, which allows us to store a more advanced "fruits" table
where each row contains a name, a list of colours and a list of "stock" objects. 

``` js
// advanced fruits table
var schema = new parquet.ParquetSchema({
  "name": { type: "STRING" },
  "colour": { type: "STRING", repeated: true },
  "stock": {
    repeated: true,
    fields: [
      "price": { type: "DOUBLE" },
      "quantity": { type: "INT64" },
    ]
  }
});
```

The above schema allows us to store the following rows:

``` js
// Row 1: Banana
{
  name: "banana",
  colours: ["yellow"],
  stock: [
    { price: 2.45, quantity: 16 },
    { price: 2.60, quantity: 420 }
  ]
}

// Row 2: Apple
{
  name: "apple",
  colours: ["red", "green"],
  stock: [
    { price: 1.20, quantity: 42 },
    { price: 1.30, quantity: 230 }
  ]
}
```

It might not be obvious why one would want to implement or use such a feature when
the same can - in  principle - be achieved by serializing the record using JSON
(or a similar scheme) and then storing it into a STRING field.

Putting aside the philosophical discussion on the merits of strict typing,
knowing about the structure and subtypes of all records (globally) means we do not
have to store this date (i.e. the field names) for every record and on top of
that allows us to compress the remaining data far more efficiently.


Supported Types & Encodings
---------------------------

We aim to be feature-complete and add new features as they are added to the
parquet specification; this is the list of currently implemented data types and
encodings:

<table>
  <tr><th>Logical Type</th><th>Primitive Type</th><th>Encodings</th></tr>
  <tr><td>UTF8</td><td>BYTE_ARRAY</td><td>PLAIN</td></tr>
  <tr><td>JSON</td><td>BYTE_ARRAY</td><td>PLAIN</td></tr>
  <tr><td>BSON</td><td>BYTE_ARRAY</td><td>PLAIN</td></tr>
  <tr><td>BYTE_ARRAY</td><td>BYTE_ARRAY</td><td>PLAIN</td></tr>
  <tr><td>TIME_MILLIS</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>TIME_MICROS</td><td>INT64</td><td>PLAIN, RLE</td></tr>
  <tr><td>TIMESTAMP_MILLIS</td><td>INT64</td><td>PLAIN, RLE</td></tr>
  <tr><td>TIMESTAMP_MICROS</td><td>INT64</td><td>PLAIN, RLE</td></tr>
  <tr><td>BOOLEAN</td><td>BOOLEAN</td><td>PLAIN, RLE</td></tr>
  <tr><td>FLOAT</td><td>FLOAT</td><td>PLAIN</td></tr>
  <tr><td>DOUBLE</td><td>DOUBLE</td><td>PLAIN</td></tr>
  <tr><td>INT32</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT64</td><td>INT64</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT96</td><td>INT96</td><td>PLAIN</td></tr>
  <tr><td>INT_8</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_16</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_32</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_64</td><td>INT64</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_8</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_16</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_32</td><td>INT32</td><td>PLAIN, RLE</td></tr>
  <tr><td>INT_64</td><td>INT64</td><td>PLAIN, RLE</td></tr>
</table>


Depdendencies
-------------

Parquet uses [thrift](https://thrift.apache.org/) (basically facebook's version of
[protocol buffers](https://developers.google.com/protocol-buffers/)) to encode the schema
and other metadata, but the actual data does not use thrift.


License
-------

Copyright (c) 2017 ironSource Ltd.

Permission is hereby granted, free of charge, to any person obtaining a copy of
this software and associated documentation files (the "Software"), to deal in the
Software without restriction, including without limitation the rights to use,
copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the
Software, and to permit persons to whom the Software is furnished to do so,
subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED,
INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A
PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT
HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE
SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

