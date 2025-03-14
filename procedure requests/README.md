# Procedure Request

## XML HL7 > JSON FHIR

A Procedure Request is mapped primarily from a PlanStatement.

| Mapped to (JSON FHIR Document Reference field) | Mapped from (XML HL7 / other source)                                                                                                                                                                                              |
|------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| meta.profile                                   | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProcedureRequest-1"`                                                                                                                                 |
| meta.security                                  | When `PlanStatement / confidentialityCode [@code]` or `EhrComposition / confidentialityCode [@code]` is present and has a value of `NOPAT`. See [Confidentiality Codes](../confidentiality code/README.md) for mapping details.   |
| identifier\[0].system                          | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                                                   |
| identifier\[0].value                           | ` PlanStatement / id \[@root] `                                                                                                                                                                                                   |
| id                                             | ` PlanStatement / id \[@root] `                                                                                                                                                                                                   |
| status                                         | `PlanStatement / text` <sup>1</sup>                                                                                                                                                                                               |
| intent                                         | fixed value = `"plan"`                                                                                                                                                                                                            |
| authoredOn                                     | ` PlanStatement / availabilityTime \[@value] ` or ` EhrComposition / availabilityTime \[@value] ` or `EhrComposition / author / time \[@value]`                                                                                   |
| occurrenceDateTime                             | ` PlanStatement / effectiveTime / center \[@value] `                                                                                                                                                                              |
| subject                                        | reference to the mapped [Patient](../patient/README.md)                                                                                                                                                                           |
| note                                           | ` PlanStatement / text `                                                                                                                                                                                                          |
| requester.agent                                | ` PlanStatement / participant `                                                                                                                                                                                                   |
| code                                           | ` PlanStatement / code ` <sup>2</sup> as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                                                                                               |
| context                                        | reference to the associated [Encounter](../encounters/README.md)                                                                                                                                                                  |

1. When `PlanStatement / text` starts with `Status:` the value immediately following this will be mapped to either `active`, `cancelled` or `completed`.
If `PlanStatement / text` does not start with `Status:` or is not present then the value `unknown` will be used.
2. If the PlanStatement code doesn't have a SNOMED code provided within either the main code, or a translation then the adaptor will use a SNOMED code of 196451000000104.

### Unmapped fields

The following Procedure Request fields are not currently populated by the adaptor:

- performer
- reasonCode
- reasonReference

<details>
    <summary>Example JSON</summary>

```
{
    "resource": {
        "resourceType": "ProcedureRequest",
        "id": "3316531F-5705-424C-9E1A-EE694FB411B4",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProcedureRequest-1"
            ]
        },
        "identifier": [
            {
                "system": "https://PSSAdaptor/2167888433",
                "value": "3316531F-5705-424C-9E1A-EE694FB411B4"
            }
        ],
        "status": "active",
        "intent": "plan",
        "code": {
            "coding": [
                {
                    "extension": [
                        {
                            "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                            "extension": [
                                {
                                    "url": "descriptionId",
                                    "valueId": "282653015"
                                },
                                {
                                    "url": "descriptionDisplay",
                                    "valueString": "Medication review"
                                }
                            ]
                        }
                    ],
                    "system": "http://snomed.info/sct",
                    "code": "182836005",
                    "display": "Review of medication"
                }
            ],
            "text": "Medication review"
        },
        "subject": {
            "reference": "Patient/180b44bf-31d8-407b-b8ca-994a3f4a226c"
        },
        "context": {
            "reference": "Encounter/3BFD78DE-03BF-44FD-96BC-CDF3DB2CC039"
        },
        "occurrenceDateTime": "2011-01-15",
        "authoredOn": "2010-01-15T10:06:46+00:00",
        "requester": {
            "agent": {
                "reference": "Practitioner/2D70F602-6BB1-47E0-B2EC-39912A59787D"
            }
        }
    }
}
```
</details>

## JSON FHIR > XML HL7

A Procedure Request is mapped to a Plan Statement.

| Mapped to (XML HL7)                              | Mapped from (JSON FHIR / other source )                                                                                                                              |
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| PlanStatement / id \[@root]                      | Fetched from resource ID or, if not valid UUID, generated by the Adaptor                                                                                             |
| PlanStatement / confidentialityCode              | `ProcedureRequest.meta.security[@code]` is present and has a value of `NOPAT`                                                                                        |   
| PlanStatement / code                             | `ProcedureRequest.code`                                                                                                                                              |
| PlanStatement / text                             | `ProcedureRequest.supportingInformation`, `ProcedureRequest.occurrencePeriod`, `ProcedureRequest.requester`, `ProcedureRequest.reasonCode`, `ProcedureRequest.notes` |
| PlanStatement / statusCode \[@code]              | fixed value = `'COMPLETE'`                                                                                                                                           |
| PlanStatement / effectiveTime / center \[@value] | `ProcedureRequest.occurrenceDateTime` or else `ProcedureRequest.occurrencePeriod`                                                                                    |
| PlanStatement / availabilityTime \[@value]       | `ProcedureRequest.authoredOn`                                                                                                                                        |
| PlanStatement / Participant / AgentRef           | `ProcedureRequest.requester.agent`                                                                                                                                   |

<details><summary>Example XML</summary>

```
<PlanStatement classCode="OBS" moodCode="INT">
    <id root="3316531F-5705-424C-9E1A-EE694FB411B4" />
    <code code="282653015" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Medication review">
    </code>
    <statusCode code="COMPLETE" />
    <effectiveTime>
        <center value="20110115"/>
    </effectiveTime>
    <availabilityTime value="20100115100646"/>
    <confidentialityCode code="NOPAT" 
    codeSystem="2.16.840.1.113883.4.642.3.47" 
    displayName="no disclosure to patient, family or caregivers without attending provider's authorization" />
    <Participant typeCode="PRF" contextControlCode="OP">
        <agentRef classCode="AGNT">
            <id root="2D70F602-6BB1-47E0-B2EC-39912A59787D"/>
        </agentRef>
    </Participant>
</PlanStatement>
```
</details>

## Further documentation

- [GP Connect ProcedureRequest](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_ProcedureRequest.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
