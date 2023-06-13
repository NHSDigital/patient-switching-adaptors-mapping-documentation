# List (Consultation) Mapping

## XML HL7 > JSON FHIR

A List (Consultation) is created from mapped [Encounters](../encounters/README.md). Where the EHR extract is used it is the root `EhrExtract` of the XML.
This is a top-level profile and represents the whole structured consultation

| Mapped to (JSON FHIR List field) | Mapped from (XML HL7 / other source)                                                  |
|----------------------------------|---------------------------------------------------------------------------------------|
| id                               | `Encounter / id [@root]`                                                              |
| meta.profile\[0]                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"` |
| status                           | fixed value = `current`                                                               |
| mode                             | fixed value = `snapshot`                                                              |
| title                            | `Encounter / type`                                                                    |
| code.coding\[0].system           | fixed value = `http://snomed.info/sct`                                                |
| code.coding\[0].code             | fixed value = `325851000000107`                                                       |
| code.coding\[0].display          | fixed value = `Consultation`                                                          |
| subject                          | this a reference to mapped [Patient](../patient/README.md) from `Encounter /          |
| date                             | `Encounter / period [@value]` or from root `EhrExtract / availabilityTime / value`    |
| orderedBy.coding\[0].system      | fixed value = `http://hl7.org/fhir/list-order`                                        |
| orderedBy.coding\[0].code        | fixed value = `system`                                                                |
| orderedBy.coding\[0].display     | fixed value = `Sorted by System`                                                      |
| encounter                        | reference to mapped [Encounter](../practioners/README.md) `Encounter / type`          |
| entry[index].item.reference      | reference to one or more mapped [List (Topic)](./LIST_TOPIC_README.md)                |

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


| Mapped to (XML HL7)                                                                            | Mapped from (JSON FHIR / other source ) |
|------------------------------------------------------------------------------------------------|-----------------------------------------|
| ``                                                                                             |                                         |
| ``                                                                                             |                                         |
| ``                                                                                             |                                         |
| ``                                                                                             |                                         |
| ``                                                                                             |                                         |
| ``                                                                                             |                                         |
| ``                                                                                             |                                         |
