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

``` VimL
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
```


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
Write Google apps script to pull data out of a Google calendar into a
spreadsheet and use to log time by different developers (if we
can do it on our company docs account, we should be able to pull from
a calendar shred by the devs?)


## 2014-01-20 09:47 Monday

Welcome back!

* [ ] What does this stuff look like on Github?
* [x] What does this stuff look like on Github?

For some reason I Can't get these to render as checkboxes.

Also, I want better Markdown syntax highlighting, maybe topoe



## 2014-01-20 10:53 Monday

### V4C Kick-off meeting

Can we do:

 - Site-map/user story/workflow plan for site with Ellie?
 - SSO with Sharepoint should be treated as a lower priority.
 - what is relationshiup with ALFIE code base and V4c?
 - What needs to be on a card for it to become a candidate?
    - What should it look like (visual design)
    - How should it behave
    - the whole journey
    - Definition of Done (it's done when you can do the following things
      with the system.) outline of steps to test manually
    - Every feature that has to be in the story should be listed before we
      start working on it.
        - If there are mock-up images, are they "the spririt" or the exact
          look of the things we have to build?
        - A list of features that have to be there, if not directly
          implied by the mock-ups.
        - Anything that is not included is not part of the story (c.f.
          story of boolean logical search, which entailed definition of a
          grammar for the search language)
    - technical approach
        - e.g save 'Outputs' they are several levels deep nested formsets
          vs e.g.  backbone
        - Both are 'unknown' to some degree
        - Don't feel comfortable enough to estimate
        - JAvascript approach don't just bung js files in a directory and
          link to them.
        - Set up front-end environment, e.g. Grunt/Require.js, etc.
        - Need javascript tests, need automatic running/ build server

 - when can we estimate cards (what to they need)

 - [ ] Marko to get met a sample of good card details/spec to look at from
   3ie

 - [x] MCS invite Elly and Marko to UML tool 
 - [ ] Talk Elly and Marko through data model 
 - [ ] SparkleShare for Elly (or Dropbox, ...?)


## 2014-01-20 14:53 Monday

Possible insight on sub-indicator: corresponds to rows in the tool Is it the
atomic level at which you measure, so that later measurements in that row
replace/override earlier ones. As opposed to being added together.


## 2014-01-20 16:19 Monday

Ask George:

- [ ] Is achievement of outputs one of the indicators for outcomes?
- [ ] There can be only one Goal, ok, but what about more than one outcome?


## 2014-01-24 15:04 Friday

A theory about sub-indicators.
 Sub-indicators:
        - are non-overalpping category spaces 
        - where later measurements replace earlier ones

## 2014-01-24 15:34 Friday

`python_2_unicode_compatible`

Helps to write polyglog python

`order` help us set the sequence of, e.g. milestones.

See 'serializers' in DjangoRestFramework.
ModelViewset in DJR don't look at reverse relationships.



## 2014-01-28 14:54 Tuesday

This is how Marko creates scoped globals variables in javascript

``` javascript
    var Aptivate = window.Aptivate = window.Aptivate || {};
```

## 2014-01-28 16:38 Tuesday


Check out what require.js does (one day)
It controls loading until dependencies are loaded
and also controls global name spaces.

## 2014-01-29 12:35 Wednesday

