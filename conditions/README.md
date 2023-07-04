# Condition Mapping

## XML HL7 > JSON FHIR

A FHIR Condition is mapped from a LinkSet, A LinkSet references different resources that are mapped from an Observation statement and Ehr Composition.

The LinkSet will gather the codes/values/naming conventions from the Ehr Composition. Therefore, the LinkSet will then map out the fields below if certain conditions are met, otherwise it will add in a default value. 

| Mapped to (JSON FHIR Condition field)                  | Mapped from (XML HL7 / other source)                                                                                                                                                               |
|--------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                     | `LinkSet / id [@root]`                                                                                                                                                                             |
| meta.profile                                           | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProblemHeader-Condition-1"`                                                                                           |
| extension[0].url                                       | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ProblemSignificance-1"`                                                                                     |
| extension[0].valueCode                                 | If major is set from the LinkSet then add `"major"` as a value, otherwise add `"minor"`                                                                                                            |
| extension[index].url                                   | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1"` <sup>1</sup>                                                                              |
| extension[index].valueReference.reference              | reference to the mapped resource <sup>1</sup>                                                                                                                                                      |
| extension[index].url                                   | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedClinicalContent-1"` <sup>2</sup>                                                                     |
| extension[index].valueReference.reference              | reference to the mapped resource <sup>2</sup>                                                                                                                                                      |
| extension[index].url                                   | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedProblemHeader-1"`  <sup>3</sup>                                                                      |
| extension[index].extension[0].url                      | fixed value = `"type"` <sup>3</sup>                                                                                                                                                                |                                                                                                                                                                                          |
| extension[index].extension[0].valueCode                | either `"child"` if `LinkSet.component.statementRef.id [@root]` references another Link Set or `"parent"` if the mapped Link Set is referenced by another Link Set in the same manner <sup>3</sup> |
| extension[index].extension[1].url                      | fixed value = `"target"` <sup>3</sup>                                                                                                                                                              |
| extension[index].extension[1].valueReference.reference | Reference to the child / parent Condition  <sup>3</sup>                                                                                                                                            |
| identifier[0].system                                   | "https://PSSAdaptor/{{odsCode}}}" - where {{osdCode}} is the ODS code of the losing practice                                                                                                       |
| identifier[0].value                                    | `LinkSet / id [@root]`                                                                                                                                                                             |
| clinicalStatus                                         | If `LinkSet / code [@code]` equals `394775005` then `inactive` otherwise `active`                                                                                                                  |
| category[0].coding[0].system                           | fixed value = "https://fhir.hl7.org.uk/STU3/CodeSystem/CareConnect-ConditionCategory-1"                                                                                                            |
| category[0].coding[0].code                             | fixed value = `problem-list-item`                                                                                                                                                                  |
| category[0].coding[0].display                          | fixed value = `Problem List Item`                                                                                                                                                                  |
| code                                                   | mapped from `ObservationSatement / code` where the `Observationstatement` is referenced in `LinkSet / component / statementRef / id [@root]`                                                       |
| subject.reference                                      | reference to the mapped [Patient](../patient/README.md)                                                                                                                                            |
| context.reference                                      | reference to the associated [Encounter](../encounters/README.md)                                                                                                                                   |
| onsetDateTime                                          | `LinkSet / effectiveTime / low [@value]` or else `LinkSet / effectiveTime / center [@value]` or else `LinkSet / availabilitytime [@value]`                                                         |
| abatementDateTime                                      | `LinkSet / effectiveTime / high [@value]`                                                                                                                                                          |
| assertedDate                                           | `ehrComposition / author / time [@value]` or else `EhrExtract / availabilityTime [@value]`                                                                                                         |
| asserter.reference                                     | [Practitioner](../practioners/README.md) referenced by `ehrComposition / participant2[0] / agentRef / id [@root]`                                                                                  |
| note.text[index]                                       | fixed value = `"Defaulted status to active : Unknown status at source"` if `linkSet / code [@code]` does not equal `394774009` (active) or `394775005` (inactive)                                  |
| note.text[index]                                       | fixed value = `"Unspecified Significance: Defaulted to Minor"` if `LinkSet / code / qualifier / name [@code]` does not equal `386134007`                                                           |
| note.text[index]                                       | `ObservationStatement / pertinentInformation / pertinentAnnotation / text` where the `observationStatement` is referenced by `LinkSet / conditionNamed / namedStatementRef / id [@root]`           |


