<!-- 
vim: sw=2 ft=ghmarkdown spell 
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
