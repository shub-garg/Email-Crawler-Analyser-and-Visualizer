<h1>Analyzing an EMAIL Archive from gmane and vizualizing the data using the D3 JavaScript library:wave:</h1>

<h3> Discription :</h3>

This is a set of tools that allow you to pull down an archive
of a gmane repository using the instructions at:

http://gmane.org/export.php

In order not to overwhelm the gmane.org server, I have used 
a copy of the messages from (Thanks to Dr. Chuck! My course instructor): 

http://mbox.dr-chuck.net/

This server will be faster and take a lot of load off the 
gmane.org server.

<img src="https://images.squarespace-cdn.com/content/v1/5769fc401b631bab1addb2ab/1541580611624-TE64QGKRJG8SWAIUS7NS/ke17ZwdGBToddI8pDm48kPoswlzjSVMM-SxOp7CV59BZw-zPPgdn4jUwVcJE1ZvWQUxwkmyExglNqGp0IvTJZamWLI2zvYWH8K3-s_4yszcp2ryTI0HqTOaaUohrI8PI6FXy8c9PWtBlqAVlUS5izpdcIXDZqDYvprRqZ29Pw0o/coding-freak.gif" width="800" height="500"/>
<h3>Requirements : </h3>

You should install the SQLite browser to view and modify the databases from:

http://sqlitebrowser.org/

<h4> Python Packages : </h4>

<h5>How to install a package :</h5>

    pip install <packagename>
    
<h5> How to upgrade :</h5>

    pip --upgrade <packagename>
   
<h5> Packages required : </h5>
Please install/upgrade the following python packages :

    [urllib, re, sqlite3, ssl, time, zlib, string]

<h3> Steps : </h3>

The first step is to spider the gmane repository.  The base URL 
is hard-coded in the gmane.py and is hard-coded to the Sakai
developer list.  You can spider another repository by changing that
base url.   Make sure to delete the content.sqlite file if you 
switch the base url.  The gmane.py file operates as a spider in 
that it runs slowly and retrieves one mail message per second so 
as to avoid getting throttled by gmane.org.   It stores all of
its data in a database and can be interrupted and re-started 
as often as needed.   It may take many hours to pull all the data
down.  So you may need to restart several times.

To give you a head-start, you may dowload 600MB of pre-spidered Sakai 
email here:

https://www.py4e.com/data_space/content.sqlite.zip

If you download this, you can "catch up with the latest" by
running gmane.py.

Here is a run of gmane.py getting the last five messages of the
sakai developer list:


    Mac: python3 gmane.py 

    Win:python gmane.py 


The program scans content.sqlite from 1 up to the first message number not
already spidered and starts spidering at that message.  It continues spidering
until it has spidered the desired number of messages or it reaches a page
that does not appear to be a properly formatted message.

Sometimes gmane.org is missing a message.  Perhaps administrators can delete messages
or perhaps they get lost - I don't know.   If your spider stops, and it seems it has hit
a missing message, go into the SQLite Manager and add a row with the missing id - leave
all the other fields blank - and then restart gmane.py.   This will unstick the 
spidering process and allow it to continue.  These empty messages will be ignored in the next
phase of the process.

The second process is running the program gmodel.py.  gmodel.py reads the rough/raw 
data from content.sqlite and produces a cleaned-up and well-modeled version of the 
data in the file index.sqlite.  The file index.sqlite will be much smaller (often 10X
smaller) than content.sqlite because it also compresses the header and body text.

Each time gmodel.py runs - it completely wipes out and re-builds index.sqlite, allowing
you to adjust its parameters and edit the mapping tables in content.sqlite to tweak the 
data cleaning process.

Running gmodel.py works as follows :

    Mac: python3 gmodel.py

    Win:python gmodel.py

<h4>The gmodel.py program does a number of data cleaing steps :</h4>

Domain names are truncated to two levels for .com, .org, .edu, and .net 
other domain names are truncated to three levels.  So si.umich.edu becomes
umich.edu and caret.cam.ac.uk becomes cam.ac.uk.   Also mail addresses are
forced to lower case and some of the @gmane.org address like the following

   arwhyte-63aXycvo3TyHXe+LvDLADg@public.gmane.org

are converted to the real address whenever there is a matching real email
address elsewhere in the message corpus.

If you look in the content.sqlite database there are two tables that allow
you to map both domain names and individual email addresses that change over 
the lifetime of the email list.  For example, Steve Githens used the following
email addresses over the life of the Sakai developer list:

s-githens@northwestern.edu
sgithens@cam.ac.uk
swgithen@mtu.edu

We can add two entries to the Mapping table

s-githens@northwestern.edu ->  swgithen@mtu.edu
sgithens@cam.ac.uk -> swgithen@mtu.edu

And so all the mail messages will be collected under one sender even if 
they used several email addresses over the lifetime of the mailing list.


You can re-run the gmodel.py over and over as you look at the data, and add mappings
to make the data cleaner and cleaner.   When you are done, you will have a nicely
indexed version of the email in index.sqlite.   This is the file to use to do data
analysis.   With this file, data analysis will be really quick.

The first, simplest data analysis is to do a "who does the most" and "which 
organzation does the most"?  This is done using gbasic.py:

    Mac: python3 gbasic.py

    Win:python gbasic.py 

How many to dump? 5
Loaded messages= 51330 subjects= 25033 senders= 1584

You can look at the data in index.sqlite and if you find a problem, you 
can update the Mapping table and DNSMapping table in content.sqlite and
re-run gmodel.py.

<h3> Visualisation & Results : </h3>

There is a simple vizualization of the word frequence in the subject lines
in the file gword.py:

    Mac: python3 gword.py

    Win:python gword.py

Range of counts: 33229 129
Output written to gword.js

This produces the file gword.js which you can visualize using the file 
gword.htm.

![](https://github.com/shub-garg/Email-Crawler-Analyser-and-Visualizer/blob/main/Code/Images/gword.PNG)
A second visualization is in gline.py.  It visualizes email participation by 
organizations over time.

    Mac: python3 gline.py 

    Win:python gline.py 

Loaded messages= 51330 subjects= 25033 senders= 1584
Top 10 Oranizations
['gmail.com', 'umich.edu', 'uct.ac.za', 'indiana.edu', 'unicon.net', 'tfd.co.uk', 'berkeley.edu', 'longsight.com', 'stanford.edu', 'ox.ac.uk']
Output written to gline.js

Its output is written to gline.js which is visualized using gline.htm.

![](https://github.com/shub-garg/Email-Crawler-Analyser-and-Visualizer/blob/main/Code/Images/gline.PNG)

As always - comments welcome.
<h2>Thank you!!</h2>
<img src="https://media.tenor.com/images/4a37815ddbf2e92d8f082ca3a0aa02fb/tenor.gif" width="500" height="300"/>
