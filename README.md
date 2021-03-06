# nhm-windshaft-app

This node application provides the tile server used by the [Ckan tiled map extension](https://github.com/NaturalHistoryMuseum/ckanext-map). It consists of a thin layer to handle worker processes, queue and prioritize requests, generate SQL and stylesheets on top of a [Windshaft](https://github.com/CartoDB/Windshaft) server.

- The URL for tiles is '/database/:database_name/table/:table_name/:z/:x/:y.(png|grid.json). Note that while this can serve tiles from different databases/tables, the geom fields must be the same across the tables (see *Configuration*);
- Supported parameters are `filter` for filters (formatted as per a CKAN 2.3 URL - ```field1:value1|field2:value2|...```) and `q` for full text search;
- Starting the application will spawn a number of workers (see *Configuration*), which can be set to restart after a number of requests. Sending a SIGHUP to the main process will cause all the workers to be restarted one by one, ensuring there are always some workers available - this is useful for providing code updates without downtime.

## Configuration
In config.js you must define the following:

- ```windshaft_port```: Port on which the server will be listening;
- ```postgres_host```: The Host for the postgres database;
- ```postgres_port```: The port for the postgres database;
- ```postgres_user```: The postgres username (**make it read only!**);
- ```postgres_pass```: The postgres password;
- ```geometry_field```: The geometry field. This cannot be configured per request, and must be constant. The default on the Ckan plugin in '_the_geom_webmercator';
- ```geometry_4326_field```: The lat/long geometry field. This cannot be configured per request, and must be constant. The default on the Ckan plugin is '_geom';
- ```id_field```: The field used to uniquely identify a row. This cannot be configured per request, and must be constant. The default on the Ckan plugin is '_id';
- ```num_workers```: The number of workers to run. If the application is CPU bound, then this should be equal to the number of CPUs and no more;
- ```worker_max_requests```: The maximum number of requests a worker will server. When reached the worker will be closed and a new one spawned. Set to 0 to disable this feature;
- ```requests_per_client```: The maximum number of requests the queue will send to the windshaft component *for each client*. There is no need for this to be higher than num_workers;
- ```windshaft_port```: Internally, the port the windshaft component listens on;
- ```query_options```: By default filters are translated into `field=value` components in the where statements. It is possible here to define alternative handlers for specific filters, by defining a dict which associated filter name with an SQL where statement. The following placeholders are available:
    - `{option_value}`: The value associated with the filter in the input. This will be escaped and quoted;
    - `{resource}`: The table name for the queried resource. This will be quoted;
    - `{geom_field}`: The geometry field name, expanded to include the table name. This will be quoted;
    - `{geom_field_label}`: The geometry field name alone. This will be quoted;
    - `{geom_field_4326}`: The lat/long geometry field name, expanded to include the table name. This will be quoted;
    - `{geom_field_4326_label}`: The lat/long geometry field name alone. This will be quoted;
    - `{unique_id_field}`: The unique id field, expanded to include the table name. This will be quoted;
    - `{unique_id_field_label}`: The unique id field name alone. This will be quoted.
    Note that the default `_tmgeom` filter is expected by the CKAN component.

## Installation

There instructions are for Ubuntu 12.04 LTS. The first thing to do is install a
recent version of [nodejs]() and [npm](). You can skip this test if you already have more recent versions.

    sudo apt-get update
    sudo apt-get install -y build-essential
    mkdir ~/build
    cd ~/build
    git clone git://github.com/joyent/node.git  
    cd node
    git checkout tags/v0.10.15
    ./configure --prefix=/usr/local
    sudo make install
    cd ..
    git clone git://github.com/isaacs/npm.git
    cd npm
    git checkout tags/v1.3.5
    sudo make install

We will need `redis`:

    sudo apt-get install -y redis-server

And mapnik. Again, we need a more recent version than is provided with
Ubuntu 12.04 LTS:

    sudo echo "deb     http://ppa.launchpad.net/mapnik/v2.2.0/ubuntu precise main" > /etc/apt/sources.list.d/mapnik.list
    sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys 4F7B93595D50B6BA 
    sudo apt-get update
    sudo apt-get install -y libmapnik libmapnik-dev mapnik-utils python-mapnik

Finally we can install nhm-windshaft-app:

    sudo git clone git://github.com/NaturalHistoryMuseum/nhm-windshaft-app.git /var/www/nhm-windshaft
    sudo chown -R www-data:www-data /var/www/nhm-windshaft
    cd /var/www/nhm-windshaft
    npm install
    
## Running

You can simply run:

    node server.js
   
If you want to run this as a service under Ubuntu 12.04LTS, save the following file as /etc/init/mapserver.conf:
    description "Map tile server"
        
    start on (local-filesystems and net-device-up IFACE=eth0)
    stop on shutdown
        
    respawn
    
    exec sudo -u www-data /usr/local/bin/node /var/www/nhm-windshaft/server.js >> /var/log/nhm-windshaft.log 2>&1

This will ensure that the service is started at boot, and is re-started if it crashes. It can be started and stopped manually using `service mapserver (start|stop)`