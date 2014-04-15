<!-- vim: sw=2 ft=ghmarkdown spell -->
# Indigo Data

- [ ] Call Tim Davies <tim@practicalparticipation.co.uk> - Practical participation Wednesday 

- Install RethinkDB http://www.rethinkdb.com/docs/install/ubuntu/

- Run a server

  $ rethinkdb create -d ~/Documents/rethinkdb
  $ rethinkdb -d ~/Documents/rethinkdb &

- Go to the [web interface](http://localhost:8080/#dataexplorer) to make sure its working.

- Create a database in the web interface:

    r.dbCreate('360giving');

- Let's [load some csv](http://rethinkdb.com/blog/1.7-release/):


