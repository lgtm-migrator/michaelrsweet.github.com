---
title: How to Use the zipc "Library"
subtitle: A Simple ZIP Container "Library"
author: Michael R Sweet
copyright: Copyright © 2017 by Michael R Sweet.
project: zipc
project_name: zipc
layout: project
...

# How to Use the zipc "Library"

## Opening a ZIP Container for Reading

The `zipcOpen` function opens a ZIP container:

    zipc_t *zc = zipcOpen("filename.epub", "r");


## Reading Files from a ZIP Container

> Note: You can only read one file from a ZIP container at a time.

The `zipcOpenFile` function opens a file within the ZIP container:

    zipc_file_t *zf = zipcOpenFile(zc, "META-INF/container.xml");

The `zc` argument is the container and "META-INF/container.xml" specifies the
filename within the container.

Once opened, the `zipcFileRead` function can be used to read arbitrary bytes
from the file in the container:

    char buffer[1024];
    ssize_t bytes = zipcFileRead(zf, buffer, sizeof(buffer));

Lines of text can be read using the `zipcFileGets` function:

    char line[1024];
    while (zipcFileGets(zf, line, sizeof(line)) == 0)
      puts(line);

XML elements and text fragments can be read using the `zipcFileXMLGets`
function:

    char xml[1024];
    while (zipcFileXMLGets(zf, xml, sizeof(xml)) == 0)
      puts(xml);

When all data has been read, call the `zipcFileFinish` function to "close"
it:

    zipcFileFinish(zf);


## Creating a ZIP Container for Writing

The `zipcOpen` function creates a ZIP container:

    zipc_t *zc = zipcOpen("filename.epub", "w");


## Adding Files to a ZIP Container

> Note: You can only add one file to a ZIP container at a time.

The `zipcCreateFile` function creates a file within the ZIP container:

    zipc_file_t *zf = zipcCreateFile(zc, "META-INF/container.xml", 1);

The `zc` argument is the container, "META-INF/container.xml" specifies the
filename within the container, and the `1` value specifies that the file should
be compressed.  You would specify `0` to add the file without compression.

Once created, the `zipcFilePrintf`, `zipcFilePuts`, `zipcFileWrite`, and
`zipcFileXMLPrintf` functions can be used to write to the file in the container:

    zipcFilePuts(zf, "<?xml version=\"1.0\" encoding=\"utf-8\"?>\n");

    zipcFilePrintf(zf, "<element>%d</element>\n", intvalue);

    zipcFileWrite(zf, buffer, length);

    zipcFileXMLPrintf(zf, "<element>%s</element>\n", stringvalue);

When all data has been written, call the `zipcFileFinish` function to "close"
it:

    zipcFileFinish(zf);

Sometimes ZIP container files contain short constant strings.  These may be
added using the `zipcCreateFileWithString` function:

    zipcCreateFileWithString(zc, "metadata", "application/epub+zip");

Files created in this way are always stored uncompressed.


## Adding Directories to a ZIP Container

Although strictly not required, you can add directories to a ZIP container
using the `zipcCreateDirectory` function:

    zipcCreateDirectory(zc, "META-INF/");

If you forget to include a trailing slash, one will be added for you.


## Copying Files to a ZIP Container

The `zipcCopyFile` function efficiently copies a file to the ZIP container:

    /* Copy text file and compress */
    zipcCopyFile(zc, "CONTENT/filename.html", "tempfile.html", 1, 1);

    /* Copy image file and don't compress */
    zipcCopyFile(zc, "CONTENT/filename.png", "tempfile.png", 0, 0);

    /* Copy binary data file and compress */
    zipcCopyFile(zc, "CONTENT/filename.dat", "tempfile.dat", 0, 1);


## Closing the ZIP Container

Once you have added all of the files you need to the container, close it with
the `zipcClose` function:

    zipcClose(zc);


## Error Handling/Reporting

The various functions either return `-1` or a `NULL` pointer on error.  You can
call the `zipcError` function to get a string describing the last error for
the specified container:

    printf("Error: %s\n", zipcError(zc));


# Reference

## zipc_t

    typedef struct _zipc_s zipc_t;

The `zipc_t` object tracks a ZIP container that was opened with the `zipcOpen`
function.


## zipc\_file\_t

    typedef struct _zipc_file_s zipc_file_t;

The `zipc_file_t` object tracks a single file in a ZIP container that was
created using the `zipcCreateFile` function.


## zipcClose

    int
    zipcClose(zipc_t *zc);

The `zipcClose` function closes a ZIP container file, writing the central
directory for every file and directory in the container.  `0` is returned on
success or `-1` on error.


## zipcCopyFile

    int
    zipcCopyFile(zipc_t *zc, const char *dstname, const char *srcname,
                 int text, int compressed);

The `zipcCopyFile` function copies an existing file `srcname` into a ZIP
container file `dstname`.  If the `text` parameter is non-zero then the file
will be stored as plain text.  If the `compressed` parameter is non-zero then
the file will be compressed when copied into the container.  `0` is returned on
success or `-1` on error.


## zipcCreateDirectory

    int
    zipcCreateDirectory(zipc_t *zc, const char *filename);

The `zipcCreateDirectory` function creates a directory in the ZIP container.
`0` is returned on success or `-1` on error.


