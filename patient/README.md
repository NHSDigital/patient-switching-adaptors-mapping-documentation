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
    <summary>Example JSON</summary>

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

A Patient record is inserted into the EhrExtracts recordTarget.

| Mapped to (XML HL7)        | Mapped from (JSON FHIR / other source ) |
|----------------------------|-----------------------------------------|
| patient / id \[@extension] | Patients NHS number                     |

## Example XML

<details>
    <summary>Example XML</summary>

```
<EhrExtract classCode="EXTRACT" moodCode="EVN">
    <id root="F5C1FDBB-A948-43BB-AB30-CEAA51FC0CC0" />
    <statusCode code="COMPLETE" />
    <availabilityTime value="20200101010101" />
    <recordTarget typeCode="RCT">
        <patient classCode="PAT">
            <id root="2.16.840.1.113883.2.1.4.1" extension="9726908817" />
        </patient>
    </recordTarget>
    ...
</EhrExtract>
```
</details>

## Further documentation

[FHIR Patient](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1)
[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
