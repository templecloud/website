This repository contains the content of scratchapixel.com in Markdown format.

What is Scratchapixel? It is a website that attempts at being a central repository of various computer graphics techniques. We try to offer readers a place in which most of that vital information is stored and organized in a single location.

- Free. 
- Written in plain English. 
- Every equation is derived.
- Techniques are studied if possible in isolation from the others. Each studied technique is implemented in a program (C++ source code provided) that can easily be compiled from a command line. No Makefile, no dependency. It's designed for people who learn better when they make things (we still present equations but we always explain who they translate into code).
- Focus on simplicity & accessibility.

What Scratchapixel is not:

- Doesn't teach you how to use 3D software
- Doesn't teach you about 3D real-time APIs such as Direct X or Vulkan. However, it will help you understand how these APIs work under the hood.
- Doesn't provide the source code of complex programs containing 100 files and an other-designed third-party dependent build setup that fails to work on your computer no matter what).

## Licence and Copyright

XX TO DO

## Editing Content

### Why and Who Can Edit the Content, and What For?

Anybody can edit the content. So you are more than welcome to do so. Although be mindful about a few things: changes won't necessarily be accepted/go to the main branch (see guidelines below). **The goal of making the project open source at this point is mostly to improve the content in terms of typos, bug fixes, grammar improvements, etc., and to translate the content of Scratchapixel to different languages**.

- Authors are not necessarily native to English and some lessons would deserve to be improved in terms of typos, grammar, etc. Rewording a sentence that's a bit strange to a native English speaker would be great.
- Fixing bad links, etc. (though we will check for those when the website is published so this should not happen).
- If you want to translate a lesson to a different language (see support for languages below)

If you wish to write original content please get in touch with us directly.

### General Structure of a document

- H1 (#) is reserved for the lesson's title, so don't use it on a page
- H2 (##) can be used for all the paragraph headings of a chapter

###  Editing rules

- Do not change the directory structure not the content of metadata files such as `info.txt` which you will find in the lessons repository. Please.
- Refrain yourself from changing a lesson's structure if possible. If you want to make that kind of change, please get in touch with us so that we can discuss this first. Please.
- When you write the heading of a paragraph (##), always insure that the first letter of every **key** word is capitalized. For instance, write "This is Lesson on Ray-Tracing" and not "This is a lesson on ray-tracing"
- Use standard markdown tags for things such as quote, italic, bold, images, etc. (see below).
- No point at the end of headings (h2)
- No word capitalization after a semicolon. For example write, `Figure 1: this is an example` and not `Figure 1: This is an example`, unless of course the word is a noun.
- Avoid double-nested lists (they don't look good on mobile).
- The word Figure (if it refers to a figure) should be written `Figure` (with a capital F).
- All texts in figures should end with a dot (this is checked by the MD to HTML app).
- While the main author of Scratchapixel makes an abusive use of smileys, avoid smileys within the content of the lessons please)).

### Markdown Rules

We are using the standard rules though we added a few that are specific to Scratchapixel. Because in Scratchapixel we want to have a little something a little more sophisticated than what's offered by the basic rules (like notes, marking a paragraph as important, etc.).

**Standard Markdown rules**:
- **Italic**: \_italic_
- **Bold**: \*\*bold**
- **Image**: \!\[Coment about the image](/relative/path/to/image.png)
- **Link**: \[Link](/relative/path/to/content)
- **Heading 2**:  \## Some text (space after \##).
- **Heading 1**:  \# Some text (space after \#). Generally do not use h1's. They are reserved to lessons' titles. Lesson's content should only contain h2's.
- **List**: start a paragraph with a - (numbered list not supported now). Don't forget to put a space after the sign -
- **Quote**: \> my quote text
- **Table**: see below
- **Inline-code**: \'example of inline code\' (surround the text with a `  - back single quote).
- **Code**. Create a new line starting with \```,  add your code then close the block with a new line starting with \```.

**Rules that are specific to Scratchapixel**:
- **Note**: use \<details>\</details>. This is the only html tag that you should be using in a text. It can be used when you want to add a not that you think is a detail or not as relevant as the core of the content.
  ```
  <details>
  <summary>This is the title of this note</summary>
  Put your content here
  </details>
  ```
  Please respect the formatting (\<details>\</details> tags need to be on their own line). You can use the \<summary>\</summary> to give the note a title.
- **Important**: use !!!. This tag can be used when you want to put the emphasis on some content. You need to start a line with three ! and close the block with another three !.
  ```
  !!!!
  This is a very important section.
  You can use multiple lines and list if you need to.
  !!!
  ```
- **Tables**: tables are not handled in the way they are typically handled in Mardown. While Markdown is "designed" to make the text sort of readible in ascii, the  accepted rule for building table in MD is just making the life of the editor a real pain when the cells contain multiple lines which is almost all the case when it comes to real world work.
  ```
  |-table{My header title 1,My header title 2}
  |-row
  |-cell
  Some text in this cell
  - the nice thing is that it accepts lists.
  - this is another list item.
  |-cell
  Some text in this other cell
  |-row
  |-cell
  I like cells.
  |-cell
  |-
  ```
  The syntax is hopefully self-explanatory. As you can see if feels similar to the way you lay a table in HTML. Be mindful to not create more cells than they are number of cells in the header row declaration. For the header cell declaration put the text of the cells better {} and separate them with a comma (leaving no space between the text and the commas and no space between the text and the {}).

### Style

For style recommendations, check the Philosophy Behind Writing Content for Scratchapixel section below.

## General File Organization

The content for all the lessons is stored under `website/lessons`. Directories under this particular directory represent broad categories. Then in each broad category, you will find the lessons organized in separate folders. Example: `lessons/3d-basic-rendering/introduction-to-ray-tracing/` is the path to the introduction-to-ray-tracing lesson. Inside that folder, you will find the chapters stored as Markdown files. The order of the lesson is defined in the `info.txt` file that is stored in the same directory. This is also where the lesson title is defined. Generally please do not edit the name of the Markdown files, not change the content of an `info.txt` file. Only change the content of an MD file if you want to.

## Pushing Changes

At the moment merges are blocked on the master branch. 

## Support for More Languages

XX TODO

## Philosophy Behind Writing Content for Scratchapixel

XX TO DO

We can always do more, but there's a limited number of hours in a day. Let's iterate instead. One step at a time. Some first version of a lesson is better than no lesson at all. And then over time, let's improve things, add more visuals, etc.

The written style is "casual". Writing "it's okay" vs "it is okay" is fine. Both versions are fine!
