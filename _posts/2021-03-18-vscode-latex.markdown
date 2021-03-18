---
layout:     post
title:      "vscode配置latex"
subtitle:   " \"在vscode中快速配置latex环境 \""
date:       2021-03-18 12:00:00
author:     "paulikarl"
header-img: "img/post-bg-2015.jpg"
catalog:    true
tags:
    - vscode
    - latex
---

>windows10+texlive2020+sumatraPDF+vscode

默认vscode已经安装完成，下载安装texlive2020和sumatraPDF

##### 1. 安装vscode插件latex workshop

##### 2. 打开settings文件，配置编译过程

```
{
    "latex-workshop.latex.tools": [
        {
            // 编译工具和命令
            "name": "xelatex",
            "command": "xelatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "-pdf",
                "%DOCFILE%"
            ]
        },
        {
            "name": "pdflatex",
            "command": "pdflatex",
            "args": [
                "-synctex=1",
                "-interaction=nonstopmode",
                "-file-line-error",
                "%DOCFILE%"
            ]
        },
        {
            "name": "bibtex",
            "command": "bibtex",
            "args": [
                "%DOCFILE%"
            ]
        }
    ],
    "latex-workshop.latex.recipes": [
        // {
        //     "name": "xelatex",
        //     "tools": [
        //         "xelatex"
        //     ],
        // },
        // {
        //     "name": "pdflatex",
        //     "tools": [
        //         "pdflatex"
        //     ]
        // },
        {
            "name": "xe->bib->xe->xe",
            "tools": [
                "xelatex",
                "bibtex",
                "xelatex",
                "xelatex"
            ]
        },
        // {
        //     "name": "pdf->bib->pdf->pdf",
        //     "tools": [
        //         "pdflatex",
        //         "bibtex",
        //         "pdflatex",
        //         "pdflatex"
        //     ]
        // }
    ],

    "latex-workshop.view.pdf.viewer": "external",
    "latex-workshop.view.pdf.viewer": "external",

    "latex-workshop.view.pdf.external.viewer.command": "G:/SumatraPDF/SumatraPDF.exe",
    "latex-workshop.view.pdf.external.viewer.args": [
        "-forward-search",
        "%TEX%",
        "%LINE%",
        "-reuse-instance",
        "-inverse-search",
        "\"C:/Users/79181/AppData/Local/Programs/Microsoft VS Code/Code.exe\" \"C:/Users/79181/AppData/Local/Programs/Microsoft VS Code/resources/app/out/cli.js\" -gr \"%f\":\"%l\"",
        "%PDF%"
    ],
    "files.autoSave": "afterDelay",
    "latex-workshop.view.pdf.external.synctex.command": "G:/SumatraPDF/SumatraPDF.exe",
    "latex-workshop.view.pdf.external.synctex.args": [
        "-forward-search",
        "%TEX%",
        "%LINE%",
        "-reuse-instance",
        "-inverse-search",
        "\"C:/Users/79181/AppData/Local/Programs/Microsoft VS Code/Code.exe\" \"C:/Users/79181/AppData/Local/Programs/Microsoft VS Code/resources/app/out/cli.js\" -gr \"%f\":\"%l\"",
        "%PDF%",
    ],
    "editor.wordWrap": "on",
    "window.menuBarVisibility": "visible",
}
```

注意：需要替换自己的sumatraPDF.exe路径和vscode相关路径即可

重启！！ok

##### 分享网址：

1. [伪代码生成](https://blog.csdn.net/golden1314521/article/details/40923377)
2. [常用数学符号](https://blog.csdn.net/Ying_Xu/article/details/51240291)
3. [表格生成器](https://www.tablesgenerator.com/)
4. [模板网站](https://www.latexstudio.net/)