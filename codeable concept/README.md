# Codeable Concept Mapping

The mapping to / from the FHIR CodableConcept and HL7Concept Descriptor (CD) 
data types happens in serval places. Where stated in the documentation, 
the following method of mapping has been used.

## XML HL7 > JSON FHIR

`coding` contains an array of provided `code` and `code / translation` elements.

| Mapped to (JSON FHIR CodeableConcept)              | Mapped from (XML HL7 CD / other)                                                                       |                                                                                                                            
|----------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| coding\[].code                                    | `code [@code]` or `code / translation [@code]`  <sup>1</sup>                                           |
| coding\[].system                                  | `code [@codeSystem] <sup>1,2</sup>                                               | 
| coding\[].display                                 | `code [@displayName] <sup>1,3</sup>          |
| coding\[].extension\[0].url                       | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid"` <sup>1,4</sup> |
| coding\[].extension\[0].extension\[0].url         | fixed value = `"decriptionId"` <sup>1,4</sup>                                                           |
| coding\[].extension\[0].extension\[0].valueId     | found by searching the adaptors SNOMED database for the preferred description ID <sup>1,4</sup>          |
| coding\[].extension\[0].extension\[1].url         | fixed value = `"descriptionDisplay"` <sup>1,4</sup>                                                   |
| coding\[].extension\[0].extension\[1].valueString | found by searching the adaptors SNOMED database for the preferred description <sup>1,4</sup>            |
| text                                               | `code [@displayName]` or else `code / originalText`                                                    |

1. If no codes are found then `code.coding` will not be populated.
2. If this is a `Read V2`, `Read V3` or `SNOMED` code system then it will be mapped to its corresponding code system reference.
If there is no corresponding code system reference, and this is an OID (i.e. `1.2.3.4.5.6`) then this will be presented as
a URN and prefixed with `urn:oid:`.
3. If this is a `SNOMED` code it is found by searching the adaptors SNOMED database for the appropriate description.
4. If this is a not a 'SNOMED' code the `extension` will not be added

## JSON FHIR > XML HL7

| Mapped to (XML HL7 CD)            | Mapped from (JSON FHIR / other source )                                                                                                          |
|-----------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| code \[@code] <sup>5</sup>        | `coding.extension\[index].value` where `coding.extension[index].url` equals `"descriptionId"` or else `coding.code`                              |
| code \[@codeSystem] <sup>5</sup>  | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                 |
| code \[@displayName] <sup>5</sup> | `coding.extension[index].value` where `coding.extension[index].url` equals `"descriptionDisplay"` or else `coding.display`                       |
| code / originalText               | `coding.text` or else `coding.display` or else `coding.extension[index].value` where `coding.extension[index].url` equals `"descriptionDisplay"` |

5. If no SNOMED code is present, the code element will only have the attribute `nullFlavor="UNK"` i.e. unknown.   
