# Organization Mapping

## XML HL7 > JSON FHIR

Organizations are generated from the HL7v3 Agent Directory entries which represent either an `AgentOrganization`
or an `AgentPerson` with an associated `representedOrganization`.

### AgentOrganization
| Mapped to (JSON FHIR Organization resource field) | Mapped from (XML HL7 / other)                                                                                |
|---------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| id                                                | `Agent / id [@root]`                                                                                         |
| meta.profile\[0]                                  | fixed value = `https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1`                  |
| identifier\[0].system                             | fixed value = `"https://fhir.nhs.uk/Id/ods-organization-code"`                                               |   
| identifier\[0].value                              | `Agent / agentOrganization / id [@root]`                                                                     |
| name                                              | `Agent / agentOrganization / name`                                                                           | 
| type                                              | `Agent / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md) |
| address\[0].use                                   | fixed value = `"work"`                                                                                       |
| address\[0].type                                  | fixed value = `"physical"`                                                                                   |
| address\[0].line[]                                | `Agent / agentOrganization / addr / streetAddressLine` <sup>1</sup>                                          |
| address\[0].postalCode                            | `Agent / agentOrganization / addr / postalCode`                                                              |
| telecom\[0].system                                | fixed value = `"phone"`                                                                                      |
| telecom\[0].use                                   | fixed value = `"work"`                                                                                       |
| telecom\[0].rank                                  | fixed value = `1`                                                                                            |
| telecom\[0].value                                 | `Agent / agentOrganization / telecom[0] [@value]`                                                            |

<details>
<summary>Example JSON</summary>

```
{
    "resource": {
        "resourceType": "Organization",
        "id": "4D60F559-16D2-43CD-88FD-E779B43D915E",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1"
            ]
        },
        "identifier": [
            {
                "system": "https://fhir.nhs.uk/Id/ods-organization-code",
                "value": "A86005"
            }
        ],
        "type": [
            {
                "coding": [
                    {
                        "system": "http://snomed.info/sct",
                        "code": "394745000",
                        "display": "General practice (organization)"
                    }
                ]
            }
        ],
        "name": "West Farm Surgery",
        "telecom": [
            {
                "system": "phone",
                "value": "01132050080",
                "use": "work",
                "rank": 1
            }
        ],
        "address": [
            {
                "use": "work",
                "type": "physical",
                "line": [
                    "31 West Farm Avenue",
                    "Newcastle Upon Tyne",
                    "Tyne and Wear"
                ],
                "postalCode": "NE12 8LS"
            }
        ]
    }
}

```
</details>

### AgentPerson with representedOrganization
| Mapped to (JSON FHIR Organization resource field) | Mapped from (XML HL7 / other)                                                               |
|---------------------------------------------------|---------------------------------------------------------------------------------------------|
| id                                                | `Agent / id [@root]` appended with `"-ORG"` <sup>2</sup>                                    |
| meta.profile\[0]                                  | fixed value = `https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1` |
| identifier\[0].system                             | fixed value = `"https://fhir.nhs.uk/Id/ods-organization-code"`                              |
| identifier\[0].value                              | `Agent / representedOrganization / id [@extention]`                                         |
| name                                              | `Agent / representedOrganization / name`                                                    |
| address\[0].use                                   | fixed value = `"work"`                                                                      |
| address\[0].type                                  | fixed value = `"physical"`                                                                  |
| address\[0].line[]                                | `Agent / representedOrganization / addr / streetAddressLine` <sup>1</sup>                   |
| address\[0].postalCode                            | `Agent / representedOrganization / addr / postalCode`                                       |
| telecom\[0].system                                | fixed value = `"phone"`                                                                     |
| telecom\[0].use                                   | fixed value = `"work"`                                                                      |
| telecom\[0].rank                                  | fixed value = `1`                                                                           |
| telecom\[0].value                                 | `Agent / representedOrganization / telecom[0] [@value]`                                     |

<details>
<summary>Example JSON</summary>

```
 {
    "resource": {
        "resourceType": "Organization",
        "id": "1E8A8446-A0C1-11ED-808B-AC162D1F16F0-ORG",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1"
            ]
        },
        "identifier": [
            {
                "system": "https://fhir.nhs.uk/Id/ods-organization-code",
                "value": "A86005"
            }
        ],
        "name": "West Farm Surgery",
        "telecom": [
            {
                "system": "phone",
                "value": "01132050080",
                "use": "work",
                "rank": 1
            }
        ],
        "address": [
            {
                "use": "work",
                "type": "physical",
                "line": [
                    "31 West Farm Avenue",
                    "Newcastle Upon Tyne",
                    "Tyne and Wear"
                ],
                "postalCode": "NE12 8LS"
            }
        ]
    }
}

