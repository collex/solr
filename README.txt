# Licensed to the Apache Software Foundation (ASF) under one or more
# contributor license agreements.  See the NOTICE file distributed with
# this work for additional information regarding copyright ownership.
# The ASF licenses this file to You under the Apache License, Version 2.0
# (the "License"); you may not use this file except in compliance with
# the License.  You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

This provides and sets up the solr instance required for Collex Catalog.
--------------------------

This is mostly the Apache Solr project, but it is set up the way it needs to be for the
Collex Catalog to work. For instance, this version of solr has been tested with the Catalog
and with the current solr index. The configuration files are written specifically for the Collex
project. And the JVM is set up to be optimized to run on Collex's server.

--------------------------------------
Collex architecture

Collex is a complex project made up of a number of subprojects that all have to be in place
for it to work. Most users will probably just need to set up the main Collex piece and point it at
the existing Catalog. If that is all you want to do, then you don't need to understand the following
architecture and you don't need to download the "solr" or "catalog" projects.

When Collex is deployed, it is branded with the name of a particular "Federation", like NINES or 18thConnect.
The website that the end user goes to will look like that federation, but the code behind it is the "collex"
project here.

When a search is done from "collex", the request is made to the "catalog" project, which is a web service
that exposes all the documents that have been stored.

The "catalog" webservice processes the request and forms the correct call to the "solr" webservice.

The documents are added to the solr index by converting RDF documents using the project "rdf-indexer".

The About section of "collex" and the News section of "collex" are two separate WordPress installations. The
recommended theme to use is in the "collex_wordpress_theme" project.

The "typewright" project can be attached to a "collex" instance if you wish by setting it up in the site.yml file
of "collex". The "typewright" project is a webservice that keeps the information about all the typewright-enabled
documents. The actual web presence of typewright is in the "collex" project under subfolders named typewright.

--------------------------------------

To run on the production machine, you will probably want to create a service, something like
the following:

--------------------------------------
#!/bin/sh

# Starts, stops, and restarts Apache Solr.
#
# chkconfig: 35 92 08
# description: Starts and stops Apache Solr

SOLR_DIR="/path/to/solr"
JAVA_OPTIONS="-Djetty.port=8983 -DSTOP.PORT=8078 -DSTOP.KEY=mustard -jar start.jar"
JAVA="/usr/bin/java"

case $1 in
    start)
        echo "Starting Solr"
        cd $SOLR_DIR
        $JAVA $JAVA_OPTIONS &
        ;;
    stop)
        echo "Stopping Solr"
        cd $SOLR_DIR
        $JAVA $JAVA_OPTIONS --stop
        ;;
    restart)
        $0 stop
        sleep 1
        $0 start
        ;;
    *)
        echo "Usage: $0 {start|stop|restart}" >&2
        exit 1
        ;;
esac
--------------------------------------

In CentOS, save the above as /etc/rc.d/init.d/solr. The details may be different for your
OS and installation.

Then, to run solr:

	sudo /sbin/service solr start

When the server reboots, this project will restart automatically.

To run on a development machine, you can start and stop it from the Collex Catalog folder by
typing the following:

	rake solr:start
	rake solr:stop

