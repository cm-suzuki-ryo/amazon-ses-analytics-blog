#  Sample SQL to retrieve SES bounce events

- [Analyzing Amazon SES event data with AWS Analytics Services](https://aws.amazon.com/blogs/messaging-and-targeting/analyzing-amazon-ses-event-data-with-aws-analytics-services/)



## Partition not used

### sesmaster (Bounce Only)

```
CREATE EXTERNAL TABLE sesmaster_bounce (
eventType string,
bounce struct < bouncedrecipients: array < struct < action: string,
diagnosticcode: string,
emailaddress: string,
status: string >>,
bouncesubtype: string,
bouncetype: string,
feedbackid: string,
reportingmta: string,
`timestamp`: string >,
mail struct < timestamp: string,
source: string,
sourcearn: string,
sendingaccountid: string,
messageid: string,
destination: string,
headerstruncated: boolean,
headers: array < struct < name: string,
value: string >>,
commonheaders: struct < `from`: array < string >,
to: array < string >,
messageid: string,
subject: string >,
tags: struct < ses_source_tls_version: string,
ses_operation: string,
ses_configurationset: string,
ses_source_ip: string,
ses_outgoing_ip: string,
ses_from_domain: string,
ses_caller_identity: string >>
)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES (
"mapping.ses_caller_identity" = "ses:caller-identity",
"mapping.ses_configurationset" = "ses:configuration-set",
"mapping.ses_from_domain" = "ses:from-domain",
"mapping.ses_operation" = "ses:opeation",
"mapping.ses_outgoing_ip" = "ses:outgoing-ip",
"mapping.ses_source_ip" = "ses:source-ip",
"mapping.ses_source_tls_version" = "ses:source-tls-version"
)
LOCATION 's3://aws-s3-ses-analytics-<aws-account-number>/partitioned/<Identity>/Bounce/'
```


```
select * from sesmaster_bounce limit 10
```


## Partition used

### sesmaster (partition-projection)

```
CREATE EXTERNAL TABLE sesmaster_partition (
eventType string,
complaint struct < arrivaldate: string,
complainedrecipients: array < struct < emailaddress: string >>,
complaintfeedbacktype: string,
feedbackid: string,
`timestamp`: string,
useragent: string >,
bounce struct < bouncedrecipients: array < struct < action: string,
diagnosticcode: string,
emailaddress: string,
status: string >>,
bouncesubtype: string,
bouncetype: string,
feedbackid: string,
reportingmta: string,
`timestamp`: string >,
mail struct < timestamp: string,
source: string,
sourcearn: string,
sendingaccountid: string,
messageid: string,
destination: string,
headerstruncated: boolean,
headers: array < struct < name: string,
value: string >>,
commonheaders: struct < `from`: array < string >,
to: array < string >,
messageid: string,
subject: string >,
tags: struct < ses_source_tls_version: string,
ses_operation: string,
ses_configurationset: string,
ses_source_ip: string,
ses_outgoing_ip: string,
ses_from_domain: string,
ses_caller_identity: string >>,
send string,
delivery struct < processingtimemillis: int,
recipients: array < string >,
reportingmta: string,
smtpresponse: string,
`timestamp`: string >,
open struct < ipaddress: string,
`timestamp`: string,
userAgent: string >,
reject struct < reason: string >,
click struct < ipAddress: string,
`timestamp`: string,
userAgent: string,
link: string >
)
PARTITIONED BY (sender_identity string, event_type string, day string)
ROW FORMAT SERDE 'org.openx.data.jsonserde.JsonSerDe'
WITH SERDEPROPERTIES (
"mapping.ses_caller_identity" = "ses:caller-identity",
"mapping.ses_configurationset" = "ses:configuration-set",
"mapping.ses_from_domain" = "ses:from-domain",
"mapping.ses_operation" = "ses:opeation",
"mapping.ses_outgoing_ip" = "ses:outgoing-ip",
"mapping.ses_source_ip" = "ses:source-ip",
"mapping.ses_source_tls_version" = "ses:source-tls-version"
)
LOCATION 's3://aws-s3-ses-analytics-<aws-account-number>/partitioned/'
TBLPROPERTIES (
 "projection.enabled" = "true",
 "projection.sender_identity.type" = "injected",
 "projection.event_type.type" = "enum",
 "projection.event_type.values" = "Bounce,Complaint,Delivery,Send,Reject,Open,Click,`Rendering Failure`,DeliveryDelay,Subscription",
 "projection.day.type" = "date",
 "projection.day.format" = "yyyy/MM/dd",
 'projection.day.range' = 'NOW-40DAYS,NOW',
 "projection.day.interval" = "1",
 "projection.day.interval.unit" = "DAYS",
 "storage.location.template" = "s3://aws-s3-ses-analytics-<aws-account-number>/partitioned/${sender_identity}/${event_type}/${day}/"
)   
```


```
SELECT * FROM sesmaster_partition
where  sender_identity ='<Domain>'
and event_type = 'Bounce'
and day ='2024/02/16'
```

```
SELECT
  bounce.bounceType as bouncebounceType
, bounce.bouncesubtype as bouncebouncesubtype
, bounce.feedbackid as bouncefeedbackid
, bounce.timestamp as bouncetimestamp
, bounce.reportingMTA as bouncereportingmta
, mail.messageId as mailmessageid
, mail.timestamp as mailtimestamp
, mail.source as mailsource
, mail.sendingAccountId as mailsendingAccountId
, mail.commonHeaders.subject as mailsubject
, mail.tags.ses_configurationset as mailses_configurationset
, mail.tags.ses_source_ip as mailses_source_ip
, mail.tags.ses_from_domain as mailses_from_domain
, mail.tags.ses_outgoing_ip as mailses_outgoing_ip
FROM sesmaster_partitione
where sender_identity ='<Domain>'
and event_type = 'Bounce'
and day = '2024/02/16'
```




