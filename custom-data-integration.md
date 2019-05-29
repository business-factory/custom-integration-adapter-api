## Custom Data Integrations

Custom Data Integrations with ROI Hunter can be done either via an API (JSON is prefered) or via FTP/SFTP (csv is prefered). This document describes the requirements and what the data can look like.

### Requirements

To match ROI hunter ad/adset/campaign data with custom data, the custom data needs to contain at least one column that can be used to match the data with ROI Hunter data. Currently, there are two options:

- FB AD ID
- ROI Hunter AD ID (this is, by default, passed by utm parameters)

If neither of those columns are available, some other options must be explored.

### Integration via FTP/SFTP

The way this is done is via a cron that continously checks FTP/SFTP (can be created by either a third party or by us) for new or modified files. If there is a new or a modified file, the entire file is parsed, matched to our ROI hunter Ads and saved. Subsequently, the data is aggregated (if possible, some columns can't be aggregated) into upper layers (Adset, campaign). An example of such a file follows:

```
fb_id,transactions,revenues,currency,views
23489034809893,50,1000.0,USD,1000
2533276472642387,66,1500.8,EUR,1294
234238789891234,2348,8259853.78,USD,234890
```

Note: The file can be also provided via HTTP.

### Integration via API

Integration via API is more complicated. First, specification is required, both for the authentication and for the way the data can be retrieved. We also need specification of how often we should refresh the data. The prefered API is HTTP REST + JSON. 
