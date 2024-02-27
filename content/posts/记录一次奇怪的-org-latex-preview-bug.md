+++
title = "记录一次奇怪的 org-latex-preview bug"
author = ["lkcp"]
date = 2024-02-01T12:36:00+08:00
tags = ["orgmode", "latex"]
draft = false
+++

## 问题 {#问题}

最近发现在使用 orgmode 记笔记时无法在 org 文档中使用 "C-c C-x C-l" 预览 latex 公式，报错是 \\(a=3329\\)

```shell
org-compile-file: File "/private/var/folders/rh/nz_99d411h3ffn40jn65798h0000gn/T/orgtexQPAbPt.dvi" wasn’t produced.  Please adjust ‘dvipng’ part of ‘org-preview-latex-process-alist’.
```

但是观察 **Org Preview Latex Output** buffer 中的 log, 可以看到，其实跟 dvipng 根本没有关系，而是

> ! I can't find file \`../../../../var/folders/rh/nz_99d411h3ffn40jn65798h0000gn/
> T/orgtexQPAbPt.tex'.

找不到这个生成的临时 tex 文件，但是当我在这个文件夹下直接 pdflatex orgtexQPAbPt.tex 是可以正确生成对应的 pdf 的。随后我又尝试在 ~/ 目录下以及 ~/repo 目录下尝试编译该文件，都没有问题。但是在 ~/OneDrive 及其子目录下都会存在这个问题。


## 发现 {#发现}

直到我尝试在 OneDrive 路径下使用相似的相对路径执行 python3 ../repo/test.py, 看到 python 的报错信息如下

```shell
Python: can't open file '/Users/username/Library/CloudStorage/OneDrive-个人/../repo/a.py': [Errno 2] No such file or directory
```

我才意识到，原来 "~/OneDrive" 只是一个软链接，实际的文件夹是在 "~/Library/CloudStorage/OneDrive-个人/", 我尝试通过这一路径打开 org 文件，然后执行 prg-latex-preview, 果然问题解决了。
但是这样会使得每次访问路径很长，不太方便，所以还得进一步找更好的解决方案，让 emacs 和 latex 达成一致。


## 解决 {#解决}

最终还是在论坛求助得到了解决：

1.  这是 orgmode 的一个 bug, 9.7 版本已经不存在这个问题了，所以可以通过更新 orgmode 的方式来解决
2.  如果不更新 orgmode
    一种方法是使得 emacs 总是访问其真实的路径，即设置
    ```emacs-lisp
    (setq find-file-visit-truename t)
    ```
    另一种方法就是设置 dvipng 的执行路径，查看 org-preview-latex-process-alist 的定义,其中对占位符给出定义如下
    Place-holders used by ‘:image-converter’ and ‘:latex-compiler’:

    -   %f    input file name
    -   %b    base name of input file
    -   %o    base directory of input file
    -   %O    absolute output file name

    所以是 dvipng 下的 %f 导致了错误的路径，更改为 "%o/%b.type" 即可。
    即更改默认的 org-preview-latex-process-alist 如下：
    ```emacs-lisp
    (setq org-preview-latex-process-alist
    '((dvipng :programs
        ("latex" "dvipng")
        :description "dvi > png" :message "you need to install the programs: latex and dvipng." :image-input-type "dvi" :image-output-type "png" :image-size-adjust
        (1.0 . 1.0)
        :latex-compiler
        ("latex -interaction nonstopmode -output-directory %o %o/%b.tex")
        :image-converter
        ("dvipng -D %D -T tight -o %O %o/%b.dvi")
        :transparent-image-converter
        ("dvipng -D %D -T tight -bg Transparent -o %O %f"))
     (dvisvgm :programs
         ("latex" "dvisvgm")
         :description "dvi > svg" :message "you need to install the programs: latex and dvisvgm." :image-input-type "dvi" :image-output-type "svg" :image-size-adjust
         (1.7 . 1.5)
         :latex-compiler
         ("latex -interaction nonstopmode -output-directory %o %f")
         :image-converter
         ("dvisvgm %f --no-fonts --exact-bbox --scale=%S --output=%O"))
     (imagemagick :programs
             ("latex" "convert")
             :description "pdf > png" :message "you need to install the programs: latex and imagemagick." :image-input-type "pdf" :image-output-type "png" :image-size-adjust
             (1.0 . 1.0)
             :latex-compiler
             ("pdflatex -interaction nonstopmode -output-directory %o %f")
             :image-converter
             ("convert -density %D -trim -antialias %f -quality 100 %O")))
    )
    ```

当 "dvi &gt; svg" 或者 "pdf &gt; png" 出现问题时则对应地修改另外两
