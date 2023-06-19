# Condition Mapping

## XML HL7 > JSON FHIR

A Condition resource is mapped from a Ehr Composition.
#TODO

| Mapped to (JSON FHIR Immunization field) | Mapped from (XML HL7 / other source)                                                                                                                                     |
|------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                       | set value = Generated GUID                                                                                                                                               |
| meta.profile\[0]                         | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1"`.                                                                                |
| meta.versionId                           | fixed value = `1521806400000`                                                                                                                                            |
| indentifier\[0].system                   | fixed value = `https://fhir.nhs.uk/Id/nhs-number`                                                                                                                        |
| indentifier\[0].value                    | `Patient / id [@extension]`                                                                                                                                              |
| managingOrganization                     | referenced to mapped AgentOrganization: [Organization](https://github.com/NHSDigital/patient-switching-adaptors-mapping-documentation/blob/main/organisations/README.md) |

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

#TODO
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

#TODO
## JSON FHIR > XML HL7

A Patient record is inserted into the EhrExtracts recordTarget.

| Mapped to (XML HL7)                                    | Mapped from (JSON FHIR / other source )                                      |
|--------------------------------------------------------|------------------------------------------------------------------------------|
| EhrExtract / recordTarget / patient / id \[@extension] | NHS number is taken from the initial EHR Request message (RCMR_IN010000UK05) |


<details>
    <summary>Example XML</summary>'

```XML
<ControlActEvent classCode=\"CACT\" moodCode=\"EVN\">
<author1 typeCode=\"AUT\">
<AgentSystemSDS classCode=\"AGNT\">
<agentSystemSDS classCode=\"DEV\" determinerCode=\"INSTANCE\">
<id root=\"1.2.826.0.1285.0.2.0.107\" extension=\"200000000359\" />
        </agentSystemSDS>
        </AgentSystemSDS>
        </author1>
<subject typeCode=\"SUBJ\" contextConductionInd=\"false\">
<EhrExtract classCode=\"EXTRACT\" moodCode=\"EVN\">
<id root=\"4B3EC6C4-D9BD-4FFE-8C29-01A5EC41B9E9\" />
<statusCode code=\"COMPLETE\" />
<availabilityTime value=\"20230613105301\" />
<recordTarget typeCode=\"RCT\">
<patient classCode=\"PAT\">
<id root=\"2.16.840.1.113883.2.1.4.1\" extension=\"9726908744\" />
        </patient>
        </recordTarget>
    
... 
    
</EhrExtract>
```

</details>

## Further documentation

[FHIR Patient](https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Patient-1)
[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
