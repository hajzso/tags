Brace Tags
==========

> The simplest static site generator

Brace Tags is a static site generator focused on simplicity. It does one thing:
solves the problem of having to repeat the same HTML code on several web pages.

The template language provided by Brace Tags only has two tags, `include` and
`is`.


## Static Site Generation 101

You can use a Brace Tags to build a multi-page static website without
duplicating navigation or footer code. Here's generally how it works:

1. Find duplicated code snippets in your HTML files. Extract them into separate
files called "partials".

2. Replace each duplicated code snippet with a special placeholder tag. It looks
like `{% include mypartial.html %}`. This is where the content from a partial
will be injected.

3. Run the `tags build` command to assemble the website from your source code.
You can put the generated site online using any static site hosting provider.


## An example Brace Tags website

Here's a simple multi-page website with `index.html` and `about.html` files. We
can add the main navigation into each page with the `include` tag.

index.html:

    <html>
      <body>
        {% include nav.html %}  <!-- include the nav partial -->
        Welcome to Brace Tags!
      </body>
    </html>


about.html:

    <html>
      <body>
        {% include nav.html %}
        Tags is very simple!
      </body>
    </html>

The navigation partial knows what page is being generated, and can adjust its
content appropriately with the `is` tag. You can use this to highlight the
current page in the nav menu.

nav.html:

    <ul>
      <li>
        <a href="/" {% is index.html %}class="active"{% endis %}>home</a>
      </li>
      <li>
        <a href="/about.html" {% is about.html %}class="active"{% endis %}>about</a>
      </li>
    </ul>        

Note: you'll need to define CSS styles separately to take advantage of the
`active` class attribute used above.

## Installing Brace Tags

Brace Tags is written in Python. Most computers today come with Python. You can
install it with `easy_install` by opening up your terminal and typing in:

    sudo easy_install brace-tags

(The sudo part will ask you to log-in. It's required because Brace Tags needs to
install the `tags` command line script.)

Brace has one external dependency, `watchdog` which is only required if you want
to use Brace to monitor a folder for changes, and recompile your site on the
fly. Before using the `--watch` option you'll need to install `watchdog` with
`sudo easy_install watchdog`.


## Using Brace Tags

Tags has two commands, the `build` command and the `serve` command. Build is
used to generate a site from a source folder.

    tags build

By default, Brace Tags compiles all the .html files in your site. Tags places
the generated site in the `_site` folder, and ignores those files during future
builds. 

Once built, the `serve` command will start a local webserver that you can use
to view the website locally with your browser. This is for testing only.

    tags serve

For more options and explanation, check out the help:

    tags --help

## Hosting your static site

There are lots of places to host a static site. Github pages and S3 are just a
few options. Of course we'd love it if you host your site with us, on
[Brace](http://brace.io)!

## Extending Brace Tags

If you're a python programmer, you can add your own tags to implement custom
functionality on top of Brace Tags.

Custom tags should look like this:

    {% mytag argument1 argument2 %}

Optionally, a tag can have a body, like this:

    {% mytag %}
      Tag Body
    {% endmytag %}

Each time Brace Tags encounters a tag in an input file it checks for a
corresponding tag function. If the function exists, it is called and the result
is substituted in the output.

In the `/tags/tags.py` file you'll find a function for each template tag. Add
your custom tag functions to that file. They should look something like this:

    @lang.add_tag
    def print3x(style, body=u'', context={}):
        ''' A tag that appends 3 copies of its body '''
        result = body + body + body
        if style == "bold":
            result = u'<b>' + result + u'</b>'
        return result

The above function defines a print3x tag that would be called like this:

    {% print3x bold %}
      <h1> ROBOTS, MAKE MY HTML! </h1>
    {% endprint3x %}
    
When adding a new tag function, here are some things you should know:

- The `add_tag` decorator adds the tag function to the template language.

- The tag's name is taken from the function's name. For example, the function
above creates a `print3x` tag. Optionally you can use the `add_tag_with_name`
decorator to supply a tag name.

- The positional arguments of the function define the tag's required arguments.
In this case the tag requires one argument, `style`.

- If you specify a `body` keyword argument, then the tag will require a body.
The body is the content between the opening tag and an end tag.

- All tag functions must accept a `context` keyword argument. This is a
dictionary containing contextual data passed in by the generator. By default,
context includes a `filename` key whose value is the file currently being
generated.


You can also define tags that accept a variable argument list like so:

    @lang.add_tag
    def whatever(*args, **kwargs):
        return str(len(args))


When called, the `*args` parameter will contain the variable argument list, and
the `body` and `context` keyword args will be in the `**kwargs` dictionary.
