## Entities query track

The Entities query track aims to measure the performance of queries generated from Entity definitions.
The track workflow is copied from the tsdb_k8s_queries, see the [README](../tsdb_k8s_queries/README.md) to get a better understanding of the operations.

### Adding a request

To add a query to benchmark, add an entry to the [operations file](./operations/default.json). The current dataset includes 1M documents with 200k entities (identified by `host.hostname`) spread over a 5 minutes period and stored in `k8s-pod` datastream. The dataset contains kubernetes pod metrics document with [this structure](https://github.com/elastic/elastic-integration-corpus-generator-tool/blob/main/assets/templates/kubernetes.pod/schema-b/gotext.tpl#L18).

You can use [the example query](./operations/default.json#L9-L14) to replicate the timestamp filters necessary to target the entire dataset.

Once the query created, add it to the bottom of the [challenges file](./challenges/default.json) with a block similar to the [example query](./challenges/default.json#L103-L122).

### Generation of data

New Corpora data can be generated with the help of [Elastic-integration-corpus-generator-tool How_to_Guide](https://github.com/elastic/observability-dev/blob/main/docs/infraobs/cloudnative-monitoring/dev-docs/elastic-generator-tool-with-rally.md).

Specific templates are implemented as part of the generator tool. Based on them, sample datasets needed for the rally track have already been generated and uploaded to public GCP bucket for reuse.

### Generation of Templates

Along with generation of data, a rally track might also need updated template files for the specific datasets that will index.

Follow below process to extract the latest index templates for specific package version:

1. Create local Elastic stack

```bash
elastic-package stack up -d -vvv --version=8.7.1
```

1. Login to Kibana (https://localhost:5601) and install specific integration. `Eg. Kubernetes Integration v1.39.1`
   The installation of package will install index templates in local Elasticsearch.

2. Export needed environmental variables

```bash
 elastic-package stack shellinit
 export ELASTIC_PACKAGE_ELASTICSEARCH_HOST=https://127.0.0.1:9200
 export ELASTIC_PACKAGE_ELASTICSEARCH_USERNAME=elastic
 export ELASTIC_PACKAGE_ELASTICSEARCH_PASSWORD=changeme
 export ELASTIC_PACKAGE_KIBANA_HOST=https://127.0.0.1:5601
 export ELASTIC_PACKAGE_CA_CERT=~/.elastic-package/profiles/default/certs/ca-cert.pem
 ```

 4. Use elastic-package dump command:

```bash
 elastic-package dump installed-objects --package kubernetes

 [output] ...
 packages exttracted to package-dump
 ```

5. Locate and use your needed index templates

```bash
 cd package-dump/index_templates
  - metrics-kubernetes.pod.json
  - metrics-kubernetes.container.json 
```

### Parameters

This track allows to overwrite the following parameters using `--track-params`:

* `bulk_size` (default: 9000)
* `touch_bulk_size` (default: 50): The bulk size for bulk requests executed during searching.
* `bulk_indexing_clients` (default: 8): Number of clients that issue bulk indexing requests.
* `touch_bulk_indexing_clients` (default: 3): Number of clients that issue bulk indexing for the bulk tasks that get executed during searching.
* `ingest_percentage` (default: 100): A number between 0 and 100 that defines how much of the document corpus should be ingested.
* `force_merge_max_num_segments` (default: unset): An integer specifying the max amount of segments the force-merge operation should use.
* `number_of_replicas` (default: 0)
* `number_of_shards` (default: 1)
* `post_ingest_sleep` (default: false): Whether to pause after ingest and prior to subsequent operations.
* `post_ingest_sleep_duration` (default: 30): Sleep duration in seconds.
