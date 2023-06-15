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

A `List (Consultation)` is mapped from references `resource` with a `resource.type` of `list` when mapping an [Encounter](../encounters/README.md) to an `EhrComposition` from a GP Connect FHIR `Encounter`

| Mapped to (XML HL7 Encounter)                   | Mapped from (JSON FHIR / other source )                                                                           |
|-------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| EhrComposition / Component [@typeCode="COMP"]`  | `resource` when the list contains a reference to `Encounter.id` and 'resource.resourceType` = `List` <sup>1</sup> |

1.  For each reference in `resource.entry.item.reference`, the resultant topic list are parsed and are mapped to a `Component` when the referenced resource `resource.resourceType` is one of the following clinic records:

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
        <component typeCode="COMP">
            <LinkSet classCode="OBS" moodCode="EVN">
                <id root="EDCE31A8-4D09-4093-80AC-5397F155ED55" />
                <code code="394774009" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                    displayName="Active Problem">
                    <originalText>Active Problem, major</originalText>
                    <qualifier inverted="false">
                        <name code="386134007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                            displayName="Significant" />
                    </qualifier>
                </code>
                <statusCode code="COMPLETE" />
                <effectiveTime>
                    <low value="20100119" />
                </effectiveTime>
                <availabilityTime value="20100119113634" />
                <component typeCode="COMP">
                    <statementRef classCode="OBS" moodCode="EVN">
                        <id root="368D4556-101B-4031-9F34-8F513CDC034E" />
                    </statementRef>
                </component>
                <component typeCode="COMP">
                    <statementRef classCode="OBS" moodCode="EVN">
                        <id root="2940BA4A-1DD6-4F65-9145-DA03EF572AD9" />
                    </statementRef>
                </component>
                <component typeCode="COMP">
                    <statementRef classCode="OBS" moodCode="EVN">
                        <id root="8BCB936E-C3CA-46E2-8F01-655F20F4C83E" />
                    </statementRef>
                </component>
                <component typeCode="COMP">
                    <statementRef classCode="OBS" moodCode="EVN">
                        <id root="4898ED73-44F1-4D68-B9F3-A3DA1ABA0FE5" />
                    </statementRef>
                </component>
                <conditionNamed typeCode="NAME" inversionInd="true">
                    <namedStatementRef classCode="OBS" moodCode="EVN">
                        <id root="EAF7104D-5957-40CE-81E9-1A02D2A3760E" />
                    </namedStatementRef>
                </conditionNamed>
                <Participant typeCode="PRF" contextControlCode="OP">
                    <agentRef classCode="AGNT">
                        <id root="686B63E5-EB8B-4353-8FF2-8387EFB38839" />
                    </agentRef>
                </Participant>
            </LinkSet>
        </component>
        <component typeCode="COMP">
            <ObservationStatement classCode="OBS" moodCode="EVN">
                <id root="EAF7104D-5957-40CE-81E9-1A02D2A3760E" />
                <code code="76953011" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                    displayName="Dry eye syndrome">
                </code>
                <statusCode code="COMPLETE" />
                <effectiveTime>
                    <low value="20100119" />

                </effectiveTime>
                <availabilityTime value="20100119" />
                <pertinentInformation typeCode="PERT">
                    <sequenceNumber value="+1" />
                    <pertinentAnnotation classCode="OBS" moodCode="EVN">
                        <text>Problem Info: Problem Notes: This is a dry
                            eyes problem</text>
                    </pertinentAnnotation>
                </pertinentInformation>
            </ObservationStatement>
        </component>
    </ehrComposition>
</component>
```
</details>

## Further documentation
[GP Connect List](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_list_consultation.html#list-consultation)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
