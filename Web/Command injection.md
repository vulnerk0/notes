# Overview
Also known as shell injection, your goal is to pass system commands to the underlying system. if you are in a black box scenario, any parameter might be vulnerable because the application might issue system commands with the HTTP parameters as arguments to the command.

## Scenario 1 - simple
some functionality on the website might be coded to perform system commands. For example, you want to fetch the stock for a product, so the application takes the product id and issues a system command like './check_stock.sh \<product-id\>' you can try to change the product id to something like &sleep 5& and check the result. I used sleep to make sure I have control over the sever because I might not be able to see the output of the command, but in this case I can see the output by using URL-encoded & command separator.

### Labs
[os command injection, simple case](https://portswigger.net/web-security/os-command-injection/lab-simple)

## Scenario 2 - blind
most of the os command injection vulnerabilities do not return an output, you can use the above mentioned technique to confirm that you indeed have command injection capabilities. If you want to see the output you  can redirect the output of the command to a directory that you can access via the web (i.e. /uploads under /var/www/html/uploads), using the greater than operand > lets you redirect output to a desired file. 

#### callback
You can also use the "callback" method which works on alot of vulnerabilities, when you use this method you want the server to send any message to you, it might be a dns lookup on a domain you own or an email or a get request to your server. In the context of command injection you can use `nslookup` to send a message to your domain - which could be a burp collaborator server - or use `curl` to send an HTTP request to your server -if you don't own the domain like ngrok-.

## Protection bypass
Please refer to this gihub page for more infomation:
[PayloadsAllTheThings-command-injection](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/Command%20Injection#bypass-without-space)
#### Blacklisted characters
One way of preventing injection attacks is blacklists, the server checks the user input for any blacklisted characters and if any are found, it will drop the request. If this is the case with command injection we should identify the character that caused the server to drop our request.
##### Identifying blacklisted characters
First thing to do is provide a valid input to the server and gradually build on it until the server drops the request. For example:
```
# Request
GET /index.php?createfile=hello.txt;ls; HTTP/2.0

# Response
invalid input

# Request
GET /index.php?createfile=hello.txt HTTP/2.0

# Response
file created

# Request 
GET /index.php?createfile=hello.txt; HTTP/2.0

# Response
invalid input
```
In this case, the semi-colon is blacklisted
#### Bypassing blacklisted characters
- **Space** : If the application has the space blacklisted we can provide a URL-encoded tab ``%09`` instead, we can also use the **${IFS}** linux env variable as it may hold a space or a tab as its value. Another bypass is the brace expansion like `{ls,-la}`
- **Selection** : What I mean by Selection is selecting a character from an environment variable. We can do this by using substitution in linux like `${PATH:0:1}`:
	- **PATH** : The name of the variable
	- **0** : The start of substitution
	- **1** : Number of characters
	- this would result in `/`
- **Character shifting** : In this technique you will get the character you want (i.e. ; ) and choose the first character before it in the ascii table which is ( : ) and use the following command to shift the character one step up the table which will give us the desired character:
	- `echo $(tr '!-}' '"-~'<<<:) # gives ;`
- The same thing applies in windows:
	- **echo %envvar:~start,-end%** : `echo %HOMEPATH:~6,-11% # gives \` CMD
	- \$env:envvar[index] : `$env:HOMEPATH[0] # gives \` POWERSHELL
	- **Get-ChildItem Env:** : print all environment variables
#### Bypassing blacklisted commands
Usually some commands will be blacklisted, these are some of the basic ways to bypass the blacklisting:
- **quote/double-quote**: If the command has a character between quotes it will ignore the quotes and run the command, while the command will not match the one in the blacklist, it will still run on the system. `w'h'o'a'mi & "wh"o"am"i
##### Linux Only
We can insert other linux only characters and the shell will ignore them. These include but not limited to ( backslash `\` and the positional parameter `$@`):`who$@ami & wh\oam\i`
##### Windows Only
there's one character specific to windows which is the caret `^` like : `who^ami` 
#### Advanced bypass techniques
- **Case Manipulation-Windows** : In windows systems, the cmd and powershell commands are case-insensitive which means you can play with the case of the command and it will execute just fine `WhOaMi || wHoAmI || WHoaMI`
- **Case Manipulation-Linux** : In linux systems the shell is case-sensitive which means you need to provide it in lowercase only, you can make use of the `tr` command and give it the desired command and make `tr` translate the command to lower case like: `$(tr "[A-Z]" "[a-z]"<<<"WhOaMi") # gives whoami` & `$(a="WhOaMi";printf %s "${a,,}") # Works in bash not zsh`
- **Reversing Commands-Linux**: In this technique we will first reverse the command using the `rev` utility and then pass it to the same tool in an inline command like : `echo "whoami" | rev # gives imaohw` then `$(rev<<<"imaohw") # gives username` You can also do it in one run like `$(rev<<<$(echo "whoami" | rev))` and if the pipe is blacklisted you can try `$(rev<<<$(rev<<<"whoami"))`
- **Reversing Commands-Windows**: We can do the same in windows, lets reverse a string `"whoami"[-1..-20] -join '' # gives imaohw` then to execute the reversed command we can use powershell subshell `iex "$()"` as follows `iex "$('imaohw'[-1..-20] -join '')"`
- **Encoding Commands-Linux** : This technique is helpful for the commands that have filtered characters or characters that may be url-decoded by the server which can mess up the command. Lets start with basic base64 encoding `echo "whoami" | base64 # gives d2hvYW1pCg==` now we want to decode then execute the command: `bash<<<$(base64 -d<<<"d2hvYW1pCg==")`
- **Encoding Commands-Windows** : The same thing applies to Windows, lets encode the string `[Convert]::ToBase64String([System.Text.Encoding]::Unicode.GetBytes('whoami')) # gives dwBoAG8AYQBtAGkA` and then we can decode the command and execute it using the subshell `iex "$([System.Text.Encoding]::Unicode.GetString([System.Convert]::FromBase64String('dwBoAG8AYQBtAGkA')))"`


### Labs
[Blind os command injection with time delays](https://portswigger.net/web-security/os-command-injection/lab-blind-time-delays)<br>
[Blind os command injection with output redirection](https://portswigger.net/web-security/os-command-injection/lab-blind-output-redirection)<br>
[Blind os command injection with out-of-band interaction](https://portswigger.net/web-security/os-command-injection/lab-blind-out-of-band)

## Injection Operators
Don't forget that the command you want to execute is initially part of a bigger command, so you want to either use inline command execution or use command separators so that the main command will execute, then yours. These are command separators on both unix and windows  systems:
- &
- && - the AND operator executes both commands at the same time
- \|
- \|\| - the OR operator executes one of the commands (the valid one)

These only work for Linux:
- ;
- \n or 0x0a (for newline, its like hitting the enter button)

As for inline command execution you can use these for Unix systems:
- \`whoami\` notice the back-ticks
- $(whoami)

Some times the command executed will be like:
`./fetch_info.sh "productid"`
notice that the product id is within a quotation, you need to escape it like:
`./fetch_info.sh ""&whoami&""`
in this example you will send ("&whoami&") to the server.

## Questions
these are questions that comes to my mind when I think of this vuln (updated regularly):

Q1 : Where can I find this vulnerability?.
A1 : Any parameter you control can be vulnerable, if you have the source code you can pinpoint the parameters because you will see a system call to execute a command.

## Side notes
- I noticed that the command separators will behave differently. For example, sometimes the & separator will give you output, unlike the back-tick which will not return output.
- Some times the injected input won't work until there is a specific parameter, check skill assessment HTB, try fuzzing for other parameters.
- You might need to URL-encode the command separators.
- Don't test for command injection in the terminal. 99% of the time you will execute commands on your machine and think it's the target server (especially with sleep).
- if the application uses black-listing or escaping the command separators that might be bypassable.
## Resources
[Owasp](https://owasp.org/www-community/attacks/Command_Injection)<br>
[Portswigger](https://portswigger.net/web-security/os-command-injection)