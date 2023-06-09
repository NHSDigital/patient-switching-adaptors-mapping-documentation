# Patient Mapping

## JSON FHIR > XML HL7

A Patient resource is mapped from a HL7 Patient.

| Mapped to (JSON FHIR Immunization field)        | Mapped from (XML HL7 / other source)                                                              |
|-------------------------------------------------|---------------------------------------------------------------------------------------------------|
| id                                              | set value = Generated GUID                                                                        |
| meta.profile\[0]                                | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1"`.         |
| meta.versionId                                  | fixed value = `1521806400000`                                                                     |
| indentifier\[0].system                          | fixed value = `https://fhir.nhs.uk/Id/nhs-number`                                                 |
| indentifier\[0].value                           | `Patient / id [@extension]`                                                                       |


## Example JSON

<details>
```
{
    "resource": {
        "resourceType": "Patient",
        "id": "82c11ce2-7d9f-49a7-930b-0a195e1e9775",
        "meta": {
            "versionId": "1521806400000",
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1"
            ]
        },
        "identifier": [
            {
                "system": "https://fhir.nhs.uk/Id/nhs-number",
                "value": "5538824210"
            }
        ],
    }
}
```
</details>


## XML HL7 > JSON FHIR

An Immunization is primarily mapped from an Observation Statement.

| Mapped to (XML HL7)                                                      | Mapped from (JSON FHIR / other source )              |
|--------------------------------------------------------------------------|------------------------------------------------------|
| ControlActEvent.[@id]                                                    | fixed value = `{{winning-asid}}`                     |
| ControlActEvent.subject.EhrExtract.[@id]                                 | fixed value = `F5C1FDBB-A948-43BB-AB30-CEAA51FC0CC0` |
| ControlActEvent.subject.EhrExtract.recordTarget.patient.[@id].extension  | fixed value = `{{nhs-number}}`                       |

## Example XML

<details>

```
<ControlActEvent classCode="CACT" moodCode="EVN">
        <author1 typeCode="AUT">
            <AgentSystemSDS classCode="AGNT">
                <agentSystemSDS classCode="DEV" determinerCode="INSTANCE">
                    <id root="1.2.826.0.1285.0.2.0.107" extension="{{winning-asid}}" />
                </agentSystemSDS>
            </AgentSystemSDS>
        </author1>
        <subject typeCode="SUBJ" contextConductionInd="false">
            <EhrExtract classCode="EXTRACT" moodCode="EVN">
                <id root="F5C1FDBB-A948-43BB-AB30-CEAA51FC0CC0" />
                <statusCode code="COMPLETE" />
                <availabilityTime value="20200101010101" />
                <recordTarget typeCode="RCT">
                    <patient classCode="PAT">
                        <id root="2.16.840.1.113883.2.1.4.1" extension="{{nhs-number}}" />
                    </patient>
                </recordTarget>
```
</details>

## Further documentation

[GP Connect Migrate a patient's structured record](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_migrate_patient_record.html)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
