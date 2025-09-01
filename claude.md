# Project Hugo Site

## Overview

This project defines the structure and content of a statically generated photography presentation web site. The 
[Hugo](https://gohugo.io/) framework is used to generate the site

## Standard Workflow

1. First think through the problem, read the codebase for relevant files, and write a plan to tasks/todo.md.
2. The plan should have a list of todo items that you can check off as you complete them
3. Before you begin working, check in with me and I will verify the plan.
4. Then, begin working on the todo items, marking them as complete as you go.
5. Please, every step of the way, give me a high level explanation of what changes you made
6. Make every task and code change you do as simple as possible. We want to avoid making any massive or complex changes. Every change should impact as little code as possible. Everything is about simplicity.
7. Finally, add a review section to the todo.md file with a summary of the changes you made and any other relevant information.

## Hugo Conventions
- The generated site content is written to the `public` directory
- Static content to be referenced by Hugo Markdown files is found under the `static` directory. 
  - The contents of the `static/books` and `static/images` files are not checking in to git; they are large binary files that do not change. 
  - You should never, under any circumstances, modify, delete or add to the files under the `static` directory; always leave that to me.
- The [Newsroom](https://github.com/onweru/newsroom) Hugo Theme, used as a foundation for the web site, is located under the `themes/newsroom` directory.
- Style sheet definitions and JavaScript files are located under the `assets` directory. 

## JavaScript Packages
- The web site's dynamic image gallery display is implemented using the [lightGallery](https://www.lightgalleryjs.com/) package. The source code for this is located under the `asssets/js` directory.

## git Conventions
- git shall be used for change management.
- Check with me before committing.
- Never push to remote; do not push to remote under any circumstances. 
- Never change any of the files or directories named in the `.gitignore` file.

## Deployment to AWS
- The generated site content is deployed to an AWS S3 bucket.
- Never deploy the site yourself; do not deploy the site under any circumstances.
- For reference only, deployment is achieved using the the `sync.sh` script, but you should never run this script.

## Development Workflow
- **Test:** `hugo server -D --config config.toml,private.toml`
- **Generate:** `hugo --config config.toml,private.toml`


