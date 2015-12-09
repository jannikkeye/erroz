# erroz

_Erroz ramazotti the way you like it._

Defining and handling errors can be quite complicated! Stop parsing error messages and append meta data like a pro :tm: 

[![Build Status](https://travis-ci.org/peerigon/erroz.svg?branch=master)](https://travis-ci.org/peerigon/erroz)
[![](https://img.shields.io/npm/v/erroz.svg)](https://www.npmjs.com/package/erroz)
[![](https://img.shields.io/npm/dm/erroz.svg)](https://www.npmjs.com/package/erroz)

__erroz__ helps you with... 
- consistent error formats
- meta data for your errors (like http status-codes) 
- templated error messages
- optional stack traces
- optional [JSend](http://labs.omniti.com/labs/jsend) format

## Example


```javascript
var DuplicateError = erroz({
    name: "Duplicate",
    code: "duplicate",
    statusCode: 409,
    template: "Resource %resource (%id) already exists"
});
```

```javascript
throw new DuplicateError({ resource: "Unicorn", id: 1 });

/*
 throw new DuplicateError();
 ^
 Duplicate: Resource Unicorn (1) already exists
 at Object.<anonymous> (/erroz/examples/staticErrorMessage.js:14:7)
 at Module._compile (module.js:456:26)
 at Object.Module._extensions..js (module.js:474:10)
 at Module.load (module.js:356:32)
 at Function.Module._load (module.js:312:12)
 at Function.Module.runMain (module.js:497:10)
 at startup (node.js:119:16)
 at node.js:902:3
 */
```

## Installation

`npm install erroz`

## Usage

### Definition 

```javascript
erroz(errorDefinition);
```

#### errorDefinition

The error definition can contain whatever you want. It's the place for your meta data which will be available on every error instance. Some attributes have a special meaning which is why they are described below: 

#####  `name`

Gives your error a name which will be shown when you throw it.

##### `template`

By setting a `template` property you can define a dynamic error message.

The _message_ will be generated by rendering the `template` with an object passed on Error construction. i.e. `new Error({ resource: "User" });` 

```javascript

var erroz = require("erroz");

var NotFoundError = erroz({
    name: "NotFound",
    template: "%resource (%id) not found"
});

throw new NotFoundError({ resource: "User", id: 1 });
```

>NotFound: User (1) not found

See _Options_ on how to define a custom renderer.

##### `message`

By setting a `message` property you can define a static error message. 

```javascript
var DuplicateError = erroz({
    name: "Duplicate",
    message: "Resource already exists"
});

throw new DuplicateError();
 ```

 > Duplicate: Resource already exists

### Throwing

#### with an object

If you throw an error with an object, the data will be used to render a template if set. In any case the data will be available at `error.data`. 

```javascript
throw new DuplicateError({ resource: "Unicorn", id: 1 });
```

> Duplicate: Resource Unicorn (1) already exists

#### with a string

If you want to throw an error with a custom errro message you can do it by calling the error with a string. This will set `error.data`to an empty object and overwrite any defined `message`or `template`. 

```javascript
throw new ForbiddenError("You are not authorized to eat my cookies");
```

> Forbidden: You are not authorized to eat my cookies

### `erroz` API 
  
#### `error.toJSON()`

You can simply convert your errors to `JSON`. 

```javascript 
var err = new DuplicateError({ resource: "Unicorn", id: 1 });

JSON.stringify(err);

/*
 {
    "name":"Duplicate",
    "code":"duplicate",
    "status":"fail",
    "statusCode":409,
    "template":"Resource %resource (%id) already exists",
    "data":{
        "resource":"Unicorn",
        "id":1
    },
    "message":"Resource Unicorn (1) already exists"
 }
 */
```

__Custom JSON format__ 

By defining a `toJSON` function on AbstractError you can customize the JSON format.

```javascript
//provide a custom `toJSON` method for all inherited errors
erroz.AbstractError.prototype.toJSON = function() {
    return {
        name: this.name,
        code: this.code
    };
};
```
 
#### `error.toJSend()`

By calling `toJSend()` you can convert your error to a JSend style object. Simply pass it to `res.send(err.toJSend());`. 
 
```javascript
var err = new DuplicateError({ resource: "Unicorn", id: 1 });

err.toJSend();

/*
 {
    "status":"fail",
    "code":"duplicate",
    "message":"Resource Unicorn (1) already exists",
    "data":{
    	"resource":"Unicorn",
    	"id":1,
    	"stack":"Duplicate: Resource Unicorn (1) already exists\n    at Object.<anonymous> (/erroz/examples/				  toJson.js:13:11)\n    at Module._compile (module.js:				  456:26)\n    at Object.Module._extensions..js (module.js:474:10)\n    at Module.load 				  (module.js:356:32)\n    at Function.Module._load (module.js:312:12)\n    at 			     Function.Module.runMain (module.js:497:10)\n    at startup (node.js:119:16)\n    at node.js:				  906:3"
    	}
}
*/
```



## Options 

__renderMessage__ 

A custom template renderer in the form

```javascript 
erroz.options.renderMessage = function(data, template) { 
    return "Ooops"; 
}
```

__includeStack__

Should the stack be included in errors? Defaults to true. 
Consider turning that off in production and send it to your logger instead. 

```javascript 
erroz.options.includeStack = false;
```

 ## Pro Tip: Using erroz with connect/express error handlers

 _PRO TIP_: Define a global error handler which does to `toJSend()` call and some logging. So you can simply `next` your errors in your route-handlers 

 ```javascript
 function myAwesomeRoute(req, res, next) {
    if(!req.awesome) {
       next(new NotAwesomeError()); 
       return; 
    }
    
    next();
 }	
 ```

 ```javascript
 app.use(function errozHandler(err, req, res, next) {
 	
 	if(err instanceof errorz.AbstractError) {
       res.status(err.statusCode).send(err.toJSend()); 
       return; 
 	} 
 	
 	//pass on all non-erroz errors
 	next(err);
});
 ```

 ## Licence 

 Unlicense