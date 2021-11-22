# Pattern Folding Parameters by Example

The folding rules for a language are specified in its "language-configuration.json" file as a series of rules:
```json
	"patternFolding": {
		"rules": [
			{ ... },
			{ ... },
			...
		]
	},
```

The examples below illustrate a number of ways to specify folding rules. Most examples are for the "C" language. At the end there is a full example for "C".

Familiarity with regular expressions is assumed for these examples.

## Simple Block Fold
```json
	{
		"foldFrom": ["{"],
		"foldTo": ["}"]
	},
```
This rule specifies that the set of lines from one "{" to a matching "}" are to be a foldable region.

Foldable regions are nested. A "{" which follows another before its matching "}" creates a new foldable region contained within the outer one.

When a region is folded all lines between the one containing the `foldFrom` and the `foldTo` values are hidden. The line containing the `foldTo` value is also hidden if it does not have any nonblank text following the `foldTo` value. When the `foldTo` line is hidden and is followed by multiple blank lines, all blank lines except one are also hidden.

## Regular Expressions
```json
	{
		"foldFrom": ["^\\s*#(pragma\\s+)?region.*$"],
		"foldTo": ["^\\s*#(pragma\\s+)?endregion.*$"],
	    "type": "region"
	},
```
All values in folding rules are regular expressions unless otherwise noted.

This example uses regular expressions to allow for a number of ways that "regions" can be defined in C. (Optional whitespace at start of line, then either `#pragma region` or just `#region`.)

The double `\\` in the example allows for these being literals in a json file. The parsing of the quoted json value will convert each `\\` to a single `\` in the actual regular expression.

This example also adds the `"type"` parameter. This can be used to associate a rule with a type name which is a string value. In future vscode might have commands which enable folding of all regions of a given type. At the moment this feature exists only for type "comment".

## Multiple From/To Values
```json
	{
		"foldFrom": ["^\\s*#if", "^\\s*#else", "^\\s*#elif"],
		"foldTo": ["^\\s*#else", "^\\s*#elif", "^\\s*#endif"]
	},
```
All regular expression values in the configuration can be lists of values. When more than one value is specified they are an "or" list - the rule applies when any expression in the list is matched.

This example also shows that the same value can be used both as a `foldFrom` and as a `FoldTo`.

The values `#else` and `#elif` mark the end of a previous region of this type and also mark the start of a new region. Because they end a previous region defined by this rule they do not create a nested fold. And because they start a new region they don't end a nested fold.

The value `#if` only marks the start of a new region and thus it can be nested.

The value `#endif` only marks the end of a region and thus will end a nested case of this rule.

## Headings and Subheadings
```json
	{
		"foldAt": ["(?<![-A-Z0-9])DIVISION(?![-$A-Z0-9])"]
	},
	{
		"foldAt": ["(?<![-A-Z0-9])SECTION(?![-$A-Z0-9])"],
		"foldInside": ["(?<![-A-Z0-9])DIVISION(?![-$A-Z0-9])"]
	},
```
We've switched to COBOL for this example. The regular expressions above use look-behind and look-ahead to allow for COBOL's syntax. For many languages we could just use values like `"\\bDIVISION\\b"` to specify keywords.

In some languages and in text documents and files such as transaction logs it can be useful to define fold regions based only on "start" indicators. I.e. to describe a structure which is like layers of headings and subheadings.

We already did that in a way in the example with `#else` and `#elif`. In this example we are being more explicit about it.

The `foldAt` keyword is just shorthand for saying that the specified value(s) are to be used both as `foldFrom` and `foldTo` values. Each value specified in `foldAt` both ends a foldable region and begins a new one.

The `foldInside` keyword specifies something new. It says that regions defined by this rule also end before these additional values. In this example `DIVISION` is like a Heading1 and `SECTION` is like a Heading2. Divisions end at the start of the next division. Sections end at the start of the next section or the start of the next division, whichever comes first.

## Groups
```json
	{
		"foldFrom": ["^\\s*#include"],
		"foldTo": ["$"],
		"group": "directive"
	},
```
The `group` parameter indicates that "groups" of lines in the source which are matched by the rule are to be foldable as a unit.

A sequence of lines is treated as a group when they satisfy the rule repeatedly with no intervening text or blank lines.

The value of the `"group"` parameter gives the group a name. Multiple rules can specify the same group name. When they do then consecutive lines which match any of those rules are grouped into a single foldable region.

## Comments
```json
	{
		"commentFrom": ["/\\*"],
		"commentTo": ["\\*/"],
		"type": "comment"
	},
```
It is important to specify rules for comments because:
* We don't want to mistakenly treat something in a comment as the start or end of a foldable region. All other rules are ignored for the text in comments.
* Comments do not interfere with other rules. For example a `"group"` of lines will still satisfy the group rule if there are comments present on the grouped lines.

## Strings
	{
		"stringFrom": ["\""],
		"stringTo": ["\""],
		"escape": "(?<!\\\\)(\\\\\\\\)*\\\\\""
	}
As with comments, we don't want to mistake something in a string literal as representing the start or end of a foldable region. All other rules are ignored for the text in strings.

The `"escape"` parameter defines the value(s) which can be used to place a string delimiter (`"` in this case) inside a string without ending it. The regular expression above matches any case of `\"` in a string which is not preceded by an odd number of `\`.

## Full Example

A configuration for "C" folding rules:
```json
"patternFolding": {
	"rules": [
		{
			"foldFrom": ["{"],
			"foldTo": ["}", "};"]
		},
		{
			"foldFrom": ["\\["],
			"foldTo": ["\\]", "\\];"]
		},
		{
			"foldFrom": ["\\("],
			"foldTo": ["\\)", "\\);"]
		},
		{
			"foldFrom": ["^\\s*#if", "^\\s*#else", "^\\s*#elif"],
			"foldTo": ["^\\s*#else", "^\\s*#elif", "^\\s*#endif"]
		},
		{
			"foldFrom": ["^\\s*#(pragma\\s+)?region.*$"],
			"foldTo": ["^\\s*#(pragma\\s+)?endregion.*$"],
		    "type": "region"
		},
		{
			"foldFrom": ["^\\s*#include"],
			"foldTo": ["$"],
			"group": "directive"
		},
		{
			"commentFrom": ["//"],
			"commentTo": ["$"],
			"type": "comment",
			"group": "comment"
		},
		{
			"commentFrom": ["/\\*"],
			"commentTo": ["\\*/"],
			"type": "comment"
		},
		{
			"stringFrom": ["\""],
			"stringTo": ["\""],
			"escape": ["(?<!\\\\)(\\\\\\\\)*\\\\\""]
		}
	]
}
```
