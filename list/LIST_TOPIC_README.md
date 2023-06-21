# List (Topic) Mapping

## XML HL7 > JSON FHIR

A List (Topic) is created from a [List (Consultation)](./LIST_CONSULTATION_README.md) member (referred to as `Consultation`) and one or more `CompoundStatement`s within the same `ehrComposition` which have a classCode of `TOPIC`
This represents Topic / Problem groupings within consultations

| Mapped to (JSON FHIR List field) | Mapped from (XML HL7 / other source)                                                          |
|----------------------------------|-----------------------------------------------------------------------------------------------|
| id                               | `CompoundStatement / id [@root]`                                                              |
| meta.profile\[0]                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"`         |
| status                           | fixed value = `current`                                                                       |
| mode                             | fixed value = `snapshot`                                                                      |
| title                            | `CompoundStatement / code/ originalText` or `CompoundStatement / code [@displayName]`         |
| code.coding\[0].system           | fixed value = `http://snomed.info/sct`                                                        |
| code.coding\[0].code             | fixed value = `25851000000105`                                                                |
| code.coding\[0].display          | fixed value = `Topic (EHR)`                                                                   |
| subject                          | this a reference to mapped [Patient](../patient/README.md) from `Consultation.subject`        |
| date                             | `CompoundStatement / AvailabilityTime [@value]` or from `Consultation.date`                   |
| orderedBy.coding\[0].system      | fixed value = `http://hl7.org/fhir/list-order`                                                |
| orderedBy.coding\[0].code        | fixed value = `system`                                                                        |
| orderedBy.coding\[0].display     | fixed value = `Sorted by System`                                                              |
| encounter                        | this a reference to mapped [Encounter](../encounters/README.md) from `Consultation.encounter` |
| entry[index].item.reference      | reference to one or more mapped [List (Heading)](./LIST_TOPIC_README.md) <sup>1</sup>         |

1. Where information within the Topic is organised as subheadings, `entry.list` will reference instances of the `List(Heading)` level.</br>
For consultations which have a flat structure (for example, clinical record entries made outside the Topic and heading structure), an artificial Topic List is generated, and entries will reference resource representing those record entries (such as, Allergies, Medications, Tests, ...).

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

| Mapped to (XML HL7 Compound Statement)                                                           | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                                                                                         |
|--------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `CompoundStatement [@classCode] [@moodCode]`                                                     | classCode fixed value = `TOPIC`, moodCode fixed value = `EVN`                                                                                                                                                                                                                                                                   |
| `CompoundStatement / id [@root]`                                                                 | new system generated UUID                                                                                                                                                                                                                                                                                                       |
| `CompoundStatement / code [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15'] [@code] [@displayName]` | If there is a `resource.extension` which is a related problem and `resource.extension.extension.url` is `target` then the values are set from the coding block in that extension.</br> If there are no related problems then a default value for `[@code]` of `394776006` and `[@displayName]` of `Unspecified problem` is used |
| `CompoundStatement / code / OriginalText`                                                        | `resource.title` or `resource.code.coding[0].text` or `resource.code.coding[0].display`                                                                                                                                                                                                                                         |
| `CompoundStatement / statusCode [@code]`                                                         | fixed value = `COMPLETE`                                                                                                                                                                                                                                                                                                        |
| `CompoundStatement / effectiveTime`                                                              | from mapped [Encounter](../encounters/README.md) referenced in `list.encounter` <sup>1</sup><sup>2</sup><sup>3</sup>                                                                                                                                                                                                            |
| `CompoundStatement / availabiltyTime [@value]`                                                   | `list.date` or from mapped [Encounter](../encounters/README.md) referenced in `list.encounter` <sup>4</sup><sup>5</sup>                                                                                                                                                                                                         |
| `CompoundStatement / components`                                                                 | contains one or more `CompoundStatements` mapped from 'list.entry' <sup>6</sup>                                                                                                                                                                                                                                                 |

1. When `Encounter` has `encounter.period.start` and `encounter.period.end` then values are set `effectiveTime / lowValue` & `effectiveTime / highValue` using `encounter.period.start` and `encounter.period.end` respectively 
2. When `Encounter` has `encounter.period.start` only, then `effectiveTime / center` is set by `encounter.period.start`
3. If there is no `encounter.period` then value `effectiveTime / center [@nullFlavour="UNK"]` is used
4. If `list.date` is not present and `encounter.period.start` is present then that value is used 
5. if `list.date` and `encounter.period.start` are not present `availabilityTime [@nullFlavour="UNK"]` is used
6. each `list.entry` is mapped to `CompoundStatement` from a [List (Heading)](./LIST_HEADING_README.md) when `list.entry.reference` has a code of `24781000000107` and, when in a flat structure is mapped from the reference to the relevant clinical record:

