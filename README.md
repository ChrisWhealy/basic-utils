# Basic Formatting Utilities

## Overview

This NPM module is a development/debugging tool designed to display the contents of one or more server-side objects as HTML tables.

This utility is designed to run inside a stateless container.  Therefore, the generated HTML has been designed such that it will ***not*** make any further calls back to the app that generated it because that app probably no longer exists.

The screen shot below shows what the first few properties of the NodeJS `process` object might look like when displayed using the HTML generated by this utility.

![Screen shot](./img/screenshot.png)

### Behaviour

* Object properties are listed in alphabetic order
* Only object properties of type `Object`, `Array`, `Map` or `Set` are considered expandable
* Click on the expand/collapse button in the `Type` column to hide or display this data
* By default:
    * Expandable object properties will be displayed in a collapsed state
    * Object properties of types `Function` and `GeneratorFunction` will be suppressed from the display
    * If you choose to display functions names (by calling `show_fns()`), their source code of will not be displayed

### But Why?

When developing NodeJS apps, I frequently want to see inside the server-side objects with which I'm working.  This is particulary true when I'm learning my way around a new framework.

To see inside objects, people typicially dump the object contents to the console using `JSON.stringify`; however, this approach has two problems:

1. The string representation created by `JSON.stringify` (even with the formatting options) can be difficult to read - especially if the object is large and deep
1. There is a more serious problem however: `JSON.stringify()` explodes when passed an object containing circular references

Therefore, rather than resorting to the tedious, trial-and-error approach of repeatedly calling `JSON.stringify` within a `try`/`catch` block, I decided to create a more user-friendly result by transforming the object into an HTML table whose rows could be expanded or collapsed as necessary.

## Usage

The `show_object` function transforms a JavaScript object into an HTML table.  If any property of this object is considered expandable (I.E. is of type `Map`, `Object`, `Array` or `Set`), then this property value will itself be transformed into a nested HTML table.

Transformation of expandable properties continues recursively until the pre-determined depth limit is reached.  This is how I avoid circular references blowing up in my face...

By default, the depth limit is set to `3`, but this can be adjusted by passing a positive integer to `set_depth_limit()`

By default, object properties of type `Function` or `GeneratorFunction` are suppressed altogether from the display.  If however, you wish to display these properties, then call function `show_fns()`.  

> ***IMPORTANT***  
> Function source code will never be displayed.

Typically, the HTML generated by this utility will form just a fragment of some larger Web page.

### Example 1: Display a Single Object as an HTML `<DIV>`

Call `show_object` to transform the contents a server-side object into an HTML `<DIV>` containing a `<TABLE>`:

```javascript
var bfu     = require('basic-formatting-utils')
var request = ... /* Get an HTTP request object from somewhere */

var obj_as_html_div = bfu.show_object("HTTP Request", request)
```

### Example 2: Display a Single Object as an Entire HTML Page

Enclose the call to `show_object` within calls to `as_body` and `as_html` in order to generate a complete HTML page that then wraps the `<DIV>` fragment.

We will also display the names of properties of type `Function` and `GeneratorFunction`:

```javascript
var bfu     = require('basic-formatting-utils')
var request = ... /* Get an HTTP request object from somewhere */

bfu.show_fns()

var obj_as_html_page = 
  bfu.as_html([]
  , bfu.as_body([]
    , bfu.show_object("HTTP Request", request)
    )
  )
```


### Example 3: Display Multiple Objects as an HTML `<DIV>`

Call `show_objects` (plural) to transform multiple objects into separate `<DIV>` elements that are then returned within a parent `<DIV>`:

```javascript
var bfu     = require('basic-formatting-utils')
var request = ... /* Get an HTTP request object from somewhere */
var event   = ... /* The HTTP event that invoked this function */

bfu.show_objects([
  {title: "HTTP request", value: request}
, {title: "HTTP event",   value: event}
])
```

### Example 4: Convenience Functions

There are two hard-coded functions that transform the NodeJS `process` and `global` objects respectively into HTML `<DIV>` fragments.

```javascript
var bfu = require('basic-formatting-utils')
    
var process_as_html_div = bfu.show_nodejs_process()
var global_as_html_div  = bfu.show_nodejs_global()
```


## API

### Low-level Utilities

| Name | Return Type | Description
|---|---|---|
| `package_version` | `String` | Returns this utility's current package version
| `sizeOf` | `Number` | For any object for which `isExpandable` returns `true` (see below), this function returns the number of enumerable elements/properties.<br>Returns `0` for all other data types

### Datatype Identifiers

