# Condition Mapping

## XML HL7 > JSON FHIR

A FHIR Condition is mapped from a LinkSet, A LinkSet references different resources that are mapped from an Observation statement and Ehr Composition.

The LinkSet will gather the codes/values/naming conventions from the Ehr Composition. Therefore, the LinkSet will then map out the fields below if certain conditions are met, otherwise it will add in a default value. 

| Mapped to (JSON FHIR Condition field)     | Mapped from (XML HL7 / other source)                                                                                                                                                     |
|-------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                        | `LinkSet / id [@root]`                                                                                                                                                                   |
| meta.profile                              | fixed value = "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ProblemHeader-Condition-1"                                                                                   |
| extension[0].url                          | fixed value = "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ProblemSignificance-1"                                                                             |
| extension[0].valueCode                    | If major is set from the LinkSet then add "major" as a value, otherwise add "minor" = `major`                                                                                            |
| extension[index].url                      | fixed value = "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1"  <sup>1</sup>                                                                     |
| extension[index].valueReference.reference | reference to the mapped resource <sup>1</sup>                                                                                                                                            |
| extension[index].url                      | fixed value = "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedClinicalContent-1" <sup>2</sup>                                                             |
| extension[index].valueReference.reference | reference to the mapped resource <sup>2</sup>                                                                                                                                            |
| identifier[0].system                      | "https://PSSAdaptor/{{odsCode}}}" - where {{osdCode}} is the ODS code of the losing practice                                                                                             |
| identifier[0].value                       | `LinkSet / id [@root]`                                                                                                                                                                   |
| clinicalStatus                            | If `LinkSet / code [@code]` equals `394775005` the `inactive` otherwise `active`                                                                                                         |
| category[0].coding[0].system              | fixed value = "https://fhir.hl7.org.uk/STU3/CodeSystem/CareConnect-ConditionCategory-1"                                                                                                  |
| category[0].coding[0].code                | fixed value = `problem-list-item`                                                                                                                                                        |
| category[0].coding[0].display             | fixed value = `Problem List Item`                                                                                                                                                        |
| code                                      | mapped from `ObservationSatement / code` where the `Observationstatement` is referenced in `LinkSet / component / statementRef / id [@root]`                                             |
| subject.reference                         | reference to the mapped [Patient](../patient/README.md)                                                                                                                                  |
| context.reference                         | reference to the associated [Encounter](../encounters/README.md)                                                                                                                         |
| onsetDateTime                             | `LinkSet / effectiveTime` or else `LinkSet / availabilitytime [@value]`                                                                                                                  |
| assertedDate                              | `ehrComposition / author / time [@value]` or else `EhrExtract / availabilityTime [@value]`                                                                                               |
| asserter.reference                        | [Practitioner](../practioners/README.md) referenced by `ehrComposition / participant2[0] / agentRef / id [@root]`                                                                        |
| note.text[index]                          | fixed value = `"Defaulted status to active : Unknown status at source"` if `linkSet / code [@code]` does not equal `394774009` (active) or `394775005` (inactive)                        |
| note.text[index]                          | fixed value = `"Unspecified Significance: Defaulted to Minor"` if `LinkSet / code / qualifier / name [@code]` does not equal `386134007`                                                 |
| mote.text[index]                          | `ObservationStatement / pertinentInformation / pertinentAnnotation / text` where the `observationStatement` is referenced by `LinkSet / conditionNamed / namedStatementRef / id [@root]` |


### notes

The following

- fixed value = `"Defaulted status to active : Unknown status at source"` if `linkSet / code [@code]` does not equal `394774009` (active) or `394775005` (inactive)

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

1. If `LinkSet / conditionNamed / namedStatementRef` is present. Could reference one of the resource types stated [here](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_problems.html#extensionactualproblem)
2. If `LinkSet / component / statementRef / id [@root]` is present. Could reference one of the resource types stated [here](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_problems.html#extensionrelatedclinicalcontent)

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
    <summary>Example XML</summary>

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

[Problems guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_problems_guidance.html)
[FHIR Condition](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_problems.html)
[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
