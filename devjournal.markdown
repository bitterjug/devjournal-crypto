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
        );
    }, {
      nonAtomic: true
    });

## 2014-04-23 16:55 Wednesday


- [x] Register bugs with RethinkDB?

  - Can't sort more than 100,000 rows (WTF!?)

  - Inserting dates for each of my 120,000 Big Lottery data rows failed "Have you shut down the server?"

## 2014-05-12 14:54 Monday

- [ ] Need to Fix Mutt: I think my default archive action, bound to 'a' should use quasi delete.

## 2014-05-13 12:40 Tuesday


