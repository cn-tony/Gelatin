Quick Start
===========

Suppose you want to convert the following text file to XML::

    User
    ----
    Name: John, Lastname: Doe
    Office: 1st Ave
    Birth date: 1978-01-01

    User
    ----
    Name: Jane, Lastname: Foo
    Office: 2nd Ave
    Birth date: 1970-01-01

The following Gelatin syntax does the job::

    # Define commonly used data types. This is optional, but
    # makes your life a litte easier by allowing to reuse regular
    # expressions in the grammar.
    define nl /[\r\n]/
    define ws /\s+/
    define fieldname /[\w ]+/
    define value /[^\r\n,]+/
    define field_end /[\r\n,] */

    grammar user:
        match 'Name:' ws value field_end:
            out.add_attribute('.', 'firstname', '$2')
        match 'Lastname:' ws value field_end:
            out.add_attribute('.', 'lastname', '$2')
        match fieldname ':' ws value field_end:
            out.add('$0', '$3')
        match nl:
            do.return()

    # The grammar named "input" is the entry point for the converter.
    grammar input:
        match 'User' nl '----' nl:
            out.open('user')
        user()

Explanation:

#. **"grammar input:"** is the entry point for the converter.
#. **"match"** statements in each grammar are executed sequentially. If
   a match is found, the indented statements in the match block are
   executed. After reaching the end of a match block, the grammar
   restarts at the top of the grammar block.
#. If the end of a grammar is reached before the end of the input
   document was reached, an error is raised.
#. **out.add('$0', '$3')** creates a node in the XML (or JSON, or YAML)
   if it does not yet exist. The name of the node is the value of the
   first matched field (the fieldname, in this case). The data of the
   node is the value of the fourth matched field.
#. **out.open('user')** creates a "user" node in the output and selects
   it such that all following "add" statements generate output relative
   to the "user" node. Gelatin leaves the user node upon reaching the
   out.leave() statement.
#. **user()** calls the grammar named "user".

This produces the following output:

::

    <xml>
      <user lastname="Doe" firstname="John">
        <office>1st Ave</office>
        <birth-date>1978-01-01</birth-date>
      </user>
      <user lastname="Foo" firstname="Jane">
        <office>2nd Ave</office>
        <birth-date>1970-01-01</birth-date>
      </user>
    </xml>

Starting the transformation
---------------------------

The following command converts the input to XML:

::

    gel -s mysyntax.gel input.txt

The same for JSON or YAML:

::

    gel -s mysyntax.gel -f json input.txt
    gel -s mysyntax.gel -f yaml input.txt

Using Gelatin as a Python Module
--------------------------------

Gelatin also provides a Python API for transforming the text::

    from Gelatin import generator
    from Gelatin.util import compile

    # Parse your .gel file.
    syntax = compile('syntax.gel')

    # Convert your input file to XML.
    builder = generator.Xml()
    syntax.parse('input.txt', builder)
    print builder.serialize()