```
</details>

If after generating the organizations above, the losing practice is not present in the Agent Directory and
needs to be referenced as an author for a [Document Reference](../document%20references/README.md), an Organization
is generated with the losing practice ODS code.

### Generated Organization
| Mapped to (JSON FHIR Organization resource field) | Mapped from (XML HL7 / other)                                  |
|---------------------------------------------------|----------------------------------------------------------------|
| id                                                | A random UUID generated by the Adaptor                         |
| identifier\[0].system                             | fixed value = `"https://fhir.nhs.uk/Id/ods-organization-code"` |
| identifier\[0].value                              | The ODS code of the losing practice                            |

<details>
<summary>Example JSON</summary>

```
{
    "resource": {
        "resourceType": "Organization",
        "id": "5f536500-f139-4524-a649-70016869a00f",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1"
            ]
        },
        "identifier": [
            {
                "system": "https://fhir.nhs.uk/Id/ods-organization-code",
                "value": "D5445"
            }
        ]
    }
}

```
</details>

1. A String is added to the line array for each streetAddressLine element. The first occurrence of `addr` is mapped,
   additional occurrences are ignored.
2. The first occurrence of `telecom` is mapped, additional occurrences are ignored.
3. Multiple practitioners could represent the same organization. Therefore, the adaptor maps the first instance of
   a represented organization, determined by ODS code, to an `Organization` and links the relevant
   [Practitioners](../practioners/README.md) via `PractionerRole` resources. The `id` is appended with `"-ORG"` as the
   [Practitioner](../practioners/README.md) mapping also uses the value mapped from `Agent / id [@root]`.

### Unmapped fields

The following Organization fields are not currently populated by the adaptor: 

- implicitRules
- language
- contained
- extension
- identifier.id
- identifier.use
- identifier.type
- identifier.type.id
- identifier.period
- identifier.assigner
- active
- type.id
- type.coding
- alias
- telecom.id
- telecom.period
- address.id
- address.text
- address.city
- address.district
- address.country
- address.period
- partOf
- contact
- endpoint

## JSON FHIR > XML HL7

GP Connect FHIR Organizations are mapped to the HL7v3 Agent Directory.  

Where an Organization is not a mapped as a `representedOrganization` of an `AgentPerson` ([Practitioner](../practioners/README.md)), the adaptor maps the 
Organization to an `AgentPerson` instead of `AgentOrganization`. This is due to compatability issues where attribution 
was given to the Organization as a performer.

### Organization as the Agent (AgentPerson)
| Mapped to (XML HL7)                 | Mapped from (JSON FHIR / other source)  |
|-------------------------------------|-----------------------------------------|
| Agent / id \[@root]                 | New UUID generated by the adaptor       |
| Agent / code \[@nullFlavor]         | fixed value = `"UNK"`                   |
| Agent / code / originalText         | fixed value = `Unknown`                 |
| Agent / agentPerson / name / family | `Organization.name`                     |

<details>
    <summary>Example XML</summary>

```
<Agent classCode="AGNT">
    <id root="38DED72A-6D4B-4838-8751-8498E579AF03"/>
    <code nullFlavor="UNK">
        <originalText>Unknown</originalText>
    </code>
    <agentPerson classCode="PSN" determinerCode="INSTANCE">
        <name>
            <family>TEMPLE SOWERBY MEDICAL PRACTICE</family>
        </name>
    </agentPerson>
</Agent>
```
</details>

### Represented Organization where a practitioner is the Agent (AgentPerson) 
| Mapped to (XML HL7)                                        | Mapped from (JSON FHIR / other source)                            |
|------------------------------------------------------------|-------------------------------------------------------------------|
| Agent / representedOrganization / name                     | `Organization.name`                                               | 
| Agent / representedOrganization / telecom \[@value]        | `Organization.telecom.value` <sup>3</sup>                         |
| Agent / representedOrganization / telecom \[@use]          | fixed value = `"WP"`                                              |
| Agent / representedOrganization / addr / streetAddressLine | `Organization.address[0].line` and `Organization.address[0].city` |
| Agent / representedOrganization / addr / postalCode        | `Organization.address[0].postalCode`                              |  

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

3. The first instance of `Organization.telecom` where `Organization.telecom.system.display` is equal to `"phone"` and `Organization.telecom.use` is equal to `"work"`

## Further documentation

- [GP Connect Organization structure definition](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
