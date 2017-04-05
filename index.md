author:            Bob Bijvoet
summary:           abcdefghij
id:                123
categories:        n/a
environments:      web
status:            draft
feedback link:     github.com/bobbijvoet
analytics account: 0

Formatting help:
https://github.com/googlecodelabs/tools/tree/master/claat/parser/md

# Pure Web Components

## Setup
Duration: 0:05

Setup an index.html file with a doctype, a element named my-element and add a script tag.

``` html
<!DOCTYPE html>

<my-element></my-element>

<script>

</script>
```


Positive
: View the index.html file in Chrome since this browser has the best support for Web Components today.

## Simple Element using HTMLElement
Duration: 0:15


We start with a very simple Custom Element by extending `HTMLElement`. On the connected lifecycle hook (more on that in the next step) we replace the contents with some elements using `this.innerHTML`.


``` js

//Inside the script tag
class MyElement extends HTMLElement {
  connectedCallback() {
    this.innerHTML = "<h1>Hello!</h1><p>I am a custom element</p>";
  }
}
```


Negative
: Always be careful using innerHTML when handling user input! There are chances of exposing XSS vulnerabilities!


Now we can define the Custom Element with the just created class.

``` js

customElements.define('my-element', MyElement);

```

View the html file in your browser and see that the element renders.

## Using the lifecycle hooks
Duration: 0:15

There are 4 important life-cycle hooks for Custom Elements



### 1. Constructor
```
constructor() {
    // An instance of the element is created.
    // Useful for initializing state, settings up event listeners, or creating shadow dom.
    super(); // always call super() first in the ctor.
}
```



Negative
: Always call super(); first!



Set up some state for your component:

`this.name = 'Bob'`

### 2. Connected
```
connectedCallback() {
    // Called every time the element is inserted into the DOM.
    // Useful for running setup code, such as fetching resources or rendering.
}



```


Use the state and show it using a template literal:

```

this.innerHTML = `<h1>Hello!</h1><p>${this.name}</p>`;
```


### 3. Disconnected
```
disconnectedCallback() {
    // Called every time the element is removed from the DOM.
    // Useful for running clean up code (removing event listeners, etc.).
}
```

Log a message when the element is removed. Remove it by typing `$('my-element').remove()` in your console.
### 4. Attribute changed
```
attributeChangedCallback(attrName, oldVal, newVal) {
    // An attribute was added, removed, updated, or replaced.
    // Also called for initial values when an element is created by the parser, or upgraded.
}
```

Negative
: Only attributes listed in the observedAttributes property will receive this callback.

We will cover attributes later.


## Using a template
Duration: 0:15

### Adding a template

Create a `<template>` element just above the `<script>` tag:

```
<template>
    <h1>My template</h1>
    <p id="greeting"></p>
</template>

```

We are going to use this template as the content for our Custom Element:

1. Give the template an id
2. Select it using getElementById
3. Clone the contents
4. Set the content of the greeting paragraph to the template string
5. Append it to your Custom Element

```
connectedCallback() {
    let template = document.getElementById('my-template');
    let content = template.content.cloneNode(true); // Parameter is for deep cloning
    content.getElementById('greeting').textContent = `Hi, ${this.name}`;
    this.appendChild(content);
}
```


Positive
: Using textContent is safer and faster because it will not parse any HTML tags.

## Element with scoped styles
Duration: 0:15
### Add some styles

Create a `<style>` tag inside of the `<template>`. Add some style for the paragraph.

``` css

    p {
        color: red;
    }


```

Add a paragraph next to your Custom Element and see that the styles are applied there as well.


### Adding a shadow root

Instead of attaching the Template content directly as a child, we create a Shadow Root, and attach the template inside.



```
this.attachShadow({mode: 'open'}); // Parameter is for deep cloning
this.shadowRoot.appendChild(content);
```

Notice that only the paragraph in the shadow dom has the styles applied.

## Styling the Custom Element
Duration: 0:10
### Using :host
Web components can style themselves too, by using the :host selector.

``` css

    :host {
        display: block;
        background: darkgray;
    }

```

Positive
: By default, custom elements are display: inline.


### The :host() function
You can also use :host to encapsulate behaviors or to style internal nodes based on the host

``` css
    :host(:hover) {
        background: lightgray;
    }

    :host(.blue) {
        background: darkblue;
        color: white;
    }

    :host(.blue) > p {
        color: green;
    }
```


Negative
: Be aware that outside styles win in terms of specificity, so users can override your top-level styles from outside.


## Life cycle hooks part 2

### Rules


Since we're allowed to create shadow dom in the constructor we can move


## Slots

### Placeholders
Slots are placeholders inside your component that users can fill with their own markup. The markup that the users adds we call light DOM. Add some light DOM:

```

<my-element>
    hello
</my-element>

```

Note that the text is not rendered. If we add a `<slot>` element to our `<template>` then we see the text get rendered.

### Fallback

Put some text in the `<slot>`, notice that it's overwritten. Now if we remove the Light DOM you'll see the text of your slot is rendered.

Negative
: For this fallback to work, make sure your Custom Element is completely empty.


### Named slots

Add a name to your slot:
```
<slot name="title"></slot>

```

Render an header in the slot:
```
<my-element>
    <h1 slot="title">Slots rule!</h1>
</my-element>
```

Create one more slot and play with the defaults.


### Element with ::slotted styles

Use the `::slotted` selector to match nodes inside slots.

``` css
::slotted(h1) {
    margin: 0;
    font-weight: 300;
    color: red;
}
::slotted(.title) {
    color: orange;
}

```


##  Variables on the Element

    ### Attributes
    ### Properties
    ### Passing variables up (eventing)


<!--## Creating our Polymer clone-->

    <!--- Element cache-->
    <!----->
