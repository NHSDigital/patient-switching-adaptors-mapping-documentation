# List (Heading) Mapping

## XML HL7 > JSON FHIR

A List (Heading) is created from a [List (Topic)](./LIST_TOPIC_README.md) member (referred to as `Topic`) and a `CompoundStatement` within the same `ehrComposition` which has a classCode of `CATEGORY` (This is a descendent of the related Topic's `CompoundStatement`). 
This represents the SOAP heading sections (Subjective, Objective, Analysis, Plan) which contain clinical record entries.

| Mapped to (JSON FHIR List field) | Mapped from (XML HL7 / other source)                                                                                                                                             |
|----------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                               | `CompoundStatement / id [@root]`                                                                                                                                                 |
| meta.profile\[0]                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"`                                                                                            |
| status                           | fixed value = `current`                                                                                                                                                          |
| mode                             | fixed value = `snapshot`                                                                                                                                                         |
| title                            | `CompoundStatement / code/ originalText` or `CompoundStatement / code [@displayName]`                                                                                            |
| code.coding\[0].system           | fixed value = `http://snomed.info/sct`                                                                                                                                           |
| code.coding\[0].code             | fixed value = `24781000000107`                                                                                                                                                   |
| code.coding\[0].display          | fixed value = `Category (EHR)`                                                                                                                                                   |
| subject                          | this a reference to mapped [Patient](../patient/README.md) from `Topic.subject`                                                                                                  |
| date                             | `CompoundStatement / AvailabilityTime [@value]` or from `Topic.date`                                                                                                             |
| orderedBy.coding\[0].system      | fixed value = `http://hl7.org/fhir/list-order`                                                                                                                                   |
| orderedBy.coding\[0].code        | fixed value = `system`                                                                                                                                                           |
| orderedBy.coding\[0].display     | fixed value = `Sorted by System`                                                                                                                                                 |
| encounter                        | this a reference to mapped [Encounter](../encounters/README.md) from `Topic.encounter`                                                                                           |
| entry[index].item.reference      | Each entry.item is a reference to a resource representing a clinical record entry which has been mapped from the source HL7 - for example, medications, allergies, problems, etc |

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

Mapped from a `resource` with a type of `list` where `list.code.coding[0].code` is `25851000000105` to a `CompoundStatement`

| Mapped to (XML HL7 CompoundStatement)                                                                                                        | Mapped from (JSON FHIR / other source )                                                                                 |
|----------------------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| `CompoundStatement [@classCode] [@moodCode]`                                                                                                 | fixed value = `CATEGORY`, moodCode fixed value = `EVN`                                                                  |
| `CompoundStatement / id [@root]`                                                                                                             | new system generated UUID                                                                                               |
| `CompoundStatement / code [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15'] [@code='394841004'] [@displayName="Other category"] / originalText` | `resource.title` or not populated if not found                                                                          |
| `CompoundStatement / statusCode [@code]`                                                                                                     | fixed value = `COMPLETE`                                                                                                |
| `CompoundStatement / effectiveTime`                                                                                                          | from mapped [Encounter](../encounters/README.md) referenced in `list.encounter` <sup>1</sup><sup>2</sup><sup>3</sup>    |
| `CompoundStatement / availabiltyTime [@value]`                                                                                               | `list.date` or from mapped [Encounter](../encounters/README.md) referenced in `list.encounter` <sup>4</sup><sup>5</sup> |
| `CompoundStatement / components`                                                                                                             | contains one or more `CompoundStatements` mapped from 'list.entry' <sup>6</sup>                                         |

1. When `Encounter` has `encounter.period.start` and `encounter.period.end` then values are set `effectiveTime / lowValue` & `effectiveTime / highValue` using `encounter.period.start` and `encounter.period.end` respectively
2. When `Encounter` has `encounter.period.start` only, then `effectiveTime / center` is set by `encounter.period.start`
3. If there is no `encounter.period` then value `effectiveTime / center [@nullFlavour="UNK"]` is used
4. If `list.date` is not present and `encounter.period.start` is present then that value is used
5. if `list.date` and `encounter.period.start` are not present `availabilityTime [@nullFlavour="UNK"]` is used
6. each `list.entry` is mapped to `CompoundStatement` from the reference to the relevant clinical record

<details><summary>Example XML</summary>

```
<component typeCode="COMP" contextConductionInd="true">
    <CompoundStatement classCode="CATEGORY" moodCode="EVN">
        <id root="394559384658936" />
        <code code="14L..00" codeSystem="2.16.840.1.113883.2.1.6.2" displayName="H/O: drug allergy" />
        <statusCode code="COMPLETE" />
        <effectiveTime>
            <center value="19781231" />
        </effectiveTime>
        <availabilityTime value="19781231" />
        <component typeCode="COMP" contextConductionInd="true">
            <ObservationStatement classCode="OBS" moodCode="ENV">
                <id root="394559384658936" />
                <code nullFlavor="UNK">
                    <originalText>Mocked code</originalText>
                </code>
                <statusCode code="COMPLETE" />
                <effectiveTime>
                    <center value="19781231" />
                </effectiveTime>
                <availabilityTime value="19781231" />
                <pertinentInformation typeCode="PERT">
                    <sequenceNumber value="+1" />
                    <pertinentAnnotation classCode="OBS" moodCode="EVN">
                        <text>Reason Ended: Patient reports no subsequent recurrence on same
                            medication Status: Resolved
                            Type: Allergy Criticality: Low Risk Last Occurred: 1978-12-31 Example
                            note text
                        </text>
                    </pertinentAnnotation>
                </pertinentInformation>
            </ObservationStatement>
        </component>
    </CompoundStatement>
</component>
```

</details>

## Further documentation
[GP Connect List](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_list_consultation.html#list-heading)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 