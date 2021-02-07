MusicBrainz-R2RML
=================

R2RML mappings for the MusicBrainz schema on an entity-by-entity basis. 

These can be run on the MusicBrainz server using ultrawrap, for which a script is provided (`dump.sh`).
You must set an environment variable `ULTRAWRAP_HOME`.

Running `musicbrainz-r2rml/dump.sh entity` (where entity is artist, track, etc.) runs the appropriate set of mappings (e.g. `mappings/artist.ttl`) to produce output in the form of NTriples (e.g. `output/artist.nt`).

A virtual machine is available (for use with VirtualBox, VMware, etc.) with a replicated MusicBrainz database.

Note that the file `musicbrainz_compile_config.properties` must reflect your DB name:
* `musicbrainz_db` is the default for a snapshot
* `musicbrainz_db_slave` is the default for a replicated database

Please report any issues on [our Jira tracker](https://tickets.metabrainz.org/), under the `LINKB` project.


## SPARQL endpoint

Uses MusicBrainz as a Virtual Knowledge Graph, where SPARQL queries are translated into SQL queries, which are then executed over the PostgreSQL database. In this setting, no materialization is needed.

Is based on [Ontop](https://ontop-vkg.org) and runs a Docker container.


### Merging the mapping files

```bash
cd MusicBrainz-R2RML
docker build -t jena -f jena-Dockerfile .
docker run jena riot --formatted=TURTLE  ./mappings/*.ttl > ./merged-mapping.ttl
```

### Download the JDBC driver

Download [the PostgreSQL JDBC driver](https://jdbc.postgresql.org/download.html) and copy it into the `jdbc` folder.



### Start the SPARQL endpoint
```bash
docker run --rm \
           -v $PWD:/opt/ontop/input \
           -v $PWD/jdbc:/opt/ontop/jdbc \
           -e ONTOP_MAPPING_FILE=/opt/ontop/input/merged-mapping.ttl \
           -e ONTOP_PROPERTIES_FILE=/opt/ontop/input/ontop.docker.properties \
           -p 8083:8080 \
           ontop/ontop-endpoint:4.1-snapshot
```