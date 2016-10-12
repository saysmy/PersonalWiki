## Drawing a SVG file [Back](./../canvas.md)

This document is to talk about hw to draw a SVG file with canvas in JavaScript.

### Read data from a SVG file

Before drawing a SVG file, you need to read data from a SVG file with `FileReader` in JavaScript. 
 
```js 
var fileReader = new FileReader(); 
``` 
 
`FileReader` is a Web API, which has given some methods for you to read a file.`readAsText` is one of the method, supported by this Web API, for reading file with its content, and `onload` is the event triggered when a file has been read: 
 
```js 
fileReader.onload = function (e) { 
     
}; 
 
fileReader.read(file); 
``` 
 
Of course, if you would like to complete a interaction of dragging and dropping a file into a canvas to read a SVG file, you can set up a event handler for listening to the `drop` event of this canvas like this: 
 
```js 
/** Drop Event Handler */ 
canvas.addEventListener('drop', function (e) { 
    /** e is where we can extract out the `file` objecj */ 
    var file = e.dataTransfer.files[0]; 
}) 
```