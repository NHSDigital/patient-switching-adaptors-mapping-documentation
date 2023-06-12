# List (Consultation) Mapping

## XML HL7 > JSON FHIR

A List (Consultation) is mapped from an [Encounter](../encounters/README.md) 

| Mapped to (JSON FHIR Referral Request field) | Mapped from (XML HL7 / other source)                                                                         |
|----------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| id                                           | `Encounter / id [@root]`                                                                                     |
| meta.profile\[0]                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"`                        |
| status                                       | fixed value = `current`                                                                                      |
| mode                                         | fixed value = `snapshot`                                                                                     |
| title                                        | `Encounter / type`                                                                                           |
| code                                         | `                                                                                                             |
| subject                                      | `Encounter / subject`                                                                                        |
| dateElement                                  | `Encounter / period [@value]` or from root `EhrExtract / availabilityTime / value`                           |
| orderedBy                                    |                                                                                                              |
| Encounter                                    | reference to mapped [Encounter](../practioners/README.md) `Encounter`                                        |
| reasonCode                                   | mapped CodeableConcept from `RequestStatement / code` <sup>1</sup>                                           |
| note                                         | mapped Annotation from `RequestStatement / text` & `RequestStatement / priorityCode` & `RequestStatement / priorityCode` |
