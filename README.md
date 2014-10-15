CDocParser
---
CDocParser is a language agnostic C and `///`-Style comments parser that uses block and line comments to make it easier to generate documentation.


## Install

```bash
$ npm install --save cdocparser
```


## Usage

CDocParser consists of two parts the `CommentExtractor` and a `CommentParser`.


```js
var CDocParser = require('cdocparser');
var extractor = new CDocParser.CommentExtractor(/* contextParser */ );
var parser = new CDocParser.CommentParser(/* Annotations */);

var comments = extractor.extract(/* code */);
var parsedComments = parser.parse(comments);

console.log(parsedComments);
```

## API

### CommentExtractor

The ComemntExtractor is used to extract C and `///`-Style comments from source and attach context information to it.

#### `new CommentExtractor(contextParser)`

Create a CommentExtractor to extract block comment like:

```
/**
 *
 *  CDocComment
 *
 */
```

You need to pass in a function that is used to generate a `context` object used to specify the context of the comment.
A context obj:

```js
{
  type : 'contextType'
}
```

The `type` attribute is mandatory, you can add as much attributes as you would like.

#### `#extract(code)`

This method will return an Array of all comments in the form of

```js
[
  {
    lines: [],
    type: 'normal|poster',
    commentRange: { start : 1, end : 2 }, 
    context: [context object generated by contextParser]
  }
]
```


### CommentParser

#### `new CommentParser(annotations, config)`

Create a new `CommentParser` where `annotaions` is an object like:
```js
{
  _: {
    alias: {
      'aliasName': 'aRealAnnotation'
    }
  },

  aRealAnnotation: {
    parse : function (annotationLine) {
    
    },
    default : function(){
      return 5;
    }
  }
}
```

This object is used to provide parser for various types of annotations. It also includes tha ability to include aliases.

#### `#parse ( comments )`

This methods takes a comments array provided by `CommentExtractor#extract` and parses all annotations. The resulting
object will look like:

```js
{
  "[context.type]" : [
    {
      description : "[Contains all comment lines without an annotation]",
      [annotationName] : [resultOfAnnotationParser]
    }
  ]
}
```


### Annotations API 

The annotations object is build up from two different kind of object. A `annotation` object and a
`alias`. 

The global strutucture looks like:
```
{
  _ : {
    [alias object]
  },
  [annotation object],
  [annotation object]
}
```


### A `annotation` object

#### Overview
```js
name : {
  parse : function(line){

  },

  autofill : function(comment){

  },

  default : function(comment){

  },

  multiple : true
}
```

Each annotation must have a `parse` method, optionaly you can have a `default` and `extend` methods. The optional `multiple` key is used to indicate if an annotation can be used mutliple times.

#### `parse` method 
The `parse` method is used to parse the actuall `string` after the `@name`. All values returned from that method
will be wrapped in an array.

##### Example:
Implementing a `name` annotation:

```js
/**
 * @name Fabrice Weinberg 
 */
```

```js
function(line){
  return {
    name : line
  }
}
```

#### `default` method 
The `default` method is used to add a default value.

##### Example:
```js
function(comment){
  return [{
    name : 'Default Name'
  }]
}
```
> Note: Please keep in mind that you need to wrap values in an Array to align with hand written annotations


#### `autofill` method 
The `autofill` method is used to extend hand written annotations by autofilled ones. 

##### Example:
```js
function(comment){
  // Access the parsed comment here. 
}
```

> Note: Extended annotations can be disabled by using the `@allowExtend` annotation.


#### `multiple` key

The `multiple` key is used to determin if this can be used mutliple times per comment. 

> Note: A warning will be emitted if a annotation is used more than once. Only the first value is used. 

## Development

Use `mocha test` to run the unit tests.

## Changelog

#### 0.6.0
  
  * Include line numbers in each found comment block. (See [PR#6](https://github.com/FWeinb/CDocParser/pull/5))

#### 0.5.0

 * Add `multiple` key, to indicate if a annoation can be used more than once per comment.

#### 0.4.0
  
  * Add `autofill` as an annotation feature.
  * Remove the array wrapping of `default` values. 

#### 0.3.8
 
  * Add type check for poster comments

#### 0.3.7

 * Fix broken API in `0.3.5` and `0.3.6`

#### 0.3.5

 * Use raw arrays returned from `default` as value.

#### 0.3.4

 * Pass in the parsed item to the `default` function

#### 0.3.3

 * Fix a bug with line comments that are indented

#### 0.3.2
 
 * Add `allowedOn` key to annotations to only apply them to comments from a specifc type

#### 0.3.0 

 * Add support for `///` comments 
 * Add a `lineNumber` function as a second parameter that will convert char indices to line numbers

#### 0.2.2

  * Add a `poster comment` to apply annotations to all items in the file that are documented.
  * Emits a `warning` if you use more than on `poster comment` per file. Only the first one will be used.

#### 0.2.1
 
  * Emits a `warning` if a annotation was not found instead of throwing an exception.

#### 0.2.0 
 
  * Throw an error if annotation was not found

#### 0.1.1
  
  * Ignore annotations that return `undefined`.

#### 0.1.0 
  
  * Restructure annotation function. Add `default` value and `parse` function.


