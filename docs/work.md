# Requesting and updating work records

Work records in Lamplight are events, sessions - things that happen at a time and place that people attend.
You can use the API to list these on your website, and add attendees to existing records.

## Requesting some records directly

Use the endpoint:

```
https://lamplight.online/api/work/some/
```

with any (or none) of the following parameters:

| Param | Type | Notes |
| --- | --- | --- |
| num | int | Number of records (optional: default is same as global settings for 'rows per page') |
| start | int | Offset to start from (optional: default = 0) |
| order | string | Field to order by. Must be a publishable field. The default is to return records sorted in ascending date order. |
| q | string | Search string. All publishable fields are searched. |
| after | string | Date string in YYYY-MM-DD format. Will return records with date on or after that given. |
| before | string | Date string in YYYY-MM-DD format. Will return records with date on or before that given. |
| workarea | int | StringID or comma-separated list of workarea IDs to filter by. Workareas and their IDs may be requested separately (see requesting workarea).|


### Return data

```json
{
  "data": [
    {
      "id": 1,
      "title": "This is a test record",
      "start_date": "2022-01-02 14:00:00",
      "end_date": "2022-01-02 15:30:00",
      "workarea": ["Drop-in"],
      "may_add_attend": true,
      "num_users_attending": 4,
      "maximum_num_users_allowed": 10
    }
  ],
  "meta": {
    "numRecords": 1,
    "totalRecords": 20
  }
}
```

The `num_users_attending` and `maximum_num_users_allowed` fields will be added if the record has been set to allow
people to add via the API.



### Fetching some work records using the php client

```php

// Set up the Client - $api_credentials defined elsewhere with keys 'lampid', 'project', 'key'
$client = new \Lamplight\Client(null, $api_credentials);
// Fetch work records starting after 1st Jan 2022
$response = $client->fetchWork()->fetchSome()->setParameterGet('after', '2022-01-01')->request();

if ($response->isSuccessful()) {
    // Ask the client for the RecordSet, with the data parsed from the response
    $work_records = $client->getRecordSet();
    // Render each with a template:
    $template = '<span class="date">{start_date}</span><div>{title}</div>';
    echo $work_records->render($template, "\n");
} else {
    // handle the error
}
```

will output

```html 
<span class="date">2022-01-02 14:00:00</span><div>This is a test record</div>
```


## Requesting one record

Requesting a single record gives more data than when requesting `some`.  Use the endpoint:

```
https://lamplight.online/api/work/one/
```

with the following parameter:

| Param    | Type | Notes                                                                                                                                         |
|----------| --- |-----------------------------------------------------------------------------------------------------------------------------------------------|
| id       | int | ID of the record                                                                                                                              |


### Return data

```json
{
  "data": [
    {
      "id": 1,
      "title": "This is a test record",
      "start_date": "2022-01-02 14:00:00",
      "end_date": "2022-01-02 15:30:00",
      "workarea": ["Drop-in", "Benefits advice"],
      "workareaText": "Drop-in",
      "subWorkareas": ["Benefits advice"],
      "description" : "Text from the record, if settings allow",
      "summary" : "Text from the record, if settings allow",
      "followup" : "Text from the record, if settings allow",
      "location": ["Cafe"],
      "location_full_details": [
        {
          "name": "Cafe",
          "address_line_1": "123 Test Road",
          "address_line_2": "Liverpool",
          "address_line_3": "etc",
          "address_line_4": "",
          "address_line_5": "",
          "postcode": "L22 0PJ",
          "description": "Cozy cafe with cracking coffee",
          "website": "",
          "disabled_access": "Fully acccessible",
          "contact_phone": ""
        }
      ],
      "may_add_atted": true,
      "num_users_attending": 4,
      "maximum_num_users_allowed": 10
    }
  ],
  "meta": {
    "numRecords": 1,
    "totalRecords": 20
  }
}
```

The `num_users_attending` and `maximum_num_users_allowed` fields will be added if the record has been set to allow
people to add via the API.  Other additional fields are provided if enabled in publishing settings in system admin.

