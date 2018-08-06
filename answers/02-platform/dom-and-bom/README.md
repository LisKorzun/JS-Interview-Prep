# DOM and BOM

### Document Object Model (DOM)
The Document Object Model (DOM) is an application programming interface (API) for HTML as well as XML.
It represents the page so that programs can change the document structure, style, and content.

The Document Object Model (DOM) represents that same document so it can be manipulated. 
The DOM is an object-oriented representation of the web page, which can be modified with a scripting language such as JavaScript.
The DOM maps out an entire page as a document composed of a hierarchy of nodes like a tree structure 
and using the DOM API nodes can be removed, added, and replaced.

![alt text](https://cdn-images-1.medium.com/max/880/1*aXkyRKW3S5t1n6x1TgmcxA.png)

The DOM is not a programming language, but without it, the JavaScript language wouldn't have any model or notion of web pages, 
HTML documents, XML documents, and their component parts (e.g. elements). 
The page content is stored in the DOM and may be accessed and manipulated via JavaScript, 
so that we may write this approximative equation:  

**API (HTML or XML page) = DOM + JS (scripting language)**

__`DOM level 1:`__

consisted of two modules: the **DOM Core**, which provided a way to map the structure of an 
XML-based document to allow for easy access to and manipulation of any part of a document, 
and the **DOM HTML**, which extended the DOM Core by adding HTML-specific objects and methods.

__`DOM level 2:`__ 

introduced several new modules of the DOM to deal with new types of interfaces:

**DOM Views** — describes interfaces to keep track of the various views of a document (that is, the document before CSS styling and the document after CSS styling)

**DOM Events** — describes interfaces for events

**DOM Style** — describes interfaces to deal with CSS-based styles

**DOM Traversal and Range** — describes interfaces to traverse and manipulate a document tree

__`DOM level 3:`__
 
 further extends the DOM with the introduction of methods to load and save documents in a 
 uniform way (contained in a new module called DOM Load and Save) as well as methods to 
 validate a document (DOM Validation). In Level 3, the DOM Core is extended to support 
 all of XML 1.0, including XML Infoset, XPath, and XML Base.

**_Note that the DOM is not JavaScript-specific, and indeed has been implemented in numerous 
other languages. For Web browsers, however, the DOM has been implemented using ECMAScript 
and now makes up a large part of the JavaScript language._**

### Browser Object Model (BOM)
Browsers feature a Browser Object Model (BOM) that allows access and manipulation of 
the browser window. Using the BOM, developers can move the window, change text in the status bar, 
and perform other actions that do not directly relate to the page content.

![alt text](https://cdn-images-1.medium.com/max/880/1*K-CM7OV1lQnnACBeIcagSw.png)

Because no standards exist for the BOM, each browser has its own implementation.
