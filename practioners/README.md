# Practitioner Mapping

## XML HL7 > JSON FHIR

The FHIR `Practitioner` and `PractitionerRole` resources are mapped from the HL7v3 `AgentDirectory` where the `Agent` is an `AgentPerson`. 

### Practitioner

| Mapped to (JSON FHIR Practitioner resource field) | Mapped from (XML HL7 / other)                                                                 |
|---------------------------------------------------|-----------------------------------------------------------------------------------------------|
| id                                                | `Agent / id [@root]`                                                                          |
| meta.profile\[0]                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Practitioner-1"` |
| name\[0].use                                      | fixed value = `official`                                                                      |
| name\[0].family                                   | `Agent / AgentPerson / name / family` or else `"unknown"`                                     |
| name\[0].given\[0]                                | `Agent / AgentPerson / name / given`                                                          |
| name\[0].prefix\[0]                               | `Agent / AgentPerson / name / prefix`                                                         |

<details>
    <summary>Example JSON</summary>

```
{
    "resource": {
        "resourceType": "Practitioner",
        "id": "C5DEFBF3-0174-BC6F-182C-B777B9C6FF43",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Practitioner-1"
            ]
        },
        "name": [
            {
                "use": "official",
                "family": "Doe",
                "given": [
                    "John"
                ],
                "prefix": [
                    "Dr"
                ]
            }
        ]
    }
}
```

</details>

### Unmapped fields

The following `Practitioner` fields are not currently populated by the adaptor:

- implicitRules
- language
- text
- contained
- extension
- modifierExtension
- identifier
- active
- name.id
- name.text
- name.suffix
- name.period
- telecom
- address
- gender
- birthDate
- photo
- qualification

### PractitionerRole

A `PractionerRole` resource will only be added if the HL7 `AgentPerson` has an associated `representedOrganization`.

| Mapped to (JSON FHIR PractitionerRole resource field) | Mapped from (XML HL7 / other)                                                                             |
|-------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|
| id                                                    | `Agent / id [@root]` appended with `-PR`                                                                  |
| meta                                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-PractitionerRole-1"`         |
| practitioner.reference                                | Reference to the mapped Practitioner (see above)                                                          |
| organization.reference                                | Reference to the [Organisation](../organisations/README.md) mapped from `Agent / representedOrganization` |
| code\[0].text                                         | `Agent / code / orginalText` or else `Agent / code [@displayName]`                                        |

<details>
    <summary>Example JSON</summary>

```
{
    "resource": {
        "resourceType": "PractitionerRole",
        "id": "94F00D99-0601-4A8E-AD1D-1B564307B0A6-PR",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-PractitionerRole-1"
            ]
        },
        "practitioner": {
            "reference": "Practitioner/94F00D99-0601-4A8E-AD1D-1B564307B0A6"
        },
        "organization": {
            "reference": "Organization/94F00D99-0601-4A8E-AD1D-1B564307B0A6-ORG"
        },
        "code": [
            {
                "text": "General Medical Practitioner"
            }
        ]
    }
}
```

</details>

### Unmapped fields

The following `PractitionerRole` fields are not currently populated by the adaptor:

- implicitRules
- language
- contained
- modifierExtension
- identifier
- active
- period
- practitioner.id
- practitioner.identifier
- practitioner.display
- organization.id
- organization.identifier
- organization.display
- code.id
- code.coding
- specialty
- location
- healthcareService
- telecom
- availableTime
- notAvailable
- availabilityExceptions
- endpoint

## JSON FHIR > XML HL7

A Practitioner is mapped to an `Agent` with a role of `AgentPerson`.  

| Mapped to (XML HL7)                 | Mapped from (JSON FHIR / other source )                                                                              |
|-------------------------------------|----------------------------------------------------------------------------------------------------------------------|
| Agent / id \[@root]                 | New UUID generated by the adaptor                                                                                    |
| Agent / code \[@nullFlavor]         | fixed value = `"UNK"`                                                                                                |
| Agent / code / originalText         | `PractionerRole.code[0].coding[0].display` <sup>1</sup>                                                              | 
| Agent / AgentPerson / name          | `Practitioner.name[0].text` <sup>2</sup>                                                                             |
| Agent / AgentPerson / name / prefix | `Practitioner.name[0].prefix[0]`                                                                                     |
| Agent / AgentPerson / name / given  | `Practitioner.name[0].given[0]`                                                                                      | 
| Agent / AgentPerson / name / family | `Practitioner.name[0].family` or `"Unknown"` <sup>2</sup>                                                            |
| Agent / representedOrganisation     | Mapped using the [Organisation](../organisations/README.md) referenced in `PractionerRole` (if present) <sup>3</sup> |  

1. Where the `practitionerRole` references the practitioner 
2. If `Practitioner.name[0].given` or `Practitioner.name[0].family` are empty and `Practitioner.name[0].text` has a string as a value, then 
`Agent / AgentPerson / name` is populated with the unstructured string. In the case where all three JSON fields are missing / empty
`Agent / AgentPerson / name / family` is set to `"Unknown"`.
3. The mapping of `representedOrganisation` is outlined in the [Organisation](../organisations/README.md) mapping documentation.

<details>
    <summary>Example XML</summary>

```
 <Agent classCode="AGNT">
    <id root="BDB45F13-D71B-474B-9A12-BB39A53B6273"/>
    <code nullFlavor="UNK">
        <originalText>General Medical Practitioner</originalText>
    </code>
    <agentPerson classCode="PSN" determinerCode="INSTANCE">
        <name>
            <prefix>Mr</prefix>
            <given>NHS</given>
            <family>Test</family>
        </name>
    </agentPerson>
    <representedOrganization classCode="ORG" determinerCode="INSTANCE">
        <name>TEMPLE SOWERBY MEDICAL PRACTICE</name>
        <telecom use="WP" value="tel:01133800000"/>
        <addr use="WP">
            <streetAddressLine>Fulford Grange</streetAddressLine>
            <streetAddressLine>Micklefield Lane</streetAddressLine>
            <streetAddressLine>Rawdon</streetAddressLine>
            <streetAddressLine>Rawdon</streetAddressLine>
            <streetAddressLine>Leeds</streetAddressLine>
            <postalCode>LS19 6BA</postalCode>
        </addr>
    </representedOrganization>
</Agent>
```
</details>

## Further documentation
[GP Connect Practitioner structure definition](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Practitioner-1)

[GP Connect PractitionerRole structure definition](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-PractitionerRole-1)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
