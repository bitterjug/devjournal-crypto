<!-- 
vim: sw=4 ft=ghmarkdown spell 
-->
# Indigo Data

- [ ] Call Tim Davies <tim@practicalparticipation.co.uk> - Practical participation Wednesday 

- Download some data from the big lottery

- Install RethinkDB http://www.rethinkdb.com/docs/install/ubuntu/

- Run a server

  $ rethinkdb create -d ~/Documents/rethinkdb
  $ rethinkdb -d ~/Documents/rethinkdb &

- Go to the [web interface](http://localhost:8080/#dataexplorer) to make sure its working.

- Create a database in the web interface:

    r.dbCreate('360giving');

- And a table:

    r.db('360giving').tableCreate('BigLottery');

- Let's [load some csv](http://rethinkdb.com/blog/1.7-release/):


Crash and burn:

    $ rethinkdb import -f BIGGrantOpenData2004_05.csv --format csv --table 360giving.BigLottery --force

    [===                                     ]   7% 
    0 rows imported in 1 table
    'utf8' codec can't decode byte 0xa3 in position 447: invalid start byte
    'utf8' codec can't decode byte 0xa3 in position 57: invalid start byte
    'utf8' codec can't decode byte 0xa3 in position 300: invalid start byte
    'utf8' codec can't decode byte 0xa3 in position 253: invalid start byte
    'utf8' codec can't decode byte 0xa3 in position 309: invalid start byte
    Errors occurred during import

Not encouraging. `0xa3` [might be the £ symbol][1]
[1]: http://www.psteiner.com/2010/09/how-to-fix-weird-characters-in-xml.html

## 2014-04-16 10:40 Wednesday

I flanged about with python `codecs` library and rewrote the files from
ISO-8859-1 (Latin-1) into UTF-8. But in Linux the command line is your friend.

* `file -i`  tells me the current encodings
* `iconv` converts between encodings;

```bash
   iconv -f ISO-8859-1 -t UTF-8 BIGGrantOpenData_2005_06.csv -o uBIGGrantOpenData_2005_06.csv
```


## 2014-04-21 21:17 Monday

Two likely JS libraries on top of d3.js:
* [nvd3](http://nvd3.org/)
* [dimple](http://dimplejs.org/)

* [ ] Convert  (coerce) the `CURRENT_AWARD` field to a real number

``` javascript
     r.table("BigLottery").update({
        amount: r.row("CURRENT_AWARD").coerceTo("Number")
     })  
```

* [ ] Add an index on the `FUNDER NAME`

``` javascript
r.table("BigLottery").indexCreate("FUNDER_NAME")
```

* [ ] Now you can find out how much each funder has given overall

``` javascript
r.table("BigLottery").group({index: "FUNDER_NAME"})('amount').sum() 
```

## 2014-04-22 17:16 Tuesday

- [ ]Look into why that last update to add the charity and company numers failed.


    r.db('360giving').table('Organization').update(function (org) {
        return r.do(
            r.db("360giving").table("BigLottery").getAll(org('id'), {index: '360G-id'}).nth(0),
            function (lottery) {
                return {
                   charityNumber: lottery('CHARITY_NUMBER'),
                   companyNumber: lottery('COMPANY_NUMBER')
                };
            }
        );Tech retrospective
    }, {
      nonAtomic: true
    });

## 2014-04-23 16:55 Wednesday


- [x] Register bugs with RethinkDB?

  - Can't sort more than 100,000 rows (WTF!?)

  - Inserting dates for each of my 120,000 Big Lottery data rows failed "Have you shut down the server?"

## 2014-05-12 14:54 Monday

- [ ] Need to Fix Mutt: I think my default archive action, bound to 'a' should use quasi delete.

## 2014-05-22 09:41 Thursday


Bug in v4c.

- Editing activity descriptions results in them having the description from one particular activity.

- Replicated DB locally
- Tried just now on a random activity (4.1.1) and didn't cause replication.
- They have the problem with the Results and Measure ment Output, so I try
  tere.  I edited 10.1.2 which formerly  had a different description and it DID
  change to :

"Inception Meeting with Research Companies (Promundo and TNS) held to discuss expectations and processes "

- This might be coming from the original activity with this id, or from one of the other clones, I guess.

- Did it again and the network tab shows the initiator of the event that caused the update.
- Tracking the stack for this, I found `postChangeElement` in `editable-text.js` is called as soon as I click to start the edit.
But not when I click outside. (on Activity 4.7.1)
- from a breakpoint on `input_view.changeElement` I find that the model recorded for Activity 10.1.1 is Activity #5
- At the `changeElement` stage, we still have the correct object, with the correct description. 
- breakpoint at the start of `input_view.commitEdit`.




## 2014-05-22 15:45 Thursday

Clue!

`$('.savable')` after loading the page gives a non-empty set. it contains the
div for the inception meeting text!  input_view has a handler on `blur` on
`.savable` which I does the update. 

## 2014-06-17 12:38 Tuesday

-  [ ] v4c: select Indicator descriptions: without editing them, drag up from
   the end of the text and keep the mouse button down and go outside the
   widget, it unselects the text and selectts the table instead. Weird.


## 2014-06-19 16:41 Thursday


Notes for v4c

The hierarchical approach to dates that you propose has some interesting consequences:

- if you expand quarterly "Results" within 


## 2014-06-25 09:33 Wednesday

The Dfid Yemen component C proposal contains the following statement of Goals:

  The goals of the proposed database system are as follows:

  1. Store qualitative and quantitative datasets from surveys and interviews, and geo-referenced photographs
  2. Searchable by project
  3. Present data on maps
  4. Produce simple reports
  5. Accessible to DFID and project partners

In the context of that programme, this has become a central part of the implied "specification"
of the system. 

I have a problem with this in that it does not in itself provide a sufficiently
guiding vision statement.  I think the problem for me is that it is written in
solution terms and not in business terms. It says what the database will do,
but does not refer to the context in which it will do it, or the business needs
it will be serving. I wonder if we would be in a better place if this were the case. 
I'm thinking of something like:

  * Serve as the canonical storage place for study data for all partners during the programme.
  * Enable DFID and project partners to access study data 
  * Be the platform by which study results are published to DFID and project partners

Hmm. Perhaps not. 

## 2014-06-25 09:45 Wednesday

Based on the initial design overview:

 - upload interface
 - GIS
 - reporting/querying

We explored a number of options while seeking clarification on the principal
workflows that the database will be required to support. Important questions 
guide the design process include:

- What point in the programme's data related work-flow will those data be
  stored in the database? Alternatives might include:

  ** The database will be the canonical repository for sharing data from
  studies during the programme. Raw data will be uploaded as soon as possible
  after they are captured in the field; the database will be the source and
  destination for any subsequent data-based operations such as off-line
  analysis, augmentation (such as coding qualitative datasets), reporting. In
  this case we might imagine the database needs to support modifying or
  updating workflows as well as the initial upload.

  ** The database will be used primarily to publish results to stakeholders. In
  this case we imagine datasets need be uploaded once only but queried many
  times.me

- What are the principal units of data that users will want to deal with.

  ** Complete datasets (e.g. the results of a single study)

  ** Individual survey results (e.g. rows of data each derived from on
  respondent)

  ** Arbitrary subsets and/or unions of rows from different surveys.


We understand that in order to meet security requirements, the data in the DB
may not specify location to more than the level of Governorates. This calls
into question the value of presenting results on a map. We need to 


The initial proposal makes indicative suggestions 

## 2014-06-30 16:39 Monday

DFID Yemen

The spreadsheet problem:

I notice a tendency for our clients to refer to an implicit "simple" and "general"
tool. 

I infer:
- They assume that a general tool is easier to  make than one tailored to their
  specific requirements
- They believe that they can cut cost by "falling back" on the general tool, by
  saving us the trouble of customizing it.
I compare this mindset with Excel sheets:
- Excel is a general tool
- People have to do some work to define custom sheets to  meet specific purposes
- IF they fall short, you still have excel, but you have to do much of the work yourself

I believe
- These people often find it hard to engage with spreadsheets themselves,
  beyond what they see on the screen
- They don't understand the abstract concepts of formulae (beyond, perhaps, sum)

Furthermore
- They transfer this mindset to our projects
- They almost certainly have no prior experience of bespoke software
  development
- They are disappointed whenever it becomes clear that if we stop work, there
  is no general solution to fall back on.
- They expect us to be able to conjure a general solution into existence to act
  as  their security blanket


Questions for us:
- Is there an equivalent general system we could use as the starting point for
  their projects? Google docs and script, for example?

- Could we build "the python shell" into all our work to provide this opener?

- What if a web-app existed where every cell could contain formulae?
- You could egg prefix with a @ and then write an expression that refers to
  other stuff that exists in some namespace
- The name space could for example be the rest of the web at that moment in
  time, via http
- Or certain parts of our domain model accessible behind the scenes via api

## 2014-07-01 14:53 Tuesday

#output 
#outies

### Frustrations with the requirements process.

Talking to Emeka on v4c.  He sent a spreadsheet to guide us on how to make the
export to Excel.  But its just a copy and paste from the plans people have been
making by hand.  My problem is that he is treating it as general guidance on
what to do.  He finds it frustrating that I don't find it sufficient to proceed
with work. I'm trying to extract and verify all the details withing it and need
to spend time asking him about every detail.

Yemen is worse because there's no one to have those detailed conversations with.
Choosing how to proceed with software development is a process of understanding
details: how systems should respond in their edge cases as well as the general
case. Sometimes we can make informed guesses based on a guiding principle, if
we feel confident that
 - We understand that principle
 - That the client understands it and agrees to it
 - That making such choices represents the best value to the client (the client
   wouldn't rather be consulted)

But there must be a source of answers to detailed questions for software
development to proceed.  And someone willing to make choices on behalf of the
client about when the answers to those questions must come from the client and
when/weather its okay for us to make them.

## 2014-07-21 21:58 Monday

Xmonad and freemind: 

  sudo apt-get install suckless-tools
  wmname LG3D
  freemindhttps://github.com/trademapper/trademapper-js



## 2014-10-30 Thursday

### 15:35

Mapping Open Contracting data with TradeMapper

You can use TradeMapper, the open source, browser based mapping tool, to
visualise any kind of international flow data, not just wildlife trade. Here is
a demonstration using Open Contracting data.

[image]

Last week [Tim Davies](http://www.timdavies.org.uk/) was in our Cambridge
office working with us on the [data
standard](http://ocds.open-contracting.org/standard/r/0__3__3/) for [Open
Contracting](http://standard.open-contracting.org/). I showed him
[TradeMapper](https://github.com/trademapper/trademapper-js) and suggested
visualising international procurement patterns using the Open Contracting
data. 

Not all open contract data has well encoded location data, but Tim took some
time out on a delayed train to create a CSV file from some European Union data
that Development Gateway have converted to the draft Open Contracting standard.
He used Open Refine to link to country names to ISO country codes that
TradeMapper uses.  

I made a column mapping, so TradeMapper can open the resulting file and we had
our proof-of-concept visualisation. Try filtering by Buyer or Supplier country.

We used a small arbitrarily chosen file for this test, so it's hard to ascribe
any meaning to the resulting image; but it demonstrates both the versatility of
TradeMapper, and the potential for visualising international Open Contracting
data, so we're hoping to do more work with it in the near future.

## 2014-11-03 Monday

### 14:35

Addendum: mapping DFID IATI data
-----------------------------------

It turns out IATI data includes recipient country codes. 
Knowing this I:
* downloaded all the [DFID IATI](http://iatiregistry.org/publisher/dfid) files as CSV,
* opened them all in [Open Refine](http://openrefine.org/),
* removed any rows without recipient country codes (some of the files are "regional" and don't have them),
* added a '_from_' column, with value 'GB' (DFID being in Great Britain),
* removed all the columns we weren't going to be able to meaningfully filter by, to keep the file size down,
* pulled out the year from the _Activity Start Data_ to give us a year to filter by,
* hacked up a column mapping for TradeMapper.

The results are [on line
here](http://trademapper.aptivate.org/?csvtype=iati&loadcsv=http://trademapper.aptivate.org/sample_data/iati/dfid-iati-with-start.csv)
I've a lot of different types of data mixed up in here (6k lines) so it still
isn't telling a clear story. But it's another demonstration of the potentialk
power of TradeMapper to bring data-flow visualisation to the desktop.

## 2014-11-04 Tuesday

### 10:50

Firefox settings: keep the tabs when full screen:

    about:config
    browser.fullscreen.autohide: false

### 11:10

There is a story to have a demo with Kashana as the title.


### 11:50

SCSS:  is controlled by [Django Assets](http://django-assets.readthedocs.org/en/latest/)
You create an asset file in your app (e.g. main/assets.py) that defines the dependencies
and you can run a management command to watch for changes and rebuild:

    $ ./manage.py assets watch

## 2014-11-05 Wednesday

### 12:00

Currently you deselect a period by clicking it again with a toggle action.
This means if you have a quarter selected, and you click on the year it belongs
to, you don't select the whole of that year, you deselect that year.

Elly wants to add an "all" button next to the quarters to select the whole
year.  I'd like clicking the year to always mean select the year.  This gives
us two ways to select the whole year. But only one way to select all the years
in the project, because there isn't a top level selector for the project as a
whole.  Elly also wants us to add an "all" button at the year level to select
the whole project.

The template creates the links for periods and adds their relevant period
`start` and end `dates` as attributes on the elements.

**Theory:**

Can we somehow create a 'all' link with the project start and end?  Probably
not as the start and end dates are not well know.  Maybe because the code
compares both start and end when deciding if the period matches the selection.
So if we had a link with start and end values that would "never" match any of 
the periods, would clicking on that have the same effect as unselecting all
the periods?:

It might, but that's not what we want: we would want to actually select the
"all" tab instead. Not sure how that would look like as a UX.

It means when you load the page, you would see the "all" tab selected next to
the years.  And when you select a year, you would first see the "all" sub-tab
selected within that.



## 2014-11-06 Thursday

### 09:28

Apparently the alfie demo is here

    http://alfie.demo.aptivate.org/

And tom has already cleared the db and set the result level to 2.  So the
problem is that setting the number of levels in the database isn't the only
thing to do.  It doesn't look right until you also fix the CSS for the levels.

[x] Fix css for results at level 4

[x] Fix css for no year selected

[ ] Make sure we have some periods and years appearing 

### 11:53

This last one is proving tricky. 

I thought we just set the start and end date of some activity and the system
detected them and filled in as many periods as we needed.  Lets check the
period settings on v4c and Alfie.

### 11:58

Turns out the periods were set OK but the system uses the milestones to work
out the extent of the project. So creating some milestones made dates appear.

### 12:01

Next thing is the project name.
In the Alfie code, this was already  made into a variable, so configurable.
Lets go look at that.

### 12:40

I want to test my django but the tests fail. 

Error about django-appconf database table not existing.
Fixed by

    $ ./manage.py test --create-dbb

To re-create the test database. Otherwise it seems to have been reusing an
existing one which hadn't been updated since the schema was changed and, in
particular, since appconf was added to the app.



## 2014-11-07 Friday

### 16:28

We're busy getting ready for the launch of the Open Contracting Data Standard
at next week's Open Government Partnership meeting in Costa Rica.  [The Open
Contracting Data Standard](http://standard.open-contracting.org/) enables
governments to publish accessible and structured releases of data and documents
at each stage of a contracting process, in accordance with the [Open
Contracting Principles](http://www.open-contracting.org/global_principles).  We
can do creative things with this open data, like [visualising it on a
map](http://aptivate.org/en/blog/2014/10/30/mapping-open-contracting-data-with-trademapper/).

Sarah will be there in person.  Over the past year she has been working with a
team from the Open Contracting Partnership, including Tim Davies of [The Web
Foundation](http://webfoundation.org/), on the technical aspects of the data
standard. The team considered a number of supply and demand use cases and
created a standard data model, and JSON Schema, to meet their needs. 

The standard itself is an evolving document that lives in its own GitHub
repository to manage its evolution. We just built a shiny new website to
showcase the standard with its documentation and examples. The site will go
live when the standard is launched next week, and replace the [existing
site](http://ocds.open-contracting.org/standard/).  Get a preview by
viewing the bleeding-edge of work-in-progress on the standard's [master
branch](http://ocds.open-contracting.org/standard/r/master/).


## 2014-11-11 Tuesday

### 12:38

We hacked TradeMapper better.

Thanks to all Fifteen people who came the hack-day Saturday and helped improve
TradeMapper, the browser-based tool for mapping trade flows.

## 2014-12-01 Monday

### 12:27

Indigo Data bug:

EG On this page: http://lin-360giving.aptivate.org/grantees/360G-phf:28197
There is a link to www.ggbkorg.in which has the internal address:

    http://lin-360giving.aptivate.org/grantees/www.ggbkorg.in

I tried to access this Organization via the admin, but there's no search and
the keys in the url `360G-phf:28197` aren't the database keys (they have
internal numeric keys).

Let's install via the [instructions](https://projects.aptivate.org/projects/indigodata/wiki):

    sudo npm install -g elasticsearch

I've got 360giving-demos checked out, pulling recent changes takes AGES!


### 13:08

Got the db image from the project installed locally but can't find any
organizations with web site urls like the ones in the bugs. I'm wondering if the data
on the server is more up to date than the image checked in maybe.

Test by ssh to the live server and try sql remotely.

### 14:37

Well that didn't fix it, sill no matches. So I'm guessing those grantee details
are not really coming from the Organization table. So where? Have alook at the view
code associated with the template.

### 14:40

Hmm.

    class GranteeDetailView(DetailView):
        model = Activity
        template_name = 'grantees/detail.html'

## 2015-01-04 Sunday

### 21:12

Upon installing docker, you need to add yourself to the docker group and restart the server
as per http://docs.docker.com/installation/ubuntulinux/

The command to add yourself to a group is:

    $ sudo gpasswd -a ${USER} docker

This doesn't actually appear to work out of the box. I might  need to log out
and back in or something. Meanwhile work round by

    newgrp docker

Which appears to start a new shell (history lost) with the new group active.

### 21:15

Oddly, however, `sudo service docker restart` doesn't work:

    docker: unrecognised command

The service is called `docker.io`:

    sudo service docker.io restart


### 21:35

The [mysql Docker image](https://registry.hub.docker.com/_/mysql/) 
says its supported in docker 1.4.1 but the default Ubuntu maintained
one is 1.0.1. Therefore as per [instructions](http://docs.docker.com/installation/ubuntulinux/):

    Sudo aptitude install apt-transport-https

    sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
    sudo sh -c "echo deb https://get.docker.com/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
    sudo aptitude remove docker.io
    sudo aptivate install lxc-docker

### 21:45

Random search for xmonad and adobe air turned up 
[the follow possible fix](http://vollerthun.eu/2011/06/13/balsamiq-for-wireframes/index.html) for the flicker problem:

The only real bug I’ve encountered in this regard was a strange
focus-flickering issue with xmonad: every tenth second or so, the focus would
switch between two windows – yes, the keyboard focus, too. I could fix it,
though:

    ?import XMonad.Actions.UpdatePointer
    main = xmonad {
    logHook = updatePointer (Relative 0.5 0.5)
    }


### 23:09

When you run `django-admin.py startproject ...` in a docker container,
you get the files created but owned by root. I changed them with 

    chown -R marks <project>

(leaving them in root group)


### 23:11

Switching the default django app from postgres to mysql:
- I found the python bindings for `MySQL-python` need to be installed.
- And then this appears to need the `mysql` binary too, so I do:

    apt-get update && apt-get install -y mysql-client

  in the `Dockerfile` to install it.

After each change to `Dockerfile`, re-run

    fig build

To get the changes applied.

### 23:32

When I try to convert the default app to work with mysql 
I get messages about  no host called 'db'. I revert to the
postgress example  and get a different error about missing
config but if I 

    fig run web cat /etc/hosts


I can see the entry for the db server.
So something is awry with the mysql setup.


### 23:37

Stuff to fix in the setup:

- [x] powerline in vim

  powerline config files changed

- [ ] the ctrlp support still seems broken so commented out of 
  `~/.dotfiles/powerline/config.json` for the moment.

- [x] tmux config copy and paste commands (what do we have on fuji?)

  xclip not installed!



### 23:39


0 packages upgraded, 1 newly installed, 0 to remove and 6 not upgraded.
Need to get 17.0 kB of archives. After unpacking 72.7 kB will be used.
Get: 1 http://gb.archive.ubuntu.com/ubuntu/ trusty/universe xclip amd64 0.12+svn84-4 [17.0 kB]

## 2015-01-05 Monday

### 22:21

- [ ] Add vimperator zenburn theme (can we do this from fresh?)

    https://github.com/vimpr/vimperator-colors

### 22:39

After a bit of hitting with a stick, I get the mysql docker image to work with fig.
I wonder if we were experiencing the non-deterministic failure behaviour where
it tries to create a database first time it runs and so isn't ready when the
client needs it.

I tested it by running 

    fig run web bash

And then running 

    mysql -hdb -uuser -ppass

where `db` is the name of the database service in `fig.yml`

## 2015-01-06 Tuesday

### 12:49



## 2015-01-07 Wednesday

### 11:41

Apparently I can mount Box storage in linux as web dav:
- http://xmodulo.com/how-to-mount-box-com-cloud-storage-on-linux.html
- http://seb.so/50gb-of-cloud-space-with-box-automatically-syncd-on-linux-with-webdav/#box_mountwebdav

## 2015-01-09 Friday

### 11:16

Apparently we can integrate an editable spreadsheet view into iPython:
http://nbviewer.ipython.org/gist/rossant/9463955

### 17:35

For next week:

[ ] make tech adoption data for multiple years

## 2015-01-12 Monday

### 10:56

Link for details of LSMS questionnaires.

    http://econ.worldbank.org/WBSITE/EXTERNAL/EXTDEC/EXTRESEARCH/EXTLSMS/0,,contentMDK:22949557~menuPK:4196952~pagePK:64168445~piPK:64168309~theSitePK:3358997~isCURL:Y~isCURL:Y,00.html

### 11:05

In this overview from the link above, there is a section on weighting the data.

    http://siteresources.worldbank.org/INTLSMS/Resources/3358986-1233781970982/5800988-1282216357396/Nigeria2010_11_GHS_Panel_BINFO_June_2014.pdf

> #### 6.14 Weighting of Data
> 
> When a sample of households is selected for a survey, these households
> represent the entire population of the country. To accurately use the datasets,
> the data must be weighted to reflect the distribution of the full population in
> the country. A population weight was calculated for the panel households. This
> weight variable (wght) has been included in the household dataset: Section A
> (secta_plantingw1 for post-planting and secta_harvestw1 for post-harvest). When
> applied, this weight will raise the sample households and individuals to
> national values adjusting for population concentrations in various areas


The data sets do have what looks like weighting fields:

    wt_wave1	wt_combined

Bit it's not obvious how to use these at the moment.

### 12:11

Don't have proper mysql setup with passwsord in /root

### 15:52

Setting up the desktop I had `mysqld` installed but our `deploy/bootstrap.py` script was failing.
http://stackoverflow.com/questions/7475223/mysql-config-not-found-when-installing-mysqldb-python-interface
is the fix. tl;dr: install `libmysqlclient-dev`


## 2015-01-14 Wednesday

### 12:05


I found that the generated CSV file has a couple of issues, and the ipython
notebook wouldn't load the file at the start at all, so I didn't manage to fix
them.  The issues are:

- [x] lack of header in first column (easy to fix by hand editing afterwards)

- [x] the states for 2010 are capitalised but the states for 2012 are just
  lower case.  


- [x] We can drop the pesticide quantity and pesticide unit columns.  

- [x] we ideally want one row per household.  So the yes/no columns would be
  combined by the rule "if any plot did use X, then the household counts as
  using X"

- [x] We'd also like the crop type to be recorded, but we're not sure about how
  to store it in columns.  Basically we want to see how many cassava farmers
  used pesticide (and so on) we think there is a mapping from hhid to the crops
  they grew

  Done but assuming no household farms both cassava and rice.

### 20:43

Here's a neat recipe to focus a window or start it if it's not already running:

    wmctrl -a FreeMind || freemind &

## 2015-01-15 Thursday

### 09:27


CUA (right handed) copy/paste commands:

    Cut 	        Copy 	        Paste
    shift+Delete 	control+Insert 	shift+Insert

### 18:04

[bootstrap modals](http://getbootstrap.com/javascript/#modals)

### 21:29

- [ ] Need to set up new sheet to verify no households farm both cassava and rice.

## 2015-01-16 Friday

### 11:36

Python c-tags not working. Is exuberant-ctags enough to make my vim sidebar work again?

    sudo aptitude instal exuberant-ctags 

fixed it ok.



### 11:44

# ATA:

The extant tests for  ATA tool mainly test filtering options. So to add the new
chart:

- [x] add the code for it in the homepage template and see what it takes to
  populate it with the right settings.

### 11:58

# Vim:

- [ ] No syntax error correction for my python. Looks like I'm lacking a linter
  for syntastic?

### 12:01

# ATA:

- [ ] The value-chain filters for nutrition were different (more sophisticated)
  than for technology adoption so they need to be cahnged.

### 12:15


- [ ] The state filters also don't work, but only because the data are
  different. States in the nutrition dataset have all-lower  case names and in
  the technology one I chose initial capitalised. Because those state names
  appear in the filter picker, I think its best to rebuild the nutrition data
  with capitalised values? Or maybe not, as the python `string.capitalize` only
  capitalizes the first letter. Whereas the 2012 data state names have irregular
  capitalization like, e.g.: "FCT Abuja", which I can't systematically get from 
  "fct abuja", so it would need a lookup. 

  So maybe for a first pass, I would re-generate technology with all lower case
  state names, because thats easier to do programmatically, and add mapping
  state names to upper case as a card for the future.

In the meantime I might remove the filtering from the code and check in a
version without filters working as an interim progress.

### 14:40

- [ ] Ungrey the technology chart heading.


### 15:10

Here's the [Python
docs](https://docs.python.org/release/3.1.5/library/stdtypes.html#old-string-formatting-operations)
on using `format` over `%` in new code.

### 15:31

The keyboard shortcut to open/close the Firefox inspector is:

    <Ctrl><Shift>i

## 2015-01-19 Monday

### 11:37

The crappy navbar footer we got for our site credit seems badly formed because
there isn't a predefined static-bottom style for navbars (becasue you can't see
them) so conceptually it should  be a footer. But I don't know how to do one of
those. And its not priority. So thinking of leaving it out for the moment and
kludging it later.

### 12:59

- [x] The header navbar changes height on small screens so the page offset
  needs to be set dynamically to cope.

  That smallest size I think we're not targeting?

### 14:42

- [ ] remove chart header text and make into title above text
- [ ] Padding between divider row and content
- [x] add explore link buttons

## 2015-01-21 Wednesday

### 16:49

- [x] `httpie` to get command line http command -- from Hamish

### 17:02

- [ ] Fid out how to commit hunks with fugitive

### 17:15

Filtering second production:
- Kogi/Benue: hide the chart altogether
- Cassava/Rice: switch to a chart across years, e.g.:

        <a
        target="_blank"
        href="//ata.livestories.com/chart/embed"
        data-show-dataset="false"
        data-colors="#0066b3,#00aaad,#00a65d,#72bf44,#fff200,#faa61a,#f58220,#ef413d,#ed1c24,#a3238e,#5c2d91,#214009"
        data-filters="filters.Crop=Rice"
        data-chart-type="column"
        data-secondary-operation="sum"
        data-operation="sum"
        data-indicators="Production,Yield Per Hectare"
        data-variables="Year"
        data-dataset="8ba9c30ca16e11e4927006909bee25eb"
        data-enable-filtering="true"
        data-height="600"
        data-width="600"
        class="insight-tile">Total Production and Avg of Yield Per Hectare across Crop</a>
        <script async src="//ata.livestories.com/static/js/embed-api.js"></script>

## 2015-01-22 Thursday

### 11:09

- [ ] Talk to Hamish about the Chart Embed object vs the dictionary of its
  parameters as the right abstraction, and why its easier to test a dict than
  the object.

### 12:09

- [ ]  Write some documentation for django_harness html_parsing mixin


### 15:40

Fix the offset for anchor links 

http://stackoverflow.com/questions/10732690/offsetting-an-html-anchor-to-adjust-for-fixed-header

## 2015-01-23 Friday

### 11:54

I'm looking at the char wdths. I  can adjust the grid widths for the production
charts to make the meta 3 and the charts 7, but the charts don't grow to that
width on large screen sizes because of the 600 width for the iframe. 
option 1) change the iframe width for all charts to something larger, e.g. 800??
then all charts get wider when the screen size is wide enough to allow the
6-col region to be larger than 600. And for the wide charts, 7-col.
Otherwise, if the normal charts are never supposed to be wider than 600, 
I have to set the chart iframe size differently in normal and wide charts.



### 14:53


ppa:libreoffice/libreoffice-4-3 

### 15:49

Bootstrap, making things stick to the top of the browser screen.

    http://getbootstrap.com/javascript/#affix

## 2015-01-27 Tuesday

### 19:37

Syntastic isn't giving me error warnings for python.

### 20:07

But pymode is. But pymode is super slow each time I write a file (init checking) it does
regenerate auto-import chache which takes tens of seconds.

### 20:56

Delete `~/.ropeproject` because it forces everything to use it.

### 20:59

To make ultisnips work again, set its working dir somewhere in a sparkleshare folder.
Maybe use a common one for both setups.


## 2015-01-30 Friday

### 11:06

Elastic search (required for authoraid) should really be installed in a docker container.

### 15:35

Redmine simple [support desk plugin](https://github.com/jfqd/redmine_helpdesk)

### 15:48

- [ ] MAke VimFiler ignore .pyc files (and others?)

    g:vimfiler_ignore_pattern			 *g:vimfiler_ignore_pattern*
		Specify the regexp pattern to ignore candidates of the source.
		This applies on the filenames of candidates.  It's not case
		sensitive.
		Note: If you use
		|<Plug>(vimfiler_toggle_visible_ignore_files)|, the ignore
		pattern is disabled.
>
		let g:vimfiler_ignore_pattern = '^\%(\.git\|\.DS_Store\)$'
<
		Default value is '^\.' (dot files pattern).

## 2015-02-03 Tuesday

### 12:20

A strategy for email foldes:

- Use procmail (now under git control) to organize into folders
- Tag the folders with their notmuch equivalent but try using the folder structure in mutt so we can move them and use imap to sunchronise
- Use notmuch mainly for search. 
- Therefore maybe add some other useful tags for notmuch.

- What to do about archiving old mail?
- Maybe use notmuch virtual folders with a date range on them ?
- But these have to be created manually each time in mutt, which sucks.

## 2015-02-04 Wednesday

### 11:38

Hamish using recent pandas 0.15.2 couldn't open any ATA data: Categorical Categories must be unique .. investigate.

### 13:22

For percentage income graph:

11,766 rows (household/plot/crop)

Of whom 



### 22:11

Here's a nice [Javascript spreadsheet-table library](http://handsontable.com/index.html)

## 2015-02-05 Thursday

### 15:23


## 2015-02-10 Tuesday

### 08:54

# Aptivate Carbon

We are concerned about our impact on the environment.
Climate change is a major concern in international development.
We want to consciously deal with the challenges of working 
in international development, without our work making things worse.
Similarly, we want to make conscious, informed choices about other 
aspects of our environmental impact; things like:

* Greenhouse Gas Emissions
* Procurement (Fair Trade, Ethical suppliers)
* Food (Organic, Vegetarian, Vegan)

We want to make these issues part of the daily conversation at Aptivate, not
just a kind of accounting exercise, so we can engage our combined creative
abilities to help find ways to improve.
For example, we would like it to become normal for us to list and discuss the
ethics and impact of proposed policy changes, experiments or other
interventions, along with their financial and social implications.

We will create an environmental policy for Aptivate to document our
commitments and to use in communications including PD.

We propose to focus on greenhouse gas emissions first, for the first year.



To do this we think we need the following:

- Draft and review an environmental policy document
- Make a map of Aptivate's "organisational ecology" and determine our system
  boundary
- Put in place a tool (e.g. spreadsheet based) that models our activities and
  greenhouse gas emissions.
- Update and publish environmental policy document



### 10:14

link for tim
https://support.office.microsoft.com/en-gb/article/Consolidate-data-in-multiple-worksheets-007ce8f4-2fae-4fea-9ee5-a0b2c9e36d9b?CorrelationId=0b8afae2-e896-4141-87f8-8e815dbc2082&ui=en-US&rs=en-GB&ad=GB
https://www.youtube.com/watch?v=MsbdMiruX68

## 2015-02-12 Thursday

### 12:09

Fix mysql root password http://ubuntu.flowconsult.at/en/mysql-set-change-reset-root-password/

### 17:35

- [ ] Write the text about productivity charts Friday, 

- [ ] Then do the non functional requirements

## 2015-02-13 Friday

### 09:13

Cool! USe `Ctrl-\` (control-backtic) to switch between tab groups in Firefox!

### 11:28

#Mutt

The `A` macro to add users to abook wasn't working because I bound | to hid the
sidebar.  So I unbound | and now I can add senders again, but can't re-bind
that bloody macro to, e.g. ¬ for some reason.


## 2015-02-18 Wednesday

### 11:23

# ATA

Add a pair of bars in the existing tech adoption chart on if any seed has been
used.  use post agric survey 11e question 3 - if they said yes or no. the
question may seem strange (have you used any seed?) of course everybody used
seed! but this is actually around if you acquired any seed based on the follow
up questions. So this should be the right one.  [Mapping on this spreadsheet](https://docs.google.com/a/livestories.com/spreadsheets/d/1-3b6zCyPBWLfhLqBiT-6rU6mTVkir5UGw1vzFl9kLg8/edit#gid=748484414)

This will require update to report in all relevant sections.

## 2015-02-19 Thursday

### 15:56

- [ ] My python setup in vim isn't describing the errors it detects.  I dont
  even know if its syntastic or pymode.

### 16:59

- [ ] Turn off line numbers in the ViMfiler pane!

## 2015-02-20 Friday

### 10:54

# ATA

chart-title-container
    height: 150px

    p
        margin-top: 15px (half the text height)

- [x] So make the expolore button 15px up from the bottom of the chart too.

- [x] Position of the explore button needs top be more than 15 px from bottom when in single column - e./g. margin bottom or similar

- [x] The traffic light text is aligned wrongly when the chart floats left.
    - seems to have inherited text alight center!
