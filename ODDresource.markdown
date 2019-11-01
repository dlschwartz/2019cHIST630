---
layout: page
title: ODD Resources
permalink: /ODDresources/
---

This page contains information and links on the basics of producing an 
ODD file for customizing the TEI guidelines. For a more detailed discussion 
of this material, see the chapter on 
["Documentation Elements"](https://www.tei-c.org/release/doc/tei-p5-doc/en/html/TD.html){:target="_blank"} 
in the TEI Guidelines. It might help to think or this page as a cheatsheet
from which you can cut-and-paste into your ODD as you work.

## Minimal Requirements for Valid ODD

### \<teiHeader\>
The easiest way to see what is minimally required in the header is to 
open a new file in oXygen and select ODD Customization\[TEI\]. If you have 
gotten far enough with TEI to be thinking about writing your own customization
I'm going to assume you understand the basics of the header and what you 
might want to do with the header in your ODD.

### [\<schemaSpec\>](https://www.tei-c.org/release/doc/tei-p5-doc/en/html/ref-schemaSpec.html){:target="_blank"}
All of the actual customization will go inside the \<schemaSpec\> element.

```
<schemaSpec ident="[add name of your customization]" start="TEI">  
    <moduleRef key="header"/>  
    <moduleRef key="core"/>  
    <moduleRef key="tei"/>  
    <moduleRef key="textstructure"/>  
</schemaSpec>
```

### [\<moduleRef\>](https://www.tei-c.org/release/doc/tei-p5-doc/en/html/ref-moduleRef.html){:target="_blank"}
The \<moduleRef\> element allows you to specify which elements you will 
include in your customization. You use the @key attribute to name the 
modules you want to include. You may include all 22 modules but you 
must at least include the four required modules shown here. 

```
<schemaSpec ident="[add name of your customization]" start="TEI">  
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

## Customizing Elements

### [\<elementSpec\>](https://www.tei-c.org/release/doc/tei-p5-doc/en/html/ref-elementSpec.html){:target="_blank"}
The \<elementSpec\> is used to make modifications to an element. These 
modifications can include changes to content models, deleting attributes 
from an element, or constraining attribute values for project specific 
needs. 
```
<elementSpec module="[add]" ident="[add]" mode="[add]">
    ...
</elementSpec>
```
* @module: The use of the @module attribute here might seem unnecessary but you 
should always use it. It is required if you will process your ODD into RelaxNG 
using Roma. You can find the TEI module that an element is part of in the TEI 
guidelines.

* @ident: The @ident attribute takes the name of the element you want to modify.

* @mode: The @mode attribute indicates what you would like to do in modifying 
the element. The acceptable values are "add", "delete", "change", and "replace". 
	* **add**: Use this value to define a new element. I recommend that you focus your 
	TEI customization on specifying a subset of the TEI P5 All that you wish to use
	rather than creating new elements (or attributes). If you choose to add an element,
	this is how you would do it.
	* **delete**: Use this value to remove an element from the customization.
	* **change**: Use this value to modify how an element functions in your customization. 
	This is the value I use most often and the option I would recommend for those starting 
	new to ODD customization.
	* **replace**: Use this to replace the TEI specifications for the given TEI element. 
	As with "add" above, this customization will take you out of the realm of specifying 
	subset of the TEI P5 All and into the realm of dramatically changing how a 
	defined TEI element should work. 
	
For more on \<elementSpec\>, see [the TEI Guidelines](https://www.tei-c.org/release/doc/tei-p5-doc/en/html/TD.html#TDcrystals){:target="_blank"}.
	
### [\<content\>](https://www.tei-c.org/release/doc/tei-p5-doc/en/html/ref-content.html){:target="_blank"}
By customizing the content of an element you state which elements may appear as children
of that element. A self-closing \<empty\> element inside [\<content\>] means that no elements 
may appear. If the aim of the customization is to specify which elements can appear,
use \<textNode\> and \<elementRef\> elements. 
```
<elementSpec ident="sponsor" module="header" mode="change">
    <content>
        <textNode/>
    </content>
</elementSpec>
```
The customization above would indicate that the only acceptable child node for a \<sponsor\> element 
is a text node. 
```
<elementSpec ident="editionStmt" module="header" mode="change">
    <content>
        <elementRef key="edition" minOccurs="1" maxOccurs="1"/>
    </content>
</elementSpec>
```
The customization above would indicate that the only acceptable child node for an \<editionStmt\>
element is an \<edition\> element. Additionally, the use of @minOccurs and @maxOccurs 
indicates that this \<edition\> can only appear once.

#### Ordering Child Elements
Note that using multiple \<elementRef\> elements will result in the requirement that the 
elements appear in the order in which they are listed. 
```
<elementSpec ident="place" module="namesdates" mode="change">
    <content>
        <elementRef key="placeName" minOccurs="1" maxOccurs="unbounded"/>
        <elementRef key="desc" minOccurs="1" maxOccurs="unbounded"/>
        <elementRef key="location" minOccurs="0" maxOccurs="unbounded"/>
        <elementRef key="event" minOccurs="0" maxOccurs="unbounded"/>
    </content>
</elementSpec>
```

If you want an un-ordered set of elements or if you simply want to be more 
intentional about element order, you can put your \<elementRef\> elements 
inside a \<sequence\> element with a @preserveOrder attribute. In the example 
below, if the value of the @preserveOrder attribute is "true," then the 
customization functions the same as the example above. If "false," then the 
elements listed can appear in any order. 
```
<elementSpec ident="place" module="namesdates" mode="change">
    <content>
        <sequence preserveOrder="[true|false]">
            <elementRef key="placeName" minOccurs="1" maxOccurs="unbounded"/>
            <elementRef key="desc" minOccurs="1" maxOccurs="unbounded"/>
            <elementRef key="location" minOccurs="0" maxOccurs="unbounded"/>
            <elementRef key="event" minOccurs="0" maxOccurs="unbounded"/>
        </sequence>
    </content>
</elementSpec>
```
Note as well that \<sequence\> without a @preserveOrder attribute has a default 
value equivalent to including the attribute with a value of "false." In general, 
I find it best to err on the side of specificity and put \<elementRef\> 
elements inside \<sequence\> along with a @preserveOrder attribute.

Where \<sequence\> allows you to indicate what elements can appear in a given context 
and whether or not they must appear in order, the \<alternate\> elements allows you 
to specify that one (and only one) of several elements appear. In the example below,
the \<editor\> element must contain either a \<persName\> element or a text node 
but it cannot contain both.
```
<elementSpec ident="editor" module="core" mode="change">
    <content>
        <alternate>
            <elementRef key="persName"/>
            <textNode/>
        </alternate>
    </content>
</elementSpec>

```
Your customization can also combine the use of \<sequence\> and \<alternate\>. 
In the example below, the customization requires a sequence with a preserved 
order. However, the second element in that sequence can be either \<orgName\> 
or \<name\> but not both.

```
<elementSpec ident="respStmt" module="header" mode="change">
    <content>
        <sequence preserveOrder="true">
            <elementRef key="resp" minOccurs="1" maxOccurs="1"/>
            <alternate>
                <elementRef key="orgName" minOccurs="1" maxOccurs="1"/>
                <elementRef key="name" minOccurs="1" maxOccurs="1"/>
            </alternate>
        </sequence>
    </content>
</elementSpec>
```
In this example from the TEI guidelines, the customization permits any one 
(and only one) of the three sequences.
```
<content>
    <alternate>
        <sequence>
            <elementRef key="sic"/>
            <elementRef key="corr"/>
        </sequence>
        <sequence>
            <elementRef key="orig"/>
            <elementRef key="reg"/>
        </sequence>
        <sequence>
            <elementRef key="abbr"/>
            <elementRef key="expan"/>
        </sequence>
    </alternate>
</content>
```

## Customizing Attributes on Elements
(forthcoming!)
