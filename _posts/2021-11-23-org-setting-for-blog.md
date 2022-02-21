---
layout: post
title: "How to set up org mode for blogging"
date:   2021-11-23 18:00:00 +0800
tags: blog emacs org pandoc
---

This blog is on how I set up my blog pulishing process in emacs org
mode.

<!--more-->

# Setting up essentials

## Org mode

I started to use org mode to document my work several years ago. Before
that I am using markdown for this purpose. After know how powerful org
mode is, I can\'t help but want to do everything in org mode.

[Org mode](https://orgmode.org/) is not only a text mode. You can use it
as your calendar, to-do list, and work report. You can insert code
snippet and even run in it to get the return results of your snippet.
You can even play games in it. But as a text mode, it gives you the
power to output you org file into different formats, including pdf,
latex, markdow, MS doc, html etc.

If that sounds a ton of funs to you, please try org mode out!

## ox-pandoc & pandoc

I have been searching for a good package to convert org to markdown. And
I discover this beautiful package
[ox-pandoc](https://github.com/emacsorphanage/ox-pandoc) that use pandoc
as the backend to convert your org file into other format.
[Pandoc](https://pandoc.org/) is one of my old friends, and I used to
write my PhD thesis in markdown and use it to convert into pdf with a
latex template. It is very versitile. If you have even been driven crazy
on converting file to different formats, you should definitely check
pandoc out.

# Set up publishing directory

## Directory structure

This is the structure that I used to organize my blog.

    myProject
    ├───org
    └───myBlog
        ├───assets
        ├───_data
        ├───_includes
        ├───_layouts
        ├───_posts
        ├───_sass
        └───_site

I write and put all my org files under `org` directory, and then I
publish all org into markdown under `myBlog/_posts/`.

## Convert org to markdown

In org mode, you can export the current org file into markdown using
`C-c C-e p` and choose from the options that you want. If you have
ox-pandoc ready, you can see a list of pandoc output options.

If you did not see the markdown option, you need to go to `ox-pandoc.el`
and uncomment the following lines as you needed:

``` elisp
;;(?k "to markdown." org-pandoc-export-to-markdown)
;;(?k "to markdown and open." org-pandoc-export-to-markdown-and-open)
;;(?K "as markdown." org-pandoc-export-as-markdown)
```

After that, you need to recompile the `ox-pandoc.el` by `M-x
    emacs-lisp-byte-compile`. Now you get your markdown option under
pandoc.

## Publish all of your organize

It would be tedious to hit the export everytime you want to convert you
org to markdown. One option org mode offers you is to publish a project.
This means that you can publish all org files under your project in one
command (you can read more on
[publishing](https://orgmode.org/manual/Publishing.html#Publishing)).

First, you need to set up a project. Simply define the source org files
and destination of the markdown files in your `init.el`:

``` elisp
(setq org-publish-project-alist
  '(("my-blog"
     :base-directory "/path/to/myProject/org/"
     :base-extension "org"
     :publishing-directory "/path/to/myProject/myBlog/_posts/"
     :publishing-function (org-pandoc-publish-to-markdown))
    ("all" :components ("my-blog"))))
```

The publish command is `C-c C-e P x` and then select `my-blog`. Then all
the org files would be executed in the publishing function and get
converted into markdown.

## Some extrac works

### Publish using ox-pandoc backend

In the previous section, I use a user-defined function
`org-pandoc-publish-to-markdown` for the conversion. The reason is that
the ox-pandoc conversion function is not compatible with the org mode
publish function. If I simply supply a ox-pandoc function
`org-pandoc-export-to-markdown` to the `publishing-function`, there will
be an error:

    Publishing file /path/to/pubtest.org using `org-pandoc-export-to-markdown'
    Initializing asynchronous export process
    Publishing file /path/to/pubtest.org using `org-pandoc-export-to-markdown'
    Process `org-export-process' exited abnormally

The reason being that the `org-pandoc-export-to-markdown` does not take
arguments used during org mode export (see this
[answer](https://emacs.stackexchange.com/a/49075/33616)). Therefore, you
need to construct a function to take such arguments and also run pandoc
conversion. Luckily, pgorczak has come up with a solution for this
function in this
[issue](https://github.com/kawabata/ox-pandoc/issues/18#issuecomment-262979338)
(thank you so much!). You only need to put these in your `init.el`:

``` elisp
(defun org-pandoc-publish-to (format plist filename pub-dir)
  (setq org-pandoc-format format)
  (let ((tempfile
   (org-publish-org-to
    'pandoc filename (concat (make-temp-name ".tmp") ".org") plist pub-dir))
  (outfile (format "%s.%s"
           (concat
            pub-dir
            (file-name-sans-extension (file-name-nondirectory filename)))
           (assoc-default format org-pandoc-extensions))))
    (let ((process
     (org-pandoc-run tempfile outfile format 'org-pandoc-sentinel
             org-pandoc-option-table))
    (local-hook-symbol
     (intern (format "org-pandoc-after-processing-%s-hook" format))))
      (process-put process 'files (list tempfile))
      (process-put process 'output-file filename)
      (process-put process 'local-hook-symbol local-hook-symbol))))

(defun org-pandoc-publish-to-markdown (p f pd)
  (org-pandoc-publish-to 'markdown p f pd))
```

### Pass front matter to markdown

In order to function normally, the markdown file needs a front matter
block to specify layout, title, time etc.

We can pass such header by adding a structure template block at the
start of the org file using `C-c C-, E`, and change the block attribute
to markdown.

    #+begin_export markdown
    ---
    layout: post
    title: "How to set up org mode for blogging"
    date:   2021-11-23 18:00:00 +0800
    categories: blog
    ---
    #+end_export

However, the ox-pandoc by default enable `--standalone` for all format.
This would automaticall create a empty front matter in the final
markdown. This will cause the header of the output markdown to look like
this:

    ---
    ---

    ---
    layout: post
    title: "How to set up org mode for blogging"
    date:   2021-11-23 18:00:00 +0800
    categories: blog
    ---

In order to fix this, we can simply turn off `--standalone` for markdown
mode only and we pass our own front matter to pandoc. Add this to your
`init.el`:

``` elisp
;; ox-pandoc default options for all output formats
(setq org-pandoc-options '((standalone . t)))
;; cancel above standalone 'markdown' format so that we can pass front matter in org mode.
(setq org-pandoc-options-for-markdown '((standalone . nil)))
```

Now you are all good to go. Go ahead and publish your blogs in one go!
