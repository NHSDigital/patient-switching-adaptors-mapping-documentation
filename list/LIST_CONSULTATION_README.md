# List (Consultation) Mapping

## XML HL7 > JSON FHIR

A List (Consultation) is created from mapped [Encounters](../encounters/README.md). Where the EHR extract is used it is the root `EhrExtract` of the XML.

| Mapped to (JSON FHIR Referral Request field) | Mapped from (XML HL7 / other source)                                                  |
|----------------------------------------------|---------------------------------------------------------------------------------------|
| id                                           | `Encounter / id [@root]`                                                              |
| meta.profile\[0]                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"` |
| status                                       | fixed value = `current`                                                               |
| mode                                         | fixed value = `snapshot`                                                              |
| title                                        | `Encounter / type`                                                                    |
| code.coding\[0].system                       | fixed value = `http://snomed.info/sct`                                                |
| code.coding\[0].code                         | fixed value = `325851000000107`                                                       |
| code.coding\[0].display                      | fixed value = `Consultation`                                                          |
| subject                                      | this a reference to mapped [Patient](../patient/README.md) from `Encounter /          |
| dateElement                                  | `Encounter / period [@value]` or from root `EhrExtract / availabilityTime / value`    |
| subject.coding\[0].system                    | fixed value = `http://hl7.org/fhir/list-order`                                        |
| subject.coding\[0].code                      | fixed value = `system`                                                                |
| subject.coding\[0].display                   | fixed value = `Sorted by System`                                                      |
| Encounter                                    | reference to mapped [Encounter](../practioners/README.md) `Encounter / type`          |

