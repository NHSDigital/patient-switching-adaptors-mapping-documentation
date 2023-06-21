# Codeable Concept Mapping

The mapping to / from the FHIR CodableConcept and HL7Concept Descriptor (CD) 
data types happens in serval places. Where stated in the documentation, 
the following method of mapping has been used.

## XML HL7 > JSON FHIR

| Mapped to (JSON FHIR CodeableConcept)              | Mapped from (XML HL7 CD / other)                                                                       |                                                                                                                            
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| coding\[0].code                                    | `code [@code]` or `code / translation [@code]`  <sup>1</sup>                                           |
| coding\[0].system                                  | fixed value = `"http://snomed.info/sct"` <sup>1</sup>                                                  | 
| coding\[0].display                                 | found by searching the adaptors SNOMED database for the appropriate description <sup>1</sup>           |
| coding\[0].extension\[0].url                       | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid"` <sup>1</sup> |
| coding\[0].extension\[0].extension\[0].url         | fixed value = `"decriptionId"` <sup>1</sup>                                                            |
| coding\[0].extension\[0].extension\[0].valueId     | found by searching the adaptors SNOMED database for the preferred description ID <sup>1</sup>          |
| coding\[0].extension\[0].extension\[1].url         | fixed value = `"descriptionDisplay"` <sup>1</sup>                                                      |
| coding\[0].extension\[0].extension\[1].valueString | found by searching the adaptors SNOMED database for the preferred description <sup>1</sup>             |
| text                                               | `code [@displayName]` or else `code / originalText`                                                    |

1. If a SNOMED CT code cannot be found `code.coding` will not be populated.

## JSON FHIR > XML HL7

| Mapped to (XML HL7 CD)            | Mapped from (JSON FHIR / other source )                                                                                                          |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| code \[@code] <sup>2</sup>        | `coding.extension\[index].value` where `coding.extension[index].url` equals `"descriptionId"` or else `coding.code`                              |
| code \[@codeSystem] <sup>2</sup>  | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                 |
| code \[@displayName] <sup>2</sup> | `coding.extension[index].value` where `coding.extension[index].url` equals `"descriptionDisplay"` or else `coding.display`                       |
| code / originalText               | `coding.text` or else `coding.display` or else `coding.extension[index].value` where `coding.extension[index].url` equals `"descriptionDisplay"` |

2. If no SNOMED code is present, the code element will only have the attribute `nullFlavor="UNK"` i.e. unknown.   
