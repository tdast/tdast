# ![tdast](./logo.png)

**T**abular **D**ata **A**bstract **S**yntax **T**ree format.

---

**tdast** is a specification for representing tabular data as an abstract
[syntax tree][syntax-tree].
It implements the **[unist][]** spec.

This document may not be released.  See [releases][] for released documents.

## Contents

*   [Introduction](#introduction)
    *   [Where this specification fits](#where-this-specification-fits)
    *   [Scope](#scope)
*   [Nodes](#nodes)
    *   [`Parent`](#parent)
    *   [`Literal`](#literal)
    *   [`Table`](#table)
    *   [`Row`](#row)
    *   [`Cell`](#cell)
    *   [`Column`](#column)
*   [Glossary](#glossary)
*   [List of utilities](#list-of-utilities)
*   [Related](#related)
*   [License](#license)

## Introduction

This document defines a format for representing tabular data as an [abstract syntax
tree][syntax-tree].  This specification is written in the style of other [syntax-tree][] specs, for potential incorporation into the ecosystem in the future.  Development started in September 2020.

### Where this specification fits

tdast extends [unist][], a format for syntax trees, and benefits from its
[ecosystem of utilities][unist-utilities].

tdast relates to [JavaScript][] in that it has an [ecosystem of
utilities][unist-utilities] for working with compliant syntax trees in
JavaScript.  However, tdast is not limited to JavaScript and can be used in other programming
languages.

tdast relates to the [unified][] project in that tdast syntax trees follow the [unist][] spec and is compatible with utilities throughout its ecosystem.

### Scope

tdast seeks to represent how tabular data from some source content (e.g. CSV, JSON), would be represented in a syntax tree.  It implements the [unist][] spec, meaning that the tree would track positional information of nodes that will be useful for remaining interoperable with unist utilities.

With associated utilities, tdast enables flexible ways to parse, transform, serialize tabular data in various content formats.  CSV, JSON, HTML tables are immediate native candidates benefiting from tdast, and future content types can be supported by implementors.

While it is possible to perform data calculations in tdast with associated utilities, it is recommended that you work with more direct data representations of the source content for these purposes.  tdast utilites to convert to/from JS objects exist for this purpose.

While tdast could be used to represent most tabular data, it is best used with simple cases of flat tabular data.


## Nodes

### `Parent`

```ts
interface Parent extends UnistParent {
  /** Array of child nodes */
  children: Array<Row | Cell | Column>;
  /** Additional data maybe attached. */
  data?: Data;
}
```

`Parent` ([`UnistParent`][dfn-unist-parent]) is an abstract node containing other nodes (said to be children).

This node is not used directly in tdast and is defined as an abstract interface.

### `Literal`

```ts
interface Literal extends UnistLiteral {
  /** Primary data value of a literal node. Loosely typed to `any` for convenience. */
  value: any;
  /** Additional data maybe attached. */
  data?: Data;
}
```

`Literal` ([`UnistLiteral`][dfn-unist-literal]) is an abstract node in containing a value.

This node is not used directly in tdast and is defined as an abstract interface.

### `Table`

```ts
interface Table extends Parent {
  /** Table node type. */
  type: 'table';
  /** Can only contain Rows for children. */
  children: Row[];
}
```

`Table` ([`Parent`][parent]) represents the node that holds all tabular data with [`Row`][row] nodes.

`Table` can be used as the [root][dfn-unist-root] of a [tree][dfn-unist-tree], but never as a [child][dfn-unist-child].


### `Row`

```ts
interface Row extends Parent {
  /** Row node type. */
  type: 'row';
  /** Index of Row in relation to other Rows in a Table. */
  index: number;
  /** Can only contain Cells or Columns for children. */
  children: Array<Cell | Column>;
}
```

`Row` ([`Parent`][parent]) holds literal nodes that contain data, such as [`Column`][column] or [`Cell`][cell] nodes.

`Row` contains an `index` field that tracks its relative position with other rows under a [`Table`][table].

### `Cell`

```ts
interface Cell extends Literal {
  /** Cell node type. */
  type: 'cell';
  /** Tracks which Column the Cell belongs to */
  columnIndex: number;
  /** Tracks which Row the Cell belongs to */
  rowIndex: number
}
```

`Cell` ([`Literal`][literal]) represents the intersection of a [`Row`][row] and [`Column`][column] of a [`Table`][table].  This intersection information is stored on the `columnIndex` and `rowIndex` properties.

Its primary data is stored on the `value` property.  `Cell` can also contain attach additional data (not relevant to tdast) under the optional `data` property.

### `Column`

```ts
interface Column extends Literal {
  /** Column node type. */
  type: 'column';
  /** Display label of a column. */
  label: string;
  /** Index of Column in relation to other Columns in a Table. */
  index: number;
  /** Optional data type useful to determine data types of Cells matching the Column. */
  dataType?: string;
}
```

`Column` ([`Literal`][literal]) is usually reserved in the first [`Row`][row] of a [`Table`][table].

`Column` contains an `index` field that tracks its relative position with other columns under a [`Table`][table].  Its primary data is represented in the `value` property.  It should have a display `label` specified as a string.

`Column` can optionally specify the `dataType` property, which informs the data types applied to [`Cell`s][cell] in a [`Row`][row].


## Glossary

- **`ast`**: a data structure representing source content as an abstract syntax tree.
- **`cell`**: the intersection of a table row and column.  A cell contains data.
- **`column`**: a table column provides definitions for cells in subsequent rows.  A column should define the `dataType` of cells under it.  The cardinality of a column is equal to the number of rows in a table.
- **`csv`**: a common tabular data format that uses comma-separated values for delimiting values.
- **`dataType`**: refers to the data type a table cell assumes.  This is usually defined by a table column.
- **`row`**: a table row contains cells.  The cardinality of a row is equal to the number of columns in a table.  The `dataType` of each cell should assume what is prescribed by the columns.
- **`table`**: an arrangement of data in rows and columns.
- **`tdast`**: represent tabular data as an abstract syntax tree with tdast.
- **`unist`**: [universal syntax tree][unist] that tdast is based on.


## List of utilities

- [`tdastscript`][tdastscript]: utility to create tdast trees.
- [`tdast-util-from-array`][tdast-util-from-array]: transform from JS array to tdast
- [`tdast-util-from-csv`][tdast-util-from-csv]: parse from CSV to tdast ([RFC-4180][] compliant)
- [`tdast-util-to-array`][tdast-util-to-array]: transform tdast to JS array, which can be manipulated externally and read back with `tdast-util-from-array`.
- [`tdast-util-to-csv`][tdast-util-to-csv]: serialize tdast to CSV ([RFC-4180][] compliant)
- [`tdast-util-to-hast-table`][tdast-util-to-hast-table]: transform tdast to a [hast][] table node
- [`tdast-util-to-html-table`][tdast-util-to-html-table]: serialize tdast to a HTML table
- [`tdast-util-to-json`][tdast-util-to-json]: serialize tdast to JSON (array form)
- [`tdast-util-to-markdown-table`][tdast-util-to-markdown-table]: serialize tdast to a markdown table


## Related
- [unist][]: Universal Syntax Tree format
- [tdastscript][]: utility to create tdast trees.


## License

[CC-BY-4.0][license] © [Chris Zhou][author]

<!-- Definitions -->
[cell]: #cell
[column]: #column
[literal]: #literal
[parent]: #parent
[row]: #row
[table]: #table

[author]: https://chrisrzhou.io
[dfn-unist-child]: https://github.com/syntax-tree/unist#child
[dfn-unist-literal]: https://github.com/syntax-tree/unist#literal
[dfn-unist-parent]: https://github.com/syntax-tree/unist#parent
[dfn-unist-root]: https://github.com/syntax-tree/unist#root
[dfn-unist-tree]: https://github.com/syntax-tree/unist#tree
[hast]: https://github.com/syntax-tree/hast
[license]: https://creativecommons.org/licenses/by/4.0/
[javascript]: https://www.ecma-international.org/ecma-262/9.0/index.html
[releases]: https://github.com/tdast/tdast/releases
[rfc-4180]: https://tools.ietf.org/html/rfc4180
[syntax-tree]: https://github.com/syntax-tree/unist#syntax-tree
[tdastscript]: https://github.com/tdast/tdastscript
[tdast-util-from-array]: https://github.com/tdast/tdast-util-from-array
[tdast-util-from-csv]: https://github.com/tdast/tdast-util-from-csv
[tdast-util-to-array]: https://github.com/tdast/tdast-util-to-array
[tdast-util-to-csv]: https://github.com/tdast/tdast-util-to-csv
[tdast-util-to-hast-table]: https://github.com/tdast/tdast-util-to-hast-table
[tdast-util-to-html-table]: https://github.com/tdast/tdast-util-to-html-table
[tdast-util-to-json]: https://github.com/tdast/tdast-util-to-json
[tdast-util-to-markdown-table]: https://github.com/tdast/tdast-util-to-markdown-table
[unified]: https://github.com/unifiedjs/unified
[unist]: https://github.com/syntax-tree/unist
[unist-utilities]: https://github.com/syntax-tree/unist#list-of-utilities
[xast]: https://github.com/syntax-tree/xast
