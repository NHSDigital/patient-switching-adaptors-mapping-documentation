# List (Consultation) Mapping

## XML HL7 > JSON FHIR

A List (Consultation) is created from mapped [Encounters](../encounters/README.md).</br>
Where the EHR extract is used it is the root `EhrExtract` of the XML.
Where `EhrComposition` is used, it refers to the EhrComposition used to construct the `Encounter`  
This is a top-level profile and represents the whole structured consultation

| Mapped to (JSON FHIR List field) | Mapped from (XML HL7 / other source)                                                                                                                                                            |
|----------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                               | `ehrComposition / id \[@root]` suffixed with `-CONS`                                                                                                                                            |
| meta.profile\[0]                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"`                                                                                                           |
| status                           | fixed value = `current`                                                                                                                                                                         |
| mode                             | fixed value = `snapshot`                                                                                                                                                                        |
| title                            | `ehrComposition / code [@displayName]` or `ehrComposition / code / originalText` or found by searching the adaptors SNOMED database for the appropriate description when `Encounter` is mapped. |
| code.coding\[0].system           | fixed value = `http://snomed.info/sct`                                                                                                                                                          |
| code.coding\[0].code             | fixed value = `325851000000107`                                                                                                                                                                 |
| code.coding\[0].display          | fixed value = `Consultation`                                                                                                                                                                    |
| subject                          | this a reference to the mapped [Patient](../patient/README.md) from the `encounter`                                                                                                             |
| date                             | `ehrComposition / effectiveTime / center` or else `EhrComposition / effectiveTime / low` or else `ehrComposition / availibiltyTime` or from root `EhrExtract / availabilityTime / value`        |
| orderedBy.coding\[0].system      | fixed value = `http://hl7.org/fhir/list-order`                                                                                                                                                  |
| orderedBy.coding\[0].code        | fixed value = `system`                                                                                                                                                                          |
| orderedBy.coding\[0].display     | fixed value = `Sorted by System`                                                                                                                                                                |
| encounter                        | reference to mapped [Encounter](../practioners/README.md)                                                                                                                                       |
| entry[index].item.reference      | reference to one or more mapped [List (Topic)](./LIST_TOPIC_README.md)                                                                                                                          |

The following List fields are not currently populated by the adaptor:
- identifier
- source
- note
- entry.flag
- entry.deleted
- entry.date


<details>
    <summary>Example JSON</summary>

```
{
    "resource": {
        "resourceType": "List",
        "id": "1E8A8448-A0C1-11ED-808B-AC162D1F16F0-CONS",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"
            ]
        },
        "status": "current",
        "mode": "snapshot",
        "title": "Surgery Consultation Note",
        "code": {
            "coding": [
                {
                    "system": "http://snomed.info/sct",
                    "code": "325851000000107",
                    "display": "Consultation"
                }
            ]
        },
        "subject": {
            "reference": "Patient/14013417-5eb8-4fb2-9916-4c1621e2533b"
        },
        "encounter": {
            "reference": "Encounter/1E8A8448-A0C1-11ED-808B-AC162D1F16F0"
        },
        "date": "2010-12-16",
        "orderedBy": {
            "coding": [
                {
                    "system": "http://hl7.org/fhir/list-order",
                    "code": "system",
                    "display": "Sorted by System"
                }
            ]
        },
        "entry": [
            {
                "item": {
                    "reference": "List/5a8d2ec6-807d-4db5-b6c0-a757bbfb5372"
                }
            }
        ]
    }
}
```
</details>

## JSON FHIR > XML HL7

A `List (Consultation)` does not generate its own XML section but instead is used to identify which Topics will be used to retrieve the `components` used when mapping an [Encounter](../encounters/README.md).</br> 

| Mapped to (XML HL7 Encounter)                   | Mapped from (JSON FHIR / other source )                                                                           |
|-------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| EhrComposition / Component [@typeCode="COMP"]`  | `resource` when the list contains a reference to `Encounter.id` and 'resource.resourceType` = `List` <sup>1</sup> |

1.  For each reference in `resource.entry.item.reference`, the resultant topic list are parsed and the mapped `Components` are inserted into the XML.

<details><summary>Example XML</summary>

```
<component typeCode="COMP">
    <ehrComposition classCode="COMPOSITION" moodCode="EVN">
        <id root="46DDDE45-9CA1-485A-9789-2C342D33CEE3" />
        <code code="109341000000100" displayName="GP to GP communication transaction"
            codeSystem="2.16.840.1.113883.2.1.3.2.4.15" />
        <statusCode code="COMPLETE" />
        <effectiveTime>
            <center value="20100119" />
        </effectiveTime>
        <availabilityTime value="20100119113634" />
        <author typeCode="AUT" contextControlCode="OP">
            <time value="20100119113634" />
            <agentRef classCode="AGNT">
                <id root="686B63E5-EB8B-4353-8FF2-8387EFB38839" />
            </agentRef>
        </author>
        <Participant2 typeCode="PRF" contextControlCode="OP">
            <agentRef classCode="AGNT">
                <id root="686B63E5-EB8B-4353-8FF2-8387EFB38839" />
            </agentRef>
        </Participant2>
        {{Retrieved Components are added here}}
    </ehrComposition>
</component>
```
</details>

## Further documentation

- [GP Connect List](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_list_consultation.html#list-consultation)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
