# List (Category) Mapping

## XML HL7 > JSON FHIR

A List (Topic) is created from a [List (Topic)](./LIST_TOPIC_README.md) member (referred to as `Topic`) and a `CompoundStatement` within the same `ehrComposition` which have a classCode of `CATEGORY`
This represents the SOAP heading sections which contain clinical record entries.

| Mapped to (JSON FHIR Referral Request field) | Mapped from (XML HL7 / other source)                                                                                                                       |
|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                           | `CompoundStatement / id [@root]`                                                                                                                           |
| meta.profile\[0]                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"`                                                                      |
| status                                       | fixed value = `current`                                                                                                                                    |
| mode                                         | fixed value = `snapshot`                                                                                                                                   |
| title                                        | `CompoundStatement / code/ originalText` or `CompoundStatement / code [@displayName]`                                                                      |
| code.coding\[0].system                       | fixed value = `http://snomed.info/sct`                                                                                                                     |
| code.coding\[0].code                         | fixed value = `24781000000107`                                                                                                                             |
| code.coding\[0].display                      | fixed value = `Category (EHR)`                                                                                                                             |
| subject                                      | this a reference to mapped [Patient](../patient/README.md) from `Topic.subject`                                                                            |
| date                                         | `CompoundStatement / AvailabilityTime [@value]` or from `Topic.date`                                                                                       |
| orderedBy.coding\[0].system                  | fixed value = `http://hl7.org/fhir/list-order`                                                                                                             |
| orderedBy.coding\[0].code                    | fixed value = `system`                                                                                                                                     |
| orderedBy.coding\[0].display                 | fixed value = `Sorted by System`                                                                                                                           |
| encounter                                    | this a reference to mapped [Encounter](../encounters/README.md) from `Topic.encounter`                                                                     |
| entry[index].item.reference                  | Each entry.item is a reference to a profile representing a clinical record entry in the source system - for example, medications, allergies, problems, etc |

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
        "id": "5a8d2ec6-807d-4db5-b6c0-a757bbfb5372",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"
            ]
        },
        "status": "current",
        "mode": "snapshot",
        "code": {
            "coding": [
                {
                    "system": "http://snomed.info/sct",
                    "code": "25851000000105",
                    "display": "Topic (EHR)"
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
                    "reference": "Observation/1E8A8478-A0C1-11ED-808B-AC162D1F16F0"
                }
            },
            {
                "item": {
                    "reference": "Condition/1E8A8479-A0C1-11ED-808B-AC162D1F16F0"
                }
            },
            {
                "item": {
                    "reference": "Observation/1E8A8480-A0C1-11ED-808B-AC162D1F16F0"
                }
            }
        ]
    }
}
```
</details>

## JSON FHIR > XML HL7