# Command Line

All Graphviz programs have a similar invocation:

```
cmd [ flags ] [ input files ]
```

For example:

```
$ dot -Tsvg input.dot
```

If no input files are supplied, the program reads from **stdin**. For example:

```
$ echo 'digraph { a -> b }' | dot -Tsvg > output.svg
```

Generates:

![img](https://www.graphviz.org/doc/info/stdin.svg)

## Flags

### `-G`*name*[=*value*]

Set a graph attribute, with default *value* = `true`

For example,

```
$ echo 'digraph { a -> b }' | dot -Tsvg -Gfontcolor=red -Glabel="My favorite letters"
```

Overrides the default `fontcolor` and `label` attributes of the graph, producing a red legend:

![img](https://www.graphviz.org/doc/info/-G.svg)

### `-N`*name*[=*value*]

Set a default node attribute, with default *value* = `true`.

For example,

```
$ echo 'digraph { a -> b }' | dot -Tsvg -Nfontcolor=red -Nshape=rect
```

Overrides the default node `fontcolor` and `shape` attributes, producing rectangular nodes with red text:

![img](https://www.graphviz.org/doc/info/-N.svg)

### `-E`*name*[=*value*]

Set a default edge attribute, with default *value* = `true`.

For example,

```
$ echo 'digraph { a -> b }' | dot -Tsvg -Ecolor=red -Earrowhead=diamond
```

Overrides the default edge `color` and `arrowhead` attributes, producing red edges with a diamond arrowhead:

![img](https://www.graphviz.org/doc/info/-E.svg)

### `-K`*layout*

Specifies which default [layout engine](https://graphviz.org/docs/layouts/) to use, overriding the default from the command name. For example, running `dot -Kneato` is equivalent to running `neato`.

### `-T`*format*[:*renderer*[:*formatter*]]

Set output language to one of the [supported formats](https://www.graphviz.org/doc/info/output.html). By default, [attributed dot](https://www.graphviz.org/docs/outputs/canon/) is produced.

Depending on how Graphviz was built, there may be multiple renderers for generating a particular output format, and multiple formatters for creating the final output. For example, a typical installation can produce [PNG](https://www.graphviz.org/docs/outputs/png/) output using either the Cairo or GD library. The desired rendering engine can be specified after a colon. If there are multiple formatting engines available, the desired one can be specified in a similar fashion after the rendering engine. Thus, `-Tpng:cairo` specifies PNG output produced by Cairo (using the Cairo’s default formatter), and `-Tpng:cairo:gd` specifies PNG output produced by Cairo formatted using the GD library.

If no renderer is specified, or a renderer but no formatter, the default one is invoked. The flag `-T*format*:` produces a list of all of the renderers available for the specified *format*, the first one listed with a prefix matching *format* being the default. Using the `-v` flag, described below, will print which format, renderer, and formatter are actually used.

### `-V`

Emit version information and exit. For example:

```
$ dot -V
dot - graphviz version 2.47.1 (20210417.1919)
```

### `-l`*library*

User-supplied, device-dependent library text. Multiple flags may be given. These strings are passed to the code generator at the beginning of output.

For PostScript output, they are treated as file names whose content will be included in the preamble after the standard preamble. If *library* is the empty string `""`, the standard preamble is not emitted.

### `-n`[*num*]

Sets no-op flag in **neato**. If set, **neato** assumes nodes have already been positioned and all nodes have a [pos](https://graphviz.org/docs/attrs/pos/) attribute giving the positions. It then performs an optional adjustment to remove node-node overlap, depending on the value of the [overlap](https://graphviz.org/docs/attrs/overlap/) attribute, computes the edge layouts, depending on the value of the [splines](https://graphviz.org/docs/attrs/splines/) attribute, and emits the graph in the appropriate format. If *num* is supplied, the following actions occur:

- *num* = 1

  Equivalent to `-n`.

- *num* > 1

  Use node positions as specified, with no adjustment to remove node-node overlaps, and use any edge layouts already specified by the [pos](https://graphviz.org/docs/attrs/pos/) attribute. **neato** computes an edge layout for any edge that does not have a **pos** attribute. As usual, edge layout is guided by the [splines](https://graphviz.org/docs/attrs/splines/) attribute.

### `-o`*outfile*

Write output to file *outfile*. For example,

```
$ echo 'digraph { a -> b }' | dot -Tsvg -o output.svg
```

Generates `output.svg`:

![img](https://www.graphviz.org/doc/info/stdin.svg)

By default, output goes to **stdout**.

### `-O`

Automatically generate output file names based on the input file name and the various output formats specified by the **-T** flags.

For example,

```
$ dot -Tsvg -O ~/family.dot ~/debug.dot
```

Generates `~/family.dot.svg` and `~/debug.dot.svg` files.

### `-P`

Automatically generate a graph that shows the plugin configuration of the current executable. e.g.

```
$ dot -P -Tsvg -o plugins.svg
```

### `-q`

Suppress warning messages.

### `-s`[*scale*]

Set input scale to *scale*. If this value is omitted, 72.0 is used. This number is used to convert the point coordinate units used in the [pos](https://graphviz.org/docs/attrs/pos/) attribute into inches, which is what is expected by neato and fdp. Thus, feeding the output of a graph laid out by one program into neato or fdp almost always requires this flag. Ignored if the `-n` flag is used.

### `-v`

Verbose mode

### `-x`

In **neato**, on input, prune isolated nodes and peninsulas. This removes uninteresting graph structure and produces a less cluttered drawing.

### `-y`

By default, the coordinate system used in generic output formats, such as [attributed dot](https://www.graphviz.org/docs/outputs/canon/), [extended dot](https://www.graphviz.org/docs/outputs/canon/), [plain](https://www.graphviz.org/docs/outputs/plain/) and [plain-ext](https://www.graphviz.org/docs/outputs/plain/), is the standard cartesian system with the origin in the lower left corner, and with increasing y coordinates as points move from bottom to top. If the `-y` flag is used, the coordinate system is inverted, so that increasing values of y correspond to movement from top to bottom.

### `-?`

Print usage information, then exit.

If multiple `-T` flags are given, drawings of the graph are emitted in each of the specified formats. Multiple `-o` flags can be used to specify the output file for each format. If there are more formats than files, the remaining formats are written to **stdout**.

Note that the `-G`, `-N` and `-E` flags override any initial attribute declarations in the input graph, i.e., those attribute statements appearing before any node, edge or subgraph definitions. In addition, these flags cause the related attributes to be permanently attached to the graph. Thus, if attributed dot is used for output, the graph will have these attributes.

## Environment Variables

### `GDFONTPATH`

List of pathnames giving directories which a program should search for fonts. Overridden by [DOTFONTPATH](https://www.graphviz.org/doc/info/command.html#DOTFONTPATH). *Used only if Graphviz is not built with the `fontconfig` library*

### `DOTFONTPATH`

List of pathnames giving directories which a program should search for fonts. Overridden by [**fontpath**](https://graphviz.org/docs/attrs/fontpath/). *Used only if Graphviz is not built with the `fontconfig` library*

### `SERVER_NAME`

If defined, this indicates that the software is running as a web application, which restricts access to image files. See [GV_FILE_PATH](https://www.graphviz.org/doc/info/command.html#GV_FILE_PATH).

### `GV_FILE_PATH`

If [SERVER_NAME](https://www.graphviz.org/doc/info/command.html#d:SERVER_NAME) is defined, image files are restricted to exist in one of the directories specified by `GV_FILE_PATH`. This last is a list of directory pathnames, separated by semicolons in Windows or by colons otherwise. Note that sometimes, when using one of the layout programs in a web script, it is not enough to use an export command but rather the variables should be set when the command is run, for example,

```bash
SERVER_NAME=xxx GV_FILE_PATH="images:etc/images:/usr/share/images" dot -Tpng -o x.png x.gv
```

Copy

Note that the image files must really reside in one of the specified directories. If the image file is specified as an absolute or relative pathname, a warning is given and only the base name is used.

### `GVBINDIR`

Indicates which directory contains the Graphviz config file and plug-in libraries. If it is defined, the value overrides any other mechanism for finding this directory. If Graphviz is properly installed, it should not be needed, though it can be useful for relocation on platforms not running Linux or Windows.

Last modified January 28, 2022: [Check anchors with hyperlink (33b99a2)](https://gitlab.com/graphviz/graphviz.gitlab.io/commit/33b99a2a1dcfed6954f7ce1bd5375f8e4e6243e1)



[Command Line | Graphviz](https://www.graphviz.org/doc/info/command.html)