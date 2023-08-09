# Document Reference Mapping

## XML HL7 > JSON FHIR

A Document Reference is primarily mapped from a Narrative Statement. Where the EHR Composition is used, it is the parent of the Narrative Statement.

| Mapped to (JSON FHIR Document Reference field) | Mapped from (XML HL7 / other source)                                                                                                                               |
|------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| meta.profile                                   | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-DocumentReference-1"`                                                                 |
| id                                             | `NarrativeStatement / id [@root]`                                                                                                                                  |
| identifier\[0].system                          | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                    |
| identifier\[0].value                           | `NarrativeStatement / id [@root]`                                                                                                                                  |
| status                                         | fixed value = `"current"`                                                                                                                                          |
| type                                           | `NarrativeStatement / reference / referredToExternalDocument / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md) |
| subject                                        | reference to the mapped [Patient](../patient/README.md)                                                                                                            |
| created                                        | `EhrComposition / AvailabilityTime`                                                                                                                                |
| indexed                                        | `EhrComposition / author / time [@value]`                                                                                                                          |
| author\[0]                                     | `NarrativeStatement / participant / agentRef[0] / id [@root]` or else `ehrComposition / Participant2 / id \[@root]`                                                |
| custodian                                      | reference to the losing [Organisation](../organisations/README.md)                                                                                                 |
| description                                    | `NarrativeStatement / text`                                                                                                                                        |
| content.attachment.url                         | The URL of the attachment in the Adaptors file storage area (S3 / Blob Storage) <sup>1</sup>                                                                       |
| content.attachment.size                        | The size of the file in the Adaptors file storage area (S3 / Blob Storage)                                                                                         |
| content.attachment.title                       | fixed value = `"GP2GP generated placeholder. Original document not available. See notes for details"` <sup>2</sup>                                                 |
| content.attachment.contentType                 | `NarrativeStatement / referredToExternalDocument / text [@mediaType]`                                                                                              |                                                       
| context.encounter                              | reference to the associated [Encounter](../encounters/README.md) <sup>3</sup>                                                                                      |                                                       

<details>
    <summary>Example JSON</summary>

```
        {
            "resource": {
                "resourceType": "DocumentReference",
                "id": "FDD5D332-0468-414A-84E5-AFDAD5B37BCE",
                "meta": {
                    "profile": [
                        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-DocumentReference-1"
                    ]
                },
                "identifier": [
                    {
                        "system": "https://PSSAdaptor/D5445",
                        "value": "FDD5D332-0468-414A-84E5-AFDAD5B37BCE"
                    }
                ],
                "status": "current",
                "type": {
                    "coding": [
                        {
                            "system": "http://snomed.info/sct",
                            "code": "37251000000104",
                            "display": "Other digital signal"
                        }
                    ],
                    "text": "Other Attachment"
                },
                "subject": {
                    "reference": "Patient/cc49817a-813d-4000-ba8f-50157aea1caa"
                },
                "created": "2013-10-25T15:53:28+00:00",
                "indexed": "2013-10-25T15:53:28.000+00:00",
                "author": [
                    {
                        "reference": "Practitioner/14CA7344-7460-4C96-BABC-AA0A2C8E96D6"
                    }
                ],
                "custodian": {
                    "reference": "Organization/ae6747c5-4872-4134-8010-296bd82f121c"
                },
                "description": "application/binary",
                "content": [
                    {
                        "attachment": {
                            "contentType": "text/plain",
                            "url": "https://example.s3.eu-west-2.amazonaws.com/BB2D2DA0-3CA3-4E51-8266-AA0917A09BD2_1C135B3C-5AA5-4BF1-A08B-E6002455B806_text.pdf?X-Amz-Algorithm=AWS4-HMAC-SHA256&X-Amz-Date=20230607T134042Z&X-Amz-SignedHeaders=host&X-Amz-Expires=3599&X-Amz-Credential=example%2Feu-west-exampleaws4_request&X-Amz-Signature=example",
                            "size": 359252
                        }
                    }
                ],
                "context": {
                    "encounter": {
                        "reference": "Encounter/26EE99BB-00FF-4596-9D8B-1D349C1D70A1"
                    }
                }
            }
        },
