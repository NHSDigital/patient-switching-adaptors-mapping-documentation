# Organisation Mapping

## XML HL7 > JSON FHIR

Organisations are generated from the HL7v3 Agent Directory entries which represent either an `AgentOrganisation`
or an `AgentPerson` with an associated `representedOrganisation`.

### AgentOrganisation
| Mapped to (JSON FHIR Organisation resource field) | Mapped from (XML HL7 / other)                                       |
|---------------------------------------------------|---------------------------------------------------------------------|
| id                                                | `Agent / id [@root]`                                                |
| identifier\[0].system                             | fixed value = `"https://fhir.nhs.uk/Id/ods-organization-code"`      |   
| identifier\[0].value                              | `Agent / agentOrganisation / id [@root]`                            |
| name                                              | `Agent / agentOrganisation / name`                                  | 
| type\[0].text                                     | `Agent / code / originalText` or `Agent / code [@displayName]`      |
| address\[0].use                                   | fixed value = `"work"`                                              |
| address\[0].type                                  | fixed value = `"physical"`                                          |
| address\[0].line[]                                | `Agent / agentOrganisation / addr / streetAddressLine` <sup>1</sup> |
| address\[0].postalCode                            | `Agent / agentOrganisation / addr / postalCode`                     |
| telecom\[0].system                                | fixed value = `"phone"`                                             |
| telecom\[0].use                                   | fixed value = `"work"`                                              |
| telecom\[0].rank                                  | fixed value = `1`                                                   |
| telecom\[0].value                                 | `Agent / agentOrganisation / telecom` <sup>2</sup>                  |


### AgentPerson with representedOrganisation
| Mapped to (JSON FHIR Organisation resource field) | Mapped from (XML HL7 / other)                                             |
|---------------------------------------------------|---------------------------------------------------------------------------|
| id                                                | `Agent / id [@root]` appended with `"-ORG"` <sup>3</sup>                  |
| identifier\[0].system                             | fixed value = `"https://fhir.nhs.uk/Id/ods-organization-code"`            |
| identifier\[0].value                              | `Agent / representedOrganisation / id [@extention]`                       |
| name                                              | `Agent / representedOrganisation / name`                                  |
| address\[0].use                                   | fixed value = `"work"`                                                    |
| address\[0].type                                  | fixed value = `"physical"`                                                |
| address\[0].line[]                                | `Agent / representedOrganisation / addr / streetAddressLine` <sup>1</sup> |
| address\[0].postalCode                            | `Agent / representedOrganisation / addr / postalCode`                     |
| telecom\[0].system                                | fixed value = `"phone"`                                                   |
| telecom\[0].use                                   | fixed value = `"work"`                                                    |
| telecom\[0].rank                                  | fixed value = `1`                                                         |
| telecom\[0].value                                 | `Agent / representedOrganisation / telecom`  <sup>2</sup>                 |

1. A String is added to the line array for each streetAddressLine element. The first occurrence of `addr` is mapped, 
additional occurrences are ignored.
2. The first occurrence of `telecom` is mapped, additional occurrences are ignored.
3. Multiple practitioners could represent the same organisation. Therefore, the adaptor maps the first instance of 
a represented organisation, determined by ODS code, to an `Organisation` and links the relevant 
[Practitioners](../practioners/README.md) via `PractionerRole` resources. The `id` is appended with `"-ORG"` as the 
[Practitioner](../practioners/README.md) mapping also uses the value mapped from `Agent / id [@root]`.

If after generating the organisations above, the losing practice is not present in the Agent Directory and 
needs to be referenced as an author for a [Document Reference](../document%20references/README.md), an Organisation 
is generated with the losing practice ODS code.

### Generated Organisation
| Mapped to (JSON FHIR Organisation resource field) | Mapped from (XML HL7 / other)                                  |
|---------------------------------------------------|----------------------------------------------------------------|
| id                                                | A random UUID generated by the Adaptor                         |
| identifier\[0].system                             | fixed value = `"https://fhir.nhs.uk/Id/ods-organization-code"` |
| identifier\[0].value                              | The ODS code of the losing practice                            |

### Unmapped fields

The following Organisation fields are not currently populated by the adaptor: 

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

GP Connect FHIR Organisations are mapped to the HL7v3 Agent Directory.  

Where an Organisation is not a mapped as a `representedOrganisation` of an `AgentPerson` ([Practitioner](../practioners/README.md)), the adaptor maps the 
Organisation to an `AgentPerson` instead of `AgentOrganisation`. This is due to compatability issues where attribution 
was given to the Organisation as a performer.

### Organisation as an AgentPerson 
| Mapped to (XML HL7)         | Mapped from (JSON FHIR / other source) |
|-----------------------------|----------------------------------------|
| Agent / id \[@root]         | New UUID gnerated by the adaptor       |
| Agent / code \[@nullFlavor] | fixed value = `"UNK"`                  |
| Agent / code / originalText | fixed value = `Unknown`                |
| Agent / name / family       | `Organisation.name`                    |

### Represented Organisation
| Mapped to (XML HL7)                                        | Mapped from (JSON FHIR / other source)                                   |
|------------------------------------------------------------|--------------------------------------------------------------------------|
| Agent / representedOrganisation / name                     | `Organisation.name`                                                      | 
| Agent / representedOrganisation / telecom                  | `Organisation.telecom.value` <sup>4</sup>                                |
| Agent / representedOrganisation / addr / streetAddressLine | `Organisation.address.line` and `Organisation.address.city` <sup>5</sup> |
| Agent / representedOrganisation / addr / postalCode        | `Organisation.address.postalCode` <sup>5</sup>                           |  


4. The first instance of `Organisation.telecom` where `Organisation.telecom.system.display` is equal to `"phone"` and `Organisation.telecom.use` is equal to `"work"`   
5. Where `Organisation.address` is the first instance. 

## Further documentation
[GP Connect Organisation Structure definition](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Organization-1)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
