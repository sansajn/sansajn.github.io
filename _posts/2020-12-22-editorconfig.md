---
layout: post
title: "General coding style configuration"
date: 2020-12-22 12:20:00 +0100
categories: IDE
comments: true
author: Adam Hlavatovic
---

What coding style do you prefer? Tabs or spaces (two, three or even four)? Unix or windows style newlines? 

It would be cool to have per project editor settings working with various editors and IDEs (like Vim, Kate, Visual Studio Code, IntelliJ IDEA, Sublime Text and many more ...) without any additional setup. I've found that [EditorConfig](https://editorconfig.org/) was made exactly for that purpose.

[EditorConfig](https://editorconfig.org/) project page says:

> EditorConfig helps maintain consistent coding styles for multiple developers working on the same project across various editors and IDEs. The EditorConfig project consists of a file format for defining coding styles and a collection of text editor plugins that enable editors to read the file format and adhere to defined styles. EditorConfig files are easily readable and they work nicely with version control systems.

## Default config

The only think we need is `.editorconfig` file placed in our project directory. Let's say we prefer indents with tabs (rendered as 3 spaces) with unix style newlines and newline ending. Our `.editorconfig` can looks like this

```ini
# see https://EditorConfig.org for further config informations

# top-most EditorConfig file
root = true

# unix-style newlines with a newline ending every file
[*]
end_of_line = lf
insert_final_newline = true
trim_trailing_whitespace = true

# default charset and identation
[*]
charset = utf-8
indent_style = tab
tab_width = 3

# makefile specific settings
[Makefile]
indent_style = tab

# YAML specific settings
[*.yml]
indent_style = space
indent_size = 2
```

> **note**: YAML files doesn't allow indentation with tabs so that is why `[*.yml]` override

Feel free to change line ending from unix (`lf`) to windows (`crlf`) style as `end_of_line` property or `indent_style` from `tab` to `spaces` if you prefer one or another.

> **tip**: list of editors with build-in EditConfig support can be found in [editorconfig.org](https://editorconfig.org/) page in *No Plugin Necessary* section 

## Adding support

List of editors that requires to install a plugin can be found in [editorconfig.org](https://editorconfig.org/) page in *Download a Plugin* section. 

### for Visual Studio Code

just install [*EditorConfig for VS Code*](https://marketplace.visualstudio.com/items?itemName=EditorConfig.EditorConfig) plugin


Are you using EditorConfig in editor not listed there? Just, let me know what needs to be done for your favorite editor or IDE in comments bellow.
