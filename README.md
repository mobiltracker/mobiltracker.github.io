# Guide

MbBook is a tool to easily create Markdown books. In this guide we'll explain how to add and edit files to Mobiltracker's MdBook. For a more in-depth explanation of MdBook and all of its features you can read the [official documentation](https://rust-lang.github.io/mdBook/).

## Installing MdBook

You can download the executable binary from the [GitHub Releases page](https://github.com/rust-lang/mdBook/releases). You can choose the right one for your OS and download it. The archive contains an `mdbook.exe` which can be used to build the books.

It's recommendable to add the path to the file to your `PATH` environment variable.

## Read the book locally

After cloning this repository, you can use the command `mbook serve` in the root of the directory to open a local server. By default it opens in `localhost:3000`.

This server rebuilds the book automatically after you modify and save any file in the `./src` folder.

## Deploy and build

This repository has an action that automatically builds the book and deploys it to our GitHub page. All you have to do is `push` your changes to the `master` branch.

> Notice that only the `master` branch is deployed automatically, if you want to save in the repository your work in progress you can create another branch and then make a pull request after you finished it.

The `./book` folder contains the builded files of the book and it's in the `.gitignore` since our production build is done automatically.

## How to edit an existing page

Since the pages are all written in Markdown, you can open them in any text editor you like and start editing. You can check the `SUMMARY.md` file to see how each page is linked to each file.

## How to create a new page

The `summary.md` file is the backbone of the book. How the links are organized there directly reflects in how it will be build.

To add a new page to the book you simply need to create a new item in the summary and link to the file you want. If the file you inserted doesn't exist, it will be automatically created.

To see everything you can do with Markdown in MdBook you can read the [documentation](https://rust-lang.github.io/mdBook/format/markdown.html).

## How to organize the Summary

There are a few things you can put on the summary:

- **Prefix Chapters:** they must come before all the numbered chapters and it apears on the summary without any number notation. It's usually used for forewords and introduction.

- **Part Title:** headers can be used as a title for the following numbered chapters.

- **Numbered Chapters:** they outline the main content of the book and can be nested, creating sub-chapters.

- **Suffix Chapters:** they are much like the Prefix Chapters, but they must come after all the numbered chapters

```markdown
# Summary

[This is a Prefix Chapter](./prefix-page-file.md)

# This is a part title

- [This is a numbered chapter](./chapter-1.md)
  - [This is a subchapter](./chapter-1-1.md)
- [This is the second chapter](./chapter-2.md)

[This is a Suffix Chapter](./suffix-page-file.md)
```

This code example renders the following summary:

![Example Summary](https://user-images.githubusercontent.com/32579593/155591366-06ee7e70-f386-442e-87f6-47e5aa58ac82.png)

For more informations you can read the [documentation](https://rust-lang.github.io/mdBook/format/summary.html)
