---
title: "Troubleshooting for Firefox showing error while downloading .xlsx file"
date: 2021-09-29T10:57:25+08:00
description: "solution for Firefox incorrectly identifies .xlsx"
menu:
  sidebar:
    name: Troubleshooting for Firefox showing error while downloading .xlsx file
    identifier: Firefox-download-excel
    parent: Web-Development
    weight: 10
<!--- hero: images/excel1.png --->
---
While downloading the excel file in firefox, it shows a message box of 'What should Firefox do with this file?', giving options are open or save the file, if choose to open it, firefox opens it as 'file.xlsx.xls' and the error comes up:
*The file format and extension of 'filename.xlsx.xls' don't match. The file could be corrupted or unsafe. Unless you trust its source, don't open it. Do you want to open it anyway?*

This is a problem especially for Firefox, if downloaded with other browsers there is no error, and it has to do with the MIME type.

Solution is to change the type from `application/vnd.ms-excel` to `application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`
```Vue
      const url = window.URL.createObjectURL(
        new Blob([data], {
          type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet',
        })
      );
```

Because `application/vnd.ms-excel` represents xls, Firefox is giving the file a proper extension based on the information provided by the server, which is the MIME type that we put for the file.

As developer.mozilla.org mentions:
> Browsers use the MIME type, not the file extension, to determine how to process a URL, so it's important that web servers send the correct MIME type in the response's Content-Type header. If this is not correctly configured, browsers are likely to misinterpret the contents of files, sites will not work correctly, and downloaded files may be mishandled.

Here are the correct Microsoft Office MIME types for HTTP content streaming:
```
Extension MIME Type
.doc      application/msword
.dot      application/msword

.docx     application/vnd.openxmlformats-officedocument.wordprocessingml.document
.dotx     application/vnd.openxmlformats-officedocument.wordprocessingml.template
.docm     application/vnd.ms-word.document.macroEnabled.12
.dotm     application/vnd.ms-word.template.macroEnabled.12

.xls      application/vnd.ms-excel
.xlt      application/vnd.ms-excel
.xla      application/vnd.ms-excel

.xlsx     application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
.xltx     application/vnd.openxmlformats-officedocument.spreadsheetml.template
.xlsm     application/vnd.ms-excel.sheet.macroEnabled.12
.xltm     application/vnd.ms-excel.template.macroEnabled.12
.xlam     application/vnd.ms-excel.addin.macroEnabled.12
.xlsb     application/vnd.ms-excel.sheet.binary.macroEnabled.12

.ppt      application/vnd.ms-powerpoint
.pot      application/vnd.ms-powerpoint
.pps      application/vnd.ms-powerpoint
.ppa      application/vnd.ms-powerpoint

.pptx     application/vnd.openxmlformats-officedocument.presentationml.presentation
.potx     application/vnd.openxmlformats-officedocument.presentationml.template
.ppsx     application/vnd.openxmlformats-officedocument.presentationml.slideshow
.ppam     application/vnd.ms-powerpoint.addin.macroEnabled.12
.pptm     application/vnd.ms-powerpoint.presentation.macroEnabled.12
.potm     application/vnd.ms-powerpoint.template.macroEnabled.12
.ppsm     application/vnd.ms-powerpoint.slideshow.macroEnabled.12

.mdb      application/vnd.ms-access
```

Only a few are listed here, for full media types pleae refer [this page](https://www.iana.org/assignments/media-types/media-types.xhtml)