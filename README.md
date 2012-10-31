# amber
--
    import "github.com/eknkc/amber"

Amber is an elegant templating engine for Go Programming Language
It is inspired from HAML and Jade

### Tags

A tag is simply a word:

    html

is converted to

    <html></html>

It is possible to add ID and CLASS attributes to tags:

    div#main
    span.time

are converted to

    <div id="main"></div>
    <span class="time"></span>

Any arbitrary attribute name / value pair can be added this way:

    a[href="http://www.google.com"]

You can mix multiple attributes together

    a#someid[href="/"][title="Main Page"].main.link Click Link

gets converted to

    <a id="someid" class="main link" href="/" title="Main Page">Click Link</a>

It is also possible to define these attributes within the block of a tag

    a
        #someid
        [href="/"]
        [title="Main Page"]
        .main
        .link
        | Click Link

### Doctypes

To add a doctype, use `!!!` or `doctype` keywords:

    !!! transitional
    // <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">

or use `doctype`

    doctype 5
    // <!DOCTYPE html>

Available options: `5`, `default`, `xml`, `transitional`, `strict`, `frameset`, `1.1`, `basic`, `mobile`

### Tag Content

For single line tag text, you can just append the text after tag name:

    p Testing!

would yield

    <p>Testing!</p>

For multi line tag text, or nested tags, use indentation:

    html
        head
            title Page Title
        body
            div#content
                p
                    | This is a long page content
                    | These lines are all part of the parent p

                    a[href="/"] Go To Main Page

### Data

Input template data can be reached by key names directly. For example, assuming the template has been
executed with following JSON data:

    {
        "Name": "Ekin",
        "LastName": "Koc",
        "Repositories": [
            "amber",
            "dateformat"
        ],
        "Avatar": "/images/ekin.jpg",
        "Friends": 17
    }

It is possible to interpolate fields using `#{}`

    p Welcome #{Name}!

would print

    <p>Welcome Ekin!</p>

Attributes can have field names as well

    a[title=Name][href="/ekin.koc"]

would print

    <a title="Ekin" href="/ekin.koc"></a>

### Expressions

Amber can expand basic expressions. For example, it is possible to concatenate strings with + operator:

    p Welcome #{Name + " " + LastName}

Arithmetic expressions are also supported:

    p You need #{50 - Friends} more friends to reach 50!

Expressions can be used within attributes

    img[alt=Name + " " + LastName][src=Avatar]

### Variables

It is possible to define dynamic variables within templates,
all variables must start with a $ character and can be assigned as in the following example:

    div
        $fullname = Name + " " + LastName
        p Welcome #{$fullname}

If you need to access the supplied data itself (i.e. the object containing Name, LastName etc fields.) you can use `$` variable

    p $.Name

### Conditions

For conditional blocks, it is possible to use `if <expression>`

    div
        if Friends > 10
            p You have more than 10 friends
        else if Friends > 5
            p You have more than 5 friends
        else
            p You need more friends

Again, it is possible to use arithmetic and boolean operators

    div
        if Name == "Ekin" && LastName == "Koc"
            p Hey! I know you..

There is a special syntax for conditional attributes. Only block attribues can have conditions;

    div
        .hasfriends ? Friends > 0

This would yield a div with `hasfriends` class only if the `Friends > 0` condition holds. It is
perfectly fine to use the same method for other types of attributes:

    div
        #foo ? Name == "Ekin"
        [bar=baz] ? len(Repositories) > 0

### Iterations

It is possible to iterate over arrays and maps using `each`:

    each $repo in Repositories
        p #{$repo}

would print

    p amber
    p dateformat

It is also possible to iterate over values and indexes at the same time

    each $i, $repo in Repositories
        p
            .even ? $i % 2 == 0
            .odd ? $i % 2 == 1

### Includes

A template can include other templates using `include`:

    a.amber
        p this is template a

    b.amber
        p this is template b

    c.amber
        div
            include a
            include b

gets compiled to

    div
        p this is template a
        p this is template b

### Inheritance

A tamplate can inherit other templates. In order to inherit another template, an `extends` keyword should be used.
Parent template can define several named blocks and child template can modify the blocks.

    master.amber
        !!! 5
        html
            head
                block meta
                    meta[name="description"][content="This is a great website"]

                title
                    block title
                        | Default title
            body
                block content

    subpage.amber
        extends master

        block title
            | Some sub page!

        block append meta
            // This will be added after the description meta tag. It is also possible
            // to prepend someting to an existing block
            meta[name="keywords"][content="foo bar"]

        block content
            div#main
                p Some content here

### License
(The MIT License)

Copyright (c) 2012 Ekin Koc <ekin@eknkc.com>

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the 'Software'), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED 'AS IS', WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

## Usage

#### type Compiler

```go
type Compiler struct {
	// Setting if pretty printing is enabled.
	// Pretty printing ensures that the output html is properly indented and in human readable form.
	// If disabled, produced HTML is compact. This might be more suitable in production environments.
	PrettyPrint bool
}
```

Compiler is the main interface of Amber Template Engine. In order to use an
Amber template, it is required to create a Compiler and compile an Amber
tempalte source to native Go template.

    compiler := amber.New()
    // Parse the input file
    err := compiler.ParseFile("./input.amber")
    if err == nil {
    	// Compile input file to Go template
    	tpl, err := compiler.Compile()
    	if err == nil {
    		// Check built in html/template documentation for further details
    		tpl.Execute(os.Stdout, somedata)
    	}
    }

#### func  New

```go
func New() *Compiler
```
Create and initialize a new Compiler

#### func  NewParsing

```go
func NewParsing(input string) (*Compiler, error)
```
Creates and initializes a new compiler and then parses the supplied amber
template string. Returns the "ready to compile" Compiler instance or and error

#### func  NewParsingFile

```go
func NewParsingFile(filename string) (*Compiler, error)
```
Creates and initializes a new compiler and then parses the supplied amber
template file. Returns the "ready to compile" Compiler instance or and error

#### func (*Compiler) Compile

```go
func (c *Compiler) Compile() (*template.Template, error)
```
Compile amber and create a Go Template (html/templates) instance. Necessary
runtime functions will be injected and the template will be ready to be
executed.

#### func (*Compiler) CompileString

```go
func (c *Compiler) CompileString() (string, error)
```
Compile template and return the Go Template source You would not be using this
unless debugging / checking the output. Please use Compile method to obtain a
template instance directly.

#### func (*Compiler) CompileWriter

```go
func (c *Compiler) CompileWriter(out io.Writer) (err error)
```
Compile amber and write the Go Template source into given io.Writer instance You
would not be using this unless debugging / checking the output. Please use
Compile method to obtain a template instance directly.

#### func (*Compiler) Parse

```go
func (c *Compiler) Parse(input string) (err error)
```
Parse given raw amber tempalte string.

#### func (*Compiler) ParseFile

```go
func (c *Compiler) ParseFile(filename string) (err error)
```
Parse the amber tempalte file in given path