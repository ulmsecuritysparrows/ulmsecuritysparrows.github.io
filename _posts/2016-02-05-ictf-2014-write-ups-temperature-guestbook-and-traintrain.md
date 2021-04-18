---
layout: post
title: "iCTF 2014 Write-Ups: Temperature, Guestbook and Traintrain"
date: 2016-02-05
tags: ["CTF","ICTF"]
---

Even though USS's team was small in size during iCTF 2014 (2015) it was pretty good at writing exploits. The Ulm Security Sparrows scored 4.771 points and achieved the 35th place of 87 teams (66 of them scored) at the iCTF2014-2015 . In the following we would like to contribute to the event and do our part to education by sharing some of our [exploits](http://uss.informatik.uni-ulm.de/wp-content/uploads/2016/02/uss_writeups_ictf2014_code.tar) in the following writeups, namely the services "Temperature", "Guestbook" and "traintrain".

## Temperature

Temperature provides a service to query and store temperature values for a supplied date and location. The service was executed directly from its python source code. It uses a plain text file (neverguess) to store its data. For adding new entries the service uses the command

    echo "'%s %s %s"' >> neverguess

where the placeholders are replaced by the user provided input which is assumed to be a date, location, and temperature value. Retrieving a value is done by executing

    cat neverguess ' grep %s ' grep %s ' awk '{ print $3 }'

 again replacing the placeholders by the user provided input which is assumed to be a date and a location. The flag_id that is given to the exploit is a date and the sought after flag is the corresponding temperature value. One way to exploit this service is that one of the input data provided must be an expression that causes the corresponding command
_grep %s_ to become a no op .

To patch this service the intended purpose of the retrieval command was reimplemented in python such that the corresponding temperature value is returned only if the user supplied date and location matches the date and location stored in the file.

## Guestbook

The Guestbook web app (like the name suggests) enables the visitors to publish their messages. The first step was identifying the service. At the first glance one could find the corresponding Apache config residing inside _/etc/apache2/sites-enabled/guestbook_. Beside the port it strikes one's eye the option "FollowSymlinks" is explicitly turned on. Little bit later more on that.
Like all other services guestbook resides in _{/usr,/var}/ctf/guestbook_. The first of them contains the cgi-bin folders with some Perl scripts along with guestbook.txt and a symbolic link to _/var/ctf/data_ with it's own _guest-book.txt_ file. The later one contains not only the public messages but also the ones marked private (which are posted through a check mark in the web interface). Since the tokens were posted as private messages we just had to call the URL "http://$HOST:$PORT/guestbook/cgi-bin/data/guestbook.txt" and parse the response to get tokens. As simple as the exploitation was the fix:

    sed 's/+ FollowSymlinks/\−FollowSymlinks/g' \
        /etc/apache2/sites-enabled/guestbook

Traintrain

Traintrain was another web app we managed to exploit. Although there are several good write ups out there we implemented our own solution and wanted to publish it. The reasons for the new implementation were a) it was fun and b) we noticed pretty late after digging through the service's source that the service really didn't change since last year (despite somebody on IRC/Mailinglist claimed otherwise).
Inside the web app you had to register an account and the log in with the new credentials. Afterwards you were redirected to the base URL. All saved data is stored inside an SQLite db file inside the service's folder (_/var/ctf/traintrain/traintrain.db_). Inside the the database file one can noticed there's also a "history" field aside to the user credentials. Every time you surfed on the traintrain website the URLs got saved there. The next step was to decompile the python bytecode of the service and dig through the source. SQL-code inside the history row of some users (already exploited by other teams) gave us the idea what to do. We didn't have to look long to find that the SQLi was triggered in the "solutions" area of the site executing the following python code in the line 346 (depending on what decompiler you use) inside the function_ solution(self, form)_:

    query = "select username, score from users where history"\
            "LIKE '%%%s%%' and not session='%s'" % (history, session)

As for the SQL injection itself, we were able to utilize the code that was injected in our own service by other teams, in order to build our own SQLi. Since the flags were located in the users table and the username was used as flag ID, we simply added a complete printout of the users table to the output of the solutions, utilizing the UNION keyword.
To store the SQLi payload we had to URL-encode it:

<pre class="lang:tsql decode:true"># before
%' or 1=1 union select username, authorization from users where 1=1 \ 
     or ':/solution%'=':/solution

# after
%2F%25%27%20or%201%3D1%20union%20select%20username%2C%20authorization\
     %20from%20users%20where%201%3D1%20or%20%27%3A%2Fsolution%25%27\
     %3D%27%3A%2Fsolution</pre>

This is hence in line 114 of the def history(self, session, path): function the visited path was URL-decoded before stored in the db:

    history = prev_history + urllib2.unquote(path)

Knowing all this we identified and replayed all the important requests to trigger the vulnerability with BurpSuite Proxy and then began to bake the Python exploit. Following brief description explains the steps have to be executed to get to the tokens:

*   Register with a self defined username, password and authorization-string
*   Login using username/password and parse out the session-id from response to attach it to all further requests
*   Fire up a request with the encoded SQLi inside the URL, which is stored in the history row afterwards
*   A request to the assignment page is needed since the value of the field "assignment" has to be extracted for the final request (why ever...)
*   Launch a POST-request to the solution-site with the "assignment" and a random "solution"-value attached in the body
*   SQLi is triggered. Now you just need to extract the values of the HTML-Table and save/return the desired one
The [exploits](http://uss.informatik.uni-ulm.de/wp-content/uploads/2016/02/uss_writeups_ictf2014_code.tar) and writeups were a joint work of matou, Viktor and winnie.
