This repository contains the content of scratchapixel.com in Markdown format.

What is Scratchapixel? It is a website that attempts at being a central repository of various computer graphics techniques. We try to offer readers a place in which most of that vital information is stored and organized in a single location.

- Free. 
- Written in plain English. 
- Every equation is derived.
- Focus on simplicity & accessibility.
- Books like PBRT (Physically Based Rendering) provide a complete software solution and chapters from the book describe diffrent parts of the program. Scratchapixel does not focused on providing a large software solution to which the lessons would refer to. Instead, each lesson focuses on one particular technique and each lesson comes with its own code. The goal is to show how the studied technique translate into code in a program that can easily be compiled from the command line and doesn't rely on a complex build system to be tested.

What Scratchapixel is not:

- Not a place where you can learn about using 3D software such as Maya or Blender.
- Doesn't teach you about 3D real-time APIs such as Direct X or Vulkan. However, it will help you understand how these APIs work under the hood.

## Licence and Copyright

All content of Scratchapixel falls under the Creative Commons “BY-NC-ND” licence (Attribution-NonCommercial-NoDerivatives license). It allows people to use the unadapted work for noncommercial purposes only, and only as long as they give credit to the creator. They may also adapt the work for their own personal use but may not share any adaptations publicly.

## Editing Content

### Why and Who Can Edit the Content, and What For?

Anybody can edit the content. So you are more than welcome to do so. However please check the guidelines below.

### Workflow for Contributors

If you want to help, first thank you). We want to avoid conflicts in the process of reviewing lessons. For this reason, as we are experimenting with this review process, we thought that to start with, we would only have a one reviewer per lesson. To do so, we made a Google Docs document in which we have listed all the lessons that are ready for being reviewed. If you see a lesson that has no reviewer/contributor yet and that you would be willing to review this lesson, add your name next to the lesson title and follow the steps below.

[https://docs.google.com/spreadsheets/d/19ljS9fyrRFchaIn_TF8L2YaZw5p89kFqo6QfTad9Av0/edit?usp=sharing](https://docs.google.com/spreadsheets/d/19ljS9fyrRFchaIn_TF8L2YaZw5p89kFqo6QfTad9Av0/edit?usp=sharing)

- You need a GitHub account. Once logged in into your account, go to [Scratchapixel's website repository]().
- Fork the repository. This will create a copy of the repository in your own account.
- How you want to edit the content after that point is entirely up to you. You can edit the content directly from GitHub (from your account, using the "forked" repo) or clone the project on your local computer to use the text editor of your choice.
- If you work with a cloned copy (remotely) you will need to install [Git Credential Manager](http://microsoft.github.io/Git-Credential-Manager-for-Windows/Docs/CredentialManager.html). 
- Once you want your edits to be pushed to the main website, create a pull request (from which we will be to able to merge your changes into the master branch). Your edits will go through an approval process before being merged to the main branch.

For the time being, it would be great if you could mostly focus on typos, layout, grammatical issues, and  improving the readibility of sentences if needs be. If you find some issues with the code snippets or the equations of course, please feel free to fix them too. 

The point of making the project open source at this point in time is to improve the quality of the current content as much as possible. If you wish to write original content or would like to suggest important changes to the structure of a lesson, please get in touch with us (email/Discord).

### General Structure of a document

- H1 (#) is reserved for the lesson's title, so don't use it on a page.
- H2 (##) used for the paragraph headings of a chapter.

###  Editing rules

- Do not change the directory structure nor the content of metadata files such as `info.txt` which you will find in the lessons repository. Thank you.
- Don't change the lesson's structure. If you want to make that kind of change, please get in touch with us first. Thank you.
- When you write the heading of a paragraph (##), the first letter of every **key** word is capitalized. For instance, write "This is Lesson on Ray-Tracing" and not "This is a lesson on ray-tracing"
- Use standard markdown tags for things such as quote, italic, bold, images, etc. (see below).
- No point at the end of headings (h2).
- No word capitalization after a semicolon. For example write, `Figure 1: this is an example` and not `Figure 1: This is an example`, unless of course the word is a noun.
- Avoid double-nested lists (they don't look good on mobile).
- The word Figure (if it refers to a figure) should be written `Figure` (with a capital F).
- All texts in figures should end with a dot (this is checked by the MD to HTML app).

### Markdown Rules

We are using some of the standard rules but we added a few that are specific to Scratchapixel. Because we want to have something a little more sophisticated (and sometimes more control too) than what's possible with the basic rules (like adding notes, marking a paragraph as important, etc.).

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
- **Code**. Create a new line starting with \```,  add your code then close the block with a new line starting with \```. You can add the word `wrap` just after the first three ! (opening tag). That will force the code within the block to wrap (it will overflow in x otherwise if the lines are longer than the column width).

**Rules that are specific to Scratchapixel**:
- **Note**: use \<details>\</details>. This is the only HTML tag that you should be used in a text. It can be used when you want to add a note that you think is a detail or not as relevant as the core of the content.
  ```
  <details>
  <summary>This is the title of this note</summary>
  Put your content here
  </details>
  ```
  Please respect the formatting (\<details>\</details> tags need to be on their own line). You can use the \<summary>\</summary> to give the note a title.
- **Important**: use !!!. This tag can be used when you want to emphasize some content. You need to start a line with three ! and close the block with another three !.
  ```
  !!!!
  This is a very important section.
  You can use multiple lines and lists if you need to.
  !!!
  ```
- **Tables**: tables are not handled in the way they are typically handled in Markdown. While Markdown is "designed" to make the text sort of readable in ASCII, the accepted rule for building a table in MD is just making the life of the editor a real pain when the cells contain multiple lines which is more often the case than not when it comes to real world work.
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
  The syntax is hopefully self-explanatory. As you can see it feels similar to the way you lay a table in HTML. Be mindful to not create more cells than they are cells in the header row declaration. For the header cell declaration put the text of the cells better {} and separate them with a comma (leaving no space between the text and the commas and no space between the text and the {}).

### Style

For style recommendations, check the Philosophy Behind Writing Content for Scratchapixel section below.

## General File Organization

The content for all the lessons is stored under `website/lessons`. Directories under this particular directory represent broad categories. Then in each broad category, you will find the lessons organized in separate folders. Example: `lessons/3d-basic-rendering/introduction-to-ray-tracing/` is the path to the introduction-to-ray-tracing lesson. Inside that folder, you will find the chapters stored as Markdown files. The order of the lesson is defined in the `info.txt` file that is stored in the same directory. This is also where the lesson title is defined. Generally please do not edit the name of the Markdown files, and do not change the content of an `info.txt` file (note that these will be replaced by json files soon). Only change the content of an MD file if you want to.

## Pushing Changes

At the moment merges are blocked on the master branch. 

## Support for More Languages

XX TODO

## Philosophy Behind Writing Content for Scratchapixel

XX TO DO

We can always do more, but there's a limited number of hours in a day. Let's iterate instead. One step at a time. Some first version of a lesson is better than no lesson at all. And then over time, let's improve things, add more visuals, etc.

The written style is "casual". Writing "it's okay" vs "it is okay" is fine. Both versions are fine!
