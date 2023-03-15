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

#### Data Verification Requirements

* Pipeline critical elements (for specific formatting validation)
  * Defined in detail in the Pipeline Critical Elements section
  below. 
* ALL elements that are available in your vendor system 

All data elements are housed in the [vendor folder](https://tylertech.sharepoint.com/sites/Client/DI/AOIC/Program%201%20%203%20Prepare%20Solution/Forms/AllItems.aspx?RootFolder=%2Fsites%2FClient%2FDI%2FAOIC%2FProgram%201%20%203%20Prepare%20Solution%2FVendor%20docs%2FData%20Elements%20%2D%20ALL&FolderCTID=0x012000E4E5E251D4298743B4D89B00DBBF4D85&View=%7B0F3FBEB1%2DB9A9%2D4E2E%2D957E%2D9E4144F8F6F9%7D)

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
Using the Bearer Token received from the Step 1, prepare an API
call to **PUT** a Message. A Message is simply a Record of Data.
This record of data should include all the data elements that are
available and specifically include the critical pipeline fields listed
below in the Pipeline Critical Elements section.

Sample cURL and PowerShell commands can be found here to
demonstrate minimal examples for each data element set.

#### Example PowerShell command 
```powershell
$headers = New-Object
"System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add("Content-Type", "application/json")
$headers.Add("Authorization", "Bearer YOUR_TOKEN_HERE")

$body = "{
    `"Events`": [      {
        `"EventType`": `"di-aoic-new-record-event`",
        `"Entities`": [
            {
                `"EntityType`": di-aoic-problem-solving-courts-
individual-background,
                `"EntityId`": `"AOIC`",
                `"LinkEntity`": true,
                `"EntityData`": {
                                    `"county`": `"cook`",
                                    `"offenderid`": `"9816`",
                                    `"dateofbirth`": `"10/20/1989`",
                                    `"pscbackgroundid`": `"4238`",
                                    `"instanceid`": `"7865`",
                                    `"name`": `"Heather Graham`",
                                    `"sexperceived`": `"Female`",
                                    `"localid`": 8147,
                        }
                }
            ]
        }
    ]
}"

response = Invoke-RestMethod 'https://api.dev.tyleralliance.com/
exchange/Messages' -Method 'PUT' -Headers $headers -Body
$body

$response | ConvertTo-Json
```
Please refer to the [API documentation](http://aoic-api.s3-website-us-west-2.amazonaws.com/) for more information.

A successful response will look like:
```json
{
  "status":"SUCCESS",
  "EnvelopeId": "UNIQUE-ENVELOPE-ID"
}
```
Repeat Step 2, utilizing the same 60-min token, to submit
additional record(s).

**Make sure to screenshot and copy the SUCCESS response
with the envelopeID, this is needed for Step 3**

## Step 3: Complete Certification
Vendors must email their SUCCESS response and EnvelopeID. **Note: Separate emails must be sent for each county program area**
* Send to: [data-certification@tylertech.com](mailto:data-certification@tylertech.com)
* Subject: Certification SUCCESS - [Program Area - County]

Tyler will confirm receipt of the email within 1 business day

Upon successful validation:
* Tyler will submit the vendor’s certification with the State and notify the vendor that certification has been completed

## Data Elements
Every entity must include each of the required elements listed in the [vendor folder](https://tylertech.sharepoint.com/sites/Client/DI/AOIC/Program%201%20%203%20Prepare%20Solution/Forms/AllItems.aspx?RootFolder=%2Fsites%2FClient%2FDI%2FAOIC%2FProgram%201%20%203%20Prepare%20Solution%2FVendor%20docs%2FData%20Elements%20%2D%20ALL&FolderCTID=0x012000E4E5E251D4298743B4D89B00DBBF4D85&View=%7B0F3FBEB1%2DB9A9%2D4E2E%2D957E%2D9E4144F8F6F9%7D) 

### Formatting data elements
Both RowID and entity type follow specific formats

#### EntityType 
```
di-[Program Name]-[Dataset Name]
```
##### Examples

| Entity-type | Description |
| ------------|-------------|
| di-aoic-courts-party-hearing          | Courts program and the Party Hearing dataset         |
| di-aoic-pretrial-individual-background          | Pretrial program and the Individual Background dataset         |
| di-aoic-probation-intake          | Probation program and the Intake dataset         |
| di-aoic-problem-solving-courts-psc-status          | PSC program and the PSC Status dataset         |

#### RowID
The row ID is produced by the CMS system and matches the following pattern: 

County, case type and vendor ID should be lowercase with no white space.
```
4 digit year-County-Case Type-Vendor ID-Unique Number
```

Example:
```
2023-cook-criminalfelony-tyl-1234
```


### Pretrial Program Pipeline Critical Elements
In addition to the Data Verification Prerequisites, the following elements must be included in every PSC record:

* county
* instanceid
* offenderid
* name
* localid
* dateofbirth*
* sexperceived*
* pretrialbackground**

*Critical for certification. May not be required for production
data submissions

**this element is the rowID. [See this section](https://spaceballone.github.io/swagger-test/docs/overview#rowid) for the row ID format

Example
```json
"Entities": [{
    "name": "Dale Bell",
    "localid":  9152,
    "instanceid": "5008",
    "offenderid": "4354",
    "dateofbirth": "1900/11/12",
    "sexperceived": "Male",                    
    "pretrialbackgroundid": "2023-cook-asdf-tyl-9152",
    "county": "cook",
}]
```

## FAQ 
1. What happens if I get a SUCCESS response but receive an
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