## zipcCreateFile

    zipc_file_t *
    zipcCreateFile(zipc_t *zc, const char *filename, int compressed);

The `zipcCreateFile` function creates a file in the ZIP container for streamed
writing.  If the `compressed` parameter is non-zero then the file will be
compressed in the container.  `NULL` is returned if the file cannot be created.

Data is written to the created file using the `zipcFilePrintf`, `zipcFilePuts`, `zipcFileWrite`, and `zipcFileXMLPrintf` functions.  The file is "closed" using
the `zipcFileFinish` function.


## zipcCreateFileWithString

    int
    zipcCreateFileWithString(zipc_t *zc, const char *filename,
                             const char *contents);

The `zipcCreateFileWithString` function creates a file in the ZIP container that
contains the `contents` string.  The file is not compressed.  `0` is returned on
success or `-1` on error.


## zipcError

    const char *
    zipcError(zipc_t *zc);

The `zipcError` function returns the last error message for a ZIP container.
`NULL` is returned if no error has occurred.


## zipcFileFinish

    int
    zipcFileFinish(zipc_file_t *zf);

The `zipcFileFinish` function "closes" a ZIP container file that was created
using the `zipcCreateFile` function.  `0` is returned on success or `-1` on
error.


## zipcFileGets

    int
    zipcFileGets(zipc_file_t *zf, char *line, size_t linesize);

The `zipcFileGets` function reads a line of text from a ZIP container file that
was opened using the `zipcOpenFile` function.  The line may be terminated by a
carriage return and line feed (Windows-style) or just a line feed (everybody
else).  The line ending is not copied to the line buffer.  `0` is returned on
success or `-1` on error.


## zipcFilePrintf

    int
    zipcFilePrintf(zipc_file_t *zf, const char *format, ...);

The `zipcFilePrintf` function writes a formatted string to a ZIP container file
that was created using the `zipcCreateFile` function.  The format string uses
the same syntax as the `printf` function.  `0` is returned on success or `-1` on
error.


## zipcFilePuts

    int
    zipcFilePuts(zipc_file_t *zf, const char *s);

The `zipcFilePuts` function writes a literal string to a ZIP container file
that was created using the `zipcCreateFile` function.  `0` is returned on
success or `-1` on error.


## zipcFileRead

    ssize_t
    zipcFileRead(zipc_file_t *zf, void *data, size_t bytes);

The `zipcFileRead` function reads a buffer of data from a ZIP container file
that was opened using the `zipcOpenFile` function.  The actual number of bytes
read is returned on success or `-1` on error.


## zipcFileWrite

    int
    zipcFileWrite(zipc_file_t *zf, const void *data, size_t bytes);

The `zipcFileWrite` function writes a buffer of data bytes to a ZIP container
file that was created using the `zipcCreateFile` function.  `0` is returned on
success or `-1` on error.


## zipcFileXMLGets

    int
    zipcFileXMLGets(zipc_file_t *zf, char *fragment, size_t fragsize);

The `zipcFileXMLGets` function reads an XML fragment from a ZIP container file
that was opened using the `zipcOpenFile` function.  The text or element from the
XML file is stored in the buffer pointed to by `fragment` whose size is
`fragsize`.  Text fragments have XML escaping removed.  `0` is returned on
success or `-1` on error.

For example, the following XML:

    <element url="https://example.com?a=1&amp;b=2">foo &amp; bar</element>

is returned as the following sequence of fragment strings:

    <element url="https://example.com?a=1&amp;b=2">
    foo & bar
    </element>

The `zipcXMLGetAttribute` function can be used to retrieve the attribute values
from XML element fragments.


## zipcFileXMLPrintf

    int
    zipcFileXMLPrintf(zipc_file_t *zf, const char *format, ...);

The `zipcFileXMLPrintf` function writes a formatted XML string to a ZIP
container file that was created using the `zipcCreateFile` function.  The format
string supports the "%%", "%d", "%f", and "%s" `printf` directives, where the
"%s" (string) directive handles escaping of special XML/HTML characters.  `0` is
returned on success or `-1` on error.


## zipcOpen

    zipc_t *
    zipcOpen(const char *filename, const char *mode);

The `zipcOpen` function opens a new ZIP container file.  The `mode` argument
specifies how the container is opened:

- "r" indicates that the ZIP container file is opened for reading.
- "w" indicates that the ZIP container file is opened for writing, replacing any
existing file of the given name.

`NULL` is returned if the file cannot be opened or created.


## zipcOpenFile

    zipc_file_t *
    zipcOpenFile(zipc_t *zc, const char *filename);

The `zipcOpenFile` function opens a file in the ZIP container.  `NULL` is
returned if the file cannot be found.


## zipcXMLGetAttribute

    const char *
    zipcXMLGetAttribute(const char *element, const char *attrname,
                        char *buffer, size_t bufsize);

The `zipcXMLGetAttribute` function retrieves the attribute value from an XML
element fragment read using the `zipcFileXMLGets` function.  The value is copied
to the buffer with escaping removed.  `NULL` is returned if the named attribute
is not present in the element.

For example, the "url" attribute value returned from the element string:

    <element url="https://example.com?a=1&amp;b=2">

will be:

    https://example.com?a=1&b=2