<details>
    <summary>Example JSON</summary>

```JSON
        {
  "resource": {
    "resourceType": "Condition",
    "id": "BF627285-8E57-46C7-BBAF-27AFBC7C23B8",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProblemHeader-Condition-1"
      ]
    },
    "extension": [
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ProblemSignificance-1",
        "valueCode": "minor"
      },
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1",
        "valueReference": {
          "reference": "AllergyIntolerance/04288662-8B7A-4350-B69B-CE155E992A7C"
        }
      },
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedClinicalContent-1",
        "valueReference": {
          "reference": "Condition/0A8290DF-1060-4C61-99FC-D0542B8A8693"
        }
      },
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedProblemHeader-1",
        "extension": [
          {
            "url": "type",
            "valueCode": "child"
          },
          {
            "url": "target",
            "valueReference": {
              "reference": "Condition/0A8290DF-1060-4C61-99FC-D0542B8A8693"
            }
          }
        ]
      }
    ],
    "identifier": [
      {
        "system": "https://PSSAdaptor/B83002",
        "value": "BF627285-8E57-46C7-BBAF-27AFBC7C23B8"
      }
    ],
    "clinicalStatus": "active",
    "category": [
      {
        "coding": [
          {
            "system": "https://fhir.hl7.org.uk/STU3/CodeSystem/CareConnect-ConditionCategory-1",
            "code": "problem-list-item",
            "display": "Problem List Item"
          }
        ]
      }
    ],
    "code": {
      "coding": [
        {
          "extension": [
            {
              "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
              "extension": [
                {
                  "url": "descriptionId",
                  "valueId": "1488801013"
                },
                {
                  "url": "descriptionDisplay",
                  "valueString": "H/O: aspirin allergy"
                }
              ]
            }
          ],
          "system": "http://snomed.info/sct",
          "code": "395102008",
          "display": "H/O: aspirin allergy"
        }
      ],
      "text": "H/O: aspirin allergy"
    },
    "subject": {
      "reference": "Patient/dc9cc923-60ed-40df-8a3e-64dc6bc89bf2"
    },
    "onsetDateTime": "2010-01-13",
    "assertedDate": "2010-01-13T11:41:26+00:00",
    "asserter": {
      "reference": "Practitioner/1E473786-E7FA-785E-C911-A8D38FB56F20"
    },
    "note": [
      {
        "text": "Unspecified Significance: Defaulted to Minor"
      },
      {
        "text": "Drug Allergy - Apsrin"
      },
      {
        "text": "Active Problem, Not Significant (Minor)"
      }
    ]
  }
}
```
</details>