Each record may also include published custom fields, which will be formatted as:

```json 
{
 "Name_of_field_in_Lamplight_with_underscores": "the value",
 "Multi_select_fields_return_array": ["green", "blue"]
}
```



### Fetching some work records using the php client

```php

// Set up the Client - $api_credentials defined elsewhere with keys 'lampid', 'project', 'key'
$client = new \Lamplight\Client(null, $api_credentials);
// Fetch work records starting after 1st Jan 2022
$response = $client->fetchWork()->fetchOne(1234)->request();

if ($response->isSuccessful()) {
    // Ask the client for the RecordSet, with the data parsed from the response
    $work_records = $client->getRecordSet();
    // Render each with a template:
    $template = '<span class="date">{start_date}</span>
        <div><h2>{title}</h2><p>{description}</p></div>';
    echo $work_records->render($template, "\n");
} else {
    // handle the error
}
```

will output

```html 
<span class="date">2022-01-02 14:00:00</span>
    <div><h2>This is a test record</h2><p>Text from the record, if settings allow</p></div>
```

### Fetching all work records.

You cannot request all record through the API. Use `some`.



## Adding attendees to work records

You can add attendees to work records by POSTing data to `https://lamplight.online/api/work/attend/`.
You will need the Lamplight IDs of the records, and an identifier for the attendee - either an email address or an ID.

There are separate configuration settings within Lamplight to enable this.

Send the data in your POST request:

| Param    | Type          | Notes                                                                             |
|----------|---------------|-----------------------------------------------------------------------------------|
| id       | int or string | Required: ID of the work record, or a comma-separated list of IDs                 |
| attendee | int or string | Required: ID of the profile, or email address of the profile already in Lamplight |
| attendee_notes | string | Optional text added to the 'notes' on the attendance in Lamplight |


Lamplight will try and add the attendee to the record, and provide a response for each indicating whether 
it was successful.  So a request to add someone to work record IDs 123 and 456 would return this:

```json
{
  "data": [
    {
      "id":123,
      "attend":true
    },
    {
      "id":456,
      "attend":false
    }
  ]
}

```

There are a number of reasons this may fail:
 - settings do not allow it
 - the particular work record does not allow adding attendees via the API
 - the particular work record is full - the number of users attending is greater than or equal to the maximum set for the record
 - with email addresses, the attendee can't be identified unambiguously - either the email isn't found, or more than one record was found with that email


### Adding attendees using the php client

To add using the client, first create a `Lamplight\Record\Work` record and then `save` it with the client:

```php

// create the record
$record = new \Lamplight\Record\Work(['id' => 123]);
// the attendee can either be an email address or a Lamplight profile ID
$record->setAttendee('testing@example.com');

// use the client to save the record
$client = new \Lamplight\Client(null, $api_credentials);
$client->save($record);

// the client creates this based on the data sent
$response = $client->getDatainResponse();
if ($response->isSuccessful()) {
    // all records attended ok:
    echo 'Great, thanks';
} else {
    // handle the error
    echo 'Please contact us directly to confirm your attendance';
}
```


## Workareas

Workareas are categories added to work and other activity records in Lamplight.  They are a hierarchical list:
each activity record must have a workarea and may have any number of sub-workareas.

The uri to request all workareas is `https://lamplight.online/api/workarea/all`.

The response will be a structure like this:

```json
{
  "data": [
    {
      "id": 1,
      "text": "First workarea",
      "children": [
        {
          "id": 192,
          "text": "A sub-workarea"
        },
        {
          "id": 89,
          "text": "Another sub-workarea"
        }
      ]
    },
    {
      "id": 23,
      "text": "Advice work - workarea with no subworkareas", 
      "children": []
    }
]}
```





## Contents

[Introduction and authentication](api.html)

[Requesting and creating profiles](profiles.html)

[Requesting and attending work records](work.html)

[Creating referral records](referral.html)

[Error codes](errors.html)

[Hints and tips](hints.html)
