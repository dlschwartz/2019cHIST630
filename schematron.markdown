---
layout: page
title: Schematron Resources
permalink: /schematron/
---

This page contains information and links on the basics of creating and 
embedding schematron rules into an ODD customization of the TEI. 
For an introductory tutorial on writing an ODD customization, see 
the page on [ODD Resources](../ODDResources/). 

### Table of Contents:
* [Introduction](#introduction)
	* [Grammar and Rules](#grammarRules)
	* [Getting Started](#gettingStarted)
	* [Namespaces](#namespaces)
* [Writing Schematron Rules](#writingRules)
	* [\<pattern\> and \<rule\> Elements](#pattern)
	* [Setting the Context](#context)
	* [\<report\> and \<assert\> Elements](#rules)
	* [The Difference between ODD and Schematron](#difference)
	* [Messages and Documentation](#messages)
	* [Errors and Warnings](#errors)
	* [XPath Functions](#xpath)
* [Embedding Rules in Your ODD](#embedding)
* [More Advanced Uses of Schematron](#advanced)
	* [The \<let\> Element](#let)
	* [Creating a List of Values](#list)
	* [Using Values in Messages](#valueOf)
	* [Linking to Your Standoff](#linking)
	
_____
_____
## <a name="introduction"/>Introduction

### <a name="grammarRules"/>Grammar and Rules
A TEI ODD generates a RelaxNG file that articulates a 
customized grammar for the TEI document it is used to 
constrain. This 'grammar-based' approach produces a generalized 
set of constrains that governs how elements and the attributes 
they take function at the level of the document. Beyond such 
general constraints, most projects need a more 'rule-based' 
approach that can accommodate constraints that only apply 
in specific contexts. Schematron allows us to write rules that function 
only in specific contexts. You can associate your TEI file(s)
with both a RelaxNG file and with a separate Schematron file. 
Here we will write and test our Schematron rules in a Schematron 
file but then embed them in our ODD before generating our 
RelaxNG.


### <a name="gettingStarted"/>Getting Started
It is easy to make mistakes writing Schematron rules directly in 
an ODD file. It is best to write and test your rules in a 
Schematron file and then cut-and-paste the rule into your ODD. 
You want to start by simply opening a new file in oXygen and selecting 
a Schematron file. When you do, you should see the following (with some 
additional material inside the root element):
```
<?xml version="1.0" encoding="UTF-8"?>
<sch:schema xmlns:sch="http://purl.oclc.org/dsdl/schematron" queryBinding="xslt2"
    xmlns:sqf="http://www.schematron-quickfix.com/validator/process">
        
        ...
        
</sch:schema>
```

### <a name="namespaces"/>Namespaces
Schematron rules can be embedded in a TEI ODD file but we have to be 
careful about namespaces. The first thing we want to do is modify the 
default namespace declarations in our Schematron file. Additing 
```<sch:ns uri="http://www.tei-c.org/ns/1.0" prefix="tei"\>``` as the 
first child element of the root will help us avoid namespace errors. 
It will require that the rules we write have the "sch:" namespace before 
Schematron elements and the "tei:" namespace before the TEI elements 
we use is writing our rules. See the discussions below for examples of 
namespaces in practice.

```
<?xml version="1.0" encoding="UTF-8"?>
<sch:schema xmlns:sch="http://purl.oclc.org/dsdl/schematron" queryBinding="xslt2"
    xmlns:sqf="http://www.schematron-quickfix.com/validator/process">
    <sch:ns uri="http://www.tei-c.org/ns/1.0" prefix="tei"\>
        ...
        
</sch:schema>
```

_____
_____
## <a name="writingRules"/>Writing Schematron Rules


### <a name="pattern"/>\<pattern\> and \<rule\> Elements
Rules appear in a \<rule\> element that appear in a \<pattern\> element.
```
<sch:pattern>
    <sch:rule context="">
    
    </sch:rule>
</sch:pattern>
``` 

### <a name="context"/>Setting the Context
The value that Schematron brings to our ODD is the ability to write 
context-specific rules. We can write general customizations for how 
a \<p\> element can function in the whole TEI document using pure ODD. 
However, if some of the behaviors of the \<p\> element differ based on 
whether it appears in the header or the body of the document then I 
need Schematron. 

```
<sch:pattern>
    <sch:rule context="tei:teiHeader//tei:p">
        
    </sch:rule>
</sch:pattern>
```

Any rule that I write inside of this \<sch:rule\> element will apply 
only when the \<p\> element appears inside \<teiHeader\>. 

Note as well the use of namespaces on all elements, both Schematron and 
TEI!


### <a name="rules"/>\<report\> and \<assert\> Elements 
The two main child elements of \<sch:rule\> are \<report\> and \<assert\>. 
It can be a bit tricky to keep these ideas separate in your mind. The 
following should help: 
* \<report\> = "send a message if the following is true"
	* The rule you write will be for something that should be excluded from 
	your data.
* \<assert\> = "send a message if the following is not true"
	* The rule you write should be for something that has to be true in 
	your data.
	
The following rule send up an error message if a \<p\> element inside 
\<teiHeader\> contains a \<note\> element. A \<note\> element inside a \<p\> 
element would still be allowed in the \<body\> but it cannot appear in the 
header.  
```
<sch:rule context="tei:teiHeader//tei:p">
    <sch:report test="tei:note">
        ...
    </sch:report>
</sch:rule>
```	

In this example, the rule indicates that the only acceptable values 
for the @resp attribute on a \<note\> element inside the \<body\> are 
"#dls", "#ewb", and "#medComp". Note that the "." in the @test attribute 
is XPath that indicates the current context, in this case the @resp attribute 
designated above.

```
<sch:rule context="tei:body/tei:note/@resp">
    <sch:assert test=". = '#dls' or . = '#ewb' or . = '#medComp'">
        ...
    </sch:assert>
</sch:rule>
```

### <a name="difference"/>The Difference between ODD and Schematron
The following example helps make the difference between ODD and Schematron 
more clear. Here we see the pure ODD customization indicating that 
in the TEI document associated with this schema only the four listed elements 
may appear as children of \<p\>. This means that is the TEI for this project, these 
four elements may in one place or another appear as children of \<p\>. 
In some contexts, however, all four are not allowed so Schematron can be used to 
set that rule.
```
<elementSpec ident="p" module="core" mode="change">
    <content>
        <sequence preserveOrder="false">
            <elementRef key="note"/>
            <elementRef key="pb"/>
            <elementRef key="persName"/>
            <elementRef key="placeName"/>
        </sequence>
    </content>
</elementSpec>
```
```
<sch:rule context="tei:teiHeader//tei:p">
    <sch:report test="tei:note">
        ...
    </sch:report>
</sch:rule>
```


### <a name="messages"/>Messages and Documentation
The writer of the Schematron rule must write the message that 
encoders get when they violate the rule by writing something 
against the rule or failing to write something required by the 
rule. The message is the text node of the \<report\> or \<assert\>
element. It should make clear to the encoder what the error is. 
```
<sch:rule context="tei:teiHeader//tei:p">
    <sch:report test="tei:note">
        A &lt;p&gt; element in the &lt;teiHeader&gt; may not 
        contain a &lt;note&gt; element.
    </sch:report>
</sch:rule>
```

Note that "&lt;" and &gt; are required instead of \< and \> because 
these symbols would be understood as code not allowed in this text node.
When the encoder sees an error message in oXygen these will be rendered 
as the expected bracket symbols.

### <a name="errors"/>Errors and Warnings 
Not every violation of a rule has to result in an error. For example, if your project 
prefers certain values for an attribute but does not require only those values 
you can indicate this with a @role attribute.

```
<sch:rule context="tei:body/tei:note/@resp" role="warning">
    <sch:assert test=". = '#dls' or . = '#ewb' or . = '#medComp'">
        Preferred values: #dls, #ewb, #medComp
    </sch:assert>
</sch:rule>
```

If encoders use a value other than one of these three, they would see 
an yellow warnign message appear in oXygen rather than a red error message. 
This can help encoders use the preferred options and if they persists
in their decision to deviate from the preference, the yellow warning 
message assists with the editing process.

### <a name="xpath"/>XPath Functions

_____ 
_____ 


## <a name="embedding"/>Embedding Rules in Your ODD

_____ 
_____ 


## <a name="advanced"/>More Advanced Uses of Schematron


### <a name="let"/>The \<let\> Element


### <a name="list"/>Creating a List of Values


### <a name="valueOf"/>Using Values in Messages


### <a name="linking"/>Linking to Your Standoff


_____
_____ 

#### Acknowledgements
The resources on this page were produced with input from the following sources:
