This repository contains the content of scratchapixel.com if Mardown format.

## Editing Content

### Why and Who Can Edit the Content, and Waht For?

Anybody can edit the content. So you are more than welcome to do so. Alghtough be mindful about a few things: changes won't necessarily be accepted / go to the main branch (see guidelines below).

Why: to improve the website's quality. 

What for?

- Authors are not necessarily native in English and some lessons would definitely designed to be improved in that regards. So anything that goes from typos, grammar issues or rewording a sentence that's a bit strange to native English speaker would be great.
- Fixing bad links, etc. (though we will check for those when the website is published to this should not happened).
- If you want to translate a lesson to different language (see support for languages below)

If you wish to write original content please get in touch with us directly.

### General Structure of a document

- H1 (#) is reserved for the lesson's title, so don't use it in a page
- H2 (##) can be used for all the paragraph headings of a chapter

###  Editing rules

- Do not change the directory structure not the content of metadata files such as info.txt which you will find in the lessons repository. Please.
- Refrain yourself from changing a lesson's structure if possible. If you want to make that kind of change, please get in touch with us so that we can discuss this first. Please.
- When you write the heading of a paragraph (##), always insure that the first letter of every **key** word is capitalized. For instance write "This is Lesson on Ray-Tracing" and not "This is a lesson on ray-tracing"
- Use standard markdown tags for things such as quote, italic, bold, images, etc.

## General File Oganizations

The content for all the lessons is stored under `website/lessons`. Directories under this particular directory represent broad categories. Then in each broad category you will find the lessons organized in separate folders. Example: `lessons/3d-basic-rendering/introduction-to-ray-tracing/` is the path to the introduction-to-ray-tracing lesson. Inside that folder you will find the chapters stored as Mardown files. The order of the lesson is defined in the `info.txt` file that is stored in the same directory. This is also where the lesson title is defined. Generally please do not edit the name of the Mardown files, not change the content of an `info.txt` file. Only change the content of a MD file if you want to.

## Pushing Changes

At the moment merges are blocked on master branch. 

## Support for More Languages

XX TODO

## Philosophy Behind Writing Content for Scratchapixel

XX TO DO