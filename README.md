# func-e
An experimental hobbyist language with a focus on events.

FuncE: The Functional Event Language
Version 0.1

# Design

## Syntax

Leading whitespace-significant. 4 spaces = 1 tab.

Prefer capitalization:

* Types and Events: Pascal
* Functions and fields: camel

## Keywords

* `var` - declares a varying field or local
* `new` - declares a non-varying field or local
* `set` - changes a field or local's value to the RHS at the time of evaluation
* `let` (not in this version) - changes a field or local's value to always reflect the RHS
* `pure` - handler or function always returns the same value for the same input; it does not reference global state
* `then` - after handling the event previously specified, handle this event
* `if` - evaluates a Bool expression; if true, continue into next indent, otherwise skip
* `else` - paired with `if`, allows different indent if the condition was false
* `else if` - paired with `if`, allows another Bool expression to be evaluated recursively
* `loop` - loops by a condition, optionally with a repetition instruction
* `return` - exits a function returning the specified value

# Example

	var Int32 charactersEntered = 0;
	var Int32 linesEntered = 0;
	var Int32 minCharacters = 2000;
	var Int32 maxCharacters = 0;
	
	pure Enter<>
		LineOut "Init with: " + paramLine; // implicit use of event's parameter
		then LineOut "Type lines, or type an empty line to quit.";
	
	LineIn<line.Count != 0>
		set charactersEntered += line.Count;
		set linesEntered++;
		if {minCharacters < line.Count < maxCharacters}
			LineOut "Fitting in!"
		if line.Count < minCharacters
			set minCharacters = line.Count;
		if line.Count > maxCharacters
			set maxCharacters = line.Count;
	
	LineIn<line.Count == 0>
		new mean = getMean(charactersEntered, linesEntered);
		new range = minCharacters + (maxCharacters - minCharacters);
		new toWrite = "
			Finished. You wrote" + charactersEntered + " chars, over " + linesEntered + " lines.
			That's " + mean + " characters per line! Range is like " + range "!
		"
		LineOut toWrite
		then Exit 0

	pure Dec128 getMean(Int32 sum, Int32 numEntries)
		return sum / numEntries;

# Builtins

## Types

* Object
	* Number
		* Int32, Literal: `0` or `[1-9][0-9]*`
		* Dec128, Literal: `[1-9][0-9]*.[0-9]+` or `0.[0-9]+`
* Bool, Literal: `false` or `true`
* String, Literal: 

## Top-level Declarations

* Global Field
	* `[var|new] type id = initialValue`
* Handler
	* `pure? eventType<restriction?>`
* Function
	* `pure? returnType id(params)`

## Statements

* Event Raise
	* `then? eventType parameter parameter;`
* Assignment
	* `[set|let] id = newValue;`
	* `set id += newValue;` (or `-=`)
	* `set id++;` or `--`
* Local declaration
	* `[var|new] type? id = initialValue;`
* Conditional
	* `if boolExpression \t body`
	* `if boolExpression \t body1 un-\t else \t body2` (and else if)
* Loop
	* `loop (condition) \t body`
	* `loop (condition; statementsAfterContinue) \t body`
* Return
	* `return expression`

## Expressions

1. Object orientation (left)
	* Method call `id(params)` -> whatever type the method returns
	* Member access `id.id2` -> whatever type the member is
2. Factorwise (left)
	* Multiplication 
		* `Int32 * Int32` -> Int32
		* `Int32 * Dec128` or `Dec128 * Int32` -> Dec128
		* `Dec128 * Dec128` -> Dec128
	* Division
		* `Number / Number` -> **Dec128**
	* Integer division
		* Same as division, but with operator `/i` and always returns `Int32`
3. Termwise (left)
	* Addition
		* Same as multiplication, but with operator `+`
	* Subtraction
		* Same as addition, but with operator `-`
	* String concatenation
		* `Object + String` or `String + Object` -> String
4. Comparison (left)
	* `<=` `>=` `<` `>`
		* `Number op Number` -> Bool
		*  `{Number <= Number <= Number}` -> Bool
	*  `==` `!=`
		*  `Object op Object` -> Bool
5. Logical Combination (left)
	* `!`
		* `!Bool` -> Bool
	* `&`
		* `Bool & Bool` -> Bool
	* `|`
		* `Bool | Bool` -> Bool
	* `^`
		* `Bool ^ Bool` -> Bool

Assignment is a statement type, not an expression. Bitwise ops are not language features.

## Globals

* Bool `true`
* Bool `false`

## Events

* Enter<String paramLine>
* Exit<Int32 exitCode>
* LineIn<String line>
* LineOut<String line>

## Handlers

* Exit: stops execution with the specified exitCode
* LineOut: prints line and an endline character to stdout

## Emitters

* On app startup, emit `Enter` with paramLine being the args on command line
* On new line of input from stdin, emit `LineIn` with line being that line

# Not in this Version

* `let` (reactive)
* User-defined events/types
* Other predefined types
* Smarter/configurable event dispatching
	* Domains
	* Call only one handler
	* Waiting for handler before continuing execution (await)
* Multithreaded handlers
* Exceptions
* Multiple files
* Compilation instead of interpretation
* .NET Interop
* Aggregate types (arrays, lists, etc.)
