<!--
vim: sw=2 ft=ghmarkdown spell
-->
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

        - If there are mock-up images, are they "the spririt" or the exact look
          of the things we have to build?
        - A list of features that have to be there, if not directly implied by
          the mock-ups.
        - Anything that is not included is not part of the story (c.f.  story
          of boolean logical search, which entailed definition of a grammar for
          the search language)
    - technical approach
        - e.g save 'Outputs' they are several levels deep nested formsets vs
          e.g.  backbone
        - Both are 'unknown' to some degree
        - Don't feel comfortable enough to estimate
        - JAvascript approach don't just bung js files in a directory and link
          to them.
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
- [ ] Editor tools like [Tern](http://ternjs.net/doc/manual.html)

## 2014-02-06 14:29 Thursday

@v4c

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


@rest-framework

## 2014-02-10 09:01 Monday

From Marko:

> I just remembered another possible problem. I *think* django rest
> doesn't return anything on POST but we want it to return created object
> so we get its id. Without it we can't update it later. Not sure how to
> do that but hopefully there's an app for it. 


@zombies

## 2014-02-10 09:34 Monday

* [Clean this shit up](http://alexkehayias.tumblr.com/post/26630944947/backbone-in-the-wild-lessons-learned-using-backbone-js) 

* [Zombies](http://lostechies.com/derickbailey/2011/09/15/zombies-run-managing-page-transitions-in-backbone-apps/)


@v4c

## 2014-02-10 11:09 Monday

* [x] Pretend the Assumptions collection we have is the one for the selected result

* [x] Build a subset of the structured data:

```
    Result: Model ------> :Collection ------*> Assumptions:Model

```

* For the time being, let ResultPage have a reference to the result object

## 2014-02-10 15:05 Monday

Finally,
[this](http://stackoverflow.com/questions/15815216/passing-arguments-to-a-backbone-views-constructor)
is how you access parameters to constructors.

Or perhaps not. Doesn't seem to work. Try this instead:

``` javascript

        initialize: function(options){
            this.stuff = options.stuff;
        }
```

## 2014-02-10 16:14 Monday

How confident am I that hierarchical models work?
  * Test update -- do we have to include the parent id or can the API infer it
    from the URL?


## 2014-02-10 16:16 Monday

We now have two two Assumptions collections: the global one and one associated
with the result.

* [x] Get data into the Result's one.
    * For the time being, just `fetch()` it. -- later we can set it up with a
      smarter URL or some query parameters to get the right ones.
 
* [x] Use that collection as the one that gets rendered on the page.

* `assumotion/list-view` needs :-
        - [ ] maybe a `render()` to display values already in the collection,
          and to call on `reset` events?
        - [x] some wiring to update its view when values change -- already has
          it


## 2014-02-10 16:45 Monday

Javascript vim options
 - [x] syntastic checkinng with jshint
 - [ ] better syntax highlighting.

       The `vim-javascript-syntax` plugin tries to highlight the arg list of
       anonymous functions as a function name. @fail @vim @javascript. Maybe
       the one it's branched from is better?

 - [x] set number
 - [x] 80 character lines
 - [ ] snippet for define([ ], function{ ... })

       Doesn't work. Maybe UltiSnips isnt getting my tabs any more.
       Since I use it rarely, I could bind it to something harder to type.

 - [ ] Find out how to remove trailing spaces



## 2014-02-10 16:58 Monday

Here's a generic 
[collection view ](https://github.com/anthonyshort/backbone.collectionview)
for Backbone.

## 2014-02-10 17:15 Monday

Just exactly how the hell do you access arbitrary argument lists in @JavaScript
functions? 
[Like this](https://javascriptweblog.wordpress.com/2011/01/18/javascripts-arguments-object-and-beyond/)


## 2014-02-11 10:56 Tuesday

We have separate definitions for collections but maybe we don't need them,
maybe they are simple customizations of  standard collections.

## 2014-02-11 12:03 Tuesday

Removed the custom collection class definitions for a Assumptions and Indicators.
There still needs to be a class definition:

This:

    new Backbone.Collection.extend({...})();

Doesn't work. The association is wrong. I guess this would work:

    (new Backbone.Collection.extend({...}))();

But that's ugly. So I'm preferring:

    var Xs = Backbone.Collection.extend({...}));
    new Xs();

## 2014-02-11 12:12 Tuesday

- [ ] Make indicator-list subview which gets installed in the whole
  middle frame under the ribbon.
- [ ] Attach it to the result.indicators
- [ ] Add a subview for indicator container
- [ ] add editabble name and description parts


## 2014-02-11 14:00 Tuesday

One day, when Git is a pain, try [Gitv](http://www.gregsexton.org/portfolio/gitv/) for @vim


## 2014-02-11 14:23 Tuesday

[Backbone.CollectionView](https://github.com/rotundasoftware/backbone.collectionView/blob/master/src/backbone.collectionView.js)
uses a collection container to "store, retrieve and shut-down" subviews.

## 2014-02-11 14:26 Tuesday


Have reconsidered 
[Backbone.CollectionView](https://github.com/rotundasoftware/backbone.collectionView/blob/master/src/backbone.collectionView.js)
for the indicator list (quick scan of functionality) and currently I'm not seeing a close mapping with what we want.



## 2014-02-11 15:46 Tuesday

Don't be tempted -- as I  just was, to move `_.bindAll(...)` to the end of
`initialize` if there are `listen_to` calls in there. It's imperative, not declarative:
you have to _do it_ to those methods before setting them as event handler callbacks.

## 2014-02-11 16:52 Tuesday

Refactored more generic `ListView` out of `AssumptionList`. Used it as the basis for Indicator list.
Problem is it currently tries to render as a list, which might not be what we want. I suspect the original
app used either embedded tables or divs, so I might have to remove the list dependency from `Listview`, or at least
make it overridable. 

## 2014-02-12 09:50 Wednesday

Turns out the tag in `Listview` is overridable in the normal way.
Simply added it to the instantiation in Result Container.

So now my list to do looks like this:

- [x] Make indicator-list sub-view which gets installed in the whole middle
  frame under the ribbon.
- [x] Attach it to the result.indicators
- [x] Add a sub-view for indicator container
- [x] add editable name and description parts
- [x] Make sure the update/save logic works and adds a new row
- [x] Can we get rid of the simple subclasses of `InputView` if they do nothing
  more than set values like template and attributes, they could be inlined
  where they are instantiated.
- [x] Check why I can't save assumptions, and maybe write a test for it.
- [x] I just added a `indicator-name.handlebars` which is _the same_ as
  `result-name.handlebars`. Refactor that shit!

- [ ] Can we have a generic xxxx-container view?

## 2014-02-12 11:29 Wednesday

- [ ] Find out how pymode removes trailing white space and do it to Javascript


## 2014-02-12 11:45 Wednesday

Here is [in-place editing](https://github.com/hendrikbeneke/backbone-editable/blob/master/backbone_editable.js.coffee) 
for backbone, in coffeescript


## 2014-02-12 11:47 Wednesday

Having touble with the difference between subclassing and instantiating.  AFIK
in a prototype based language these should be the same. But I'm guessing the
Backbone's View only copies certain properties into instances when you
instantiate. Whereas somehow when I create my own subclass and instantiate that
I can pass anything through. Eek.

## 2014-02-12 14:34 Wednesday

Turns out the functructor for `Backbone.View` only copies properties from `options` 
to `this` if they are listed in `viewOptions`. 

```javascript
  var View = Backbone.View = function(options) {
    ...
    options || (options = {});
    _.extend(this, _.pick(options, viewOptions));
    this.initialize.apply(this, arguments);
    ...
  };

  // List of view options to be merged as properties.
  var viewOptions = ['model', 'collection', 'el', 'id', 'attributes', 'className', 'tagName', 'events'];
```

But the same arguments that the constructor gets are also passed to `initialize`.
So I can explicitly copy the additional ones I want over onto `this` inside
my `initialize`.

```javascript
    var ListView = Backbone.View.extend({

        initialize: function (options) {
            this.items = options.collection;
            this.itemView = options.itemView;
            this.newModelOptions = options.newModelOptions;
            this.options = _.clone(options);
      ...
```

## 2014-02-12 16:21 Wednesday

Abortive attempt to refactor out the remaining duplication
of simple views by writing init methods on our generic views
that copy their required parameters from `options`.  This
is a bit tricky because extend puts new features on the prototype
whereas instantiation passes them to the constructor -- whuc then
passes them on to initialize. And in there if we copy values
to `this`, we risk overwriting values added with `.extend()`
with `undefined`, if the value isn't in options. And at the moemnt
we seem to do some of each.


## 2014-02-12 22:40 Wednesday

There is a cunning `_.once()` in Underscorejs, but it doesn't do what
I hoped it would: Instead of caching the results of a method per
object, it caches the results of the the function once forever,
irrespective of object.  There is also `memorize`, but this
is similar: caches multiple results for efficiency but not
per object. 



## 2014-02-12 23:38 Wednesday

Finnally managed to do the refactor I spent all afternoon attempting
where the small, declarative subclasses of `InputView` view get replaced
wtih inline declarations. Reduced tne overall number of lines.

I'm noping we can make `InputView` into an even more generic `EditableView`
(stareted referring to it ass that in anticipation) that will allow us
to generically specify:

 - the input element
 - placeholder text, 
 - etc

Because we can set that stuff as options on the view and also pass them
to the template ccontext. 



## 2014-02-12 23:56 Wednesday

Next priority is to link the assumptions and indicators to the selected result objectt.

 - [ ] Check out if what nested routes in the API do if you post something
 with the wrong key value.
 - [ ] Merge that branch with dev and try build the urls for the sub-models based on their owning ones.
 - [ ] See what happens when you save POST to those urls

## 2014-02-13 09:36 Thursday

Here's a good trick. BaseView needs `template_selector` to work.  I want to be
able to supply this either when extending or instantiating.  Extend simply adds
it to the prototype, but instantiation needs to look for it in the provided
options argument and copy it over, which by default it won't  because it's not
one of the standard `viewOptions` that Backbone.View copies accross. My
current solution below:

``` javascript
      constructor: function (options) {
          Backbone.View.apply(this, arguments);
          _.extend(this, _.pick(options, 'template_selector'));
      },
```

 - override `constructor` to do this, to leave `initialize` free for subclasses
   to redefinne safely.
 - call the parent class's constructor explicitly (the only way to do it)
 - `_pick` the require options from the options parameter not to get any other
   stuff being used for other purposes.
 - `extend` `this` so as not to set the instance attribute if its not defined
   in `options`. (I had a problem befoproe setting it to `undefined`)

## 2014-02-13 10:40 Thursday

Looking at the nested routes version of the API:

- [ ] The trailing slash is back: how do we get rid of that?
- [x] Even worse! `/logframes/1/` and `/logframes/2/` give the same set of
  logframes, so even though the urls are nested, it's not iterpreting the key
  value at all at the moment.

## 2014-02-13 12:44 Thursday

OK I spent all morning trying to figure out how nested routes might
possibly work; how they might select one object within another.
AFACT it's only the url structure that it deals with and I Can't see
how to select the queryset to work with. Pfft!

Ok back to the drawing board.

Let's fish out our properties with query parameters. Do we have those?


## 2014-02-13 15:38 Thursday

URL scheme design (after talking with Hamish).

- Each Collection in the client has a url, 

- It wants to be able to get from that url to get its contents

- Since I have collections _per_ containing object (like assumptions _per_
  result) I want a get to that url to get the assumptions for that result.  So
  I'd like to be able to go to `/logframe/1/results` to get all the results for
  the first logframe.

- They also want to be able to put/post to those urls to create/update objects
  And it might be convenient if we coul just put `{ 'description': "blah" }` to
  `/result/5/assumptions` to create Assumption `{ result: 5, description: "blah" }`

- Hamish says there are problems with urls of the form `/logframe/1/results/2`
  beacuse what if result 2 actually belongs to logframe 4? Do we have to
  validate this?  He suggests `/logframe/1/results` woul list the owned
  results, but `/results/n` would be the cannical url for any given result.

- The problem I have with this is that I don't want my collections to have
  different urls to use for getting and putting.

## 2014-02-13 16:31 Thursday

Since we have to roll our own filtering by parts of the URL anyways, there
isn't much difference between using url components and query parameters (apart
from it is possible to use some [standard django filtering
framework](http://www.django-rest-framework.org/api-guide/filtering#djangofilterbackend)
to add filtering options).

 - When you create a nested router, you can pass the lookup field which
   determines the name of the regexp match parameter for the parent single-item
   url. IT gets the name of the lookup field from the parent appended to it
   (`pk` by default) so in this case below:

``` python
    logframe_router = routers.NestedSimpleRouter(
        router,
        r'logframes',
        lookup='log_frame_',
    )
    logframe_router.register(r'results', ResultViewSet)
```

 I end up inside the view with `self.kwargs = { 'log_frame_pk': 2 }` or
 something, which I can pass directlyu to the manager to filter the results objects:


``` python
  class ResultViewSet(viewsets.ModelViewSet):
      model = Result
      serializer_class = ResultSerializer

      def get_queryset(self):
          return Result.objects.filter(**self.kwargs)
```

  

## 2014-02-13 17:10 Thursday

- [ ] We might not want the python api views in @v4c to support get for
   individual objects, instead we might jsut support lists and put/post?

- [ ] Error checking for urls of the form: `/logframe/1/result/2/assumptions`
  if result 2 doesn't belong to logframe 1. This is particularly important for
  posting to these, where we might try and infer the id of the owning object
  from the url. -- Though we could probably get away with a sloppy version that
  only looks at the nearest parameter, so posting to
  `/logframe/1/result/2/assumptions` wouldn't care if result 2 belonged to
  logframe 1, but would create an assumption with result = 2, unless it
  specified a different result id.  A more elegant solution would be t do
  proper checking but we might wait until later for that.
  
          

## 2014-02-13 17:18 Thursday

- [x] Make sure we can post to these urls to create, 
- [ ] Make object creation on nested routes fill in the owning object's key
  value from the URL if it is not set. (This isn't currently urgent because
  the client side stores the owning object in list view, but I would like 
  to see if that would go away.)

## 2014-02-14 09:29 Friday

Here' because I don't know where else to put it, is 
[Radio Swiss Jazz](http://www.radioswissjazz.ch/live/mp3.m3u)


## 2014-02-14 10:24 Friday

Merged nested_routes into master. 

We currently set the `urlRoot` explicitly in models AND collections.
I don't want to do that.  What inference can Backbone do for itself?

``` javascript
    url: function() {
      var base =
        _.result(this, 'urlRoot') ||
        _.result(this.collection, 'url') ||
        urlError();
      if (this.isNew()) return base;
      return base.replace(/([^\/])$/, '$1/') + encodeURIComponent(this.id);
    },
```

Models derive their url from url base, by adding their id to the end.
The base comes either from `urlRoot`, or from `this.collection.url`.


## 2014-02-14 15:08 Friday

USer Interface Events
- Enter: save
- Click outside
  ASK if ou meant to save or not
- Escape calcel

-

## 2014-02-14 15:13 Friday

- [x] Check if source is using the same data field as the description!

## 2014-02-17 09:43 Monday

Addin milestones 

 - [x] Seems odd that the collection classes I have defined in
   `results.js` don't have model set. How do they know what to instantiate
   when adding new ones? 

   A: I think they just instantiate `Backbone.Model`, and the only
   behaviour that I require them to have is that they work out 
   their url from their containing collection; which they do.

## 2014-02-17 11:31 Monday

 - [x] Make the milestones  filtered by their owning logframe in the API

## 2014-02-17 12:44 Monday

I wanted a non editable view for milestones.  So I split out the generic view
rendering from our `list-view` into a `static-list` and leaving `list-view` as
a subclass that adds the unsaved item behaviour.  In the process I re-bound
`addEmptyItem` to `rese` event instead of explicitly calling `fetch({ success:
this.addEmptyItem });`.  The problem is that the reset event is triggered once
the elements are all removed, but not after they have all been added.  In fact
you can't know when they have all been added, because `fetch` inserts them one
at a time with `addd`. So I think we need to bind `addEmptyItem` to the `sync`
events, if such exist.

## 2014-02-17 13:56 Monday

The above is quite true. Fetch calls add by default but you can make it do
reset by passing `{ reset: true }` which is kinda what we want when setting up
the data for the first time.

## 2014-02-17 14:16 Monday

Next up: Render the measurements. Where do those come from?  Are they in the
model yet? The plan says they should be owned by the indicators.

## 2014-02-17 16:34 Monday

What I'm trying to do now, is:

- Render a table row for the values
- Each with a sub-view tag to trigger a sub-view
- And a data tag to identify the milestone column it belongs to
- Then in the `subviewCreators`, or in side the created view, see if we can
  pick up these data values to choose the right `Target` model to manipulate. 

## 2014-02-17 17:28 Monday

 - The Result page has url `/logframe/1/` but there should be only one Logframe
   in the system, so maybe the url should be  `/result/<n>` which is the result
   uniqe ID

 - Make fixture with v4c logframe data in.

 - Deploy to staging server.

## 2014-02-18 09:05 Tuesday

Update strategy for rendering targets:

- Create a view with both the milestones list and all the available targets
- Iterate over the milestones, which define  the structure, and see if
there is a target value for each one
- If there is, render an editable view for that target value
- If not, create an unsaved target view


## 2014-02-18 11:36 Tuesday

 - [x] The targets table for unsaved indicators needs to listen to save events
   on its owning indicator and re-render

 - [ ] The `<thead>` in my table doesn't have the title row in it.  Forked
   backbone-subviews to try and fix the bug where it only works with divs. I
   can make the qUnit tests run in my browser, but they stop if I add one. Not
   sure what to do.

## 2014-02-18 15:58 Tuesday

 - Make the Milestone collections sort by their order value.  There isn't,
   as yet, any way to edit those order variables, except for hacking it in the
   admin interface. But at least we could set up the right milestones that way,
   with a bit of work.

 - Delete buttons!

 - Validate numeric inputs for Target values

 - Click outside the input box should save, not cancel


## 2014-02-19 15:17 Wednesday

NOTE! 
Django Debug toolbar breaks the app! Don't use it.


 - I'd like to find a way to make our EditbaleXXX templates more generic.
   We could pass the enclosing element type in as a context parameter.  Since
   they are mainly a single element, we in fact generate them programmatically
   which would be even neater.

## 2014-02-19 15:37 Wednesday

Sub-indicators:

Need to get the sub-indicators collection into the data-model, 

  - [x] add to the Indicator model
  - [x] Make the gui show any existing subindicator(s)?
  - [x] Make the api send a 'total' sub-indicator if there aren't any?
  - Fix it so you can't save Targets unless the subindicator in that row is saved,
    and so you can't save subindicators until the indicator is saved, etc.
  - Format numerics with commas
  - Validate numerics

 
Sub-indicator view doesn't always show the single 'total' subindicator I have 
created: this I think is because the data is being loaded asynchronously, and it
renders before the data arrives. So there needs to be a sub-indicator list
view that watches the relevant collection, and renders when there are changes
and is in charge of adding rows when new subindicators come into existence.

## 2014-02-20 11:01 Thursday

Revised to do list:

 - [x] The Result page has url `/logframe/1/` but there should be only one
   Logframe in the system, so maybe the url should be  `/result/<n>` which is 
   the result uniqe ID

 - [x] Changing the result number in the url has no effect, always go to the first one.

 - [ ] Make fixture with v4c logframe data in.

 - [x] Deploy to staging server.

 - [x] Fix it so you can't save Targets unless the subindicator in that row is saved,
       and so you can't save subindicators until the indicator is saved, etc.

 - [ ] Format numerics with commas

 - [ ] Validate numeric inputs for Target values

 - [ ] Make the Milestone collections sort by their order value.  There isn't,
   as yet, any way to edit those order variables, except for hacking it in the
   admin interface. But at least we could set up the right milestones that way,
   with a bit of work.

 - [ ] Delete buttons!

 - [ ] Click outside the input box should save, not cancel

 - [ ] The `<thead>` in my table doesn't have the title row in it.  Forked
   backbone-subviews to try and fix the bug where it only works with divs. I
   can make the qUnit tests run in my browser, but they stop if I add one. Not
   sure what to do.

 - [ ] NEed the ability to add new results at each level of the hierarchy,
   where does this go? In the overview page?


 - [ ] Might need a default contribution weighting too?

 - [ ] Impact weighting box floats over result description on result page

 - [ ] Add other relevant data to Aptivate.data on page and use to set-up in browser initial data structure

 - [ ] Protect the logframe content with user permissions, at least login required

 - [ ] Use precompiled handlebars templates

 - [ ] The ugly black tabs are caused by $tabs-background not being in scope somehow in the relevant file

 - [x] After vanilla deploy, the server didn't have permssion to access 

    OSError at /logframe/1/result/2/
    [Errno 13] Permission denied: '/var/django/v4clogframe/current/django/website/static/.webassets-cache'


 - [ ] I tried re-using the sub-indicator view for each render of the target row, but this stopped
       it responding to click events for some reason I can't yet figure out

There's an interesting problem with getting the id of a newly created subindicator to use
in its target values: the new ID doesn't come back from the server until 'some time later' 
asynchronously.

We need to not render the target values until the subindicator's id is known.
Which means we need to be listening to its sync events. By calling render on the subindicator row
view when the subindictor saves, and by not drawing the target values until there _is_ a subindicator
we can get the desired behaviour for creating subindicators.

Now there is a sumilar probelm with new indicators:

- Saving a new indicator now triggers `sync()` on its subindicators -- to pick up the default subindicator.
- That row looks good: the newly pulled subindicator has its parent indicator id and Target values in that row are saved ok.
- But the empty subindicator that appears under the newly-appeared default one is not behaving properly: it doesn't know it's owning indicator, and so target values in that row are not getting saved
- If I reload he page, it looks the same, with the 'Total' sbindicator and an unsaved subindicator below, but _now_ I the subindicator appears to know its owning indicator and crates a properly functioning row.

## 2014-02-21 12:58 Friday

 - [ ] Dump logframe from Staging server

## 2014-02-21 17:00 Friday

 - [ ] Entering white space in an input item and saving makes it
   impossible to click to edit again.

 - [ ] Clearing out all the text and pressing enter/tab doesn't send a
   save with empty string back to the server so the old value is
   preserved.

 - [ ] White space in long description fields gets collapsed; map line breaks
   to `<br>`?

 - [ ] While trying to use Jquery to animate adding new items, I found out we
   do unnecessary re-draws: edit the description of an indicator and it
   re-renders all the target values.  See the following excerpt from
   `static-list.js`
 

```javascript
         addItem: function (item) {
            // TODO: re-use existing sub-views if possible?
            var $el = new this.itemView({ model: item }).render().$el.hide();
            this.$el.append($el);
            $el.show(500);
        },

```


## 2014-02-26 16:53 Wednesday

Rewrite list views using Backbone.Subviews:

- Render the list as a list of placeholders with sufficient info for their
  relevant views to be created. This means:

- The parent object should include the related objects attribute as a list of
  IDs. 

- The Subview creators might pull out the models from the relevant collection
  (which might also be attached to the parent model, but directly, not in the
  attributes dict) using, e.g. the id from the placeholder.

- To put the placeholders in the right place the parent object should be the context
  for rendering the template.

## 2014-02-26 17:09 Wednesday

- I'm guessing that this becomes a base view, i.e. its render is rendering a template 
with `this.model` as the context.


## 2014-02-26 20:15 Wednesday

- Update the api to send 1:n relations as a list of ids

## 2014-02-26 21:12 Wednesday

I think we're looking for indicators too early in the process; that somehow they have
not been fetched yet.

## 2014-02-27 09:30 Thursday

Race conditions between asynchronously loading data with fetch and rendering
the page.  Before we did't see these because the list view was listening to add
and reset methods on the collection, but now it just renders itself and uses
the forward foreign key pointers from its model to infer the existence of the
related objects even when those have not yet been loaded. 

- On the plus side, we let the objects take care of their  nesting structure
  themselves, and it gives a good way to process the internal foreign key data
  representations (lists of ids)

- On the minus side, we now have to have a loaded collection for the related
  objects before we can even begin to think about rendering the owner.

  - Unless we program it defensively, somehow?


## 2014-02-27 09:55 Thursday

What if instead of having a model to render, template list has a collection,
and assumes it can render the id of its contents. Then we could load them up
with the nested Collections alreadyy in the in-page data structure, the urls
and stuff would stay the same, and the change events for adding items etc would
come from the correct collections, other wise, for example, we end up listening
to add/reset evcents on a collection attahed  to `/api/indicators` and when one
of those is added, how do we know which list to re-render?

## 2014-02-27 10:21 Thursday

Made template-list use a collection instead of a model. This avoids the problem
of finding a generic way to tell the template which attribute of the model
contains the list we're interested in iterating.


## 2014-02-27 12:09 Thursday

Grrr! Backbone.subviews!
It caches the sub-views it creates in a hash keyed by the value of `data-subview`.
So when I create a bunch of them in a loop, they all have the same selector
which means it uses the one already in the cache. Maybe the caching algorithm 
needs to know -- in some generic way, if there are multiple instances of the same
subview creator and cache them by name and identity somehow. Damnit.


## 2014-02-27 14:37 Thursday

Here's an interesting idea:

```javascript
  this.$el.data( "view", this );
```

## 2014-02-27 17:34 Thursday

Use the DOM to store a ref to the view responsible for an element.

## 2014-02-27 17:34 Thursday

OK, Subviews fixed to accept and cache multiple views of the same kind and the
app using it in a couple of places. Next thing to do is to extend it to 
provide the functionality of the add-one-unsaved-item view.


## 2014-02-28 15:33 Friday

Rewriting `list-view` as `addone-list`.  The old one overrides `addAll` and
adds the additional item there. The new one doesn't have `addAll`, it just uses
`render`. So, when do we add the new one? Render is effectively rendering the
`this.collection`, so we want the new, unsaved item to be in the collection by
the time we render. So let's try adding it in `initialize`

## 2014-02-28 17:07 Friday

  I think that by making addone-list call its parent (template-list)'s
  constructor I am registering a render method on add events that calls the wrong
  render: the one from base-view not the one from addone-list.


Nope, that's not it. It's this: by adding an unsaved item, we're adding
one that has no id. But the template-list view uses the `id` attribute
to choose which subview to create. 




## 2014-03-03 10:43 Monday

    
Now initially the addone-list renders the empty item ok though there's a bug
which means the empty assumption doesn't save - no url set?  It's a bit of a
dog's breakfast to avoid problems of recursion when adding the empty item to
the collection triggers render first time through.

i really want the position of the unsaved item to be determined by the
template. that means being able to exclude it from the template iteration,
eithe with template logic or by removing it fro the collection that gets
passed in, then the template can ask for it explicitly by setting
data-subview-id="new".

and if I do that, to make marko's requested max size feature I'll have to
override templatecreator for liss to return null if the collection is full.

## 2014-03-03 12:11 Monday

Now I want to work backward from the template I want to be able to write:

``` handlebars
{{#each items}}
<div data-subview="itemView" data-subview-id="{{this.id}}"></div>
{{/each }}
<div class="unsaved" data-subview="itemView" data-subview-id="new"></div>
```

- [x] The unsaved item does not appear in the list of items to iterate
- [x] The template may explicitly ask for the unsaved item by  using 'new' as
  the id.
- [x] The `subviewCreators` can return undefined for the unsaved item when the
  list has reached its max size
- [ ] The template can add classes to the items which get propagated onto the
  subview elements.

I can foresee a problem where the unsaved item view remains with a handle on
the formerly unsaved item which is now saved.  Using `add()` with fallback to
return the existing object only works for objects that have an ID. For the new
one, we get back a fresh one each time.

## 2014-03-03 14:48 Monday

Does the reset that loads initial data into the collection remove the unsaved item
and thus it no longer knows the collection it belongs to? And if so, what the hell do we do next?


## 2014-03-03 16:31 Monday

- [x] Add `itemViewOptions` in list view used when instantiated `itemView`.

## 2014-03-03 18:06 Monday


There IS a reset event when the data is loaded.
This removes the model from the collection.
Then we get two new models, 1 and 2 in the page.
Their placeholders cause calls to `subviewCreator `which finds out the relevant objects ok.
The new placeholder DOESN'T because we already created the view for the new item
and it is in the subviews cache. So that view has been silently returned to the
page, but the model is already removed from the collection. 


## 2014-03-04 16:47 Tuesday

Testing with qUnit sucks because I want to test the logic of the templates too
but currently I have to include them in the containing html file to make the tests
run. What I really want is to load them dynamically as part of running the test.
What's the answer to that?


## 2014-03-05 10:29 Wednesday

- [ ] The list views used int he milestone/target view don't lend themselves
  easily to the re-written views with templates because there are many (one for
  each milestone/sub-indicator combination) unsaved items. The new view needs
  each of these to have a unique cache value. 

  Maybe I can use the row/column values as the cache values for unsaved items
  and actual id's for saved ones. That's going to be a bit more complicated.


## 2014-03-05 12:07 Wednesday

- [x] When using the new list class for result overviews, there will no longer
  be a need to set the element type to `div`.

## 2014-03-05 15:10 Wednesday

Change how `TemplateList `gets its items to the template context after Marko
addes `getTemplateData`

## 2014-03-05 17:14 Wednesday

- [ ] Replace marko's "work-round" for passing level or something to the subviews
with a solution using `itemViewOptions`.

## 2014-03-06 15:03 Thursday

- [ ] Uniform spelling and punctuation 
- [ ] modularise the API code in python when we rewrite it
- [ ] modularise the API code in python when we rewrite it



## 2014-03-06 16:58 Thursday

- [ ] Change existing template use to use: single tempalte ??/


one reason we get everything wrapped in a div is because of how BaseView works:

``` javascript
            this.$el.html(
                template(
                    this.getTemplateData ? this.getTemplateData(modelJson) : modelJson
                )
            );
```

This says insert the result of rendering the template inside $el. So we neer
loose the el. This explains why we have unnecessary divs round all the editable
items.

The template for editable-field is heading towards `<{{tag}}
class="{{classes}}" ...>{{content}}</{{tag}}> `i.e. totally generic but we have
all those controls for the created element.

How about the basic design assumption for editable fields is that there is an
attribute of the model that we want to present in either a) an editable tag,
e.g. input, or a presentation tag, e.g. div/span/h1, etc. And we just do it
program aerially, and only use a program aerially in the case where the
rendering entails more complex html?

## 2014-03-07 09:26 Friday

- [ ] I can expand unsaved results and add save activities against them, well I
  cant' that fails.
- [ ] We need to guard against saving with the field is empty, the back end
  doesn't like it.




## 2014-03-07 11:18 Friday

Maybe activities should live in their own app? Might be easier if 
They had a generic relation to attach them to resuts?

## 2014-03-07 11:54 Friday

- [ ] Make name fields optional in activities and budgetlines
- [ ] We currently don't enforce, e.g. the 256 max character length on name fields, etc.


## 2014-03-10 09:41 Monday

- [ ] Leave commas in while editing amount fields

- [ ] Better "information design" for amount totals -- marko doesn't like the
  bigger fonts and right justified totals

- [ ] Discussion about how we tackle Delete

- [ ] Overview page should start with Goal and Outcome expanded, showing outputs, probabl???
  - [ ] Open the dashboard with top two levels expanded (show outputs)

- [ ] Discuss how we tackle oob updates (someone added something to the tree I have to refresh the page)

- [ ] Results don't have indicators and shit, remove edit button

- [x] Talk to Luke Everett about linking to sharepoint documents

- [ ] v4c login css (From ruforum) needs reworking

- [ ] Conditional loading, to reduce page size

- [ ] Design the feedback process for, e.g. server errors

- [ ] Print stylesheets?

## 2014-03-12 15:31 Wednesday


- [ ] Can't enter empty amount field for TA. If I clear the field I get `.00` !!

## 2014-03-14 10:35 Friday

- [x] Create new indicator creates two blank subindicator items in the table

## 2014-03-14 15:38 Friday

### Filter by lead:

- [ ] Does the Impact really match?  

  yes if you think of this as a computer scientist would: hierarchically but
  possibly not if you consider the goal and outcome to be different things.  If
  we were to change the user interface design to resemble the mockups: with the
  single impact and outcome separate from the outputs, then the message we
  would be sending would mean those top level items do NOT match. Since there
  are only one of each there is no information in saying they match (by
  transitivity).  So maybe we should make both these changes.
  

## 2014-03-17 10:41 Monday

- [ ] When following link to accept system invitation I get "Djano Administration" page,
need to add some branding

## 2014-03-17 12:25 Monday

- [ ] The world "null" appears in indicator descriptions when I try to edit them.

## 2014-03-17 14:40 Monday

- [ ] In Marko's rework of loading data from the flat API we have the
  subcollections owned by their views. I would prefer to have them owned by the
  relevant model  object as before so that there is a single data structure for
  the app and the data structure is effectively isolated there, the views don't
  need to have knowledge of how the data model objects are related to one
  another, they just get given a collection, (or subcollection) and render it.
  This is really bugging me. I don't like the idea that the relationships
  between objects might be defined more than once if we had different views
  showing the same object relation in different ways. Instead of just rendering the
  collection from the data model, they would each recreate that collection and
  define the data dependency in the view. Yickk!

  Another reason I don't like it is it introduces dependency on Aptivate.logframe
  all over the place, whereas if I just passed in a collection object or
  it found one on the given object, it would be easier to create testing configurations.


## 2014-03-18 09:46 Tuesday

Adding status updates

- Add [django-cuser](https://github.com/Alir3z4/django-cuser)-
  
  Not such a good idea, the Ruforum user stuff has switched out the user model and Cuser
  doesn't know how to work with different ones. Marko says `request.user` has current user.
  Maybe just put that into `aptivate.data` and use to fill the field in js?

- [x] Add model for status types 
- [x] Add admin for status types
- [x] Create model for status updates 
  - With datetime field
- [x] Add API for status types and status updates
- [x] Add status list to page
- [x] Put status codes into Aptivate.data
- [x] Remove unneeded status codes in Aptivate.logframe.statuscodes 
- [x] Add user (back-end?)
- [x] Add description
- [x] Add date
- [-] Sort by date?

## 2014-03-18 11:15 Tuesday

- [ ] Should status updates be updateable at all? Only by their author?
- [x] Better text representation of status updates? Include code/date?

## 2014-03-18 12:39 Tuesday

- [ ] Do we still get unnecessary divs on our list views? I thought that was
  cleared up now? But I notice Marko renders acctivitiy lines in separate table
  from the headings for some reason?

## 2014-03-18 14:15 Tuesday

- [ ] In the models file Marko has divided the models according to what page
  they sit on, I disagree with this way of structuring things around the pages,
  I want things structured around the data model.

- [ ]  Something is wrong with our encapsulation. In order to add a new view, I have to change:
  - models.js to add the model
  - collections.js to add the collection, which duplicates the url in the model
  - app.js to instantiate the collection
  - router.js to put data into it.

## 2014-03-19 15:17 Wednesday

- [ ] Collapsable toggle for status updates to display most recent 
  - [ ] Css it to look active with the finger link
  - [ ] Make status date default to today so they can't enter an update without a date
  - [ ] New status not being added to bottom of list: possibly use auto-allocated dates with timestamps to do ordering?

- [ ] Fix colours on search bar
- [ ] Zebra stripe tables to make them more readable in wide screen


## 2014-03-20 09:22 Thursday

- [ ] Fill in today's date by default in status updates
- [ ] Sorting by date still not working? 
- [ ] I'm not seeing link buttons on Pen RTE




## 2014-03-20 10:52 Thursday

- [ ] Clearning the search filter criteria after demo left some items greyed out

## 2014-03-20 11:53 Thursday

if Emeka gets to senior management to agree to 5th iteration, and asks for additional iteration
can we, in 4 and 5th iteration 

## 2014-03-24 15:41 Monday

- [ ] If I hit the clear filter button when some of my outputs or results don't have activities, they get greyed out, this is a false match.


## 2014-03-25 09:57 Tuesday

- [ ] I cam click on the ionput field under a multi line input in result heading if it has mutiple lines, add bug

## 2014-03-27 14:28 Thursday

- [ ]total budgets missing in activities
