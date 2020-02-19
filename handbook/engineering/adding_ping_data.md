# Adding or changing pings

This page outlines the process for adding or changing the data collected from Sourcegraph instances through pings.

## Ping philosophy

Pings are the only data Sourcegraph receives from installations. Our users and customers trust us with their most sensitive data. We must preserve and build this trust through only careful additions and changes to pings.

All ping data must be:

- Anonymous (with only one exception—the email address of the initial site installer)
- Aggregated (e.g. number of times a search filter was used per day, instead of the actual search queries)
- Non-specific (e.g. no repo names, no usernames, no file names, no specific search queries, etc.)

## Adding data to pings

Treat adding new data to pings as having a very high bar. Would you be willing to send an email to all Sourcegraph users explaining and justifying why we need to collect this additional data from them? If not, don’t propose it.

1. Write an RFC describing the problem, data that will be added, and how Sourcegraph will use the data to make decisions. The Business Operations team must be a required reviewer (both @Dan and @EricBM). 
You will be asked the following questions:
    - Why was this particular metric/data chosen?
    - What business problem does collecting this address?
    - What specific product or engineering decisions will be made by having this data?
    - Will this data be needed from every single installation, or only from a select few? Will it be needed forever, or only for a short time?
    - Have you considered alternatives? E.g., collecting this data from Sourcegraph.com, or adding a report for admins that we can request from some number of friendly customers?
2. When the RFC is approved, use the [life of a ping documentation](https://docs.sourcegraph.com/dev/architecture/life-of-a-ping) and [an example PR](https://github.com/sourcegraph/sourcegraph/pull/8374) to implement the change. At least one member of the Business Operations team must approve the resulting PR before it can be merged.
    - Ensure a CHANGELOG entry is added, and that the two sources of truth for ping data are updated along with your PR:
      - Pings documentation: https://docs.sourcegraph.com/admin/pings
      - The Site-admin > Pings page, e.g.: https://sourcegraph.com/site-admin/pings
3. Determine if any transformations/ETL jobs are required, and if so, add them to the [script](https://console.cloud.google.com/storage/browser/_details/sg-analytics-data/dataflow/pipelines/udf/transform.js?project=telligentsourcegraph&authuser=0&angularJsUrl=%2Fstorage%2Fbrowser%2F_details%2Fsg-analytics-data%2Fdataflow%2Fpipelines%2Fudf%2Ftransform.js%3Fproject%3Dtelligentsourcegraph%26authuser%3D1).
4. Open a PR to change [the schema](https://github.com/sourcegraph/analytics/tree/master/BigQuery%20Schemas) with Business Operations (EricB and Dan) as approvers. Keep in mind:
	- Check the data types sent in the JSON match up with the BigQuery schema (e.g. a JSON '1' will not match up with a BigQuery integer). 
	- Every field in the BigQuery schema should not be non-nullable (i.e. `"mode": "NULLABLE"` and `"mode": "REPEATED"` are acceptable). There will be instances on the older Sourcegraph versions that will not be sending new data fields, and this will cause pings to fail.


## Changing the BigQuery schema

Commands:
- To update schema: `bq --project_id=$PROJECT update --schema $SCHEMA_FILE $DATASET.$TABLE`, replacing `$PROJECT` with the project ID, `$SCHEMA_FILE` with the path to the schema JSON file generated above, and `$DATASET.$TABLE` with the dataset and table name, separated by a dot. 
- To retrieve the current schema : `bq --project_id=$PROJECT --format=prettyjson show $DATASET.$TABLE > schema.json` with the same replacements as above. 

To update the schema: 
1. Run the update schema command on a test table.
2. Once the test is complete, run the update schema command on the production table. 
