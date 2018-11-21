# Basic Formatting Utilities

## Overview

This NPM module contains a simple set of utilities to format a JavaScript object into an HTML table as per the screen shot below.

![Screen shot](./img/screenshot.png)

Nested objects, arrays or maps will be displayed in a collapsed form.  Click on the expand/collapse icons in the `Type` column to hide or display this data.

### But Why?

I wrote this utility primarily to overcome the problem that many standard objects (such as HTTP `request` or `response` objects) contain circular references that cause `JSON.stringify` to explode.

## Usage

Let's say that for the purposes of debugging, you want to display the contents of some server-side objects in the browser.  In this example, your NodeJS app receives an HTTP request.  Your event handler function is passed `event` and `context` objects and in addition to these, you also want to see inside NodeJS's `process` object.

Your code would look something like this:

```javascript
var utils = require('basic-formatting-utils')
    
var format_response = (evt, ctx) =>
  utils.as_html([],
    utils.as_body([],
      utils.create_content([
        {title: "Context Variables", value: ctx}
      , {title: "Event Variables",   value: evt}
      , {title: "NodeJS process",    value: process}
      ])
    )
  )

// *****************************************************************************
// PUBLIC API
// *****************************************************************************
module.exports = {
 handler: (event, context) => format_response(event, context)
}
```

The coding shown above that packages the response from the call to `create_content` into an HTML `body` element inside an `html` element is optional.

Typically, this utility will supply its HTML simply as a fragment of the overall Web page generated by some larger app.

The `create_content` function creates an HTML table representation of an object containing nested objects, but only down to some predetermined recursion depth.  The default is 3, but this can be adjusted by passing a positive integer to `set_depth_limit()`

## API

### Low-level Utilities

| Name | Return Type | Description
|---|---|---|
| `typeOf` | `String` | Returns the actual data type of the object
| `isNull` | `Boolean` | Does exactly what it says on the tin...
| `isUndefined` | `Boolean` | Does exactly what it says on the tin...
| `isNullOrUndef` | `Boolean` | Does exactly what it says on the tin...
| `isNumeric` | `Boolean` | Does exactly what it says on the tin...
| `isArray` | `Boolean` | Does exactly what it says on the tin...
| `isMap` | `Boolean` | Does exactly what it says on the tin...
| `isexpandable` | `Boolean` | Returns true only for those objects whose content can be expanded.<br>Currently, this is only true for `Object`, `Array` or `Map`
| `sizeOf` | `Number` | Returns the number of enumerable elements/properties within an expandable object.<br>Returns `0` for all other data types
| `push` | The modified array | A wrapper for `Array.prototype.push` that pushes a new element onto the end of an array then returns the modified array rather than the much less useful index of the newly added element.<br>This version of `push` can now be used during chained `reduce` operations.
| `unshift` | The modified array | A wrapper for `Array.prototype.unshift` that appends the new element at index `0` and returns the modified array rather than the much less useful length of the new array.<br>This version of `unshift` can now be used during chained `reduce` operations.
| `node_list_to_array` | `Array` | Converts JavaScript's array-like `NodeList` object into a standard `Array` that can be mapped over
| `timestamp_to_str` | `String` | Formats a timestamp into a more human-readable form

### HTML Utilities

| Name | Return Type | Description
|---|---|---|
| `as_html_el` | `Function` | A partial function that accepts some HTML tag name such as `p` or `table` or `img` etc., and returns a function requiring two arguments.  See `as_<tag_name>` below.
| `as_<tag_name>` | `String` | <p>A set of functions generated by calling the partial function `as_html_el`.</p><p>E.G. to create a function that generates an HTML paragraph element, you could write:<pre>var as\_p = as\_html\_el("p")</pre>Function `as_p` then requires two parameters:<ol><li>An array of the element's property values.  <br>Pass an empty array if the element does not need any defined properties</li><li>The element's content in a form that is either a string, or where calling that object's `toString()` function returns something useful</li></ol><p>Any content passed to an empty HTML element (such as `img`) will be ignored</p><p>E.G. To generate an HTML `<table>` element where the `border` is defined and all the table rows have been built up in some accumulator array `acc`, the call would be something like:</p><pre>as_table(["border=\\"1\\""], acc.join(""))</pre> 
| `get_depth_limit` | `Number` | Returns the current recursion depth limit for displaying nested objects
| `set_depth_limit` | `Number` | Sets a new recursion depth limit for displaying nested objects. <br>The default depth is 3.
| `create_content` | `String` | Takes a single argument of an array of objects, where each object contains the following two properties<ol><li>`title` - Some header text to describe the object.<br>Do not include any formatting or encoded characters in this description as it is used to generate the value of the collapsible `DIV`'s `id` property.</li><li>`value` - The object to be displayed</li></ol>Returns a `DIV` element containing the following children:<ol><li>A small style sheet</li><li>One or more `DIV` elements for each object it receives where each `DIV` contains a heading followed by the object represented as an HTML table</li><li>A small block of JavaScript code containing the code to populate each arrow image's `src` property and the expand/collapse functions</li></ol>
