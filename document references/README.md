# Document Reference Mapping

## XML HL7 > JSON FHIR

A DocumentReference is primarily mapped from a NarrativeStatement. Where the EhrComposition is used, it is the parent of the NarrativeStatement.  

| Mapped from (XML HL7)                                              | Mapped from (Other)                                                                                                 | Mapped to (JSON FHIR)          |
|--------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------|--------------------------------|
| NarrativeStatement / id \[@root]                                   |                                                                                                                     | id                             |
|                                                                    | https://PSSAdaptor/{{losingOdsCode}} <br/> (where the {{losingOdsCode}} is the ODS code of the losing organisation) | identifier\[0].system          |
|                                                                    | fixed value = "current"                                                                                             | status                         |
| NarrativeStatement / reference / referredToExternalDocument / code |                                                                                                                     | type                           |
|                                                                    | reference to the mapped [Patient](../patient/README.md)                                                             | subject                        |
| EhrComposition / AvailabilityTime                                  |                                                                                                                     | created                        |
| EhrComposition / author                                            |                                                                                                                     | indexed                        |
| NarrativeStatement / participant / agentRef / id \[@root]          |                                                                                                                     | author                         |
|                                                                    | reference to the losing [Organisation](../organisations/README.md)                                                  | custodian                      |
| NarrativeStatement / text                                          |                                                                                                                     | description                    |
|                                                                    | The URL of the attachment in the Adaptors file storage area (S3 / Blob Storage) <sup>1</sup>                        | content.attachment.url         |
|                                                                    | The size of the file in the Adaptors file storage area (S3 / Blob Storage)                                          | content.attachment.size        |
|                                                                    | fixed value = "GP2GP generated placeholder. Original document not available. See notes for details" <sup>1</sup>    | content.attachment.title       |
|                                                                    |                                                                                                                     | content.attachment.contentType |
|                                                                    | reference to the associated [Encounter](../encounters/README.md) <sup>2</sup>                                       | context.encounter              |

1. The `content.title` field will only be populated if a GP2GP absent attachment placeholder has been sent. However, because placeholders are uploaded to storage the `content.attachment.url` field should always be populated.  
2. The Encounter referenced in `context.encounter` is mapped from the parent EhrComposition. However, this is not done if the EhrComposition has a SNOMED conceptId of Non-consultation data (196401000000100) or Non-consultation medication data (196391000000103)

## JSON FHIR > XML HL7

| Mapped from (JSON FHIR)                                       | Mapped from (Other)                                                                                                  | XML HL7                                                                      |
|---------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------|
|                                                               | NarrativeStatement ID generated by the Adaptor                                                                       | NarrativeStatement / id \[@root]                                             |
| DocumentReference.created <br/> or DocumentReference.indexed  |                                                                                                                      | NarrativeStatement / AvailabilityTime                                        |
| DocumentReference.author                                      |                                                                                                                      | NarrativeStatement / Participant / AgentRef                                  |
|                                                               | identical to the generated NarrativeStatement ID                                                                     | NarrativeStatement / referredToExternalDocument / id \[@root]                |
| DocumentReference.content.attachment.contentType <sup>3</sup> | see below <sup>3</sup>                                                                                               | NarrativeStatement / referredToExternalDocument / text \[@mediaType]         |
|                                                               | <reference value="file://localhost/{{filename}}"/> <br/> where {{filename}} is generated by the Adaptor <sup>4</sup> | NarrativeStatement / referredToExternalDocument / text / reference \[@value] |


3. If the Adaptor has generated a placeholder the media type will have a fixed value of `plain/text`. 
4. If the attachment is present, the generated filename will consist of the narrativeStatement ID and a file extension derived from the MIME content type. If the Adaptor has generated a placeholder the 
filename will be "AbsentAttachment{{narrativeStatementId}}.txt" e.g. AbsentAttachmentB2BAE43C-C540-4A0B-9EB7-D36D2F106F69.txt

### Placeholder generation

The adaptor will generate a placeholder if:
* The `DocumentReference.content.attachment.contentType` is not supported
* The `DocumentReference.content.attachment.title` field is not present or contains an empty string.
* The `DocumentReference.content.attachment.url` is missing or there is an error retrieving the document binary 

The content of the placeholder will be mapped as follows:

```
The file could not be included with the Electronic Record
{{description}}
{{odsCode}}:{{conversationId}}
Reason:06:Unable to determine problem
```

Where:

- `{{description}}` = `DocumentReference.description`
- `{{odsCode}}` = ODS code of the losing practice
- `{{conversationID}}` = the conversation ID of the migration