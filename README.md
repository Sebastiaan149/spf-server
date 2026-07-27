This repository contains code modified from **[Triple Pattern Fragments](http://www.hydra-cg.com/spec/latest/triple-pattern-fragments/)**, to contain **Star Pattern Fragments**.

# Linked Data Fragments Server <img src="http://linkeddatafragments.org/images/logo.svg" width="100" align="right" alt="" />
On today's Web, Linked Data is published in different ways,
which include [data dumps](http://downloads.dbpedia.org/3.9/en/),
[subject pages](http://dbpedia.org/page/Linked_data),
and [results of SPARQL queries](http://dbpedia.org/sparql?default-graph-uri=http%3A%2F%2Fdbpedia.org&query=CONSTRUCT+%7B+%3Fp+a+dbpedia-owl%3AArtist+%7D%0D%0AWHERE+%7B+%3Fp+a+dbpedia-owl%3AArtist+%7D&format=text%2Fturtle).
We call each such part a [**Linked Data Fragment**](http://linkeddatafragments.org/).

The issue with the current Linked Data Fragments
is that they are either so powerful that their servers suffer from low availability rates
([as is the case with SPARQL](http://sw.deri.org/~aidanh/docs/epmonitorISWC.pdf)),
or either don't allow efficient querying.

Instead, this server offers **[Star Pattern Fragments](https://relweb.cs.aau.dk/spf/)**.
Each Star Pattern Fragment offers:

- **data** that corresponds to a _star pattern_
  _([example](http://example.org/example?s=http%3A%2F%2Fexample.org%2Ftopic&triples=2&star=[p1,http%3A%2F%2Fexample.org%2Fpred1;p2,http%3A%2F%2Fexample.org%2Fpred2]))_.
- **metadata** that consists of the (approximate) total triple count
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=rdf%3Atype&object=))_.
- **controls** that lead to all other fragments of the same dataset
  _([example](http://data.linkeddatafragments.org/dbpedia?subject=&predicate=&object=%22John%22%40en))_.

This is a **Java** implementation based on Jena. 

## Build
Execute the following command to create a WAR and JAR file:
```
$ mvn install
```
## Deploy stand alone
The server can run with Jetty from a single jar as follows:

```
java -jar ldf-server.jar [config.json]
```

The `config.json` parameters is optional and is default the `config-example.json` file in the same directory as `ldf-server.jar`.

## Deploy on an application server
Use an application server such as [Tomcat](http://tomcat.apache.org/) to deploy the WAR file.

Create an `config.json` configuration file with the data sources (analogous to the example file) and add the following init parameter to `web.xml`:

    <init-param>
      <param-name>configFile</param-name>
      <param-value>path/to/config/file</param-value>
    </init-param>
  
If no parameter is set, it looks for a default `config-example.json` in the folder of the deployed WAR file.

## Status
This is software is still under development. It currently only supports:
- HDT data sources
- Turtle output

## Local setup with Java 21
The server has been checked with Java 21. From the workspace root, enter the SPF server folder and build the project:

```
cd spf-server
mvn clean package
```

This creates the standalone server jar at:

```
target/ldf-server.jar
```

The local configuration file `config.json` is set up for the data in the workspace-level `data` folder:

```
../data/dataset.hdt
```

In that config, the datasource is named `spf`. That name is the URL path you query. If you rename the datasource in the config, replace `/spf` in the URLs below with the new datasource name.

## Run the local server
Run the server from the `spf-server` folder:

```
java -jar target/ldf-server.jar config.json -p 8080
```

The server listens on all local interfaces for the selected port. Use `http://localhost:8080/...` from the same machine.
If port `8080` is already being used by another server, choose a free port with `-p` and replace `8080` in the URLs below with that port.

## Usage
To access the server, the following endpoints are available:

| URL | Expected result | Meaning |
|-----|-----------------|---------|
| `http://localhost:8080/` | `404` | The root path is not a query endpoint in this server. Query a configured datasource path instead. |
| `http://localhost:8080/spf` | `200` | The configured local HDT datasource. Without query parameters this returns the first Triple Pattern Fragment page for the whole dataset in Turtle. `spf` could be replaced with the actual name of the datasource in the configuration file. |
| `http://localhost:8080/spf?subject=&predicate=&object=` | `200` | Triple Pattern Fragment query. Empty `subject`, `predicate`, and `object` values mean variables. |
| `http://localhost:8080/spf?subject=http%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FUser0&predicate=&object=` | `200` | Triple Pattern Fragment query for triples about `User0`. |
| `http://localhost:8080/spf?subject=%3Fs&predicate=http%3A%2F%2Fschema.org%2Femail&object=%3Fo&values=(%3Fs)%20%7B%20(%3Chttp%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FUser0%3E)%20%7D` | `200` | Bindings-restricted Triple Pattern Fragment query. The `values` parameter switches the server to brTPF mode. |
| `http://localhost:8080/spf?s=http%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FUser0&triples=2&star=%5Bp1%2Chttp%3A%2F%2Fschema.org%2Femail%3Bo1%2C%3Femail%3Bp2%2Chttp%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FuserId%3Bo2%2C%3FuserId%5D` | `200` | Star Pattern Fragment query. The `triples` parameter switches the server to SPF mode, and `star` contains the predicate/object pairs. |

- `http://localhost:8080/spf`: the configured local HDT datasource. Without query parameters this returns the first Triple Pattern Fragment page for the whole dataset in Turtle.
- `http://localhost:8080/spf?subject=&predicate=&object=`: a Triple Pattern Fragment query. Empty `subject`, `predicate`, and `object` values mean variables.
- `http://localhost:8080/spf?subject=http%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FUser0&predicate=&object=`: a Triple Pattern Fragment query for triples about `User0`.
- `http://localhost:8080/spf?subject=%3Fs&predicate=http%3A%2F%2Fschema.org%2Femail&object=%3Fo&values=(%3Fs)%20%7B%20(%3Chttp%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FUser0%3E)%20%7D`: a bindings-restricted Triple Pattern Fragment query. The `values` parameter switches the server to brTPF mode.
- `http://localhost:8080/spf?s=http%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FUser0&triples=2&star=%5Bp1%2Chttp%3A%2F%2Fschema.org%2Femail%3Bo1%2C%3Femail%3Bp2%2Chttp%3A%2F%2Fdb.uwaterloo.ca%2F~galuc%2Fwsdbm%2FuserId%3Bo2%2C%3FuserId%5D`: a Star Pattern Fragment query. The `triples` parameter switches the server to SPF mode, and `star` contains the predicate/object pairs.

The server only registers Turtle output, so use an `Accept: text/turtle` header when querying with `curl`:

```
curl -H 'Accept: text/turtle' 'http://localhost:8080/spf?subject=&predicate=&object='
```

The root URL `http://localhost:8080/` is not the query endpoint. Query the configured datasource URL, for example `http://localhost:8080/spf`.
