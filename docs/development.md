# Dev setup
Clone the code repositories:

```shell
git clone https://github.com/hoover/docker-setup
git clone https://github.com/hoover/snoop2
git clone https://github.com/hoover/search
git clone https://github.com/hoover/ui
```

When creating collections or updating settings use the `-d` option. E.g.:
```shell
./createcollection -c <collection_name> -d
./updatesettings -d <collection_names_list>
```

On `createcollection` the `-d` option will add development settings for the new
collection. On `updatesettings` the option `-d` will add development settings
to the collections in the list. E.g. `-d collection1 collection2`. If the list
was empty it will add development settings to all existing collections.

It will generate the following code in the docker-compose.override.yml:
```yaml
  snoop-rabbitmq:
    ports:
      - "5672:5672"

  snoop-tika:
    ports:
      - "9998:9998"

  search-pg:
    ports:
      - "5432:5432"

  search-es:
    ports:
      - "9200:9200"

  ui:
    volumes:
      - ../ui:/opt/hoover/ui:cached

  search:
    volumes:
      - ../search:/opt/hoover/search:cached
```

For each collection it will add the following setting:
```yaml
  snoop-pg--testdata:
    ports:
      - "5433:5432"
  snoop-worker--testdata:
    volumes:
      - ../snoop2:/opt/hoover/snoop:cached
  snoop--testdata:
    volumes:
      - ../snoop2:/opt/hoover/snoop:cached
```

This will mount the code repositories inside the docker containers to run the
local development code.

To remove development from a list of collections use the following command:
```shell
./updatesettings -r <collection_names_list>
```

Leave the collection list empty to remove development for all collections.

# Docker images
Docker-hub builds images based on the Hoover GitHub repos triggered by pushes
to the master branches: [snoop2][], [search][], [ui][].

[snoop2]: https://hub.docker.com/r/liquidinvestigations/hoover-snoop2/
[search]: https://hub.docker.com/r/liquidinvestigations/hoover-search/
[ui]: https://hub.docker.com/r/liquidinvestigations/hoover-ui/

You can also build images locally. For example, the snoop2 image:

```shell
cd snoop2
docker build . --tag snoop2
```

Then add this snippet to `docker-compose.override.yml` to test the image
locally, and run `docker-compose up -d` to (re)start the containers:

```yaml
version: "2"

services:

  snoop-worker:
    image: snoop2

  snoop:
    image: snoop2
```

### Testing
For Snoop and Search tests based on [pytest][] can be executed using this commands:

[pytest]: https://docs.pytest.org/en/latest/

```shell
docker-compose run --rm snoop pytest
docker-compose run --rm search pytest
```

The test definitions can be found in the `testsuite` folder of each project. Individual tests can be started
using:

```shell
docker-compose run --rm snoop pytest testsuite/test_tika.py
```