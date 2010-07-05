==============================================================================================================
osm2pgrouting Import Tool
==============================================================================================================

osm2pgrouting is a command line tool that makes it very easy to import OpenStreetMap data into a pgRouting database. It builds the routing network topology automatically and creates tables for feature types and road classes. osm2pgrouting was primarily written by Daniel Wendt and is currently hosted on the pgRouting project site: http://pgrouting.postlbs.org/wiki/tools/osm2pgrouting

.. note::

	There are some limitations though especially regarding network size. The current version of osm2pgrouting needs to load all data into memory, which makes it fast but also requires a lot or memory for large datasets. An alternative tool to osm2pgrouting without the network size limitation is *osm2po*. It's available as under a "freeware license" (not open source license unfortunately)
	

Raw OpenStreetMap data contains much more features and information than need for routing. Also the format is not suitable for pgRouting out-of-the-box. An ``.osm`` XML file consists of three major feature types:

* nodes
* ways
* relations

The data of Barcelona.osm for example looks like this:

.. literalinclude:: code/osm_sample.osm
	:language: xml

Detailed description of all possible OpenStretMap types and classes can be found here:  http://wiki.openstreetmap.org/index.php/Map_features.

When using osm2pgrouting, we take only nodes and ways of types and classes specified in ``mapconfig.xml`` file that will be imported into the routing database:

.. literalinclude:: code/mapconfig_sample.xml
	:language: xml

The default ``mapconfig.xml`` is installed in ``/usr/share/osm2pgrouting/``.


-------------------------------------------------------------------------------------------------------------
Create pgRouting database
-------------------------------------------------------------------------------------------------------------

Before we can run osm2pgrouting we have to create PostgreSQL database and load PostGIS and pgRouting functions into this database. Therefor open a terminal window and execute the following commands:

.. code-block:: bash

	# become user "postgres" (or run as user "postgres")
	sudo su postgres

	# create routing database
	createdb routing
	createlang plpgsql routing

	# add PostGIS functions
	psql -d routing -f /usr/share/postgresql/8.4/contrib/postgis-1.5/postgis.sql
	psql -d routing -f /usr/share/postgresql/8.4/contrib/postgis-1.5/spatial_ref_sys.sql

	# add pgRouting core functions
	psql -d routing -f /usr/share/postlbs/routing_core.sql
	psql -d routing -f /usr/share/postlbs/routing_core_wrappers.sql
	psql -d routing -f /usr/share/postlbs/routing_topology.sql
	
An alternative way with PgAdmin III and SQL commands. Start PgAdmin III (available on the LiveDVD), connect to any database and open the SQL Editor:

.. code-block:: sql

	-- create routing database
	CREATE DATABASE "routing";
	
Then connect to the ``routing`` database and open a new SQL Editor window:
	
.. code-block:: sql

	-- add plpgsql and PostGIS/pgRouting functions
	CREATE PROCEDURAL LANGUAGE plpgsql;

Next open ``.sql`` files with PostGIS/pgRouting functions as above and load them to the ``routing`` database.
	
.. note::

	PostGIS ``.sql`` files can be on different locations. This depends on your version of PostGIS and PostgreSQL. The example above is valid for PostgeSQL/PostGIS versions installed on the LiveDVD.
	
	
-------------------------------------------------------------------------------------------------------------
Run osm2pgrouting
-------------------------------------------------------------------------------------------------------------

The next step is to run ``osm2pgrouting`` converter, which is a command line tool, so you need to open a terminal window.

We take the default ``mapconfig.xml`` configuration file and the ``routing`` database we created before. Furthermore we take ``~/Desktop/pgrouting-workshop/data/sampledata.osm`` as raw data. This file contains only OSM data from downtown Barcelona to speed up  data processing time.
	
.. code-block:: bash

	osm2pgrouting 	-file "~/Desktop/pgrouting-workshop/data/sampledata.osm" \
					-conf "/usr/share/osm2pgrouting/mapconfig.xml" \
					-dbname routing \
					-user postgres \
					-clean
					
A list of all possible parameters:

* required

  * ``file    <file>      -- name of your osm xml file``
  * ``dbname  <dbname>    -- name of your database``
  * ``user    <user>      -- name of the user, which have write access to the database``
  * ``conf    <file>      -- name of your configuration xml file``
	
* optional

  * ``host    <host>      -- host of your postgresql database (default: 127.0.0.1)``
  * ``port    <port>      -- port of your database (default: 5432)``
  * ``passwd  <passwd>    --  password for database access``
  * ``clean               -- drop peviously created tables``

.. note::

	If you get permission denied error for postgres users you can set connection method to ``trust`` in ``/etc/postgresql/8.4/main/pg_hba.conf`` and restart PostgreSQL server with ``sudo service postgresql-8.4 restart``.


Depending on the size of your network the calculation and import may take a while. After it's finished connect to your database and check the tables that should have been created:

.. code-block:: bash

	psql -U postgres -d routing -c "\d"
	
If everything went well the result should look like this:
	
.. code-block:: sql

		             List of relations
	 Schema |        Name         |   Type   |  Owner   
	--------+---------------------+----------+----------
	 public | classes             | table    | postgres
	 public | geometry_columns    | table    | postgres
	 public | nodes               | table    | postgres
	 public | spatial_ref_sys     | table    | postgres
	 public | types               | table    | postgres
	 public | vertices_tmp        | table    | postgres
	 public | vertices_tmp_id_seq | sequence | postgres
	 public | ways                | table    | postgres
	(8 rows)
