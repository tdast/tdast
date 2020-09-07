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

tdast relates to the [unified][] project in that tdast syntax trees follow the unist spec and is compatible with utilities throughout its ecosystem.

### Scope

tdast seeks to represent how tabular data from some source content (e.g. CSV, JSON), would be represented in a syntax tree.  It implements the [unist][] spec, meaning that the tree would track positional information of nodes that will be useful for remaining interoperable with unist utilities.

With associated utilities, tdast enables flexible ways to parse, transform, serialize tabular data in various content formats.  CSV, JSON, HTML tables are immediate native candidates benefiting from tdast, and future content types can be supported by implementors.

While it is possible to perform data calculations in tdast with associated utilities, it is recommended that you work with more direct data representations of the source content for these purposes.  tdast utilites to convert to/from JS objects exist for this purpose.

While tdast could be used to represent most tabular data, it is best used with simple cases of flat tabular data.


## Nodes

### `Parent`

```ts
interface Parent extends UnistParent {
  children: Array<Row | Column | Cell>;
}
```

**Parent** ([**UnistParent**][dfn-unist-parent]) represents a node in tdast containing other nodes (said to be children).

Its content is limited to only other tdast content.

### `Literal`

```ts
interface Literal extends UnistLiteral {
  value: string;
}
```

**Literal** ([**UnistLiteral**][dfn-unist-literal]) represents a node in tdast containing a value.

Its value field is a string.

### `Table`

```ts
interface Table extends Parent {
  type: 'table';
  children: Array<Row>;
}
```

**Table** ([**Parent**][parent]) represents the node that holds all tabular data, explicitly [**row**][row] nodes.

**Table** can be used as the [root][dfn-unist-root] of a [tree][dfn-unist-tree], never as a [child][dfn-unist-child].


### `Row`

```ts
interface Row extends Parent {
  type: 'row';
  index: number;
  children: Array<Column | Cell>;
}
```

**Row** ([**Parent**][parent]) holds literal nodes containing data, explicitly [**column**][column] or [**cell**][cell] nodes.

**Row** contains an `index` field that tracks its relative position with other rows under a [**table**][table].

### `Cell`

```ts
interface Cell extends Literal {
  type: 'cell';
  value: string;
  data?: any;
}
```

**Cell** ([**Literal**][literal]) represents the intersection of a [**row**][row] and [**column**][column] of a table.  Its primary data/value is represented in the `value` property.

It can also contain attach additional data (not relevant to tdast) under the optional `data` property.

### `Column`

```ts
interface Column extends Cell {
  type: 'column';
  label: string;
  index: number;
  dataType?: string;  // e.g. 'string' | 'number' | 'boolean' | 'null' | 'undefined' | 'nan'
}
```

**Column** ([**Cell**][cell]) is a [**cell**][cell] that is usually reserved in the first [**row**][row] of a [**table**][table].  Its primary data/key is represented in the `value` property.  It should have a display `label` specified as a string.

**Column** can optionally specify the `dataType` property, which informs the data types applied to [**cells**][cell] in a [**row**][row].

**Column** contains an `index` field that tracks its relative position with other columns under a [**table**][table].

## Glossary

- **`cell`**: the intersection of a table row and column.  A cell contains data.
- **`column`**: a table column contains cells.  A column should define the `dataType` of cells under it.  The cardinality of a column is equal to the number of rows in a table.
- **`csv`**: comma-separated values file/content.
- **`dataType`**: refers to the data type a table cell assumes.  This is usually defined by a table column.
- **`hast`**: hypertext abstract syntax tree.  See the official [spec][hast] for more details.
- **`mdast`**: markdown abstract syntax tree.  See the official [spec][mdast] for more details.
- **`row`**: a table row contains cells.  The cardinality of a row is equal to the number of columns in a table.  The `dataType` of each cell should assume what is prescribed by the columns.
- **`table`**: an arrangement of data in rows and columns.
- **`tdast`**: represent tabular data as an abstract syntax tree with tdast.


## List of utilities

> Note: The following utilities are a work in progress. They will be linked when implemented.

- `tdast-util-from-object`: parse from JS object to tdast
- `tdast-util-from-csv`: parse from CSV to tdast
- `tdast-util-from-json`: parse from JSON to tdast
- `tdast-util-to-hast-table`: transform tdast to a hast table node
- `tdast-util-to-html-table`: serialize tdast to a HTML table
- `tdast-util-to-markdown-table`: serialize tdast to a markdown table
- `tdast-util-to-csv`: serialize tdast to CSV
- `tdast-util-to-object`: transform tdast to JS object, which can be manipulated externally and read back with `tdast-util-from-object`.
- `tdast-util-to-json`: serialize tdast to JSON
- `tdast-util-find-cells`: return cell nodes that pass a provided row/column test.
- `tdast-util-reduce`: compute a reduced value on rows/columns with the provided reducer.
- `tdast-util-filter`: filter/drop rows/columns that pass a test.
- `tdast-util-sort`: sort rows/columns based on a sorting function


## Related
- [unist][]
    — Universal Syntax Tree format
- [hast][]
    — Hypertext Abstract Syntax Tree format
- [mdast][]
    — Markdown Abstract Syntax Tree format
- [nlcst][]
    — Natural Language Concrete Syntax Tree format


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
[mdast]: https://github.com/syntax-tree/mdast
[nlcst]: https://github.com/syntax-tree/nlcst
[releases]: https://github.com/tdast/tdast/releases
[syntax-tree]: https://github.com/syntax-tree/unist#syntax-tree
[unified]: https://github.com/unifiedjs/unified
[unist]: https://github.com/syntax-tree/unist
[unist-utilities]: https://github.com/syntax-tree/unist#list-of-utilities
[xast]: https://github.com/syntax-tree/xast
