# Proposals #

Formal proposals for changes to the code, new libraries, etc

Sections are a rough attempt at organization.

## Sections ##

* [Core](core)
* [Syntax](syntax)
* [Libraries](libraries)


## How do I make a new proposal? ##

Do you have an idea but aren't confident with forming a proper proposal? You may open an [issue](https://github.com/imp-erial/imperial/issues) instead.

For those of you who have a solid idea and are comfortable with making a formal proposal, basically just make a PR with the markdown for it. It's fine to commit drafts and such here but only proposals with the status `Final` will be reviewed for acceptance.

For the most part, any valid GFM Markdown will be accepted but keep the following points in mind:

* *italics* are intended to represent key names (and specialized types) in prose.
* **bold** indicates a literal string that is meant to be written as-is, such as a keyword.
* <u>underline</u> indicates a named variable to be replaced with something when actually writing RPL. For instance, an argument to a CLI program like gz <u>file</u>
* CAPS can be used similarly to underline but for when it's in a code block.
* Please use <tt>\`\`\`rpl</tt> to start code blocks for RPL, even though GitHub doesn't (and likely never will) format them. Do not use indentation for any codeblocks.

