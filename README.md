# Figg

- [Syntax](#syntax)
  - [Properties](#properties)
  - [Comments](#comments)
  - [Whitespace](#whitespace)
- [Value types](#value-types)
  - [Booleans](#booleans)
  - [Numbers](#numbers)
  - [Strings](#strings)
  - [Lists](#lists)
  - [Maps](#maps)
- [Examples](#examples)
  - [package.json](#packagejson)
  - [babel.config.js](#babelconfigjs)
  - [jest.config.js](#jestconfigjs)

## Syntax

### Properties

Properties are the most common feature, as they declare a key-value pair within its current [block scope](#maps), and are returned from the figg file when parsed. If defined at the top-level of a file, this is referred to as the root scope.

Declaring a property requires a unique name on the left-hand side (with no leading whitespace unless in a [map](#maps)), followed by a space, an equals sign operator (`=`) denoting assignment, another space, and lastly a [value](#value-types) on the right-hand side. An unquoted property name supports the characters `a-z`, `A-Z`, `0-9`, `_` and _must not_ begin with a number, while a quoted property name supports any character and _must_ be wrapped with double quotes.

```
propInCamelCase = 123
prop_in_snake_case = "abc"
"@scope/dep" = "v1.0.0"
```

Multiple properties are declared across multiple lines, with 1 property per line. Each property must have a trailing newline character (`\n`), excluding the last line in the file (which is optional).

```
propFoo = "foo"
propBar = "bar"
```

Invalid property names, duplicates, or whitespace will fail with a syntax error.

```
# INVALID, spaces required around = operator
noSpacing=123

# INVALID, must not start with a number
1stItem = "abc"

# INVALID, multiple properties must be on separate lines
propFoo = "foo" propBar = "bar"

# INVALID, property has already been declared
prop = 123
prop = 456
```

In the context of a language, these properties would be accessed as a key on an object or hashmap that was returned from a parser function. Let's demonstrate this with JavaScript.

```js
const cfg = parseFigg(`
foo = "abc"
bar {
	baz = 123
}
`);

cfg.foo; // -> "abc"
cfg.bar.baz; // -> 123
```

### Comments

Comments are declared with a leading hash and space (`# `), followed by the comment message. They can appear above any token, or trailing any line.

```
# Comment is above a property
prop = "above"

prop = "inline" # Comment is trailing a property
```

Multiple line comments are declared with a leading hash and space (`# `) for each line. There is no "doc block" like syntax.

```
# This comment will span multiple lines.
# So we prepend each line with a hash.
prop = "multi"
```

### Whitespace

A few rules around whitespace:

- There is _no_ limit to the amount of newlines that can exist between lines.
- Properties/Comments in the root scope must _not_ have a leading space or tab character.
- Properties/Comments in an object scope must be indented with a tab character (`\t`) for each object depth.
- Comments must have a space between the leading hash (`#`) and trailing message.

## Structure

## Value types

The following types are acceptable values to assign to both properties and variables.

### Booleans

Booleans are a primitive type for declaring a binary true/false (or on/off) value. The value is represented by the `true` and `false` keywords (lowercase only). 

```
propTruthy = true
propFalsy = false
```

### Numbers

Numbers are a [double-precision 64-bit binary format (IEEE 754)](https://en.wikipedia.org/wiki/Floating-point_arithmetic) value, like `double` in Java or C#. This means it can represent fractional values, but there are some limits to what it can store. A number only keeps about 17 decimal places of precision, with the largest value a number can hold being 1.8E308.

```
propInt = 123
propFloat = 456.78
```

For large numbers or better readability, the `_` character can be used to separate the digits per thousand.

```
propInt = 123_456_789 # 123,456,789
```

### Strings

TODO

### Lists

Lists are a structural type that contain other [values](#value-types). A list is declared with an opening bracket (`[`), followed by zero or more [values](#value-types) separated by trailing commas, and then a closing bracket (`]`). Lists can be declared inline or span multiple lines. When spanning multiple lines, each node must be indented with a tab character (`\t`), and the trailing comma on the last line is optional.

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

> Since lists are a structural type, the assignment operator (`=`) can be omitted when declaring a property.

### Maps

Maps are a structural type that contain nested [properties](#properties) and declare a block scope. A map is declared with an opening brace (`{`), followed by zero or more [properties](#properties) separated by newlines (`\n`), and then a closing brace (`}`). All nodes within the map must be indented with a tab character (`\t`), unless there's a single node, in which it can be inlined.

```
# Empty map
{}

# String value map
{
	foo = "a"
	bar = "b"
	"baz/qux" = "c"
}

# Inline map of 1 property
{ prop = 123 }
```

Maps can also contain other maps.

```
one {
	two {
		three {
			prop = "value"
		}
	}
}
```

> Since maps are a structural type, the assignment operator (`=`) can be omitted when declaring a property.

## Examples

### package.json

```
name = "figg"
description = "The coolest configuration format."
keywords = ["figg", "config"]
license = "MIT"
main = "./index.js"
peerDependencies {
	react = ">=17.0.0"
}
devDependencies {
	"@boost/common" = "^2.1.3"
}
```

### babel.config.js

```
plugins ["relay"]

presets [
	["@babel/preset-react", { runtime = "automatic" }],
	["@babel/preset-env", {
		modules = false
		targets { node = "current" }
	}],
]

overrides [
	{
		files = "**/*.ts"
		presets ["@babel/preset-typescript"]
	}
]
```

### jest.config.js

```
coverageThreshold {
	global {
		branches = 5
		functions = 5
		lines = 5
		statements = 5
	}
}

moduleNameMapper {
	"\\.(scss|css|jpg|jpeg|png|gif)$" = "identity-obj-proxy"
}

testEnvironment = "jsdom"
testRunner = "jest-circus/runner"
```
