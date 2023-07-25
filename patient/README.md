# Patient Mapping

## XML HL7 > JSON FHIR

A Patient resource is mapped from a HL7 Patient.

| Mapped to (JSON FHIR Immunization field) | Mapped from (XML HL7 / other source)                                                                                                                                     |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                       | set value = Generated GUID                                                                                                                                               |
| meta.profile\[0]                         | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1"`.                                                                                |
| meta.versionId                           | fixed value = `1521806400000`                                                                                                                                            |
| indentifier\[0].system                   | fixed value = `https://fhir.nhs.uk/Id/nhs-number`                                                                                                                        |
| indentifier\[0].value                    | `Patient / id [@extension]`                                                                                                                                              |
| managingOrganization                     | referenced to mapped AgentOrganization: [Organization](https://github.com/NHSDigital/patient-switching-adaptors-mapping-documentation/blob/main/organisations/README.md) |

<details>
    <summary>Example JSON</summary>

```JSON
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

## Unmapped fields

 - implicitRules
 - language
 - text
 - contained
 - extension
 - modifierExtension
 - identifier.use
 - identifier.type
 - identifier.type.coding
 - active
 - name
 - gender
 - birthDate
 - deceasedDateTime
 - address
 - maritalStatus
 - multipleBirthBoolean 
 - generalPractitioner
 - link

## JSON FHIR > XML HL7

A Patient record is inserted into the EhrExtracts recordTarget.

| Mapped to (XML HL7)                                    | Mapped from (JSON FHIR / other source )                                      |
|--------------------------------------------------------|------------------------------------------------------------------------------|
| EhrExtract / recordTarget / patient / id \[@extension] | NHS number is taken from the initial EHR Request message (RCMR_IN010000UK05) |


<details>
    <summary>Example XML</summary>

```XML
<ControlActEvent classCode=\"CACT\" moodCode=\"EVN\">
<author1 typeCode=\"AUT\">
<AgentSystemSDS classCode=\"AGNT\">
<agentSystemSDS classCode=\"DEV\" determinerCode=\"INSTANCE\">
<id root=\"1.2.826.0.1285.0.2.0.107\" extension=\"200000000359\" />
        </agentSystemSDS>
        </AgentSystemSDS>
        </author1>
<subject typeCode=\"SUBJ\" contextConductionInd=\"false\">
<EhrExtract classCode=\"EXTRACT\" moodCode=\"EVN\">
<id root=\"4B3EC6C4-D9BD-4FFE-8C29-01A5EC41B9E9\" />
<statusCode code=\"COMPLETE\" />
<availabilityTime value=\"20230613105301\" />
<recordTarget typeCode=\"RCT\">
<patient classCode=\"PAT\">
<id root=\"2.16.840.1.113883.2.1.4.1\" extension=\"9726908744\" />
        </patient>
        </recordTarget>
    
... 
    
</EhrExtract>
```

</details>

## Further documentation

- [FHIR Patient](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
