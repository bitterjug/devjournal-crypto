
Writing note.sh
=================

[strftime](http://www.cplusplus.com/reference/ctime/strftime/)
is a C++ function exposed in Vim.

## 2013-12-23 21:16 Monday

Make the time-stamps sub-headings and format them so they are sortable and then,
in theory at least, I can:

 *  Insert main headings above them to indicate what I'm working on.
 *  Sort by main heading and then sub-heading (if I'm consistent with the main
    headings) to get all the note son one heading.


## 2013-12-23 23:40 Monday

 *  Bindings for xmonad to invoke
 *  Spent way too long working out how to open and close Voom window.
    finally put it in a vim binding.



## 2013-12-29 22:56 Sunday

Install 13.10, tried:

  * Geary email. Looks cool, Set up for gmail, not as sophisticated
    search (yet) as notmuch
  * Haskell Platofrm finally installs
  * We have tmux 1.8 with support for proper copy and paste
  * Played with conky which is weird because it doesn't seem to have
    a widow border
  * Finally switched back 
  * And, strangely, I seem to have filled half my SSD already, with
    what?
    

## 2013-12-30 13:09 Monday

Apparently the upgrade has stopped my `cambridgeenergy.localhost`
working. Visiting that address takes me to the default page for the
server. So the `etc/hosts` entry appears to be still there.  Also,
`/etc/apache2/sites-enabled` and `/etc/apache2/sites-available` both
contain the `cambridgeneergy` configuration file, with the former
being a sym-link to the latter.

## 2014-01-01 12:50 Wednesday

Fixed broken apache2, eventually, by:

1. Change the virtual host config file name to end with .conf
2. Added the following inside the `<VirtualHost>` block to turn
   on access.

    <Directory "">
    Options  All
    AllowOverride All
    Require all granted
    </Directory>


OSI POINT Iteration 30
=======================

## 2014-01-06 16:14 Monday

  * Prepared sample spreadsheet and sent to client
  * Searching for the code, apprently we use Piston to generte the spreadsheet
    (CSV) which kinda makes sense as its a rendering of data.


## 2014-01-06 16:56 Monday

Looks like the way to proceed is to 

  - modify the list of field names from get_fieldnames in csv_handler.py
    and give names to all the new fields (by apending the tree level to each)

  - Then change the serializer to map the budget fields to their appropriately
    postfixed col names.

## 2014-01-07 10:44 Tuesday

  * Thinking about how to turn a list of column headers from 

        [    ...
            'budget',
            ...
        ]

    into:

        [    ...
            'budget_pfa',
            'budget_so',
            'budget_ma',
            'budget_sa',
            ...
        ]
        
    Found `itertools.chain` which chains iterables together and can be used to
    make a simple shallow version of join/flatten. But it requires all elements
    to be iterable, whereas I want to flatten `['a', 'b', ['c','d'], 'e']`
    into `['a', 'b', 'c', 'd', 'e']`. So  I'm skipping it.



## 2014-01-07 16:44 Tuesday

Working on Filtering by type of planned spending.

  * Follow the example of category_of_work that Daniel did
  * Start by asserting the field appears in the form
  * The get_forward and get_reverse joins methods are about navigating the tree
    from the given start model up to the level where the filtered value is
    stored.
  * Also, see what else Daniel had to touch in order to implement category_of_work filtering.



## 2014-01-08 11:38 Wednesday

  * Daniel's tests look at the generated queries. They don't entail going to
    the database which is probably a good idea. So the test is effectively that
    the correct filter is gerated.  There needs to be another test that this query
    gives the right results too, I guess.

  * Apparently I can get a tf:SomeTreeFilter object and do 

        tf.queries['category_of_work'] 
        
    and it will return a list of ... query stuff. 

## 2014-01-08 17:15 Wednesday

Reproduced Daniel's code, need to merge to dev and staging and deploy. And drop a tag.


## 2014-01-09 15:02 Thursday

Working on Hide unwanted values from drop-down lists.  It turns out that
`BudgetCode` objects already have a link to a set of planning year objects that
determine when they appear. I'm going to re-factor this into a superclass
(at least in the model) and use the same approach for the other values.

Except user might have to be handled differently.

I want to build something like this for ALFIE.



## 2014-01-09 17:45 Thursday

Frioday morning:

- test and deploy fix to OSI live
- Deploy to staging to get server back u

## 2014-01-10 17:57 Friday


Stuck debugging

    Cannot resolve keyword 'budgetcode' into field. Choices are:
    approved_budget, id, locked, money_type, priorityfocusarea, year

in OSI when accessing `BudgetCode.valid_years`. e.g.:

    http://127.0.0.1:8000/admin/accounts/budgetcode/2/

note taking script
===================

## 2014-01-11 20:21 Saturday

What I want this script to do:

  - Each time called add thet ime as a ### heading
  - Firs time called each day, add the date and day name as a ## heading
  - I can still add new # headings with topic

Or maybe:

  - Instead of adding #headings I could add project tags on time lines like
    this:

        ### 21:21 OSI

    To avoid havig to type them much, the script could copy the last
    one each time and maybe put the cursor at that point so it's
    easy to change them?

I want to re-write the shell script as a vim script to do the searching and
replacing.

The vim function should:

  - only work on *markdown* buffers (return if file type is not
    markdown)
  - generate the date string
  - generate the time string
  - check if the date string appears in the buffer
  - if it date string does not already appear in the file, apend
    it (with surrounding blank lines)
  - append the date string with blank lines

Perhaps it should run each time the focus goes in the note buffer
so I can leave it open, but this might end up being annoying.
Probably better to have a global key-binding  that triggers the
command in vim if its open, and if its not open, opens it and
triggers the commend.

Then we need to get the synchronisation working with Git.  In
fact if we sync with Git it might not be necessary to insert the
time stamps because I could use the git log to see the history.



## 2014-01-11 21:40 Saturday

If the new lines are appended using normal commands:
emulating keystrokes, then they can be _undone_ with
'u'?


## 2014-01-11 22:22 Saturday

The script below (`tmp/note.vim`) inserts the time and date stamp.

    " Note
    "

    if exists('Note_loaded')
        delfun Note_add
    endif

    function Note_add()
        let l:datestamp = strftime('%F %A')
        let l:timestamp = strftime('%R')
        let l:lastline = line('$')
        let l:dateheader = ['', '## ' . l:datestamp]
        let l:timeheader = ['', '### ' . l:timestamp, '', '']
        call append(l:lastline, l:timeheader)
        call append(l:lastline, l:dateheader)
    endfunction

    let Note_loaded = 1

Still need to:

 -  make inserting the date conditional (use `search()`)
 -  move the cursor to the end of the file
 -  make sure the text can be removed with 'u'


## 2014-01-12 Sunday

### 14:55 

This version inserts the date only if it isn't already there

    " Note
    "

    if exists('Note_loaded')
        delfun Note_add
    endif

    function Note_add()
        let l:date = '## ' . strftime('%F %A')
        let l:time = '### ' . strftime('%R')
        let l:lastline = line('$')
        call append(l:lastline, ['', l:time, '', ''])
        if !search(date,'w')
            call append(l:lastline, ['', l:date])
        endif
    endfunction

    let Note_loaded = 1



## 2014-01-12 20:13 Sunday

Maybe I want to switch to *orgmode* as it supports putting
to-do items in amongst notes, which is a use case I have 
already started using.


## 2014-01-12 21:36 Sunday

Reading about *orgmode*, got distracted refactoring my vimrc 
so multiple `autocmds` become one with a call to a function.
See the benefit of learning vimscript finally?

## 2014-01-12 23:38 Sunday

I don't like the way orgmode presents embedded code, markdown just lets me
indent it. But it has the option to collect TODOs from all over the place and
present them, which might be useful.  It's date stamp stuff is really for
preparing future dated todos. Which is a bit esoteric. More likely a list of
todos for a given project. An if Im using time stamping to journal progress,
going back and checking todos from earlier might seem strange. I can see how it
might work though where you just add a todo when you're thinking about
something. And then get the whole list using the plugin commands. And 
work through them. I've not been very good at working through todos
since switching to GTG.

## 2014-01-15 12:44 Wednesday

Upgraded to Saucy Salamander this morning.  Freemind doesn't work. Installing
oracle java 8 according to:
http://ubuntuhandbook.org/index.php/2013/07/install-oracle-java-6-7-8-on-ubuntu-13-10/

## 2014-01-15 12:58 Wednesday

Java 8 download failed.  I now have a broken had to :

    sudo rm /var/lib/dbkg/lock
    sudo rm /var/cache/debconf/*


## 2014-01-15 14:48 Wednesday

Oracle java 8 installed, still cant open dialogue boxes in freemind.
Maybe it needs an earlier version, or maybe Oracle broke it.

cambridgeenergy.org.uk
=======================

## 2014-01-15 22:24 Wednesday

Moved search box to top menu but don't know how to position
it aligned with the menu items. For some reason it shows up
as aligned with the top of the page, not down to the bottom 
of its containing div. CSS is such a drag.

IT might be possible to just remove it from the template
and put it back with some sort of widget config?

## 2014-01-16 21:33 Thursday

Zenburn theme for gnome terminal
Via `gconf-editor`:


    pallette:
         #3F3F3F3F3F3F:#FFFF00000000:#EFEFEFEFEFEF:#E3E3CECEABAB:#DFDFAFAF8F8F:#CCCC93939393:#7F7F9F9F7F7F:#DCDCDCDCCCCC:#3F3F3F3F3F3F:#FFFFCFCFCFCF:#EFEFEFEFEFEF:#E3E3CECEABAB:#DFDFAFAF8F8F:#CCCC93939393:#8C8CD0D0D3D3:#DCDCDCDCCCCC
    foreground:
        #EFEFEFEFEFEF
    background:
        #3F3F3F3F3F3F

## 2014-01-17 09:09 Friday

- [ ] Try https://github.com/Shougo/neobundle.vim on vim?  And maybe some other
  async stuff like the shell?

## 2014-01-17 15:47 Friday

Idea:
Write google apps script to pull data out of a google calendar into a
spreadsheet and use to log time by different developeers (if we
can do it on our company docs account, we should be able to pull from
a calendar shred by the devs?)


## 2014-01-20 09:47 Monday

Welcome back!

 - [ ] What does this stuff look like on Github?
 - [x] What does this stuff look like on Github?




