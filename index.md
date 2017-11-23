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

1. Setup an HTML file with a doctype, a element named my-element and add a script tag.


Positive
: View the HTML file in Chrome since this browser has the best support for Web Components today.

``` html
<!DOCTYPE html>

<my-element></my-element>

<script>

</script>
```

Open Chrome Dev Tools Console

1. Open the Console (opt+cmd+j)


## Simple element
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

View the HTML file in your browser and see that the element renders.

## Pre-style unregistered elements

### Use the selector

Use `my-element:not(:defined)` to style your component before its definition.

``` css

my-element {
    display: block;
    width: 200px;
    height: 200px;
}
my-element:not(:defined) {
    background:red;

}

my-element:defined {
    background:blue;

}

```

Once you proved this, remove the style tag.

## Life cycle hooks part 1
Duration: 0:15

There are 4 important life-cycle hooks for Custom Elements

### 1. Constructor
``` js
constructor() {
    // An instance of the element is created.
    // Useful for initializing state, settings up event listeners, or creating shadow dom.
    super(); // always call super() first in the constructor.
}
```



Negative
: Always call super(); first!



Add some intial state to the constructor for your Custom Element:

`this.firstname = 'Bob'`

### 2. Connected
``` js
connectedCallback() {
    // Called every time the element is inserted into the DOM.
    // Useful for running setup code, such as fetching resources or rendering.
}



```


Use the initial state and show it using a template literal. Add this to the connectedCallback:

``` js

this.innerHTML = `<h1>Hello!</h1><p>${this.firstname}</p>`;
```

Now your component renders the initial that you provided.

### 3. Disconnected
``` js
disconnectedCallback() {
    // Called every time the element is removed from the DOM.
    // Useful for running clean up code (removing event listeners, etc.).
}
```

Log a message when the element is removed. Remove it by typing `$('my-element').remove()` in your console.
### 4. Attribute changed
``` js
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

``` js
<template>
    <h1>My template</h1>
    <p id="greeting"></p>
</template>

