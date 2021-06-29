
## `local sqlpp = require'sqlpp'`

SQL Preprocessor.

Features:

 * `#if #elif #else #endif` conditionals
 * `$foo` text substitutions
 * `$foo(args...)` macro substitutions
 * `?` and `:foo` quoted-value substitutions
 * `??` and `::foo` quoted-name substitutions
 * `{foo}` unquoted substitutions
 * symbol values (for encoding `null` and `default`)
 * removing comments (for [mysql_client])
 * separating queries at `;`

## API Summary
------------------------------------------- ----------------------------------
`sqlpp.new() -> spp`                        create a preprocessor instance
`spp.query(sql, [params]) -> sql`           preprocess a query
`spp.queries(sql, [params]) -> {sql1,...}`  preprocess queries separated by `;`
`spp.params(sql, [params]) -> sql, names`   substitute params only
`spp.string(s) -> s`                        string formatting stub
`spp.number(x) -> s`                        number formatting stub
`spp.boolean(v) -> s`                       boolean formatting stub
`spp.name(s) -> s`                          name formatting stub
`spp.value(v) -> s`                         format any value
`spp.rows(rows[, indent]) -> s`             format a list of rows
`spp.tsv_rows(opt, s) -> rows`              convert a tab-separated list to a list of rows
`spp.tsv(opt, s) -> s`                      format a tab-separated list
`spp.keyword.KEYWORD -> symbol`             get a symbol for a keyword
`spp.keywords[SYMBOL] = keyword`            set a keyword for a symbol
`spp.subst'NAME text...'`                   create a substitution for `$NAME`
`function spp.macro.NAME(...) end`          create a macro to be used as `$NAME(...)`
__module system__
`function sqlpp.package.NAME(spp) end`      extend the preprocessor with a module
`spp.require(name)`                         load a module into a preprocessor state
__modules__
`spp.require'mysql_quote'`                  MySQL module for correct quoting
`spp.require'mysql_ddl'`                    MySQL module for DDL commands
`spp.require'mysql_domains'`                MySQL module for type domains
`spp.require'mysql'`                        MySQL module for all of the above
------------------------------------------- ----------------------------------

## Preprocessor

### `sqlpp.new() -> spp`

Create a preprocessor instance that modules can be loaded into.

### `spp.query(sql, [params]) -> sql`

Preprocess a SQL query, including hashtag conditionals, macro substitutions,
quoted and unquoted param substitutions, symbol substitutions and removing comments.

Notes on value expansion:

  * `{foo, bar}` expands to `foo, bar`, which is mostly useful for `in (?)`
  expressions (also because `{}` expands to `null` instead of empty string
  which would result in `in ()` which is invalid syntax in MySQL).

### `spp.queries(sql, [params]) -> {sql1, ...}`

Preprocess multiple SQL queries separated by `;`.

### `spp.keyword.KEYWORD -> symbol`

Get a symbol object for a keyword. Symbols are used to encode special SQL
values in parameter values, for instance using `spp.keyword.default`
as a parameter value will be substituted by the keyword `default`.

### `spp.keywords[SYMBOL] = keyword`

Set a keyword to be substituted for a symbol object. Since symbols can be
any Lua objects, you can add a symbol from an external library to be used
directly in SQL parameters. For instance:

```lua
pp.keywords[require'cjson'.null] = 'null'
```

This enables using JSON null values as SQL parameters.

### `spp.subst'NAME text...'`

Create an unquoted text substitution for `$NAME`.

### `function spp.macro.NAME(...) end`

Create a macro to be used as `$NAME(...)`. Param args are expanded before
macros.

## Modules

### `spp.require'mysql_quote'`

MySQL module for specific quoting of values: `inf` and `nan` expand to `null`,
booleans expand to `1` and `0`.

### `pp.require'mysql_ddl'`

MySQL module adding DDL commands.

Load the MySQL module

### `pp.require'mysql_domains'`
