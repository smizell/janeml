JaneML
------

JaneML is a fun thought-experiment in thinking about JSON formats differently, specifically ways in which JSON can be used to express XML-like markup alongside Lisp-like markup. While covering all of the elements of HTML5, it aims to make additions to what HTML5 can already do. Some things to note about JaneML.

1. It is pure JSON (though not idiomatic)
1. It is a superset of HTML5
1. It allows for expressing XML-like markup
1. It is also very Lisp-like

Please proceed cautiously! 

## Basic Syntax

1. Each element is a JSON array
1. The first item of each array MUST be a string, which is the name of the element. Any HTML element is a valid.
1. The second item of the array MUST be a JSON object, which contains attributes and their values. Valid types for attributes values are strings, numbers, booleans, null, a the special `list` element. Any HTML5 attribute is valid (depending on the element).
1. The array MAY have more items. Every item after the attribute object is content for the element, which MAY be a string, number, boolean, null, or a new element array. Objects MUST NOT be used as items of the elements other than for the attribute object (of if specified for specific elements). Elements without additional items SHOULD be considered self-closing.

```json
["janeml", {},
  ["body", {},
    ["h1", {}, "Page Title!"],
    ["p", {}, "This is a paragraph in the page"],
    ["h2", {}, "Test Form"],
    ["form", {"method": "POST", "action": "/test"},
      ["input", {"type": "text", "name": "test_var"}],
      ["input", {"type": "submit"}]]]]
```

### Mixed Content

JaneML, though JSON, can do mixed content like in HTML.

```json
["janeml", {},
  ["body", {},
    ["h1", {}, "Page Title!"],
    ["p", {}, "This is a ", ["strong", {}, "paragraph"], " in the page"],
```

## Shorthand

Shorthand element names may also be used for readability. 

1. Any element that starts with a `#` or `.` MUST be considered a `div` element
1. Using a `#` followed by a string denotes the `id` of the element
1. Using a `.` followed by a string denotes the `className` of the element
1. Element names, `id` values, and `className` values MAY be chained

```json
["janeml", {},
  ["body", {},
    ["h1", {}, "Shorthand Examples"],
    [".example", {},
      ["p", {}, "This parent element .example is a div"],
      ["p.class-example", {}, "This p element has a class"],
      ["p#id-example", {}, "This p element has an id"],
      ["p.chain.example", {}, "This p element has chained classes"]]]]
```

## Extensions, Additions, and Changes to HTML

There are some changes and additions made to the HTML.

### HTTP Verbs

JaneML supports all HTTP verbs in forms, whereas HTML only supports `GET` and `POST`.

```json
["janeml", {},
  ["body", {},
    ["form", {"method": "PUT", "action": "/customers/1"},
      ["input", {"type": "text", "name": "full_name", "value": "John Doe"}],
      ["input", {"type": "submit"}]]]]
```

### Class Attributes

Because of issues with JavaScript, the attribute name `className` MUST be used in place of the HTML attribute `class`.

```json
["janeml", {},
  ["body", {},
    ["p", {"className": "example"}, "Example of class name"]
```

### List Element

Because of the fact that all JSON arrays are considered elements (or potentially functions), a special element `list` MAY be used to denote a list for an attribute.

```json
["janeml", {},
  ["body", {},
    [".container", {},
      ["a", {"rel": ["list", {}, "self", "collection"], "href": "/example"},
        "Example of list element"]]]]
```

If this example is converted to HTML, the `rel` value SHOULD be handled accordingly.

```html
<html>
  <body>
    <div class="container">
      <a rel="self collection" href="/example">Example</a>
    </div>
  </body>
</html>
```

### Template Link

A template link uses a special `linkt` element that works like a normal HTML `link` element but with URI templates.

1. The `href` attribute of the link MUST be considered a URI template
1. The `uri-param` element works just like an `input` element, except it is for URI template parameters

```json
["janeml", {},
  ["body", {},
    ["linkt", {"rel": "customer", "href": "/customers/{id}"},
      ["uri-param", {"type": "text", "name": "id"}]]]]
```

### Template Form

A template form uses a special `formt` element that works like a normal HTML `form` element but with URI templates.

1. The `action` attribute of the form MUST be considered a URI template
1. The `uri-param` element works just like an `input` element, except it is for URI template parameters

```json
["janeml", {},
  ["body", {},
    ["formt", {"method": "PUT", "action": "/customers/{id}"},
      ["uri-param", {"type": "text", "name": "id"}],
      ["input", {"type": "text", "name": "full_name", "value": "John Doe"}],
      ["input", {"type": "submit"}]]]]
```

### JSON Element (Experimental)

For use in forms or just for expressing data, a special `json` element may be used for JSON data. This functions like all other elements except that it MUST only have one item at most, and it MAY either be an array or an object.