```

</details>

1. Because placeholders are uploaded to storage the `content.attachment.url` field should always be populated.
2. The `content.title` field will only be populated if a GP2GP absent attachment placeholder has been sent.  
3. The Encounter referenced in `context.encounter` is mapped from the parent EhrComposition. However, this is not done if the EhrComposition has a SNOMED conceptId of Non-consultation data (196401000000100) or Non-consultation medication data (196391000000103).

### Unmapped fields

The following Document Reference fields are not currently populated by the adaptor:
- masterIdentifier
- context.practiceSetting
- context.related


## JSON FHIR > XML HL7

A Document Reference is mapped to a Narrative Statement and an absent attachment placeholder (if one is created)

| Mapped to (XML HL7)                                                          | Mapped from (JSON FHIR / other source )                                                         |
|------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------|
| NarrativeStatement / id \[@root]                                             | NarrativeStatement ID (UUID) generated by the Adaptor                                           |
| NarrativeStatement / statusCode \[@code]                                     | fixed value = `"complete"`                                                                      |
| NarrativeStatement / AvailabilityTime                                        | `DocumentReference.created` or else `DocumentReference.indexed`                                 |
| NarrativeStatement / Participant / AgentRef                                  | `DocumentReference.author`                                                                      |
| NarrativeStatement / referredToExternalDocument / id \[@root]                | NarrativeStatement ID (UUID) generated by the Adaptor (identical to the UUID above)             |
| NarrativeStatement / referredToExternalDocument / text \[@mediaType]         | DocumentReference.content.attachment.contentType <sup>4</sup>                                   |
| NarrativeStatement / referredToExternalDocument / text / reference \[@value] | "file://localhost/{{filename}}" - where `{{filename}}` is generated by the Adaptor <sup>5</sup> |

<details>
    <summary>Example XML</summary>

```
<NarrativeStatement classCode="OBS" moodCode="EVN">
    <id root="212660C4-097A-404B-B90C-1C8ACE1ED63D"/>
    <text>Type: Advance directive report Custodian Org: TEMPLE SOWERBY MEDICAL PRACTICE Description: (19-Jan-2010)</text>
    <statusCode code="COMPLETE"/>
    <availabilityTime value="20100119"/>
    <Participant typeCode="PRF" contextControlCode="OP">
        <agentRef classCode="AGNT">
            <id root="F108C5E5-6A63-4472-AD1B-4F9F8ED1237B"/>
        </agentRef>
    </Participant>
    <reference typeCode="REFR">
        <referredToExternalDocument classCode="DOC" moodCode="EVN">
            <id root="212660C4-097A-404B-B90C-1C8ACE1ED63D"/>
            <code code="37251000000104" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Other digital signal"/>
            <text mediaType="image/jpeg">
                <reference value="file://localhost/212660C4-097A-404B-B90C-1C8ACE1ED63D.jpg"/>
            </text>
        </referredToExternalDocument>
    </reference>
</NarrativeStatement>
```

</details>

4. If the Adaptor has generated a placeholder the media type will have a fixed value of `plain/text`. 
5. If the attachment is present, the generated filename will consist of the NarrativeStatement ID and a file extension derived from `DocumentReference.content.attachment.contentType`. If the Adaptor has generated a placeholder the 
filename will be "AbsentAttachment<b>{{narrativeStatementId}}</b>.txt" e.g. AbsentAttachmentB2BAE43C-C540-4A0B-9EB7-D36D2F106F69.txt.

### Placeholder generation

The adaptor will generate a placeholder if:
* `DocumentReference.content.attachment.contentType` MIME type is not supported.
* `DocumentReference.content.attachment.title` is present and doesn't contain empty string.
* `DocumentReference.content.attachment.url` is missing or there is an error retrieving the document binary. 

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

## Further documentation

- [GP Connect DocumentReference](https://developer.nhs.uk/apis/gpconnect-1-6-0/access_documents_development_documentreference.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