1. If `LinkSet / conditionNamed / namedStatementRef` is present. Could reference one of the resource types stated [here](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_problems.html#extensionactualproblem)
2. If `LinkSet / component [index] / statementRef / id [@root]` is present. Could reference another Condition or one of the resource types stated [here](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_problems.html#extensionrelatedclinicalcontent)
3. If `LinkSet / component / statementRef / id [@root]` references a Link Set or another Link Set references the mapped Link set in the same manner.  


### notes

The following

- fixed value = `"Defaulted status to active : Unknown status at source"` if `linkSet / code [@code]` does not equal `394774009` (active) or `394775005` (inactive)

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

A FHIR Condition is mapped to a HL7 LinkSet and an associated ObservationStatement

| Mapped to (XML HL7 LinkSet)                     | Mapped from (JSON FHIR / other source )                                                                                                                                                                                        |
|-------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id [@root]                                      | Unique ID generated by the Adaptor                                                                                                                                                                                             |
| code [@code]                                    | if `Observation.clinalStatus` equals `"active"` then `"394774009"` (Active problem) else `"394775005"` (Inactive problem)                                                                                                      |
| code [@displayName]                             | if `Observation.clinalStatus` equals `"active"` then `"Active Problem"` else `"Inactive Problem"`                                                                                                                              |
| code / originalText                             | Same as `code [@displayName]` and appended with `Condition.extension[index].value` <sup>4</sup>                                                                                                                                |
| code / qualifier [@inverted]                    | fixed value = `"false"`                                                                                                                                                                                                        |
| code / qualifier / name [@code]                 | if `Condition.extension[index].value` <sup>4</sup> equals `"Major"` then `"386134007"` else if equals `"Minor"` then `"394847000"`                                                                                             |
| code / qualifier / name [@codeSystem]           | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                                                                                               |
| code / qualifier / name [@displayName]          | if `Condition.extension[index].value` <sup>4</sup> equals `"Major"` then `"Significant"` else if equals `"Minor"` then  `"Unspecified significance"`                                                                           |
| statusCode [@code]                              | fixed value = "COMPLETE"                                                                                                                                                                                                       |                                                                                                                           
| effectiveTime / low [@value]                    | `Condition.onsetDateTime` or else `<low nullFlavor="UNK"/>`                                                                                                                                                                    |
| effectiveTime / high                            | `Condition.abatementDateTime`                                                                                                                                                                                                  |
| availabilityTime [@value]                       | `Condition.assertedDate`                                                                                                                                                                                                       |
| component / statementRef [@classCode]           | fixed value = `"OBS"`                                                                                                                                                                                                          |
| component / statementRef [@moodCode]            | fixed value = `"EVN"`                                                                                                                                                                                                          |
| component / statementRef / id [@root]           | `Condition.extension[index].value` <sup>5</sup>                                                                                                                                                                                |
| conditionNamed / namedStatementRef [@classCode] | fixed value = `"OBS"`                                                                                                                                                                                                          |
| conditionNamed / namedStatementRef [@moodCode]  | fixed value = `"EVN"`                                                                                                                                                                                                          |
| conditionNamed / namedStatementRef / id [@root] | Either the ID of the mapped [Observation](../observations/README.md) referenced by the actual problem extension <sup>6</sup> or a reference to the [Generated Observation Statement](#generated-observation-statement) (below) |
| Participant [@typeCode]                         | fixed value = `"PRF"`                                                                                                                                                                                                          |
| Participant / agentRef / id [@root]             | the mapped [Practioner](../practioners/README.md) referenced by `Conditioner.asserter`                                                                                                                                         |

### Generated Observation Statement

An Observation Statement is generated when the Condition being mapped doesn't contain an actual problem extension <sup>6</sup> 
with a reference to an Observation.

| Mapped to (XML HL7 ObservationStatement)          | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                                                                                                                                  |
|---------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id [@root]                                        | Unique ID generated by the Adaptor                                                                                                                                                                                                                                                                                                                                       |
| code                                              | If the actual problem is an `AllergyIntolerance`, `Immunization` or `MedicationRequest` then `code` is mapped as a [CD for Transformed Actual Problem Header](#cd-for-transformed-actual-problem-header). <br/> Otherwise it is mapped from `Condition.code` as outlined in [Codeable Concept - JSON FHIR > XML HL7](../codeable%20concept/README.md#json-fhir--xml-hl7) |
| statusCode [@code]                                | fixed value = `"COMPLETE"`                                                                                                                                                                                                                                                                                                                                               |
| effectiveTime / low [@value]                      | `Condition.onsetDateTime` or else `<low nullFlavor="UNK"/>`                                                                                                                                                                                                                                                                                                              |
| effectiveTime / high                              | `Condition.abatementDateTime`                                                                                                                                                                                                                                                                                                                                            | 
| availabilityTime [@value]                         | `Condition.assertedDate`                                                                                                                                                                                                                                                                                                                                                 |
| pertinentInformation / sequenceNumber [@value]    | fixed value = `"+1"`                                                                                                                                                                                                                                                                                                                                                     |
| pertinentInformation / pertinentAnnotation / text | Concatenated as outlined in [Pertinent Annotation Text for Generated Observation](#pertinent-annotation-text-for-generated-observation)                                                                                                                                                                                                                                  |

<details>
    <summary>Example XML</summary>

```XML

<component typeCode="COMP">
    <LinkSet classCode="OBS" moodCode="EVN">
        <id root="6FF44B07-3590-4995-B9E8-C723F962BA73"/>
        <code code="394775005" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Inactive Problem">
            <originalText>Inactive Problem, major</originalText>
            <qualifier inverted="false">
                <name code="386134007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Significant"/>
            </qualifier>
        </code>
        <statusCode code="COMPLETE"/>
        <effectiveTime>
            <low value="20100113"/>
            <high value="20100323"/>
        </effectiveTime>
        <availabilityTime value="20100113140445"/>
        <component typeCode="COMP">
            <statementRef classCode="OBS" moodCode="EVN">
                <id root="11c13385-04a4-44d8-ba4a-B6F137A9BD47"/>
            </statementRef>
        </component>
        <conditionNamed typeCode="NAME" inversionInd="true">
            <namedStatementRef classCode="OBS" moodCode="EVN">
                <id root="D122054B-9740-44F3-9592-604F9352C9BA"/>
            </namedStatementRef>
        </conditionNamed>
        <Participant typeCode="PRF" contextControlCode="OP">
            <agentRef classCode="AGNT">
                <id root="4835D5BA-B9BF-4D49-A4DC-79C536328A3F"/>
            </agentRef>
        </Participant>
    </LinkSet>
</component>
<component typeCode="COMP">
<ObservationStatement classCode="OBS" moodCode="EVN">
    <id root="D122054B-9740-44F3-9592-604F9352C9BA"/>
    <code code="152306018" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Allergy to peanuts">
    </code>
    <statusCode code="COMPLETE"/>
    <effectiveTime>
        <low value="20100113"/>
        <high value="20100323"/>
    </effectiveTime>
    <availabilityTime value="20100113140445"/>
    <pertinentInformation typeCode="PERT">
        <sequenceNumber value="+1"/>
        <pertinentAnnotation classCode="OBS" moodCode="EVN">
            <text>Problem Info: Related Problem: Child of Allergy to aspirin Problem Notes: This is a peanut allergy
                (Evolved into H/O: aspirin allergy 13-Jan-2010)
            </text>
        </pertinentAnnotation>
    </pertinentInformation>
</ObservationStatement>
</component>
```

</details>

4. Where `extension[index].url` equals `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ProblemSignificance-1"`
5. For each `extension[index].value`, where `extension[index].url` equals `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedClinicalContent-1"`
   or `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1"`  
   and:`extension[index].value` does not reference:
- a [List](../list/README.md)
- an [Encounter](../encounters/README.md)
  or:
- a suppressed [MedicationRequest](../medication%20requests/README.md) (with `MedicationRequest.status` == `"stopped"` and `MedicationRequest.intent` == `"order"`)

6. where `extension[index].url` equals `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1"`
7. where `extension[index].url` equals `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedProblemHeader-1"`

#### CD for transformed actual problem header

| Mapped to (XML HL7 CD) | Mapped from (JSON FHIR / other source )         |
|------------------------|-------------------------------------------------|
| code \[@code]          | fixed value = `"55607006"`                      |
| code \[@codeSystem]    | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15` |
| code \[@displayName]   | fixed value = `"Problem"`                       |

#### Pertinent Annotation Text for Generated Observation 

Where the condition has an Actual Problem extension <sup>6</sup> and the referenced problem is a `AllergyIntolerance`, 
`Immunization` or `MedicationRequest` the following will be added to `pertinentInformation / pertinentAnnotation / text`:

- `"Transformed {{resourceType}} problem header"` where `{{resourceType}}` is the resource type of the actual problem
- `"Originally coded: {{codeText}}"` where `{{codeText}}` is `Condition.code.text` or 
`Condition.coding[0].extension[index].extenstion[index2].value` where `extension[index].url` == `https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid`
and `extension[index2].url` == descriptionDisplay. If the text field or the description display extension are not populated
the string will not be added.

Where `Condition.note` contains annotations, `annotation.text` will be added in the following manner:

- `"Problem Notes: {{annotation1}}, {{annotation2}}, {{annotation3}} ... "` 

Where the condition has related problem extensions <sup>7</sup>, the following will be added:  

- `"Related Problem: {{relationshipType}} {{conditionDisplay}}"`. Where `{{relationshipType}}` is the value of the 
related problem's `type` extension i.e. `parent` or `child` or `Unknown relationship with ` if not present. And 
`{{conditionDisplay}}` is the `code.coding[index].display` value of the related problem, where `code.coding[index].system` 
is SNOMED CT (`"http://snomed.info/sct"`) 


## Further documentation

[Problems guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_problems_guidance.html)
[FHIR Condition](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_problems.html)
[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