- A LogFrame should know what its milestones are
- These might be a subset of the available reporting periods (this relationship
  is different from  the Taxonomy-Term relationship. Taxonomy might be
  'reporting periods' and have more time points than we choose to show as
  milestones in the LogFrame
- But I feel uncomfortable with all Terms having an optional link to Logframe 
  in case they play the role of milestone

- However milestones are qualativively different from other tags because they
  have an associated end date or date period. 
  
- An Indicator should know if and what its sub-indicators are. There might be
  none.
- I think it makes sense for Indicator to have an associated sub-indicator
  Taxonomy whose members are the sub-indicator rows 

* The real decision behind whether Milestones are a kind of taxonomy-term or
  not is whether it will make it easier to build generic filtering. I just
  don't know yet.


## 2014-01-29 16:49 Wednesday

* Eventually our models (which are in models/models) will be separated
into separate javascript files and loaded only when needed using require.

## 2014-01-31 19:52 Friday

- `drf-routes-nested` is somewhat tricky to set up with `HyperlinkedModelSerializer`
for some reason. Switching our logframe and result serializers to ModelSerializer
enabled one level of nesting.

- Switching the logframe serializer back to Hyperlinkedmodelserializer 
gets our url represented as a full url. So long as I don't include the 'id'
field in the field set. If I do, I get 

    Exception at /api/logframes/1/

    Could not resolve URL for hyperlinked relationship using view name
    "result-detail". You may have failed to include the related model in your
    API, or incorrectly configured the `lookup_field` attribute on this field.
    
I think it might be trying to use our url conf to lookup "result-detail"
and identify which url pattern to use for reverse.

- Hyperlinkedmodelserializer don't include id by default, instead they have a
  url field.
- I get the error if I try and force them to have an 'id' field.
- Maybe we don't need it if we have URLs?
- I also get the problem if I try to include linked results.  but I'm not sure
  I really want to at the moment because instead of a list of urls for the
  results associated with my logframe, I can go to `/logframe/n/results/` to
  get a list of them.

- I can get results to render with Hyperlinkedmodelserializers so long as
  I don't include any linking fields in the field set. In particular I
  can't link to parent -- another Result, because it is trying to find a
  view called 'result-detail'.

- The instructions say that to use Hyperlinkedmodelserializer we must make
  sure our url patterns are named. This is where it's looking for
  `result-detail`.

## 2014-02-03 12:27 Monday

Backbone nested views: render sub-view in out-of-dom element
and then add to dom.


## 2014-02-03 14:44 Monday

Here's the require config that loads backbone_sync to fix csrf errors 
witht the django back end.

     var require = {
         paths: {
             'jquery': 'lib/jquery',
             'underscore': 'lib/underscore',
             'backbone_base': 'lib/backbone',
             'backbone': 'lib/backbone_sync',
             'handlebars': 'lib/handlebars',
         },                
         shim: {            
             'underscore': { 
                 exports: '_' 
             },                
             'backbone_base': { 
                 deps: ['underscore', 'jquery'],    
                 exports: 'Backbone'     
             },           
             'backbone': {
                 deps: ['backbone_base'],
                 exports: 'Backbone'  
             },          
             'handlebars': {           
                 exports: 'Handlebars'  
             }          
         }
     };

And here is backbone_sync

    (function() {
      var _sync = Backbone.sync;
      Backbone.sync = function(method, model, options){
        options.beforeSend = function(xhr){
          var token = $('meta[name="csrf-token"]').attr('content');
          xhr.setRequestHeader('X-CSRFToken', token);
        };
        return _sync(method, model, options);
      };
    })();

## 2014-02-03 15:14 Monday

When the element is empty you can't click on it. 
So add placeholder text, and mark the element with a class so 
that you can hide it in print style-sheets.

## 2014-02-03 15:36 Monday


Handlebars templates -- do we want a new template tag to load them verbatim
to avoid having to escape them with explicitly verbatim django tags?

## 2014-02-03 15:41 Monday

[Flexbox](http://www.w3.org/TR/css3-flexbox/) for making things that are table-like.

## 2014-02-03 16:52 Monday

[Backgrid](http://backgridjs.com/) a library of backbone widgets to use in grid display.


## 2014-02-03 16:54 Monday

Javascript developmentin vim:

- [ ] check syntax on save -- syntastic plugin?
- [ ] highlighting: highlight function names as headings 
    and method names in objects
# v4c

## 2014-02-06 14:29 Thursday

 * Use a backbone router -- to trigger different entry point 
   functions .

 * Create overview and results v-pages in separate folder
 pages/result.js

    create the right result from the results collection
    and use it to instantiate a view to present it

 pages/overview.js

 * Main would use a router to call the main function from the
   apropriate page.

 * Main would load the relevant data from Aptivate.data into
 collections 


* in principle when creating a result, we could also create an empty
  indicator  at the server side (because result without indicator is not useful)

## 2014-02-06 14:54 Thursday

See [This article](http://elucidblue.com/2012/12/24/making-qunit-play-nice-with-requirejs/) for a way to make Qunit work with require.js

    define(['qunit'], function(qunit) {
        var testFiles = ['test', 'modules', 'here'];
        require(testFiles, qunit.load);
    });
    
## 2014-02-06 23:27 Thursday

Added `assumptions` to Aptivate data. Boy that was hard.
I hate those tests!

Next up, populate the assumption collection (For a logframe) from this dataset (using reset?)
Check how assumptions part of interface behaves.

## 2014-02-08 15:07 Saturday

`Collection.fetch()` is asynchronous and non-blocking so doing 

    this.assumptions.fetch()
    this.assumptions.add(...empty one...)
    
doesn't put the new one at the end. The second line completes before the first one.
OK I'm going to switch `fetch` to `reset` soon, when I load data from `Aptivate.data`
but in the  meantime, here's what works:

            this.assumptions.fetch({
                success: this.addEmptyAssumption
            });


NOTE: `addEmptyAssmption` had to have `this` bound in order to work.

## 2014-02-08 20:55 Saturday

[This article](http://alexkehayias.tumblr.com/post/26630944947/backbone-in-the-wild-lessons-learned-using-backbone-js) 
by a django dev writing backbone talks about how to bootstrap your app with a view for the whole thing, and other hints.

Also recommends one view/model per file when working with require


## 2014-02-08 21:37 Saturday

Main does too much! Main is loaded asynchronously, so this data set up cannot
be relid upon and should be moved to a separate script upon which other modules
may explicitly depend to get the instantiated collections and models.  See:
http://requirejs.org/docs/api.html#data-main 




## 2014-02-09 20:11 Sunday

Holy Crap! Blocks don't create new scope in Javascript.
And variable definitions (not assignments) get moved (or 
[hoisted](http://www.adequatelygood.com/JavaScript-Scoping-and-Hoisting.html))
to the top of their enclosing function scope before execution.

## 2014-02-09 21:41 Sunday

No, seriously, this asynchronous malarky is just mental.  Don't initialize
members in their declaration if the value depends on anything else. This
bit me where I was picking up values of our global variable
`Aptivate.collections`.  This gets set up in `main.js` in what I'm calling
'run time' but the body of the class definition is executed at `load
time`, which we cannot rely upon when loading asynchronously with require.


I'm wondering whether instead of a global variable we'd be better
keeping these things in a module object and managing dependencies
explicitly with require. Having written that down it sounds like
a complete no brainer. Hmm.

## 2014-02-09 21:44 Sunday

And another thing: we using `backbone-subview` to let the dom elements
tell us which views to attach where. And we're creating a view per field
for the inline editing. IT might be possible to use a per-model view and
let the dom elements specify which field they are interested in.
See "Inline binding declarations" in 
[epoxyjs](http://epoxyjs.org/tutorials.html#simple-bindings)

## 2014-02-09 21:52 Sunday

How do we manage one-to-many relationships among models?  E.g. Assumptions
per Result? At the time of writing I have a single global collection for
the assumptions, and its url that of all the assumptions in the API. If I
were to `fetch()` them, I'd end up with many more than I want. Should I
populate the collection with all the `Assumptions`, and select only those
I want by magic logic, or create separate collections for each `Result`?
And how to associate those collections with the `Result` model objects?

The [Backbone docs](http://backbonejs.org/#FAQ-nested) say it's
common to nest collections inside models. The example shows an example
of a collection whose url is specific to its owning model object (implying
nested url scheme for API, which my gut still tells me we should be using)

[This tutorial](http://css.dzone.com/articles/backbone-relational-tutorial)
refers to something called [backbone-relational](http://backbonerelational.org/)

## 2014-02-09 22:19 Sunday

Our `collections` module had implicit dependency on our `models` module
because it referred to model names via Aptivate.<model>. But we can't
guarantee the order in which `models` and `collections` get loaded so this
dependency is not necessarily met by referring to the global variable.
Explicit dependencies looking ever more appealing.


#rest-framework

## 2014-02-10 09:01 Monday

From Marko:

> I just remembered another possible problem. I *think* django rest
> doesn't return anything on POST but we want it to return created object
> so we get its id. Without it we can't update it later. Not sure how to
> do that but hopefully there's an app for it. 


#zombies

## 2014-02-10 09:34 Monday

* [Clean this shit up](http://alexkehayias.tumblr.com/post/26630944947/backbone-in-the-wild-lessons-learned-using-backbone-js) 

* [Zombies](http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/)


#v4c

## 2014-02-10 11:09 Monday

* How confident that hierarchical models work?

  * Test update -- do we have to include the parent id or can the API infer it
    from the URL?

* Pretend the Assumptions collection we have is the one for the selected result

* Build a subset of the structured data:

```
    Result: Model ------> :Collection ------*> Assumptions:Model

```

* For the time being, let ResultPage have a reference to the result object
