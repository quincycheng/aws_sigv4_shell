# Generate AWS sigv4 with shell script


Generating AWS Sigv4 signatures is a crucial step in securing and authenticating requests made to AWS services. In this blog, we will explore how to generate a Sigv4 signature using a Linux Shell script.

AWS Sigv4 signature is generated using several components, including the access key, secret key, region, service, and the request being made. The signature is generated using the HMAC-SHA256 algorithm and is attached to the request as an Authorization header.

Here is a sample Shell script that demonstrates how to generate an AWS Sigv4 signature:

```shell
#!/bin/bash

# AWS Access Key ID
access_key="YOUR_ACCESS_KEY_ID"

# AWS Secret Access Key
secret_key="YOUR_SECRET_ACCESS_KEY"

# Region of the AWS service being accessed
region="us-east-1"

# AWS Service being accessed
service="s3"

# Date in the format YYYYMMDD
date=$(date -u +'%Y%m%d')

# Time in the format YYYYMMDDTHHMMSSZ
time=$(date -u +'%Y%m%dT%H%M%SZ')

# HTTP request method
http_method="GET"

# Endpoint of the AWS service being accessed
endpoint="https://YOUR_BUCKET_NAME.s3.amazonaws.com"

# Canonical Request
canonical_request="${http_method}\n/\n\nhost:${service}.${region}.amazonaws.com\nx-amz-content-sha256:UNSIGNED-PAYLOAD\nx-amz-date:${time}\n\nhost;x-amz-content-sha256;x-amz-date\nUNSIGNED-PAYLOAD"

# String to sign
string_to_sign="AWS4-HMAC-SHA256\n${time}\n${date}/${region}/${service}/aws4_request\n$(echo -n "${canonical_request}" | sha256sum | awk '{print $1}')"

# Signing key
k_secret="AWS4${secret_key}"
k_date=$(echo -n "${date}" | openssl dgst -sha256 -hex -mac HMAC -macopt "key:${k_secret}" | awk '{print $2}')
k_region=$(echo -n "${region}" | openssl dgst -sha256 -hex -mac HMAC -macopt "key:${k_date}" | awk '{print $2}')
k_service=$(echo -n "${service}" | openssl dgst -sha256 -hex -mac HMAC -macopt "key:${k_region}" | awk '{print $2}')
k_signing=$(echo -n "aws4_request" | openssl dgst -sha256 -hex -mac HMAC -macopt "key:${k_service}" | awk '{print $2}')
signature=$(echo -n "${string_to_sign}" | openssl dgst -sha256 -hex -mac HMAC -macopt "key:${k_signing}" | awk '{print $2}')

# Authorization header
authorization_header="AWS4-HMAC-SHA256 Credential=${access_key}/${date}/${region}/${service}/aws4_request, SignedHeaders=host;x-amz-content-sha256;x-amz-date, Signature=${signature}"

# Make the request
curl -v -H "Authorization: ${authorization_header}" -H "x-amz-content-sha256: UNSIGNED-PAYLOAD" -H "x-amz-date: ${time}" "${endpoint}"
```

This script generates a Sigv4 signature for a GET request to an S3 bucket. The script can be customized by replacing the access key, secret key, region, service, and endpoint with the appropriate values for the AWS service being accessed.

In conclusion, generating an AWS Sigv4 signature using a Linux Shell script is an essential step in securing and authenticating requests made to AWS services. With the sample script provided in this blog, you can easily generate Sigv4 signatures for your AWS requests.
