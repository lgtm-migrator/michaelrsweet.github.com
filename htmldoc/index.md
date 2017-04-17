---
title: HTMLDOC
subtitle: HTML Conversion Software
author: Michael R Sweet
copyright: Copyright © 1997-2017 by Michael R Sweet
project: htmldoc
project_name: HTMLDOC
layout: project
---

HTMLDOC is a program that reads HTML source files or web pages and generates
corresponding HTML, PostScript, or PDF files with an optional table of contents.
HTMLDOC is provided under version 2 of the GNU General Public License.

HTMLDOC was developed in the 1990's as a documentation generator for my previous
company, and has since seen a lot of usage as a report generator embedded in web
servers.  However, it does not support many things in "the modern web", such
as:

- Cascading Style Sheets (CSS): While I have experimented with adding CSS
  support to HTMLDOC, proper CSS support is non-trivial especially for paged
  output (which is not well supported by CSS)
- Encryption: HTMLDOC currently supports the older (and very insecure) PDF 1.4
  (128-bit RC4) encryption.  I have looked at supporting AES (256-bit)
  encryption...
- Forms: HTML forms and PDF forms are very different things.  While I have had
  many requests to add support for PDF forms over the years, I have not found a
  satisfactory way to do so.
- Tables: HTMLDOC supports HTML 3.2 tables, which means no support for TBODY,
  THEAD, or TFOOT.  I have looked at supporting THEAD and TBODY...
- Unicode: While HTMLDOC does support UTF-8 for "Western" languages, there is
  absolutely no support for languages that require dynamic rewriting or
  right-to-left text formatting.  Basically this means you can't use HTMLDOC to
  format Arabic, Chinese, Hebrew, Japanese, or other languages that are not
  based on latin-based alphabets that read left-to-right.

I plan to do a limited [1.9 feature release](https://github.com/michaelrsweet/htmldoc/milestone/3)
of HTMLDOC to add support for Markdown input and EPUB output.  Otherwise my
focus is on addressing security, build, and formatting issues and not on
extending HTMLDOC to support CSS or full Unicode.