* [AllergyIntolerance](../allergy%20intolerances/README.md)
* [Condition](../conditions/README.md)
* [DocumentReference](../document%20references/README.md)
* [Immunization](../immunisations/README.md)
* [MedicationRequest](../medication%20requests/README.md)
* [Observation](../observations/README.md)
* [ProcedureRequest](../procedure%20requests/README.md)
* [ReferralRequest](../referral%20requests/README.md)
* [DiagnosticReport](../diagnostic%20reports/README.md)


<details><summary>Example XML</summary>

```
<component typeCode="COMP">
    <CompoundStatement classCode="TOPIC" moodCode="EVN">
        <id root="394559384658936" />
        <code nullFlavor="UNK">
            <originalText>Mocked code</originalText>
        </code>
        <statusCode code="COMPLETE" />
        <effectiveTime>
            <low value="20100113152000" />
            <high value="20100113162000" />
        </effectiveTime>
        <availabilityTime value="20100123140354" />

        <component typeCode="COMP" contextConductionInd="true">
            <CompoundStatement classCode="CATEGORY" moodCode="EVN">
                <id root="394559384658936" />
                <code nullFlavor="UNK">
                    <originalText>Mocked code</originalText>
                </code>
                <statusCode code="COMPLETE" />
                <effectiveTime>
                    <low value="20100113152000" />
                    <high value="20100113162000" />
                </effectiveTime>
                <availabilityTime value="20100714163232" />

                <component typeCode="COMP">
                    <PlanStatement classCode="OBS" moodCode="INT">
                        <id root="394559384658936" />
                        <code nullFlavor="UNK">
                            <originalText>Mocked code</originalText>
                        </code>
                        <statusCode code="COMPLETE" />
                        <effectiveTime>
                            <center nullFlavor="UNK" />
                        </effectiveTime>
                        <availabilityTime value="20100113152950" />
                    </PlanStatement>
                </component>

                <component typeCode="COMP">
                    <NarrativeStatement classCode="OBS" moodCode="EVN">
                        <id root="394559384658936" />
                        <text>observation comment</text>
                        <statusCode code="COMPLETE" />
                        <availabilityTime value="20100113152950" />
                    </NarrativeStatement>
                </component>

                <component typeCode="COMP">
                    <LinkSet classCode="OBS" moodCode="EVN">
                        <id root="394559384658936" />
                        <code code="394774009" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                            displayName="Active Problem">
                            <originalText>Active Problem, minor</originalText>
                            <qualifier inverted="false">
                                <name code="394847000" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                                    displayName="Unspecified significance" />
                            </qualifier>
                        </code>
                        <statusCode code="COMPLETE" />
                        <effectiveTime>
                            <low value="20200906" />
                            <high value="20201004000000" />
                        </effectiveTime>
                        <availabilityTime value="20200907101202" />
                        <component typeCode="COMP">
                            <statementRef classCode="OBS" moodCode="EVN">
                                <id root="394559384658936" />
                            </statementRef>
                        </component>
                        <component typeCode="COMP">
                            <statementRef classCode="OBS" moodCode="EVN">
                                <id root="394559384658936" />
                            </statementRef>
                        </component>
                        <component typeCode="COMP">
                            <statementRef classCode="OBS" moodCode="EVN">
                                <id root="394559384658936" />
                            </statementRef>
                        </component>
                        <conditionNamed typeCode="NAME" inversionInd="true">
                            <namedStatementRef classCode="OBS" moodCode="EVN">
                                <id root="394559384658936" />
                            </namedStatementRef>
                        </conditionNamed>
                        <Participant typeCode="PRF" contextControlCode="OP">
                            <agentRef classCode="AGNT">
                                <id root="394559384658936" />
                            </agentRef>
                        </Participant>
                    </LinkSet>
                </component>

            </CompoundStatement>
        </component>

        <component typeCode="COMP">
            <ObservationStatement classCode="OBS" moodCode="EVN">
                <id root="394559384658936" />
                <code nullFlavor="UNK">
                    <originalText>Mocked code</originalText>
                </code>
                <statusCode code="COMPLETE" />
                <effectiveTime>
                    <center value="20100630055900" />
                </effectiveTime>
                <availabilityTime value="20100630055900" />
                <pertinentInformation typeCode="PERT">
                    <sequenceNumber value="+1" />
                    <pertinentAnnotation classCode="OBS" moodCode="EVN">
                        <text>Primary Source: true immunization note</text>
                    </pertinentAnnotation>
                </pertinentInformation>
                <Participant typeCode="PRF" contextControlCode="OP">
                    <agentRef classCode="AGNT">
                        <id root="394559384658936" />
                    </agentRef>
                </Participant>
            </ObservationStatement>
        </component>
    </CompoundStatement>
</component>
```
</details>

## Further documentation
[GP Connect List](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_list_consultation.html#list-topic)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 