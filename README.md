The Goal
--

* backup all direct messages (1:1)
* backup all private / team chanels
* normalize the data and present it in a useful format


The python script in this repo _does_ support extracting public data as well, but I am going to skip this.  It's trivial for a slack admin to get the entire public export.  There are already a variety of tools that work well with the public chanel export. [Like this wonderful flask app](https://github.com/hfaran/slack-export-viewer).



Thanks to github user [minniel](https://gist.github.com/minniel) for collecting the two gists that power this. I only made small ~~changes~~ hacks to the python script to handle errors and rate-limit the requests to the slack API.


The Plan
--

There's a few parts to this whole thing.

1. set up your dependencies
2. get an API token
3. extract everything from the slack API
4. combine it into a useful package


The Prep
--

Make sure you have the necessary software installed to do this.


php version:
```
$:php -v
PHP 5.6.30 (cli) (built: Oct 29 2017 20:30:32)
Copyright (c) 1997-2016 The PHP Group
Zend Engine v2.6.0, Copyright (c) 1998-2016 Zend Technologies
```

python version:
```
$:python3 --version
Python 3.6.2
```

osx version:
```
$:sw_vers
ProductName:	Mac OS X
ProductVersion:	10.12.6
BuildVersion:	16G1212
```


I've not done a whole lot of testing and i'm not going to do a whole lot of support.  I'll merge your PR's though.

TL;DR:
[![works badge](https://cdn.rawgit.com/nikku/works-on-my-machine/v0.2.0/badge.svg)](https://github.com/nikku/works-on-my-machine)




The Doing
--


1. get this repo cloned

```
$ cd ~
$ mkdir slack_exports
$ cd slack_exports
$ git clone git@github.com:kquinsland/slack-private-messages-exporter.git
# or https://github.com/kquinsland/slack-private-messages-exporter.git
```


2. set up venv & load dependencies into venv

```
$ python3 -m venv ./venv
(venv) $ pip3 install slacker
```

3. get an API token from here ([Legacy tokens | Slack](https://api.slack.com/custom-integrations/legacy-tokens))
  

4. run the exporter! 
    
    - This will take a long time!  
      - I have ~ 3 years of slack history
      - sack's API lets you get 100 messages at a time
      - you can make an api call about once every second
        - i had the best results when waiting 2 seconds per API call
      - it took a little under 2 hours for me to get my DM, MPDM, and private chanels!
      
    - There is *zero* support for resuming an export!
      - Do this when you know your computer will powered and connected to a network for a long time.
      - Do this from a *reliable* internet connection.  I have hacked in some very basic error handeling. The export won't stop, but you'll probably have to go get the missing data yourself.
    
Your output will look something like this:

```
(venv) $:python3 slack_history.py --token xx-yy-zz --skipChannels
Successfully authenticated for team YourTeam and user karl
found 421 users
writing metadata

found private channels:
some_project: (7 members)
<...SNIP...>
mpdm-person1--person2--person3-1: (3 members)
<...SNIP...>
some_team: (9 members)
other-team: (7 members)
different-team: (8 members)
different-team-private: (9 members)
<...SNIP...>
getting history for private channel some_project with id XXXYYYZZZ1
Sleeping a second to avoid rate limits....
<...SNIP...>
found direct messages (1:1) with the following users:
slackbot
<...SNIP...>
getting history for direct messages with slackbot
Sleeping a second to avoid rate limits....
<...SNIP...>
```


5. turn the JSON into something useful.

copy the json from history into working dir:
```
(venv) ./history$:cp -R * ../working/
```

go to working dir
```
(venv) ./history$:cd ../working/
```

copy users and channel info for DMs and private channels
```
(venv) ./working$:cp users.json channels.json direct_message/
(venv) ./working$:cp users.json channels.json private_channels/
```

copy in the viz.php
```
(venv) ./working$:cp ../viz.php direct_message/
(venv) ./working$:cp ../viz.php private_channels/
```

do the needful
```
(venv) ./working$:cd direct_message/
(venv) ./direct_message$:php viz.php
=====
Combining JSON files for #aConversation
=====
..
2 messages exported to /Users/karl.quinsland/Development/slack_exp/working/direct_message/../slack2html/json/aConversation.json
<...SNIP...>
```


Move to the slack2html for DMs into something else
```
(venv) ./direct_message$:mv ../slack2html/ ../slack2html-DMS/
```

Switch to private channels
```
(venv) ./direct_message$:cd ../private_channels/
```

Repeat....
```
(venv) ./private_channels$:php viz.php
=====
Combining JSON files for #some-team
=====
........................................................
3,213 messages exported to /Users/karl.quinsland/Development/slack_exp/working/private_channels/../slack2html/json/some-team.json
<...SNIP...>
```

Just as before; rename the exported html dir to something meaningful
```
(venv) ./private_channels$:mv ../slack2html/ ../slack2html-private/

```
Open the index.html and you'll be good to go :)
```
(venv) ./private_channels$:open ../slack2html-private/html/index.html
```
