# Introduction to SSI Injection

---

Server-Side Includes (SSI) is a technology web applications use to create dynamic content on HTML pages. SSI is supported by many popular web servers such as [Apache](https://httpd.apache.org/docs/current/howto/ssi.html) and [IIS](https://learn.microsoft.com/en-us/iis/configuration/system.webserver/serversideinclude). The use of SSI can often be inferred from the file extension. Typical file extensions include `.shtml`, `.shtm`, and `.stm`. However, web servers can be configured to support SSI directives in arbitrary file extensions. As such, we cannot conclusively conclude whether SSI is used only from the file extension.

---

## SSI Directives

SSI utilizes `directives` to add dynamically generated content to a static HTML page. These directives consist of the following components:

- `name`: the directive's name
- `parameter name`: one or more parameters
- `value`: one or more parameter values

An SSI directive has the following syntax:

Code: ssi

```ssi
<!--#name param1="value1" param2="value" -->
```

For instance, the following are some common SSI directives.

#### printenv

This directive prints environment variables. It does not take any variables.

Code: ssi

```ssi
<!--#printenv -->
```

#### config

This directive changes the SSI configuration by specifying corresponding parameters. For instance, it can be used to change the error message using the `errmsg` parameter:

Code: ssi

```ssi
<!--#config errmsg="Error!" -->
```

#### echo

This directive prints the value of any variable given in the `var` parameter. Multiple variables can be printed by specifying multiple `var` parameters. For instance, the following variables are supported:

- `DOCUMENT_NAME`: the current file's name
- `DOCUMENT_URI`: the current file's URI
- `LAST_MODIFIED`: timestamp of the last modification of the current file
- `DATE_LOCAL`: local server time

Code: ssi

```ssi
<!--#echo var="DOCUMENT_NAME" var="DATE_LOCAL" -->
```

#### exec

This directive executes the command given in the `cmd` parameter:

Code: ssi

```ssi
<!--#exec cmd="whoami" -->
```

#### include

This directive includes the file specified in the `virtual` parameter. It only allows for the inclusion of files in the web root directory.

Code: ssi

```ssi
<!--#include virtual="index.html" -->
```
## Exploitation
see this scenario. We have a web server that has a login page which once given the correct credentials redirects us to a `.shtml` page and prints our username on top of the page, we can register a new user with an SSI payload like `<!--exec cmd="whoami" -->`. and once we login again, if the server is vulnerable to SSI it will parse the payload and execute the `whoami` command and prints it instead of the username.