| Name | Return Type | Description
|---|---|---|
| `typeOf` | `String` | Returns the actual data type of the object
| `isOfType` | `Function` | Partial function that returns a function to check for a specific data type.<br>E.G. If you want your own type checking function to test for an `ArrayBuffer`, then you could write<pre>var isArrayBuffer = isOfType("ArrayBuffer")</pre>
| `isArray` | `Boolean` | Does exactly what it says on the tin...
| `isBigInt` | `Boolean` | Does exactly what it says on the tin...
| `isExpandable` | `Boolean` | Returns `true` only for objects of type `Object`, `Array`, `Map` or `Set`.
| `isFunction` | `Boolean` | Returns `true` for objects of type `Function` or `GeneratorFunction`
| `isMap` | `Boolean` | Does exactly what it says on the tin...
| `isNull` | `Boolean` | Does exactly what it says on the tin...
| `isNullOrUndef` | `Boolean` | Does exactly what it says on the tin...
| `isNumber` | `Boolean` | Does exactly what it says on the tin...
| `isNumeric` | `Boolean` | Returns `true` if the argument is either of type `Number` or `BigInt`
| `isObject` | `Boolean` | Returns `true` both for a standard JavaScript `Object` and the NodeJs objects `process` and `global`.<br>The latter two objects are included in this test because they return their own names when passed to `typeOf()`.  The NodeJS object `WebAssembly` also behaves this way, but is deliberately ignored here.
| `isSet` | `Boolean` | Does exactly what it says on the tin...
| `isSymbol` | `Boolean` | Does exactly what it says on the tin...
| `isUndefined` | `Boolean` | Does exactly what it says on the tin...

### HTML Utilities

| Name | Return Type | Description
|---|---|---|
| `as_html_el` | `Function` | A partial function that accepts an HTML tag name and returns a function requiring two arguments.<br>See `as_<tag_name>` below.
| `as_<tag_name>` | `String` | <p>A set of functions generated by calling the partial function `as_html_el`.</p><p>E.G. to create a function that generates an HTML paragraph element, you could write:<pre>var as\_p = as\_html\_el("p")</pre>Function `as_p` then requires two parameters:<ol><li>An array of the element's property values.  <br>Pass an empty array if the element does not need any defined properties</li><li>The element's content in a form that is either a string, or where calling that object's `toString()` function returns something useful</li></ol><p>Any content passed to an empty HTML element (such as `img`) will be ignored</p><p>E.G. To generate an HTML `<table>` element having some `id` property and where all the table rows have been built up in some accumulator array called `acc`, the call would be something like:</p><pre>as_table(<br>  ["id='someTableId'"]<br>, acc.join("")<br>)</pre> 
| `get_depth_limit` | `Number` | Returns the current recursion depth limit for displaying nested objects
| `set_depth_limit` | `Number` | Sets a new recursion depth limit for displaying nested objects. <br>The default depth is 3.
| `show_fns` | `Boolean` | Switches on the display of object properties of type `Function`
| `hide_fns` | `Boolean` | Switches off the display of object properties of type `Function`

### Main Entry Point

| Name | Return Type | Description
|---|---|---|
| `show_objects` | `String` |<p>Takes an array as a single argument in which each element is an object containing the following two properties</p><ol><li>`title` - Object description<br>Do not include any formatting or encoded characters in this description as it is used to generate the value of the collapsible `DIV`'s `id` property.</li><li>`value` - The object to be displayed</li></ol>E.G. To display some HTTP request object `req`, you would write:<pre>show_objects([<br>  {title: "HTTP request", value: req}<br>])</pre>Returns a `DIV` element containing the following children:<ol><li>A small style sheet</li><li>One or more `DIV` elements for each received object, each of which contains:<ul><li>The object's title</li><li>The object represented as an HTML table</li></ul></li><li>A small block of JavaScript that:<ul><li>Populates each arrow image's `src` property</li><li>Defines the `expand`/`collapse` functions</li></ul></li></ol>
| `show_object` | `String` | Convenience function for calling `show_objects` when only a single object needs to be displayed.<br>E.G. To display some existing HTTP request object `req`, you would write:<pre>show_object("HTTP request", req)</pre>


### Convenience Functions for NodeJS Objects

| Name | Return Type | Description
|---|---|---|
| `show_nodejs_global` | `String` | Transforms the NodeJS `global` object into an HTML `<DIV>` fragment
| `show_nodejs_process` | `String` | Transforms the NodeJS `process` object into an HTML `<DIV>` fragment


### Date/Time Functions

| Name | Return Type | Description
|---|---|---|
| `datetime_by_timezone` | `String` | A partial function that receives the offset (in minutes) of a given timezone from UTC and returns a function, that when passed a `new Date()` object, returns the current time in that timezone as a human readable string.<br><pre>var datetime\_jst = datetime\_by\_timezone(540)  /* Japan Standard Time is UTC+9 \*/<br>var tokyo\_time   = datetime\_jst(new Date())<br><br>var datetime\_brt  = datetime\_by\_timezone(-180)  /* Brasilia Time is UTC-3 \*/<br>var brasilia\_time = datetime\_brt(new Date())</pre>
| `datetime_<timezone>` | `String` | When passed a `new Date()`, returns the current time for the particular zone.<br>The only pre-configured timezone functions are:<ul><li>`UTC-8.0` US Pacific Standard Time (`_pst`)</li><li>`UTC-5.0` US Eastern Standard Time (`_est`)</li><li>`UTC+0.0` Greenwich Mean Time (`_gmt`)</li><li>`UTC+1.0` Central European Standard Time (`_cet`)</li><li>`UTC+5.5` India Standard Time (`_ist`)</li></ul>


