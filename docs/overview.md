---
title: Overview
noindex: true
---


# ![logo](https://www.tylertech.com/Portals/0/Logo-NavBar.jpg?ver=Js0wL8bzpXBsBHn_bv-Kjg%3d%3d) Tyler Data Pipeline Certification 
These instructions and resources are intended to walk through
the technical process of certifying data in the Tyler Judicial
Analytics Pipeline. This is a prerequisite to be granted access
to the staging pipeline.

Vendors will leverage their vendor specific authorization token
and the Judicial Analytics Pipeline to upload and validate data. A
connection will qualify for certification and verification if a
`SUCCESS` response is received and there are no errors within the
Pipeline’s internal logs.

Each Vendor should submit all available data elements and
specifically include all pipeline critical elements noted in the
Pipeline Critical Elements section below. The submission is
considered acceptable if no validation errors are triggered and
the required data elements contain a value suitable for its
respective datatype. **Certification will occur by program per
county.**

**Important: For courts integration, a completed Court Data
Elements Mapping Tool must be returned to AOIC before
certification can begin.**

## Prerequisites

The following are required to begin the Certification
Process:

* A minimum of 10 records reflecting data requirements per
program (Probation, Pretrial, PSC, and Courts)
* Vendor specific Credential Pair 
* Submission of the Court Mapping Tool to AOIC *(Courts
Program only)

Data Requirements

