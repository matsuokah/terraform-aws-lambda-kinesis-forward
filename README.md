# AWS %description% Terraform module

Terraform module and Lambda for saving JSON log records from Kinesis Data Streams to S3.

![terraform v0.11.x](https://img.shields.io/badge/terraform-v0.11.x-brightgreen.svg)

## Prerequisites
1. Records in Kinesis stream must be valid JSON data. Non-JSON data will be **ignored**.
    1. gzipped JSON, [CloudWatch Logs subscription filters log format](https://docs.aws.amazon.com/ja_jp/AmazonCloudWatch/latest/logs/SubscriptionFilters.html) are supported.
    2. Logs without either of necessary keys listed below will be saved as `unknown` as well.
2. JSON data must have the following keys (key names are modifiable via variables):
    1. `log_type`: Log type identifier. Used for applying log type whitelist 
3. Recommended keys (necessary if target stream has [lambda-kinesis-to-s3](https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-to-s3) or other modules attached):
    1. `log_id`: Any unique identifier. Used to avoid file overwrites on S3. Also is useful to search for a specific log record.
    2. `time`: Any timestamp supported by [dateutil.parser.parse](https://dateutil.readthedocs.io/en/stable/parser.html#dateutil.parser.parse). ISO8601 with milli/microseconds recommended.

## Usage
```HCL
resource "aws_kinesis_stream" "stream" {
  name             = "stream"
  shard_count      = "1"
  retention_period = "24"
}

resource "aws_kinesis_stream" "target" {
  name             = "target"
  shard_count      = "1"
  retention_period = "24"
}

module "kinesis_forward" {
  source  = "baikonur-oss/lambda-kinesis-forward/aws"

  lambda_package_url = "https://github.com/baikonur-oss/terraform-aws-lambda-kinesis-forward/releases/download/v1.0.0/lambda_package.zip"
  name               = "kinesis_forward"

  memory     = "1024"
  batch_size = "100"

  kinesis_stream_arn = "${aws_kinesis_stream.stream.arn}"
  target_stream_name = "${aws_kinesis_stream.target.name}"
}

```

Warning: use same module and package versions!

### Version pinning
#### Terraform Module Registry
Use `version` parameter to pin to a specific version, or to specify a version constraint when pulling from [Terraform Module Registry](https://registry.terraform.io) (`source = baikonur-oss/lambda-kinesis-forward/aws`).
For more information, refer to [Module Versions](https://www.terraform.io/docs/configuration/modules.html#module-versions) section of Terraform Modules documentation.

#### GitHub URI
Make sure to use `?ref=` version pinning in module source URI when pulling from GitHub.
Pulling from GitHub is especially useful for development, as you can pin to a specific branch, tag or commit hash.
Example: `source = github.com/baikonur-oss/terraform-aws-lambda-kinesis-forward?ref=v1.0.0`

For more information on module version pinning, see [Selecting a Revision](https://www.terraform.io/docs/modules/sources.html#selecting-a-revision) section of Terraform Modules documentation.


<!-- Documentation below is generated by pre-commit, do not overwrite manually -->
<!-- BEGINNING OF PRE-COMMIT-TERRAFORM DOCS HOOK -->
## Inputs

| Name | Description | Type | Default | Required |
|------|-------------|:----:|:-----:|:-----:|
| batch\_size | Maximum number of records passed for a single Lambda invocation | string | n/a | yes |
| failed\_log\_s3\_bucket | S3 bucket name for saving failed logs (ES API errors etc.) | string | n/a | yes |
| failed\_log\_s3\_prefix | Path prefix for failed logs | string | n/a | yes |
| handler | Lambda Function handler (entrypoint) | string | `"main.handler"` | no |
| lambda\_package\_url | Lambda package URL (see Usage in README) | string | n/a | yes |
| log\_bucket | Target S3 bucket to save data to | string | n/a | yes |
| log\_id\_field | Key name for unique log ID | string | `"log_id"` | no |
| log\_path\_prefix | Log file path prefix | string | n/a | yes |
| log\_retention\_in\_days | Lambda Function log retention in days | string | `"30"` | no |
| log\_timestamp\_field | Key name for log timestamp | string | `"time"` | no |
| log\_type\_field | Key name for log type | string | `"log_type"` | no |
| log\_type\_field\_whitelist | Log type whitelist (if empty, all types will be processed) | list | `<list>` | no |
| log\_type\_unknown\_prefix | Log type prefix for logs without log type field | string | `"unknown"` | no |
| memory | Lambda Function memory in megabytes | string | `"256"` | no |
| name | Resource name | string | n/a | yes |
| runtime | Lambda Function runtime | string | `"python3.7"` | no |
| source\_stream\_name | Source Kinesis Data Stream name | string | n/a | yes |
| starting\_position | Kinesis ShardIterator type (see: https://docs.aws.amazon.com/kinesis/latest/APIReference/API_GetShardIterator.html ) | string | `"TRIM_HORIZON"` | no |
| tags | Tags for Lambda Function | map | `<map>` | no |
| target\_stream\_name | Target Kinesis Data Stream name | string | n/a | yes |
| timeout | Lambda Function timeout in seconds | string | `"60"` | no |
| timezone | tz database timezone name (e.g. Asia/Tokyo) | string | `"UTC"` | no |
| tracing\_mode | X-Ray tracing mode (see: https://docs.aws.amazon.com/lambda/latest/dg/API_TracingConfig.html ) | string | `"PassThrough"` | no |

<!-- END OF PRE-COMMIT-TERRAFORM DOCS HOOK -->

## Contributing

Make sure to have following tools installed:
- [Terraform](https://www.terraform.io/)
- [terraform-docs](https://github.com/segmentio/terraform-docs)
- [pre-commit](https://pre-commit.com/)

### macOS
```bash
brew install pre-commit terraform terraform-docs

# set up pre-commit hooks by running below command in repository root
pre-commit install
```