```

We are going to use this template as the content for our Custom Element:

1. Add an id to the template
2. Get the template by id and assign it to a let
3. Assign the contents of the template to a let using cloneNode
4. Set the textContent of the greeting paragraph (by selecting it by id) to the template string
5. Append the cloned node to your Custom Element

``` js
connectedCallback() {
    let template = document.getElementById('my-template');
    let content = template.content.cloneNode(true); // Parameter is for deep cloning
    content.getElementById('greeting').textContent = `Hi, ${this.firstname}`;
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

Add a paragraph next to your Custom Element and see that the styles are applied there as well. Since we are not leveraging the shadow DOM the styles are applied globally.


### Adding a shadow root

Now let's add shadow DOM. So instead of attaching the template content directly as a child, we create a shadow root, and attach the template inside.

``` js
this.attachShadow({mode: 'open'}); // Parameter is for deep cloning
this.shadowRoot.appendChild(content);
```

Notice that now only the paragraph in the shadow DOM has the styles applied.

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
: Be aware that outside styles win (in terms of specificity), so users can override your top-level styles from outside.


## Life cycle hooks part 2
Duration: 0:10

### Rules of the constructor

The constructor has two important rules:
1. You cannot attach or manipulate your children.
   this.appendChild() or this.innerHTML is NOT allowed
   However, manipulating the shadowRoot is OK
2. You cannot inspect or modify your attributes.
   Attributes are part of the DOM, and the constructor() is called
   before your element is aware of the DOM, so it won't work

### Move the shadow dom code

Since we're allowed to create shadow dom in the constructor we can move the code from connectedCallback to the constructor. So let's do that!


``` js
constructor() {
    super();
    let template = document.getElementById('my-template');
    let content = template.content.cloneNode(true);
    this.attachShadow({mode: 'open'});
    this.shadowRoot.appendChild(content);
}

```

Now the connectedCallback should be empty.

## Using slots
Duration: 0:15

### What are slots?
Slots are placeholders inside your component that users can fill with their own markup. The markup that the users adds we call light DOM. Add some light DOM:

``` html

<my-element>
    hello
</my-element>

```

Note that the text is not rendered.

1. Add a slot element to the template to see the text get rendered

``` html
<template id="my-template">
    <slot></slot>
</template>
```

### Fallback

Put some text in the `<slot>`, notice that it's overwritten. Now if we remove the Light DOM you'll see the text of your slot is rendered.

Negative
: For this fallback to work, make sure your Custom Element is completely empty.


### Named slots

Add a name to your slot:
``` html
<slot name="title"></slot>

```

Render an header in the slot:
``` html
<my-element>
    <h1 slot="title">Slots rule!</h1>
</my-element>
```

Create one more slot and play with the defaults.


### Element with ::slotted styles

Use the `::slotted` selector to match nodes inside slots.

``` css
::slotted(h1) {
    color: red;
}
::slotted(.title) {
    color: orange;
}

```
Positive
: Slotted elements retain the style they had from the scope where they are defined


## Attributes on the Element
Duration: 0:30

### Attributes

Users of your Custom Elements can set attributes which you can use inside of the component.

1. Add an attribute called 'firstname' to your Custom Element
2. Read the attribute inside of the connectedCallback hook using this.getAttribute();
3. If the attribute is set, update the firstname property and the greeting text.


``` js
connectedCallback() {
    let firstname = this.getAttribute('firstname');
    if(firstname){
        this.firstname = firstname;
        this.shadowRoot.getElementById('greeting').textContent = `Hi, ${this.firstname}`;
    }
}
```

Nice. We get the attribute value. But we can make this more simple...

### Using properties, getters and setters

We can make life easier by creating getters and setters for firstname.

1. Create the 'private' variable _firstname
2. Create a getter for firstname which returns the private var

``` js
get firstname() {
    return this._firstname;
}
```



Positive
: Private does not exists in javascript.
By prepending props with an underscore you should know to not use it as a part of the component's API.



3. Create a setter in which we check if _firstname has changed and if so, update the greeting
4. Make sure that the initial assignment of this.firstname happens after you created the shadow root
5. Remove the lines where we set the greeting except for in the firstname setter.


``` js
set firstname(val) {
    if (val === this._firstname) return;
    if (val) {
        this._firstname = val;
        this.shadowRoot.getElementById('greeting').textContent = `Hi, ${this.firstname}`;
    }
}
```

Now, change the connectedCallback to: `this.firstname = this.getAttribute('firstname');`


### Setting a default value

Remove the initial assignment in the constructor to this.firstname, and add an else statement to the firstname setter

``` js
set firstname(val) {
    if { ...
    } else {
        this.firstname = 'Default Bob';
    }
}

```


### Observing changed attributes

If you update your Custom Elements attribute 'firstname' via the console using `$('my-element').setAttribute('firstname', 'something');`, you will notice that the greeting won't get updated. We need to observe our attribute in order to update the greeting.

1. Add a static get observedAttributes() which return an array with the element 'firstname'
2. In the attributeChangedCallback check if the value actually has changed
3. Then use the firstname setter to update it to the new attribute value

``` js

static get observedAttributes() {
    return ['firstname'];
}

attributeChangedCallback(attrName, oldVal, newVal) {
    if (newVal != this[attrName]) {
        this[attrName] = newVal;
    }
}

```

Run the previous setAttribute command from your console again and notice that the greeting updates.

## Reflecting properties to attributes
Duration: 0:20

### Setting the attribute when properties change

Often we want the properties of Custom Elements to be reflected in the attributes. If you run `$('my-element').firstname = 'something';` you see that the attribute does not update. In the setter we can use setAttribute to reflect the property.

``` js
//Inside if(val) of set firstname()
this.setAttribute('firstname', this._firstname);
```

Inspect this common pattern in which the getters and setters reflect and modify the boolean attribute 'disabled':

``` js

static get observedAttributes() {
    return ['disabled'];
}

get disabled() {
    return this.hasAttribute('disabled');
}

set disabled(val) {
    if (val) {
        this.setAttribute('disabled', '');
    } else {
        this.removeAttribute('disabled');
    }
}

attributeChangedCallback() {
    // When disabled, update keyboard/screen reader behavior.
    if (this.disabled) {
        this.setAttribute('tabindex', '-1');
        this.setAttribute('aria-disabled', 'true');
    } else {
        this.setAttribute('tabindex', '0');
        this.setAttribute('aria-disabled', 'false');
    }
}

```

Implement this pattern in your Custom Element. Tab through the page and see the difference the disabled attribute now makes.



1. Let's add some styles for our disabled Custom Element

``` css
    :host([disabled]) {
        opacity: 0.2;
    }

```

Run `$('my-element').disabled = true` in your console to see the difference.


## Passing variables up (eventing)
Duration: 0:40

###  Creating a parent element

Often you want to notify the parent elements that stuff changed. So let's create a parent element:

1. Create a new class named MyParent which extends HTMLElement
2. In the constructor add a slot element to the shadow root

``` js
class MyParent extends HTMLElement {
    constructor() {
        super();

        this.attachShadow({mode: 'open'});
        var slot = document.createElement('slot');
        this.shadowRoot.appendChild(slot);
    }
}
```

Define it using customElements.define

``` js
    customElements.define('my-parent', MyParent);
```

Compose the HTML in the body so that my-element is rendered inside of my-parent

``` html
<my-parent>
    <my-element firstname="testtest" id="my-element">
        <h1 slot="title">Slots rule!</h1>
    </my-element>
</my-parent>
```

Everything still looks the same, right? We just render our my-element inside the default slot.

### Setting the clickCount and adding the click listener

Now from my-element we want to dispatch an event with the clickCount to the parent when it has been clicked.

1. Set the initial count prop in the constructor
2. Add an click listener in the constructor

``` js
//Inside the constructor of my-element
this.clickCount = 0;
this.addEventListener('click', this.onClick);

```
### Adding the click handler

1. Create a click handler which increments the clickCount prop and dispatches an 'click-count-changed' CustomEvent containing the details (count)
2. The CustomEvent should have composed set to true to bubble through the shadow DOM


``` js
//Add the click handler to my-element
onClick() {
    this.clickCount++;
    this.dispatchEvent(new CustomEvent('click-count-changed', {
        detail: {count: this.clickCount}, bubbles: true, composed: true
    }));
}
```

### Catch the event in the parent
In my-parent we listen for the 'click-count-changed' event.

1. Add an event listener in the constructor for 'click-count-changed'
2. Handle the event by logging it


``` js
//Inside the constructor of parent-element
this.addEventListener('click-count-changed', this.clickCountUpdated);

//Add the handler for the click-count-changed event
clickCountUpdated(e) {
    console.log(e.detail.count);
}
```

The constructed my-element is encapsulated and unaware of its surroundings. It sends out events to notify it's parents that something has changed.

### Display the count in my-parent

Add a span with an id to the shadowRoot in the constructor:

``` js
let span = document.createElement('span');
span.setAttribute('id', 'count');
this.shadowRoot.appendChild(span);
```

In the click-count-changed handler add:

``` js
this.shadowRoot.getElementById('count').textContent = e.detail.count;
```


## Finish
Duration: 0:00
### Thanks
I want to thank you for completing this Codelab! Please do not hesitate to provide feedback!

### Your conclusions?

Let's discuss Web Components? What do you think of the spec?

### Pros

- Easy to use
- Not much code needed for simple components
-  Shadow DOM is very powerful

### Cons
- We can only use strings as attributes, for object or other types we need transforming (that's why Polymer asks for property types)
- For reflection, we need to transform camelCase property names to kebab-case attributes
- Templating is doable, but easier when we have templating (like {{var}}), could virtual DOM help us here?
- Bindings must set up manually

### Conclusion
- Web Components are pretty low-level
- No binding
- No templating

## BONUS: Browser support
Duration: 0:15

### Supporting old browsers

Chrome is supporting all this, but what if you want to [support other browsers...](https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#support)



## BONUS: Custom properties
Duration: 0:15

### Use vars to change stying of Custom Elements

We provide Custom Elements to our users, but what if they want [different styling of these components?
](https://developers.google.com/web/fundamentals/getting-started/primers/shadowdom#stylefromoutside)