* Access to data for each of the critical and optional elements listed in the [vendor folder](https://tylertech.sharepoint.com/sites/Client/DI/AOIC/Program%201%20%203%20Prepare%20Solution/Forms/AllItems.aspx?RootFolder=%2Fsites%2FClient%2FDI%2FAOIC%2FProgram%201%20%203%20Prepare%20Solution%2FVendor%20docs%2FData%20Elements%20%2D%20ALL&FolderCTID=0x012000E4E5E251D4298743B4D89B00DBBF4D85&View=%7B0F3FBEB1%2DB9A9%2D4E2E%2D957E%2D9E4144F8F6F9%7D)

## Step 1: Get an Authorization (Bearer) Token
The Certification Instance of the Pipeline is access-controlled
such that each vendor has its own credential pair. A credential
pair consists of a Client ID and Client Secret. Vendors should
have received a Client ID and Client Secret via a secure email.

Vendors will use this pair to get a bearer token that will allow
them to interact with the Judicial Analytics Pipeline.

Note: this command will provide a bearer token that is valid for
60 minutes.

Sample cURL and PowerShell commands can be found here to
demonstrate minimal examples for each data element set. 

#### Example curl command 

```
curl https://tyler-alliance-system-dev.auth.us-east-1.amazoncognito.com/oauth2/token -X POST -H "Content-Type: application/x-www-form-urlencoded" --user YOUR_CLIENT_ID:YOUR_CLIENT_SECRET -d "grant_type=client_credentials"
```
Please refer to the [API documentation](http://aoic-api.s3-website-us-west-2.amazonaws.com/) for more information.

## Step 2: Submit Messages to the Pipeline API
### Build the message
Every message consists of a series of entities, as well as metadata to help route the message appropriately

An example message might look like
```json
{
  "Events": [
    {
      "Entities": [
        {
          "EntityId": "AOIC",
          "EntityType": "di-aoic-problem-solving-courts-individual-background",
          "LinkEntity": true,
          "EntityData": {
              "name": "Dale Bell",
              "localid": 9152,
              "instanceid": "5008",
              "offenderid": "4354",
              "dateofbirth": "1900/11/12",
              "sexperceived": "Male",
              "pretrialbackgroundid": "2023-cook-asdf-tyl-9152",
              "county": "cook"
           },
          "UpdateData": true,
          "ValidateEntity": true
        }
      ],
      "EventType": "di-aoic-new-record-event"
    }
  ],
  "PartnerId": "1000",
  "ResolveMappings": true,
  "TriggeredBy": "jane.doe@example.com",
  "ValidateEnvelope": true
}
```
A message starts with an `Envelope`. Details for each attribute of the `Envelope` are:

| field            | allowed_type | required | description                                                                                     |
|------------------|--------------|----------|-------------------------------------------------------------------------------------------------|
| Events           | array        | Y        | An array of events. For more details. See next section                                          |
| PartnerID        | string       | Y        | Your partner ID                                                                                 |
| ResolveMappings  | boolean      | N        |                                                                                                 |
| TriggeredBy      | string       | Y        | The user triggering the event                                                                   |
| ValidateEnvelope | boolean      | N        | Validate that the envelope is well formatted. Should be set to true unless instructed otherwise |

### Events
Envelopes may consist of one or more `events`. Details for each attribute of the `Event` are:

| field            | allowed_type | required | description                                                                                     |
|------------------|--------------|----------|-------------------------------------------------------------------------------------------------|
| Entities           | array        | Y        | An array of events. For more details. See next section                                        |
| EventType         | string       | Y        | Always set to `di-aoic-new-record-event`                                                      |                           |

### Entities
Entities are the most important part of the message. This specifies the program, the record and the values associated with that record. 
Details for each attribute are as follows

| field          | allowed_type | required | description                                                                                                                      |
|----------------|--------------|----------|----------------------------------------------------------------------------------------------------------------------------------|
| EntityID       | string       | Y        | Always set to `AOIC`                                                                                                             |
| EntityType     | string       | Y        | Maps to the specific program and data in the format `di-[Program Name]-[Dataset Name]`. See [below for more examples](#Examples) |
| EntityData     | object       | Y        | The record associated with this entity                                                                                           |
| LinkEntity     | boolean      | N        | Links this entity to another. Can be ignored                                                                                     |
| UpdateData     | boolean      | N        | Whether or not to update an existing message                                                                                     |
| ValidateEntity | boolean      | N        | Validate that the entity and the data is well formatted. Should be set to true unless instructed otherwise                       |

### Entity Data
Every entity data object must contain the set of required elements listed in the [vendor folder](https://tylertech.sharepoint.com/sites/Client/DI/AOIC/Program%201%20%203%20Prepare%20Solution/Forms/AllItems.aspx?RootFolder=%2Fsites%2FClient%2FDI%2FAOIC%2FProgram%201%20%203%20Prepare%20Solution%2FVendor%20docs%2FData%20Elements%20%2D%20ALL&FolderCTID=0x012000E4E5E251D4298743B4D89B00DBBF4D85&View=%7B0F3FBEB1%2DB9A9%2D4E2E%2D957E%2D9E4144F8F6F9%7D), in the format specified

These should be passed as attributes in the object. For example
```json
"entityData": {
     "name": "Dale Bell",
     "localid": 9152,
     "instanceid": "5008",
     "offenderid": "4354",
     "dateofbirth": "1900/11/12",
     "sexperceived": "Male",
     "pretrialbackgroundid": "2023-cook-asdf-tyl-9152",
     "county": "cook"
  }
```

#### Notes on the entity data object
There are a couple of critical items to watch out for when building the entityData object:
* **Data Elements** - Each entity **must** include the required elements listed in the [vendor folder](https://tylertech.sharepoint.com/sites/Client/DI/AOIC/Program%201%20%203%20Prepare%20Solution/Forms/AllItems.aspx?RootFolder=%2Fsites%2FClient%2FDI%2FAOIC%2FProgram%201%20%203%20Prepare%20Solution%2FVendor%20docs%2FData%20Elements%20%2D%20ALL&FolderCTID=0x012000E4E5E251D4298743B4D89B00DBBF4D85&View=%7B0F3FBEB1%2DB9A9%2D4E2E%2D957E%2D9E4144F8F6F9%7D), in the format specified. Failure to include these elements, or to format them in the right way could result in certification failure
* **RowID** - Each object has a unique rowID. The RowID **must** follow a specific format: `4 digit year-County-Case Type-Vendor ID-Unique Number`. For example`2023-cook-criminalfelony-tyl-1234`



### Send the message
Using the Bearer Token received from the Step 1, and the message prepared in the previous section, send an API
call to **PUT** the Message built in the previous section. 

Sample cURL and PowerShell commands can be found [here](https://tylertech.sharepoint.com/sites/Client/DI/AOIC/Program%201%20%203%20Prepare%20Solution/Forms/AllItems.aspx?csf=1&web=1&e=XIcIAS&cid=a0eb1b19%2D1106%2D4d00%2D8323%2D8d93d5213bbc&RootFolder=%2Fsites%2FClient%2FDI%2FAOIC%2FProgram%201%20%203%20Prepare%20Solution%2FVendor%20docs%2FCertification%20directions%2Fcertification%5Fexample%5Fcommands&FolderCTID=0x012000E4E5E251D4298743B4D89B00DBBF4D85) to
demonstrate minimal examples for each data element set.

### Validate the response
A successful response will look like:
```json
{
  "status":"SUCCESS",
  "EnvelopeId": "UNIQUE-ENVELOPE-ID"
}
```
If you receive a successful response, repeat Step 2, utilizing the same 60-min token, to submit
additional record(s).

**Make sure to screenshot and copy the SUCCESS response
with the envelopeID, this is needed for Step 3**

## Step 3: Complete Certification
Vendors must email their SUCCESS response and EnvelopeID. **Note: Separate emails must be sent for each county program area**
* Send to: [data-certification@tylertech.com](mailto:data-certification@tylertech.com)
* Subject: Certification SUCCESS - [Program Area - County]

Tyler will confirm receipt of the email within 1 business day

Upon successful validation Tyler will submit the vendor’s certification with the State and notify the vendor that certification has been completed


## Examples
The following section provides a series of data element examples for each program

### Pretrial Program Pipeline Critical Elements
`In addition to the Data Verification Prerequisites, `the following elements must be included in every PSC record:

* county
* instanceid
* offenderid
* name
* localid
* dateofbirth [1]
* sexperceived [1]
* pretrialbackground [2]

##### Example
```json
"Entities": [
  "entityData": {
     "name": "Dale Bell",
     "localid": 9152,
     "instanceid": "5008",
     "offenderid": "4354",
     "dateofbirth": "1900/11/12",
     "sexperceived": "Male",
     "pretrialbackgroundid": "2023-cook-asdf-tyl-9152",
     "county": "cook"
  }
}]
```

##### Notes
[1] Must be present for certification

[2] RowID. [See this section](https://spaceballone.github.io/swagger-test/docs/overview#rowid) for the row ID format



### Probation Program Pipeline Critical Elements
In addition to the Data Verification Prerequisites, the
following elements must be included in every Probation
record:

* county
* instanceid
* offenderid
* name
* localid
* dateofbirth [1]
* sexperceived [1]
* probationbackgroundid [2]

##### Example
```json
"Entities": [
  "entityData": {
     tbd
  }
}]
```
##### Notes
[1] Must be present for certification

[2] RowID. [See this section](https://spaceballone.github.io/swagger-test/docs/overview#rowid) for the row ID format

### Problem Solving Courts Program Pipeline Critical Elements
In addition to the Data Verification Prerequisites, the
following elements must be included in every PSC record:
* county
* instanceid
* offenderid
* name
* localid
* dateofbirth [1]
* sexperceived [1]
* pscbackgroundid [2]
##### Example
```json
"Entities": [
  "entityData": {
     tbd
  }
```
##### Notes
[1] Must be present for certification

[2] RowID. [See this section](https://spaceballone.github.io/swagger-test/docs/overview#rowid) for the row ID format


### Courts Program Pipeline Critical Elements
In addition to the Data Verification Prerequisites, the
following elements must be included in every Courts record:
* county
* statusdate
* casestatusrowid [1]
##### Example
```json
"Entities": [
  "entityData": {
     tbd
  }
```
##### Notes
[1] Must be present for certification

## FAQ 
1. Where can I find more information about the API?
   2. Please refer to the [API documentation](http://aoic-api.s3-website-us-west-2.amazonaws.com/) for more information.
2. What happens if I get a SUCCESS response but receive an
error message for data validation?
   * If there is an error in your submission, you’ll receive an
   email from AWS Notifications within a few minutes of your
   SUCCESS response.
   * See the notification email you received. Each error will
   have Error Details, which indicate the type of error
   encountered. Once fixed, please try to upload again. If you
   have issues resolving your issue, please reach out directly
   to the Tyler team at data-certification@tylertech.com.
   After you correct an error and re-submit, you may receive
   another error message calling out another error in the
   submission. Repeat the retry process until all errors are
   corrected. 
2. I have multiple counties and program areas to submit.
Can I reuse my Token to repeat Step 2? 
   * Yes. The Token can be used to repeat Step 2 several times.
   However, the token is time-sensitive and will need to be
   recreated at 60 minutes.

3. What does a response of "message":"Unauthorized" mean? 
   * It means there is an issue with your bearer token, a token
   is valid for 60 minutes, so you may need to recreate it.

4. How do I complete the certification reimbursement
process? 
   * Reach out to the AOIC with reimbursement specific
   questions. They are providing a FAQ for things related to
   this process.

5. Who can I contact for assistance?
   * Please contact data-certification@tylertech.com for
   assistance, questions, or live support.

6. What are the data elements necessary for certification of
each Program?
   * See the Pipeline Critical Elements section above or review
   this SharePoint folder for .txt examples of each data
   element reporting program’s PowerShell or cURL (Mac)
   PUT call.

## Need Help?

Our team is here to help! We are offering various options to
support you throughout this process:

* Office Hours: We are excited to offer Office Hours each
Wednesday 12:00pm Central Daylight Time through April
26th. Each vendor has already been added to this standing
meeting and can join at any time. Simply bring your questions
or issues and we’ll be ready to help via screen-share.

* 1:1 Meetings: In addition to Office Hours, our team is
scheduling time with each vendor to walk through questions
and/or through the process described above. If you would like
to schedule time with us, please notify us at data-
certification@tylertech.com. Otherwise, our team will reach
out to you starting the week of March 13th to schedule time
proactively.

* Email Help: If you prefer, you can also reach out to us at
  [data-certification@tylertech.com](mailto:data-certification@tylertech.com). Our dedicated team of
experts will respond as quickly as possible between 8am –
8pm Central Daylight Time, within 1 business day, excluding
weekends. When writing in, it is helpful if you consider the
following guidelines to help ensure that we can respond and
resolve your question as quickly and efficiently as possible:
  * Describe your question or issue 
  * Share relevant context about the issue, including links,
  screenshots and specific steps taken