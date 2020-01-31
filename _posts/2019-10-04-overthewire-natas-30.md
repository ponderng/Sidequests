---
title: 'OverTheWire: NATAS 30'
date: 2019-10-04T15:24:53-04:00
categories:
  - OverTheWire
toc: true
toc_sticky: true
---
<img class="alignnone size-large wp-image-381" src="http://dustinwatts.me/wp-content/uploads/2019/10/2019-10-04_15h26_33-1024x283.png" alt="" width="640" height="177" srcset="http://dustinwatts.me/wp-content/uploads/2019/10/2019-10-04_15h26_33-1024x283.png 1024w, http://dustinwatts.me/wp-content/uploads/2019/10/2019-10-04_15h26_33-300x83.png 300w, http://dustinwatts.me/wp-content/uploads/2019/10/2019-10-04_15h26_33-768x213.png 768w, http://dustinwatts.me/wp-content/uploads/2019/10/2019-10-04_15h26_33.png 1095w" sizes="(max-width: 640px) 100vw, 640px" />

This is another basic login form. The unique thing about this one is the sourcecode is in PERL.

<pre class="lang:perl decode:true ">#!/usr/bin/perl
use CGI qw(:standard);
use DBI;

print &lt;&lt;END;
Content-Type: text/html; charset=iso-8859-1

&lt;!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN"&gt;
&lt;head&gt;
&lt;!-- This stuff in the header has nothing to do with the level --&gt;
&lt;link rel="stylesheet" type="text/css" href="http://natas.labs.overthewire.org/css/level.css"&gt;
&lt;link rel="stylesheet" href="http://natas.labs.overthewire.org/css/jquery-ui.css" /&gt;
&lt;link rel="stylesheet" href="http://natas.labs.overthewire.org/css/wechall.css" /&gt;
&lt;script src="http://natas.labs.overthewire.org/js/jquery-1.9.1.js"&gt;&lt;/script&gt;
&lt;script src="http://natas.labs.overthewire.org/js/jquery-ui.js"&gt;&lt;/script&gt;
&lt;script src=http://natas.labs.overthewire.org/js/wechall-data.js&gt;&lt;/script&gt;&lt;script src="http://natas.labs.overthewire.org/js/wechall.js"&gt;&lt;/script&gt;
&lt;script&gt;var wechallinfo = { "level": "natas30", "pass": "&lt;censored&gt;" };&lt;/script&gt;&lt;/head&gt;
&lt;body oncontextmenu="javascript:alert('right clicking has been blocked!');return false;"&gt;

&lt;!-- morla/10111 &lt;3  happy birthday OverTheWire! &lt;3  --&gt;

&lt;h1&gt;natas30&lt;/h1&gt;
&lt;div id="content"&gt;

&lt;form action="index.pl" method="POST"&gt;
Username: &lt;input name="username"&gt;&lt;br&gt;
Password: &lt;input name="password" type="password"&gt;&lt;br&gt;
&lt;input type="submit" value="login" /&gt;
&lt;/form&gt;
END

if ('POST' eq request_method && param('username') && param('password')){
    my $dbh = DBI-&gt;connect( "DBI:mysql:natas30","natas30", "&lt;censored&gt;", {'RaiseError' =&gt; 1});
    my $query="Select * FROM users where username =".$dbh-&gt;quote(param('username')) . " and password =".$dbh-&gt;quote(param('password')); 

    my $sth = $dbh-&gt;prepare($query);
    $sth-&gt;execute();
    my $ver = $sth-&gt;fetch();
    if ($ver){
        print "win!&lt;br&gt;";
        print "here is your result:&lt;br&gt;";
        print @$ver;
    }
    else{
        print "fail :(";
    }
    $sth-&gt;finish();
    $dbh-&gt;disconnect();
}

print &lt;&lt;END;
&lt;div id="viewsource"&gt;&lt;a href="index-source.html"&gt;View sourcecode&lt;/a&gt;&lt;/div&gt;
&lt;/div&gt;
&lt;/body&gt;
&lt;/html&gt;
END
</pre>

From quick examination of the source, it&#8217;s clear that we&#8217;re intended to do SQL injection here. In fact, we don&#8217;t even have to extract the password, it should be given to us if we are able to get ANY successful result from the query. So a very simple SQL injection will do. Problem is, and the central challenge here, is the quote() function that&#8217;s supposed to be sanitizing our inputs. There must be a known way around quote().

A few minutes of searching Google and I found the [answer](https://stackoverflow.com/questions/40273267/is-perl-function-dbh-quote-still-secure) I was looking for. Turns out param() from CGI.pm introduces a vulnerability to quote(). From the post:

> You see, `param` is context-sensitive. In scalar context, if the parameter has a single value (`name=foo`), it returns that value, and if the parameter has multiple values (`name=foo&name=bar`) it returns an arrayref. In _list_ context, it returns a list of values, whether there are zero, one, or many. The argument list of a method (such as `quote`) is a list context. That means that someone using your app can cause `quote` to receive _two_ values, and `quote`&#8216;s optional second argument is an SQL data type that the first argument should be treated as. If the data type is a non-string type like `NUMERIC`, then `quote` will pass its first argument through _without any quoting_. This constitutes an opportunity for SQL injection.

To exploit the vuln, simply send <span class="lang:xhtml highlight:0 decode:true crayon-inline">username=natas31&password=&#8221;or 1=1&password=2</span> in the POST data.