![Shadow Dom Representation](https://mdn.mozillademos.org/files/15788/shadow-dom.png)

## What is the Shadow DOM?

What Is the Shadow DOM? So what exactly is this mysterious-sounding shadow DOM? According to the W3C: 

>Shadow DOM is an adjunct tree of DOM nodes. These shadow DOM subtrees can be associated with an element, but do not appear as child nodes of the element. Instead the subtrees form their own scope. For example, a shadow DOM subtree can contain IDs and styles that overlap with IDs and styles in the document, but because the shadow DOM subtree (unlike the child node list) is separate from the document, the IDs and styles in the shadow DOM subtree do not clash with those in the document.


Before you append your `Shadow ROOT` it needs to be created a `Shadow HOST` inside of the parent DOM.

## Shadow Host
A shadow host is a DOM node that contains a shadow root. It is a regular element node within the parent page that hosts the scoped shadow subtree. Any child nodes that reside under the shadow host are still selectable, with the exception of the shadow root. It is a special kind of [DocumentFragment](https://developer.mozilla.org/en-US/docs/Web/API/DocumentFragment)

## Shadow Root
A shadow root is an element that gets added to a shadow host. The shadow root is the root node for the shadow DOM branch. Shadow root child nodes are not returned by DOM queries even if a child node matches the given query selector. Creating a shadow root on a node in the parent page makes the node upon which it was created a shadow host.

- **Shadow host:** The regular DOM node that the shadow DOM is attached to.

- **Shadow tree:** The DOM tree inside the shadow DOM.

- **Shadow boundary:** the place where the shadow DOM ends, and the regular DOM begins.

- **Shadow root:** The root node of the shadow tree.

## CREATING A SHADOW ROOT
Creating a shadow root is a straightforward process. First a shadow host node is selected, and then a shadow root is created in the shadow host.

### <p align="center"> TIP </p>
<p align="center"> To inspect shadow DOM branches using the Chrome debugger, check the “Show Shadow DOM” box under the “Elements” section in the “General” settings panel of the debugger.
</p>

```javascript
<div id="host"></div>


var host = document.querySelector('#host');
var root = host.createShadowRoot();
```

 It is possible to attach multiple shadow roots to a single shadow host. However, only the last shadow root attached is rendered. A shadow host follows the LIFO pattern (last in, first out) when attaching shadow roots
 
 ## Style Encapsulation
 Any styles defined in the shadow DOM are scoped to the shadow root.
 
 ## Styling the Host Element
 In some cases you will want to style the host element itself. This is easily accomplished by creating a style anywhere within the parent page, because the host element is not part of the shadow root. This works fine, but what if you have a shadow host that needs different styling depending on the contents of the shadow root? And what if you have multiple shadow hosts that need to be styled based on their contents? As you can imagine, this would get very difficult to maintain. Fortunately, there is a new selector, `:host`, that provides access to the shadow host from within the shadow root. This allows you to encapsulate your host styling to the shadow root:
 
 ```html
 <head>
    <template id="template">
        <style>
            :host {
                border: 1px solid red;
                padding: 10px;
            }
        </style>
        My host element will have a red border!
    </template>
</head>
<body>
    <div id="host"></div>
    <script type="text/javascript">
        var template = document.querySelector('#template')
        var root = document.querySelector('#host').createShadowRoot();
        root.appendChild(template.content);
    </script>
</body>
 ```
 

Also it can be easly override.

```html
<head>
    <template id="template">
        <style>
            :host {
                border: 1px solid red;
                padding: 10px;
            }
        </style>
        My host element will have a blue border!
    </template>
</head>
<body>
    <div id="host" style="border: 1px solid blue;"></div>
    <script type="text/javascript">
        var template = document.querySelector('#template')
        var root = document.querySelector('#host').createShadowRoot();
        root.appendChild(template.content);
    </script>
</body>
```

The :host selector also has a functional form that accepts a selector, `:hostselector`, allowing you to set styles for specific hosts. This functionality is useful for theming and managing states on the host element.

## Styling Shadow Root Elements from the Parent Page

Encapsulation is all well and good, but what if you want to target specific shadow root elements with a styling update? What if you want to reuse templates and shadow host elements in a completely different application?

In either case you might not have control over the shadow root’s contents, or the update process might take a significant amount of time, which would block your development. Additionally, you might not want control over the content.

Fortunately, the drafters of the W3C specification thought of these cases (and probably many more), so they created a selector that allows you to apply styling to shadow root elements from the parent page.

The `::shadow` pseudoelement selects the shadow root, allowing you to target child elements within the selected shadow `root:`

```html
<head>
    <style>
        #host::shadow p {
            color: blue;
        }
    </style>
    <template><p>I am blue.</p></template>
</head>
<body>
    <div id="host"></div>
    <script type="text/javascript">
        var template = document.querySelector('template');
        var root = document.querySelector('#host').createShadowRoot();

        root.appendChild(template.content);
    </script>
</body>
```
The `::shadow` pseudoelement selector can be used to style **nested shadow roots**:

```html
<head>
    <style>
        #parent-host::shadow #child-host::shadow p {
            color: blue;
        }
    </style>
    <template id="child-template"><p>I am blue.</p></template>
    <template id="parent-template">
        <p>I am the default color.</p>
        <div id="child-host"></div>
    </template>
</head>
<body>
    <div id="parent-host"></div>
    <script type="text/javascript">
        var parentTemplate = document.querySelector('#parent-template');
        var childTemplate = document.querySelector('#child-template');
        var parentRoot = document.querySelector('#parent-host')
            .createShadowRoot();
        var childRoot;

        parentRoot.appendChild(parentTemplate.content);
        childRoot = parentRoot.querySelector('#child-host').createShadowRoot();
        childRoot.appendChild(childTemplate.content);
    </script>
</body>
```

Sometimes targeting individual shadow roots using the `::shadow` pseudoelement is very inefficient, especially if you are applying a theme to an entire application of shadow roots. Again, the drafters of the W3C specification had the foresight to anticipate this use case and specified the `/deep/` combinator. The `/deep/` combinator allows you cross through all shadow roots with a single selector:

```css
    /* colors all <p> text within all shadow roots blue */
    body /deep/ p {
        color: blue;
    }

    /* colors all <p> text within the child shadow root blue */
    #parent-host /deep/ #child-host # p {
        color: blue;
    }

    /* targets a library theme/skin */
    body /deep/ p.skin {
        color: blue;
    }
```

Also it is possible to manipulate the content held inside of the shadow dom. `contentWindow` of the `HTMLIFrameElement` can be used under this circumstances. 

>The contentWindow property returns the Window object of an element. You can use this Window object to access the iframe's document and its internal DOM. This attribute is read-only, but its properties can be manipulated like the global Window object. MDN

Consider you have an iframe and its Shadow DOM Tree and you want to access a span element declared with foo id.

```javascript
// HTMLIFrame​Element 
let someIframe = document.getElementBy("idOfIframe");

// Global Object of the Iframe
let frameWindow = someIframe.contentWindow;

// Dom Tree of the Target Window
let domTreeofFrame = frameWindow.document;

let textSpan = domTreeofFrame.getElementById("foo");
```

## Events and the Shadow DOM
At this point you might be thinking that projecting nodes instead of cloning them is a great optimization that will help to keep changes synchronized—but what about events bound to these projected nodes?

In these cases you can still determine the shadow root of the projected node by examining the path property of the event object. Some events are never retargeted, though, which makes sense if you think about it. For instance, how would a `scroll` event be retargeted? If a user scrolls one projected node, should the others scroll? The events that are not retargeted are:

- abort
- error
- select
- change
- load
- reset
- resize
- scroll
- selectstart 

Events that happen in shadow DOM have the host element as the target, when caught outside of the component.

## Sending message from sub to parent
The `window.postMessage()` can be used to communicate window elements which is a method safely enables cross-origin communication between `Window` objects; e.g., between a page and a pop-up that it spawned, or between a page and an iframe embedded within it.

Normally, scripts on different pages are allowed to access each other if and only if the pages they originate from share the same protocol, port number, and host (also known as the "same-origin policy"). `window.postMessage()` provides a controlled mechanism to securely circumvent this restriction (if used properly).

If the message want to sent inside of the child it can be used `window.parent.postMessage`

#### Resources
https://github.com/praveenpuglia/shadow-dom-in-depth
https://javascript.info/shadow-dom-events
https://developer.mozilla.org/en-US/docs/Web/Web_Components/Using_shadow_DOM
https://stackoverflow.com/questions/55678082/turn-iframe-elements-into-text-data-javascript/55678608#55678608
https://developer.mozilla.org/en-US/docs/Web/API/Window/postMessage
https://robertnyman.com/html5/postMessage/postMessage.html
https://gist.github.com/pbojinov/8965299
https://www.oreilly.com/library/view/modern-javascript/9781491971420/ch05.html
