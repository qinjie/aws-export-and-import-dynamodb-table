# AWS Export and Import DynamoDB Table



Exporting and importing DynamoDB Data is used to be a painful process using AWS Data Pipeline. Here is the [official guide](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/DynamoDBPipeline.html).



AWS has recently added an long-awaited feature to export DynamoDB data to S3 bucket without going through Data Pipeline, which was announced in [this blog post](https://aws.amazon.com/blogs/aws/new-export-amazon-dynamodb-table-data-to-data-lake-amazon-s3/). 



## Import and Export using AWS CLI

Following commands uses AWS CLI and jq to export and import dynmodb tables.
* Download [jq](https://stedolan.github.io/jq/) and add it to system path
* Install AWS CLI and setup AWS profile
* If AWS CLI profile is not the default profile, set the AWS_PROFILE environment variable, `set AWS_PROFILE=xxx` for Windows or `export AWS_PROFILE=xxx` for Linux.



### Export DynamoDB table to JSON file

Change AWS_PROFILE.

```
SET AWS_PROFILE=default
```

Or add `--endpoint-url http://localhost:8000` to export from local database.

Following script scan the table, transform the output of the scan to shape into the format of the batchWriteItem and dump the result into a file.

```
aws dynamodb scan --table-name url_shortener | jq "{\"url_shortener\": [.Items[] | {PutRequest: {Item: .}}]}" > url_shortener.json
aws dynamodb scan --table-name whitespace_users | jq "{\"whitespace_users\": [.Items[] | {PutRequest: {Item: .}}]}" > whitespace_users.json
aws dynamodb scan --table-name whitespace_settings | jq "{\"whitespace_settings\": [.Items[] | {PutRequest: {Item: .}}]}" > whitespace_settings.json
```
### Import JSON into DynamoDB table

Change AWS_PROFILE.

```
SET AWS_PROFILE=capdev
```

Add `--endpoint-url http://localhost:8000` to import into local database.

```
aws dynamodb batch-write-item --request-items file://url_shortener.json
aws dynamodb batch-write-item --request-items file://whitespace_users.json
aws dynamodb batch-write-item --request-items file://whitespace_settings.json
```



## Export Table using Management Console

<img src="https://raw.githubusercontent.com/qinjie/picgo-images/main/image-20211105164510225.png" alt="image-20211105164510225" style="zoom:67%;" />

The exported data is saved in a subfolder in `s3://<BUCKET>/AWSDynamoDB/`. Folder name is a generated string and can be found in the Export details, which is not related to table name.

<img src="https://raw.githubusercontent.com/qinjie/picgo-images/main/image-20211105165851147.png" alt="image-20211105165851147" style="zoom: 67%;" />



