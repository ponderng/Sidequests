---
title: 'OverTheWire: NATAS 29'
date: 2019-09-27T08:04:40-04:00
categories:
  - OverTheWire
toc: true
toc_sticky: true
---
<img class="alignnone wp-image-336 size-full" src="/assets/uploads/2019/09/2019-09-26_14h35_42.png" alt="" width="1098" height="293" srcset="/assets/uploads/2019/09/2019-09-26_14h35_42.png 1098w, /assets/uploads/2019/09/2019-09-26_14h35_42-300x80.png 300w, /assets/uploads/2019/09/2019-09-26_14h35_42-768x205.png 768w, /assets/uploads/2019/09/2019-09-26_14h35_42-1024x273.png 1024w" sizes="(max-width: 1098px) 100vw, 1098px" />

This level presents you with several articles from an old ezine about perl hacking.

<img class="alignnone wp-image-337 size-full" src="/assets/uploads/2019/09/2019-09-26_16h10_58.png" alt="" width="1079" height="685" srcset="/assets/uploads/2019/09/2019-09-26_16h10_58.png 1079w, /assets/uploads/2019/09/2019-09-26_16h10_58-300x190.png 300w, /assets/uploads/2019/09/2019-09-26_16h10_58-768x488.png 768w, /assets/uploads/2019/09/2019-09-26_16h10_58-1024x650.png 1024w" sizes="(max-width: 1079px) 100vw, 1079px" /> 

The page script &#8220;/index.pl&#8221; is what&#8217;s processing our selection. Here it is in Burp Suite:  
<img class="alignnone wp-image-338 size-full" src="/assets/uploads/2019/09/2019-09-26_16h12_38.png" alt="" width="1225" height="770" srcset="/assets/uploads/2019/09/2019-09-26_16h12_38.png 1225w, /assets/uploads/2019/09/2019-09-26_16h12_38-300x189.png 300w, /assets/uploads/2019/09/2019-09-26_16h12_38-768x483.png 768w, /assets/uploads/2019/09/2019-09-26_16h12_38-1024x644.png 1024w" sizes="(max-width: 1225px) 100vw, 1225px" /> 

### Objective 1 &#8211; sourcecode

I read some of the ezine issues and tried a few things to the input for &#8220;index.pl&#8221;. Eventually figured out that the trick is [perl piping](https://docstore.mik.ua/orelly/perl3/prog/ch16_03.htm). The documentation says

> &#8220;Perl&#8217;s <tt class="literal">open</tt> function opens a pipe instead of a file when you append or prepend a pipe symbol to the second argument to <tt class="literal">open</tt>. This turns the rest of the arguments into a command, which will be interpreted as a process (or set of processes) that you want to pipe a stream of data either into or out of. &#8220;

Given the insight that [piping can lead to command injection](https://nets.ec/Command_Injection#Perl), all we should have to do is add &#8220;|&#8221; to the beginning of the &#8220;file&#8221; variable and then we have total control! My first goal is to get the source for &#8220;index.pl&#8221; since the page doesn&#8217;t give us the link for it. <span class="lang:xhtml highlight:0 decode:true crayon-inline ">?file=|cat+index.pl</span> should work. But that alone doesn&#8217;t get anything.

The clue here is the &#8220;file&#8221; variable itself. The script is looking up a file, but there&#8217;s no extension on it, which could mean the extension is being forced (honestly, it took me forever to guess this by searching Google for anything related to perl injection). [Null byte injection](http://projects.webappsec.org/w/page/13246949/Null%20Byte%20Injection) is how to solve that issue.

Trying <span class="lang:xhtml highlight:0 decode:true crayon-inline ">?file=|cat+index.pl%00</span> works!

<img class="alignnone wp-image-339 size-full" src="/assets/uploads/2019/09/2019-09-26_16h37_05.png" alt="" width="1229" height="773" srcset="/assets/uploads/2019/09/2019-09-26_16h37_05.png 1229w, /assets/uploads/2019/09/2019-09-26_16h37_05-300x189.png 300w, /assets/uploads/2019/09/2019-09-26_16h37_05-768x483.png 768w, /assets/uploads/2019/09/2019-09-26_16h37_05-1024x644.png 1024w" sizes="(max-width: 1229px) 100vw, 1229px" /> 

### Objective 2 &#8211; password

Of course the first thing to try would be <span class="lang:xhtml highlight:0 decode:true crayon-inline ">?file=|cat+/etc/natas_webpass/natas30%00</span> , but that doesn&#8217;t work! Surprise! We only get a &#8220;meeeeeep!&#8221; close to the end of the page.

Looking in the source shows the culprit:

<pre class="lang:perl decode:true ">if($f=~/natas/){
    print "meeeeeep!&lt;br&gt;";
}</pre>

It&#8217;s a simple word filter, and there are a few tricks to getting around those.

  1. [Single Character Wildcard](https://medium.com/secjuice/waf-evasion-techniques-718026d693d8) &#8211; The question mark is as wildcard for a single character in the Bash shell.
  2. [Concatenated Strings](https://medium.com/secjuice/web-application-firewall-waf-evasion-techniques-2-125995f3e7b0) &#8211; In Bash, there is something called string literal concatenation, meaning that adjacent string literals are concatenated, without any operator. An example would be <code class="lt mr ms mt mu b">"Hello, ""World"</code> which has the value <code class="lt mr ms mt mu b">"Hello, World"</code>.

Some examples of strings to use for the &#8220;file&#8221; variable:

  1. <span class="lang:python highlight:0 decode:true crayon-inline">|cat+/etc/na?as_webpass/na?as30%00</span>
  2. <span class="lang:python highlight:0 decode:true crayon-inline">|cat+&#8217;/etc/na&#8221;tas_webpass/nat&#8221;as&#8217;30%00</span>

Using either of these will show the password at the bottom of the response HTML.