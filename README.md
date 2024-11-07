**Convert Apache Access Logs into Open Cybersecurity Schema Framework (OCSF) Schema**

This solution demonstrates the steps required to convert Apache Access logs into OCSF Parquet format.  The converted files can be stored in an Amazon S3 Bucket. Amazon Security Lake Consumers will use OCSF logs from the s3 bucket for downstream analytical processes. 

This document will show how you can convert Apache Access logs to OCSF format using python scripts before sending it to the S3 bucket backed by Amazon Security Lake

**Open Cybersecurity Schema Framework (OCSF)**

The Open Cybersecurity Schema Framework is an open-source project that delivers an extensible framework for developing schemas and a vendor-agnostic core security schema. Vendors and other data producers can adopt and extend the schema for their specific domains. Data engineers can map differing schemas to help security teams simplify data ingestion and normalization so that data scientists and analysts can work with a common language for threat detection and investigation. The goal is to provide an open standard adopted in any environment, application, or solution while complementing existing security standards and processes. You can find more information here: https://github.com/ocsf/

**Configure Apache to Write logs in Custom JSON Format**

**Method 1:**

To configure Apache to write access logs in a custom JSON format directly, you can use the `CustomLog` directive along with the `mod_log_config` module. This involves modifying your Apache configuration file (usually `httpd.conf` or `apache2.conf`). Below are the steps to achieve this:

*Step 1: Enable mod_log_config*

Ensure that the `mod_log_config` module is enabled in your Apache configuration. This module provides the necessary functionality to customize log formats.

apache
LoadModule log_config_module modules/mod_log_config.so


You can typically find this line in your Apache configuration file. Make sure it's uncommented (i.e., remove any '#' character at the beginning of the line) or add it if it's missing. Afterward, restart Apache to apply the changes.

*Step 2: Define Your Custom Log Format*

In the same configuration file, define your custom log format using the `LogFormat` directive. This format will specify how the log entries should be written in JSON format. For example:

Apache

LogFormat "{ \"Time\":%{%s}t, \"RemoteIP\":\"%a\", \"Host\":\"%V\", \"Port\":\"%p\", \"Request\":\"%U\", \"Query\":\"%q\", \"File\":\"%f\", \"Method\":\"%m\", \"Status\":\"%s\", \"UserAgent\":\"%{User-agent}i\", \"Referer\":\"%{Referer}i\" }" jsonlog



In this example, we're creating a custom JSON format with specific fields like timestamp, IP address, request method, URL, response code, user agent, and referrer. You can customize this format to include additional fields as needed.

*Step 3: Configure Custom Log File*

Next, configure the `CustomLog` directive to use your custom log format and specify the log file where the JSON-formatted logs should be written. Replace `path/to/custom/access.log` with the actual path to your log file.

Apache

CustomLog "/var/log/apache2/access-json.log" jsonlog


*Step 4: Restart Apache*

After making these changes, save your Apache configuration file and restart Apache to apply the new log format.

bash
sudo service apache2 restart


Now, Apache will log access entries in the specified JSON format to the custom log file you configured. You can access these logs for analysis, parsing, or further processing in your applications.


**Method 2:**

Customers may express concern about making modifications to the Apache configuration files on their operating systems, particularly those who adhere to an organizational policy governing the use of hardened OS images. To facilitate the conversion of the default Apache access logs into a customized JSON format, we have provided a Python script, "apache2json.py," within this repository.
Depending on your specific requirements, you have the flexibility to tailor the following attributes to execute the script successfully:

python

Define the path to your existing Apache access log file

input_log_path = 'access_log'

Specify the directory where you intend to store the resulting JSON output

output_json_path = 'access_log_n.json'

Utilize the regular expression pattern below to parse Apache log entries

log_pattern = r'(?P<ip>[\d\.]+) - - \[(?P<time>[^\]]+)\] "(?P<request>[^"]+)" (?P<status>\d+) (?P<bytes_sent>\d+)'

To execute the script, simply run the following command:

bash

python3 apache2json.py

Upon execution, the converted JSON file will be saved in the directory specified within the script. This JSON file can subsequently be used for the purpose of converting to the OCSF schema Apache Parquet format.
Converting JSON to OCSF Schema Apache Parquet Format 

In the preceding step, the Apache access logs are transformed into JSON format as show below

Eg.., 
{ "Time":1693658106, "RemoteIP":"171.67.70.84", "Host":"52.66.212.90", "Port":"80", "Request":"/", "Query":"", "File":"/usr/share/httpd/noindex/index.html", "Method":"GET", "Status":"403", "UserAgent":"Mozilla/5.0 zgrab/0.x", "Referer":"-" }
{ "Time":1693658345, "RemoteIP":"139.162.84.205", "Host":"ip-172-31-34-4.ap-south-1.compute.internal", "Port":"80", "Request":"/", "Query":"", "File":"/usr/share/httpd/noindex/index.html", "Method":"GET", "Status":"403", "UserAgent":"Mozilla/5.0 (X11; Linux x86_64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/109.0.0.0 Safari/537.36", "Referer":"-" }


To convert these JSON logs into the Apache Parquet format compliant with the OCSF (Open Cybersecurity Schema Format) schema, you can execute the provided Python script available in this repository.

Please run the following command to initiate the conversion process:

bash

python3 json2ocsf.py

It is essential to ensure that you adjust the source file path according to the location of your JSON logs. The "json2ocsf.py" Python file can be tailored to meet your specific log format or other unique requirements. For illustrative purposes, Apache Access logs are used as an example, and reference is made to the OCSF schema for the purpose of restructuring the JSON file. (https://schema.ocsf.io/1.0.0/classes/http_activity)

The resulting output will be provided in a Parquet format that conforms to the OCSF schema. This Parquet file can be conveniently transferred to the custom log data source S3 bucket associate with your security lake. Subsequently, you have the option to manually construct the table structure or utilize the Glue crawler to automate schema generation.

Following this, manual data refreshing and immediate querying using Athena can be performed, enabling swift access to your transformed log data.

For detailed instructions on establishing a custom log source within your security lake, 

Please refer to the following links:

https://docs.aws.amazon.com/security-lake/latest/userguide/custom-sources.html

**References:**

https://aws.amazon.com/security-lake/
https://schema.ocsf.io/1.0.0/classes/http_activity

