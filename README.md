# Figg

- [Syntax](#syntax)
  - [Properties](#properties)
  - [Attributes](#attributes)
  - [Directives](#directives)
    - [schema](#schema)
    - [extends](#extends)
  - [Comments](#comments)
  - [Whitespace](#whitespace)
- [Value types](#value-types)
  - [Primitives](#primitives)
    - [Booleans](#booleans)
    - [Numbers](#numbers)
    - [Strings](#strings)
      - [Literals](#literals)
  - [Structurals](#structurals)
    - [Lists](#lists)
    - [Tuples](#tuples)
    - [Maps](#maps)
- [Schema](#schema)
  - [Primitives](#primitives1)
  - [Lists](#lists1)
  - [Tuples](#tuples1)
  - [Maps](#maps1)
  - [Properties](#properties1)
  - [Attributes](#attributes1)
- [Examples](#examples)
  - [package.json](#packagejson)
  - [tsconfig.json](#tsconfigjson)
  - [babel.config.js](#babelconfigjs)
  - [jest.config.js](#jestconfigjs)

## Syntax

### Properties

Properties are the most common feature, as they declare a key-value pair within its current [block scope](#maps), and are returned from the figg file when parsed. If defined at the top-level of a file, this is referred to as the document scope.

Declaring a property requires a unique name on the left-hand side (with no leading whitespace if in the root scope), followed by a space, and lastly a [value](#value-types) on the right-hand side. An unquoted property name supports the characters `a-z`, `A-Z`, `0-9`, `_` and _must not_ begin with a number, while a quoted property name supports any character and _must_ be wrapped with [double quotes](#strings) or [backticks](#literals).

```
propInCamelCase 123
prop_in_snake_case "abc"
"@scope/dep" "v1.0.0"
`\.(js|ts)$` "glob"
```

Multiple properties are declared across multiple lines, with 1 property per line. Each property must have a trailing newline character (`\n`), excluding the last line in the file (which is optional).

```
propOne "foo"
propTwo "bar"
```

In the context of a language, these properties would be accessed as a key on an object or hashmap that was returned from a parser function. Let's demonstrate this with JavaScript.

```js
const cfg = parseFigg(`
foo "abc"
bar {
	baz 123
}
`);

cfg.foo; // -> "abc"
cfg.bar.baz; // -> 123
```

### Attributes

Attributes are a mechanism for assigning metadata to [properties](#properties), and are declared above a property node with a leading `@`, followed by the attribute name. The attribute name supports the characters `a-z`, `A-Z`, `_`.

```
@production
prop "value"
```

Multiple attributes can be assigned to a property by stacking them vertically separated by a newline (`\n`) or horizontally separated by a space (`\s`).

```
@deprecated
@production
prop "value"

# Equivalent
@deprecated @production
prop "value"
```

By default, an assigned attribute has an internal value of `true`, but this value can be customized through a single argument passed to the attribute like a function call. This syntax requires an opening parenthesis (`(`) on the left-hand side, followed by a single [primitive value](#primitives), a closing parenthesis on the right-hand side (`)`), and must be declared after the attribute name.

```
@env("development")
prop "value"

@logLevel(5)
prop "value"
```

Because attributes assign unique metadata to properties, we now have the ability to declare multiple properties _of the same name_ that can be differentiated by consumers based on these attributes. For example, we can declare different property values per environment.

```
@development
logLevel "debug"

@staging
logLevel "warning"

@production
logLevel "error"
```

### Directives

While attributes define metadata for properties, a directive defines metadata, instructions, and other useful information for the document. A directive is defined at the top-level of the document, and must be declared before all other node types.

A directive is represented by a leading hash (`#`), followed by an opening bracket (`[`), the name of the direction, a closing bracket (`]`), a colon and space (`: `), and lastly the value of the directive. Since the directive is treated as a pseudo comment, the value is declared as-is, and does not follow any primitive value semantics.

```
#[example]: some value
```

> Custom directives are not allowed in user-land, instead Figg provides a handful of built-in directives for common scenarios.

#### schema

The `schema` directive declares structural type information for the current document by linking to a [`*.figgs` file](#schema) using a fully qualified HTTP(S) URL.

```
#[schema]: https://raw.githubusercontent.com/milesj/figg/master/example.figgs

prop "value"
```

#### extends

The `extends` directive is a built-in feature for declaring additional `*.figg` files that the current document will extend and merge with.

```
#[extends]: ./some/relative/path/to/example.figg

prop "value"
```

Multiple extends are supported by declaring a directive for each file path. The order in which these are defined does matter, as subsequent extends will merge over the previous, with the document itself merging over all extends.

```
#[extends]: ./some/relative/path/to/example.figg
#[extends]: ../../another/path/to/example.figg

prop "value"
```

### Comments

Comments are declared with a leading hash and space (`# `), followed by the comment message. They can appear above any token, or trailing any line. The space after the hash is required to disambiguate between directives.

```
# Comment is above a property
prop "above"

prop "inline" # Comment is trailing a property
```

Multiple line comments are declared with a leading hash and space (`# `) for each line. There is no "doc block" like syntax.

```
# This comment will span multiple lines.
# So we prepend each line with a hash.
prop "multi"
```

### Whitespace

A few rules around whitespace:

- There is _no_ limit to the amount of newlines that can exist between lines.
- Nodes in the root scope must _not_ have a leading space or tab character.
- Nodes in a nested scope (within lists and maps) must be indented with a tab character (`\t`) for each depth.
- Comments must have a space between the leading hash (`#`) and trailing message.

## Value types

The following types are acceptable values to assign to both properties and variables.

### Primitives

#### Booleans

Booleans are a primitive type for declaring a binary on/off value. The value is represented by the `true` and `false` keywords. 

```
enabled true
disabled false
```

#### Numbers

Numbers are a [double-precision 64-bit binary format (IEEE 754)](https://en.wikipedia.org/wiki/Floating-point_arithmetic) primitive value, like `double` in Java or C#. This means it can represent fractional values, but there are some limits to what it can store. A number only keeps about 17 decimal places of precision, with the largest value a number can hold being 1.8E308.

```
int 123
float 456.78
```

Numbers can also express binary (`0b`), octal (`0o`), and hexadecimal (`0x`) literals.

```
binary 0b10101101
octal 0o666
hex 0xdeadbeef
```

For large numbers or better readability, the `_` character can be used to separate the digits per thousand.

```
bigint 123_456_789 # 123,456,789
```

#### Strings

Strings are a primitive type that declare a sequence of UTF-8 characters, represented by a set of wrapping double quotes (`"`).

```
str "This is a string"
```

Strings can also span multiple lines without extra syntax. When using this form, newlines (`\n`) are converted into spaces (`\s`). Furthermore, when nested within a block scope, the additional lines must also be indented accordingly.

```
str "This is a 
string that contains
multiple lines"

block {
	str "This is a 
	string that contains
	multiple lines"
}
```

To preserve newlines (`\n`) and declare lengthy strings, like paragraphs, you can use the triple double quote syntax, which requires leading `"""\n` and trailing `\n"""` characters. The same indentation rules apply here as well.

```
para """
Lorem ipsum dolor sit amet, consectetur adipiscing elit. 
Sed posuere diam vitae posuere ornare. Nullam vel urna ex. 

Phasellus et ipsum quis justo vehicula interdum nec ut sem. 
Curabitur posuere fermentum facilisis.
"""
```

Nested double quotes and special characters must be [escaped](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#escape_sequences). To avoid escaping, you can use a [literal value](#literals).

```
double "This \"string\" has nested quotes"
single "This 'string' uses single quotes instead"
white "Has\ttabs and \nnew lines"
file "C:\\some\\path"
```

##### Literals

Literals are a special kind of string that do _not_ allow escaping or newlines (`\n`). This is perfect for Windows style paths, regex patterns, shell scripts, and more. Literals are represented by wrapping backticks instead of double quotes.

```
str `This contains "double" and 'single' quotes`
file `C:\some\path`
regex `foo/.*?`
```

> If your literal contains a backtick, unfortunately you'll need to use a non-literal string and apply that correct escaping.

### Structurals

#### Lists

Lists are a structural type that contain other [values](#value-types) of a _variable_ length, with all the values being of a specific type. A list is declared with an opening bracket (`[`), followed by zero or more [values](#value-types) separated by trailing commas, and then a closing bracket (`]`). Lists can be declared inline or span multiple lines. When spanning multiple lines, each node must be indented with a tab character (`\t`), and the trailing comma on the last line is optional.

```
# Empty list
[]

# Number list inline
[1, 2, 3]

# String list multiline
[
	"foo",
	"bar",
	"baz",
]
```

Lists can also contain other lists.

```
matrix [
	[0, 1, 2],
	[3, 4, 5],
	[6, 7, 8],
]
```

#### Tuples

Tuples are a structural type that contain other [values](#value-types) of a _fixed_ length, with every value being a mixed type. A tuple is declared with an opening bracket (`(`), followed by one or more [values](#value-types) separated by trailing commas, and then a closing bracket (`)`). Tuples can be declared inline or span multiple lines. When spanning multiple lines, each node must be indented with a tab character (`\t`), and the trailing comma on the last line is optional.

```
# Tuple inline
(123, "name")

# Tuple multiline
(
	123,
	"name",
	{ prop "value" },
)
```

Tuples can also contain other tuples.

```
matrix (
	(0, 1, 2),
	(3, 4, 5),
	(6, 7, 8),
)
```

#### Maps

Maps are a structural type that contain nested [properties](#properties) and declare a block scope. A map is declared with an opening brace (`{`), followed by zero or more [properties](#properties) separated by newlines (`\n`), and then a closing brace (`}`). All nodes within the map must be indented with a tab character (`\t`), unless there's a single node, in which it can be inlined.

```
# Empty map
{}

# String value map
{
	foo "a"
	bar "b"
	"baz/qux" "c"
}

# Inline map of 1 property
{ prop 123 }
```

Maps can also contain other maps.

```
one {
	two {
		three {
			prop "value"
		}
	}
}
```

## Schema

A schema is a secondary file with the extension `.figgs`, that can be coupled with the primary `.figg` file to provide structural type information. This information can then be fed into editors and tooling to provide static analysis, linting, type checking, autocompletion, and more. A schema defines types and shapes similar to other languages like TypeScript or GraphQL.

### Primitives

The primitive types are represented with the `boolean`, `number`, and `string` keywords respectively. They are the lowest level of type and are always used for composition and assignment.

```
document {
	propOne boolean
	propTwo number
	propThree string
}
```

### Lists

### Tuples

### Maps

## Examples

### package.json

Document:

```
name "figg"
description "The coolest configuration format."
keywords ["figg", "config"]
license "MIT"
main "./index.js"
peerDependencies {
	react ">=17.0.0"
}
devDependencies {
	"@boost/common" "^2.1.3"
}
```

Schema:

```
document {
	name string
	description string
	keywords List<string>
	license string
	main string
	peerDependencies Map<string>
	devDependencies Map<string>
}
```

### tsconfig.json

Document:

```
#[extends]: ./tsconfig.options.json

include ["**/*"]

exclude [
	"build/**/*",
	"node_modules"
]

compilerOptions {
	strict true
	esModuleInterop true
	module "esnext"
	paths {
		"~/*": ["src/*"]
	}
}
```

Schema:

```
type ModuleType = "esnext" | "commonjs"

type CompilerOptions = {
	strict boolean
	esModuleInterop boolean
	module ModuleType
	paths: Map<List<string>>
}

document {
	include List<string>
	exclude List<string>
	compilerOptions CompilerOptions
}
```

### babel.config.js

Document:

```
plugins ["relay"]

presets [
	("@babel/preset-react", { runtime "automatic" }),
	("@babel/preset-env", {
		modules false
		targets { node "current" }
	}),
]

overrides [
	{
		files ["**/*.ts"]
		presets ["@babel/preset-typescript"]
	}
]
```

Schema:

```
type EntryWithOptions = Tuple<string, Map>

type Entry = string | EntryWithOptions

type Override = {
	files List<string>
	plugins List<Entry>
	presets List<Entry>
}

document {
	plugins List<Entry>
	presets List<Entry>
	overrides List<Override>
}
```

### jest.config.js

Document:

```
coverageThreshold {
	global {
		branches 5
		functions 5
		lines 5
		statements 5
	}
}

moduleNameMapper {
	`\.(scss|css|jpg|jpeg|png|gif)$` "identity-obj-proxy"
}

testEnvironment "jsdom"
testRunner "jest-circus/runner"
```

Schema:

```
type Threshold = {
	branches number
	functions number
	lines number
	statements number
}

document {
	coverageThreshold Map<Threshold>
	moduleNameMapper Map<string>
	testEnvironment string
	testRunner string
}
```
