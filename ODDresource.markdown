---
layout: page
title: ODD Resources
permalink: /ODDresources/
---

This page contains information and links on the basics of producing an 
ODD file for customizing the TEI guidelines. It might help to think 
of this as a cheatsheet from which you can cut-and-paste into your 
ODD as you work.

### Minimal Requirements for Valid ODD

#### \<teiHeader\>
The easiest way to see what is minimally required in the header is to 
open a new file in oXygen and select ODD Customization\[TEI\]. If you have 
gotten far enough with TEI to be thinking about writing your own customization
I'm going to assume you understand the basics of the header and what you 
might want to do with the header in your ODD.

#### \<schemaSpec\> 
All of the actual customization will go inside the \<schemaSpec\> element.

```
<schemaSpec ident="oddex1" start="TEI">  
    <moduleRef key="header"/>  
    <moduleRef key="core"/>  
    <moduleRef key="tei"/>  
    <moduleRef key="textstructure"/>  
</schemaSpec>
```

#### \<moduleRef\>
The \<moduleRef\> element allows you to specify which elements you will 
include in your customization. You use the @key attribute to name the 
modules you want to include. You may include all 22 modules but you 
must at least include the four required modules shown here. 

```
<schemaSpec ident="oddex1" start="TEI">  
    <moduleRef key="header"/>  
    <moduleRef key="core" include="title"/>  
    <moduleRef key="tei"/>  
    <moduleRef key="textstructure" except="div1 div2 div3 div4 div5 div6 div7"/>  
</schemaSpec>
```
In addition to the @key attribute, the \<moduleRef\> element takes @include 
and @except attributes. The @include attribute specifies the 
elements from that particular module that will be included in the customization 
while the @except attribute indicates that all of the elements from that module 
will be allowed in the customization except the ones listed. Leaving off 
@include and @except simply brings over into your customization all of the 
elements that are a part of the module indicated.

### Customizing Elements

#### \<elementSpec\>
The \<elementSpec\> is used to make modifications to an element.
```

```
