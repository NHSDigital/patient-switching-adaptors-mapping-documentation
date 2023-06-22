# Condition Mapping

## XML HL7 > JSON FHIR

A Condition is derived from a LinkSet, A LinkSet is made of different resources is mapped from an Observsation  Ehr Composition.


| Mapped to (JSON FHIR Immunization field)                 | Mapped from (XML HL7 / other source)                                                                                                                                                                        |
|----------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                       | set value = Generated GUID                                                                                                                                                                                  |
| meta / profile                                           | fixed value = "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProblemHeader-Condition-1"                                                                                                      |
| extension[0] / url                                       | fixed value = "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ProblemSignificance-1"                                                                                                |
| extension[0] / valueCode                                 | fixed value = `major`                                                                                                                                                                                       |
| extension[1] / url                                       | fixed value = "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1"                                                                                                      |
| extension[1] / valueReference / reference                | fixed value = `Observation/10B9023B-A997-4449-AF63-EF3015E4C7B5`                                                                                                                                            |
| extension[2] / url                                       | fixed value = "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedClinicalContent-1"                                                                                             |
| extension[2] / valueReference / reference                | fixed value = `Observation/FE85DB22-274E-4B1D-9698-5C4809E9CD42`                                                                                                                                            |
| identifier[0] / system                                   | Use value if present, if not then set as = "https://PSSAdaptor/2167888433"                                                                                                                                  |
| identifier[0] / value                                    | Use value if present, if not then set as = `31EA7C21-BE35-4837-91A5-D66D8C375338`                                                                                                                           |
| clinicalStatus                                           | Use value if present, if not then set as = `active`                                                                                                                                                         |
| category[0] / coding[0] / system                         | fixed value = "https://fhir.hl7.org.uk/STU3/CodeSystem/CareConnect-ConditionCategory-1"                                                                                                                     |
| category[0] / coding[0] / code                           | fixed value = `problem-list-item`                                                                                                                                                                           |
| category[0] / coding[0] / display                        | fixed value = `Problem List Item`                                                                                                                                                                           |
| code / coding[0] / extension[0] / url                    | fixed value = "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid"                                                                                                                     |
| code / coding[0] / extension[0] / extension[0] / url     | fixed value = `descriptionId`                                                                                                                                                                               |
| code / coding[0] / extension[0] / extension[0] / valueId | fixed value = `150085018`                                                                                                                                                                                   |
| code / coding[0] / extension[0] / system                 | fixed value = "http://snomed.info/sct"                                                                                                                                                                      |
| code / coding[0] / extension[0] / code                   | fixed value = `90560007`                                                                                                                                                                                    |
| code / coding[0] / extension[0] / display                | fixed value = `Gout`                                                                                                                                                                                        |
| subject / reference                                      | fixed value = `Patient/97ed9655-3f78-4e7c-a4ee-dc9c6e760494`                                                                                                                                                |
| context / reference                                      | fixed value = `Encounter/6D2FB37E-26B4-4838-B6D2-CA96E1F10227`                                                                                                                                              |
| onsetDateTime                                            | fixed value = `2010-03-23T15:36:00+00:00`                                                                                                                                                                   |
| assertedDate                                             | fixed value = `2010-03-23T15:51:27+00:00`                                                                                                                                                                   |
| asserter / reference                                     | fixed value = `Practitioner/2D70F602-6BB1-47E0-B2EC-39912A59787D`                                                                                                                                           |
| note / text[0]                                           | Use value if present, if not then generate notes that are derived from a list of fixed values = `Significance : Major\n Episodicity : First\n This is problem Problem Info: Problem Notes: This is problem` |
| note / text[1]                                           | Use value if present, if not then generate notes that are derived from a list of fixed values = `Active Problem, major`                                                                                     |


<details>
    <summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Condition",
    "id": "31EA7C21-BE35-4837-91A5-D66D8C375338",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProblemHeader-Condition-1"
      ]
    },
    "extension": [
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ProblemSignificance-1",
        "valueCode": "major"
      },
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1",
        "valueReference": {
          "reference": "Observation/10B9023B-A997-4449-AF63-EF3015E4C7B5"
        }
      },
      {
        "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedClinicalContent-1",
        "valueReference": {
          "reference": "Observation/FE85DB22-274E-4B1D-9698-5C4809E9CD42"
        }
      }
    ],
    "identifier": [
      {
        "system": "https://PSSAdaptor/2167888433",
        "value": "31EA7C21-BE35-4837-91A5-D66D8C375338"
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
                  "valueId": "150085018"
                }
              ]
            }
          ],
          "system": "http://snomed.info/sct",
          "code": "90560007",
          "display": "Gout"
        }
      ]
    },
    "subject": {
      "reference": "Patient/97ed9655-3f78-4e7c-a4ee-dc9c6e760494"
    },
    "context": {
      "reference": "Encounter/6D2FB37E-26B4-4838-B6D2-CA96E1F10227"
    },
    "onsetDateTime": "2010-03-23T15:36:00+00:00",
    "assertedDate": "2010-03-23T15:51:27+00:00",
    "asserter": {
      "reference": "Practitioner/2D70F602-6BB1-47E0-B2EC-39912A59787D"
    },
    "note": [
      {
        "text": "Significance : Major\n Episodicity : First\n This is problem Problem Info: Problem Notes: This is problem"
      },
      {
        "text": "Active Problem, major"
      }
    ]
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

A Condition is made from a mixture of resources that is made into a LinkSet.

| Mapped to (XML HL7)                                    | Mapped from (JSON FHIR / other source )                                      |
|--------------------------------------------------------|------------------------------------------------------------------------------|
| EhrExtract / recordTarget / patient / id \[@extension] | NHS number is taken from the initial EHR Request message (RCMR_IN010000UK05) |


<details>
    <summary>Example XML</summary>'

```XML
<component typeCode="COMP">
    <LinkSet classCode="OBS" moodCode="EVN">
        <id root="2A2EBA69-75D0-4273-88FD-AC6EF5B6C57E" />
        <code code="394776006" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Unspecified problem">
            <originalText>Health Administration</originalText>
            <qualifier inverted="false">
                <name code="386134007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Significant" />
            </qualifier>
        </code>
        <statusCode code="COMPLETE" />
        <effectiveTime>
            <low value="19781231" />
        </effectiveTime>
        <availabilityTime value="20100114095333" />
        <component typeCode="COMP">
            <statementRef classCode="OBS" moodCode="EVN">
                <id root="BA2E4425-1BC9-47C2-863C-761C8D37BF37" />
            </statementRef>
        </component>
        <conditionNamed typeCode="NAME" inversionInd="true">
            <namedStatementRef classCode="OBS" moodCode="EVN">
                <id root="230D3D37-99E3-450A-AE88-B5AB802B7137" />
            </namedStatementRef>
        </conditionNamed>
    </LinkSet>
</component>
```

</details>

## Further documentation

[FHIR Condition](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_problems_guidance.html)
[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
