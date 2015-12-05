%%%%%%%%%%%%%%%%%%%%%%%%%%%

    Title = "Using mmark to create I-Ds, RFCs and books"
    abbrev = "mmark2rfc"
    category = "info"
    docName = "draft-gieben-mmark2rfc-00"

    [pi]
    private = "yes"
    footer = ""
    header = "mmark"

    date = 2015-10-10T00:00:00Z

    [[author]]
    initials="R."
    surname="Gieben"
    fullname="R. (Miek) Gieben"

%%%%%%%%%%%%%%%%%%%%%%%%%%%

.# Abstract

![mmark logo](logo/mmark120px.png)

This document describes an markdown variant called mmark [@?mmark-ref].

Write RFCs using markdown. Mmark (written in Go) provides an advanced markdown
dialect that processes a single file to produce internet-drafts in XML format.
Internet-drafts written in mmark can produce XML2RFC v2 *and* XML2RFC v3 output.

<!--
[![Build Status](https://travis-ci.org/miekg/mmark.svg?branch=master)](https://travis-ci.org/miekg/mmark)
[![GoDoc](https://godoc.org/github.com/miekg/mmark?status.svg)](https://godoc.org/github.com/miekg/mmark)
-->

{mainmatter}

# Mmark

Mmark is a fork of blackfriday which is a [Markdown][1] processor implemented in
[Go][2]. It supports a number of extensions, inspired by Leanpub, kramdown and
Asciidoc, that allows for large documents to be written. It is specifically
designed to write internet drafts (I-Ds) and RFCs for the IETF. With mmark you can create
a single file that serves as input into the XML2RFC processor.

It also allows for writing large documents such as technical books, like my
[Learning Go book](https://github.com/miekg/learninggo).
Sample text output of this book (when rendered as an I-D) can
[be found here](https://gist.githubusercontent.com/miekg/0251f3e28652fa603a51/raw/7e0a7028506f7d2948e4ad3091f533711bf5f2a4/learninggo.txt).
It is not perfect due to limitations in xml2rfc version 2. Fully rendered
HTML version [can be found here](http://miek.nl/go).

See `mmark2rfc.md` as an example I-D for how to use mmark. Or see the rfc/ subdirectory
with some April 1st RFC that have been typeset in mmark.

It can currently output HTML5, XML2RFC v2 and XML2RFC v3 XML. Other output
engines could be added.

It adds the following syntax elements to [black friday](https://github.com/russross/blackfriday/blob/master/README.md):

* Definition lists.
* More enumerated lists.
* Table and codeblock captions.
* Table footer.
* Subfigures.
* Quote attribution.
* Including other files.
* [TOML][3] titleblock.
* Inline Attribute Lists.
* Indices.
* Citations.
* Abstract/Preface.
* Parts.
* Asides.
* Notes.
* Main-, middle- and backmatter divisions.
* Example lists.
* HTML Comment parsing.
* BCP14 (RFC2119) keyword detection.
* Include raw XML references.
* Abbreviations.
* Super- and subscript.
* Callouts in code blocks.

Mmark is forked from blackfriday which started out as a translation from C of [upskirt][4].

A simular effort is [kramdown-rfc2629](https://github.com/cabo/kramdown-rfc2629) from Carsten Bormann.

There is no pretty printed output if you need that pipe the output through `xmllint --format -`.

## Usage

In the mmark subdirectory you can build the mmark tool:

    % cd mmark
    % go build
    % ./mmark -h
    Mmark Markdown Processor v1.0
    ...

To output v2 xml just give it a markdown file and:

    % ./mmark/mmark -xml2 -page mmark2rfc.md

Making a draft in text form:

    % ./mmark/mmark -xml2 -page mmark2rfc.md > x.xml \
    && xml2rfc --text x.xml \
    && rm x.xml && mv x.txt mmark2rfc.txt

Outputing v3 xml is done with the `-xml` switch. There is not yet
a processor for this XML, but you should be able to validate the
resulting XML against the schema from the XML2RFC v3 draft. I'm trying
to stay current with the latest draft for the V3 spec:
<https://tools.ietf.org/html/draft-hoffman-xml2rfc-21>

# Extensions

In addition to the standard markdown syntax, this package
implements the following extensions:

*   **Intra-word emphasis supression**. The `_` character is
    commonly used inside words when discussing code, so having
    markdown interpret it as an emphasis command is usually the
    wrong thing. Blackfriday lets you treat all emphasis markers as
    normal characters when they occur inside a word.

*   **Tables**. Tables can be created by drawing them in the input
    using a simple syntax:

    ```
    Name    | Age
    --------|-----:
    Bob     | 27
    Alice   | 23
    ```

    Tables can also have a footer, use equal signs instead of dashes for
    the separator.
    If there are multiple footer lines, the first one is used as a
    starting point for the table footer.

    ```
    Name    | Age
    --------|-----:
    Bob     | 27
    Alice   | 23
    ======= | ====
    Charlie | 4
    ```

    If a table is started with a *block table header*, which starts
    with a pipe or plus sign and a minimum of three dashes,
    it is a **Block Table**. A block table may include block level elements in each
    (body) cell. If we want to start a new cell reuse the block table header
    syntax. In the exampe below we include a list in one of the cells.

    ```
    |+-----------------------------------------------|
    | Default aligned |Left aligned| Center aligned  |
    |-----------------|:-----------|:---------------:|
    | First body part |Second cell | Third cell      |
    | Second line     |foo         | **strong**      |
    | Third line      |quux        | baz             |
    |------------------------------------------------|
    | Second body     |            | 1. Item2        |
    | 2 line          |            | 2. Item2        |
    |================================================|
    | Footer row      |            |                 |
    |-----------------+------------+-----------------|
    ```

    Note that the header and footer can't contain block level elements.

    Row spanning is supported as well, by using the
    [multiple pipe syntax](http://bywordapp.com/markdown/guide.html#section-mmd-tables-colspanning).

*   **Subfigure**. Fenced code blocks and indented code block can be
    grouped into a single figure containing both (or more) elements.
    Use the special quote prefix `F>` for this.

*   **Fenced code blocks**. In addition to the normal 4-space
    indentation to mark code blocks, you can explicitly mark them
    and supply a language (to make syntax highlighting simple). Just
    mark it like this:

        ``` go
        func getTrue() bool {
            return true
        }
        ```

    You can use 3 or more backticks to mark the beginning of the
    block, and the same number to mark the end of the block.

*   **Autolinking**. Mmark can find URLs that have not been
    explicitly marked as links and turn them into links.

*   **Strikethrough**. Use two tildes to mark text that
    should be crossed out.

*   **Short References**. Internal references use the syntax `[](#id)`,
    usually the need for the title within the brackets is not needed,
    so mmark has the shorter syntax `(#id)` to cross reference in the
    document.

*   **Hard line breaks**. With this extension enabled
    newlines in the input translate into line breaks in
    the output. This is activate by using two trailing spaces before
    a new line. Another way to get a hard line break is to escape
    the newline with a \. And yet another another way to do this is
    to use 2 backslashes it the end of the line.\\

*   **Includes**, support including files with `{{filename}}` syntax. This is only
    done when include is started at the beginning of a line.

*   **Code Block Includes**, use the syntax `<{{code/hello.c}}[address]`, where
    address is the syntax described in <https://godoc.org/golang.org/x/tools/present/>, the
    OMIT keyword in the code also works.

    So including a code snippet will work like so:

        <{{test.go}}[/START OMIT/,/END OMIT/]

    where `test.go` looks like this:

    ``` go
    tedious_code = boring_function()
    // START OMIT
    interesting_code = fascinating_function()
    // END OMIT
    ```
    To aid in including HTML or XML framents, where the `OMIT` key words is
    probably embedded in comments, lines which end in `OMIT -->` are also excluded.

    Of course the captioning works here as well:

        <{{test.go}}[/START OMIT/,/END OMIT/]
        Figure: A sample program.

    The address may be omitted: `<{{test.go}}` is legal as well.

    Note that the special `prefix` attribute can be set in an IAL and it
    will be used to prefix each line with the value of `prefix`.

        {prefix="S"}
            <{{test.go}}

    Will cause `test.go` to be included with each line being prefixed with `S`.

*   **Indices**, using `(((item, subitem)))` syntax. To make `item` primary, use
    an `!`: `(((!item, subitem)))`. Just `(((item)))` is allowed as well.

*   **Citations**, using the citation syntax from pandoc `[@RFC2535 p. 23]`, the
    citation can either be informative (default) or normative, this can be indicated
    by using the `?` or `!` modifer: `[@!RFC2535]`. Use `[-@RFC1000]` to add the
    cication to the references, but suppress the output in the document.
    The highest modifier seen determines the final type, i.e. once a citation is
    declared normative it will stay normative, but informative will be upgraded
    to normative.

    If you reference an RFC or I-D the reference will be contructed
    automatically. For I-Ds you may need to add a draft sequence number, which
    can be done as such: `[@?I-D.blah#06]`. If you have other references
    you can include the raw XML in the document (before the `{backmatter}`).
    Also see **XML references**.

    If you reference an I-D without a sequence number it will create a reference
    to the *last* I-D in citation index.

    Once a citation has been defined (i.e. the reference anchor is known to mmark)
    you can use `@RFC2535` is a shortcut for the citation.

*   **Captions**, table and figure/code block captions. For tables add the string
    `Table: caption text` after the table, this will be rendered as an caption. For
    code blocks you'll need to use `Figure: `

         Name    | Age
         --------|-----:
         Bob     | 27
         Alice   | 23
         Table: This is a table.

     Or for a code block:

         ``` go
         func getTrue() bool {
             return true
         }
         ```
         Figure: Look! A Go function.

*   **Quote attribution**, after a blockquote you can optionally use
    `Quote: http://example.org -- John Doe`, where
    the quote will be attributed to John Doe, pointing to the URL:

         > Ability is nothing without opportunity.
         Quote: http://example.com -- Napoleon Bonaparte

*   **Abstracts**, use the special header `.# Abstract`. Note that the header
    name, when lowercased, must match 'abstract'.

*   **Notes**, use the special header `#. Title`, if the Title does *not*
    match (lowercase) abstract it will be a note.

*   **Asides**, any paragraph prefixed with `A>` .

*   **Subfigures**, any paraphgraph prefix with `F>` will wrap all images and
    code in a single figure.

*   **{frontmatter}/{mainmatter}/{backmatter}** Create useful divisions in your
    document.

*   **IAL**, kramdown's Inline Attribute List syntax, but took the CommonMark
     proposal, thus without the colon after the brace `{#id .class key=value key="value"}`.
     IALs are used for the following (block) elements:
     * Table
     * Code Block
     * Fenced Code Block
     * List (any type)
     * Section Header
     * Image
     * Quote
     * ...

*   **Definitition lists**, the markdown extra syntax.

         Apple
         :   Pomaceous fruit of plants of the genus Malus in
             the family Rosaceae.

         Orange
         :   The fruit of an evergreen tree of the genus Citrus.

*   **Enumerated lists**, roman, uppercase roman and normal letters can be used
     to start lists. Note that you'll need two space after the list counter:

         a.  Item2
         b.  Item2

*   **TOML TitleBlock**, add an extended title block prefixed with `%` in TOML.
    See the examples RFC in the rfc/ subdirectory. An alternate way of
    typsetting the TOML title block is wrapping it between two lines consisting
    out of 3 or more `%%%`s. Added benefit of wrapping it in `%%%`-lines is that
    the block may be indented (TOML disregards whitespace) 4 spaces, making it
    look like code in other markdown renderers (think Github).

*   **Unique anchors**, make anchors unique by adding sequence numbers (-1, -2,
    etc.) to them. All numeric section get an anchor prefixed with `section-`.

*   **Example lists**, a list that is started with `(@good)` is subsequently
    numbered i   throughout the document. First use is rendered `(1)`, the
    second one `(2)` and so on. You can reference the last item of the list with
    `(@good)`.

*   **HTML comments** An HTML comment in the form of `<!-- Miek Gieben -- really
    -->` is detected and will be converted to a `cref` with the `source`
    attribute set to "Miek Gieben" and the comment text set to "really".

*   **XML references** Any XML reference fragment included *before* the back
    matter, can be used as a citation reference.

*   **BCP 14** If a RFC 2119 word is found enclosed in `**` it will be rendered
    as an `<bcp14>` element: `**MUST**` becomes `<bcp14>MUST</bcp14>`.

*   **Abbreviations**: See
    <https://michelf.ca/projects/php-markdown/extra/#abbr>, any text defined by:

         *[HTML]: Hyper Text Markup Language

     Allows you to use HTML in the document and it will be expanded to
     `<abbr title="Hyper Text Markup Language">HTML</abbr>`. If you need text that looks like
     an abbreviation, but isn't, escape the colon:

         *[HTML]\: HyperTextMarkupLanguage

*   **Super and subscripts**, for superscripts use '^' and for subscripts use '~'. For example:

         H~2~O is a liquid. 2^10^ is 1024.

     Inside a sub/superscript you must escape spaces.
     Thus, if you want the letter P with 'a cat' in subscripts, use `P~a\ cat~`, not `P~a cat~`.

*   **Parts**, use the special part header `-#` to start a new part. This follows the header
     syntax, so `-# Part {#part1}` is a valid part header.

*   **Math support**, use `$$` as the delimiter. If the math is part of a paragraph it will
     be displayed inline, if the entire paragraph consists out of math it considered display
     math. No attempt is made to parse what is between the `$$`.

*   **Callouts**, in codeblocks you can use `<number>` to create a callout, later you can
     reference it:

             Code  <1>
             More  <1>
             Not a callout \<3>

         As you can see in <1> but not in \<1>. There is no <3>.

     You can escape a callout with a backslash. The backslash will be removed
     in the output (both in sourcecode and text). The callout identifiers will be remembered until
     the next code block. The above would render as:

                 Code <1>
                 Code <2>
                 Not a callout <3>

             As you can see in (1, 2) but not in <1>. There is no <3>.

     Note that callouts are only detected with the IAL `{callout="yes"}` or any other
     non-empty value is defined before the code block.
     Now, you don't usualy want to globber your sourcecode with callouts as this will
     lead to code that does not compile. To fix this the callout needs to be placed
     in a comment, but then your source show useless empty comments. To fix this mmark
     can optionally detect (and remove!) the comment and the callout, leaving your
     example pristine. This can be enabled by setting `{callout="//"}` for instance.
     The allowed comment patterns are `//`, `#` and `;`.


[1]: http://daringfireball.net/projects/markdown/ "Markdown"
[2]: http://golang.org/ "Go Language"
[3]: https://github.com/toml-lang/toml "TOML"
[4]: http://github.com/tanoku/upskirt "Upskirt"
[5]: http://github.com/russross/blackfriday "Blackfriday"

<reference anchor='mmark-ref' target='http://github.com/miekg/mmark' >
 <front>
  <title abbrev='Mmark for RFCs'>Mmark: a powerful markdown processor in Go geared for the IETF</title>
  <author initials='R.' surname='Gieben' fullname='R. (Miek) Gieben'>
  <organization>Atoom Inc.</organization>
  <address>
  <email>miek@miek.nl</email></address></author>
  <date year='2015' month='October' />
 </front>
<format type='Web' target='http://github.com/miekg/mmark' />
</reference>

{backmatter}