```json
["janeml", {},
  ["body", {},
    ["form", {"method": "PUT", "action": "/customers/1", "enctype": "application/json"},
      ["json", {}, {"id": "4",
                    "name": "John Doe",
                    "email": "john@example.com"}]]]]
```

In HTML5, the specific media type may be specified for a form using `enctype`, which allows for specifying that JSON Patch may be used.

```json
["janeml", {},
  ["body", {},
    ["form", {"method": "PATCH",
              "action": "/customers/1",
              "enctype": "application/json-patch+json"},
      ["json", {}, {"id": "4",
                    "name": "John Doe",
                    "email": "john@example.com"}]]]]
```

Other form elements may be used with the `json` element, but their element names MUST be prefixed with a `~` to ensure there is no collision with real data.

```json
["janeml", {},
  ["body", {},
    ["form", {"method": "PUT", "action": "/customers/1", "enctype": "application/json"},
      ["json", {}, {"id": "4",
                    "name": ["~input", {"type": "text", "name": "name", "value": "John Doe"}],
                    "email": "john@example.com"}]]]]
```

If there is collision even with the `~`, a new character may be specified using the meta attribute `json-char`.

```json
["janeml", {},
  ["head", {},
    ["meta", {"name": "json-char", "content": "@"}],
  ["body", {},
    ["form", {"method": "PUT", "action": "/customers/1", "enctype": "application/json"},
      ["json", {}, {"id": "4",
                    "name": ["@input", {"type": "text", "name": "name", "value": "John Doe"}],
                    "email": "john@example.com"}]]]]]
```

### XML Element (Experimental)

To express XML data in forms, the special `xml` element MAY be used. Inside of the `xml` element, any XML element MAY be used.

```json
["janeml", {},
  ["body", {},
    ["form", {"method": "PUT", "action": "/customers/1", "enctype": "application/xml"},
      ["xml", {},
        ["id", {}, 4],
        ["name", {}, "John Doe"],
        ["email", {}, "john@example.com"]]]]]
```

Like with the `json` element, other form elements may be used within the `xml` element, though they MUST be prefixed with a `~`.

```json
["janeml", {},
  ["body", {},
    ["form", {"method": "PUT", "action": "/customers/1", "enctype": "application/xml"},
      ["xml", {},
        ["id", {}, 4],
        ["name", {},
          ["~input", {"type": "text", "name": "name", "value": "John Doe"}]],
        ["email", {}, "john@example.com"]]]]]
```

## Benefits

1. JSON is supported in all the major programming languages. This means you can write HTML in native code. 
1. Since it can easily be converted to XML, all XML tooling can be used
1. Since it is a superset of HTML, jQuery-like CSS selectors can be used to select elements
1. If used in the browser, it can be easily inserted into the DOM
1. It could be used with frameworks like React.js to define components in pure JavaScript or JSON. In other words, it could be used for templating both on server and client-side.
1. Since JaneML is a superset of HTML, CSS stylesheets may be used to define styling for JaneML documents

## FAQ

### Isn't it harmful to create a new HTML spec?

HTML dominates the browser world because it is supported by all of the browsers. Because of this, we have to rely on browsers to implement the HTML spec. Since JaneML's intent is for JSON, and since clients outside of the browser can adhere to whatever spec that is desired, it seemed like a great opportunity to extend on what HTML already has done and make it available in JSON.

### Doesn't this render all JSON tooling useless?

In a way, it does. It does allow for a lot of other tooling to be used, though that couldn't be used before. For example, XPath libraries may be used to access paths in the document.

## Appendix: Lisp Experiment

I'm nervous to even go here, but because this essentially is mimicking how Lisp works, it could be used to create a Lisp-like language in JSON. Many JSON document formats are created with some way to express how they should be processed. This would allow for a way to express that kind of processing right in the document in a format that just about any programming language can parse.

```json
["janeml", {},
  ["def", "data", {"customers": [{"id": 4,
                                  "first_name": "John",
                                  "last_name": "Doe"},
                                 {"id": 5,
                                  "first_name": "Jane",
                                  "last_name": "Doe"}]}],
  ["defn", "customer-url", ["customer"],
    ["concat", "/customers/", ["customer", "id"]]],
  ["body", {},
    ["h1", {}, "Customer List"],
    ["ul", {},
      ["for", ["customer", ["data", "customers"]],
        ["li", {}, 
          ["a", {"href": ["customer-url", ["customer"]]},
            ["concat", ["customer", "first_name"], ["customer", "last_name"]]]]]]]]
```

## References

These libraries and specs have similar ideas that are expressed here. JaneML aims to go a slightly different direction than these, though.

1. [JSONML](http://jsonml.org) - JSON as XML
1. [JTHON](https://github.com/jesusabdullah/jthon) - JSON with lisp
1. [Hiccup](https://github.com/weavejester/hiccup) - Really neat library for use with Clojure
1. [Jade](http://jade-lang.com/) - Jade Templating language
1. [React.js](http://facebook.github.io/react/index.html) - React.js and its compiled JSX
