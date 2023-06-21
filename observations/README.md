# Observation Mapping

# XML HL7 > JSON FHIR

FHIR Observations are used throughout the Structured Record and the 
[GP Connect Documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured.html) outlines how they
are used. As a result they are mapped from multiple HL7 components as stated below, where the items in **bold** are 
FHIR `Observation` resource types described in the following documentation.

* Observation statements  map to **Uncategorised Data** and **Test Results**
* Request statements map to **Self Referral**
* Compound statements map to **Test Group Header**, **Blood Pressure** and **Componentised Observation Header**
* Narrative statements map to **Filing Comments**

## Uncategorised data

Uncategorised data is mapped from an `ObservationStatement` to to an `Observation` in the following way, if it is not
deemed to be an
[AllergyIntolerance](../allergy%20intolerances/README.md), blood pressure (see below)
or [Immunization](../immunisations/README.md).

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                                            |
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `ObservationStatement / id \[@root]`                                                                                                                     |
| meta / profile\[0]                               | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                             |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                          |
| identifier\[0].value                             | `ObservationStatement / id \[@root]`                                                                                                                     |
| status                                           | fixed value = `"final"`                                                                                                                                  |
| code                                             | `ObservationStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                              |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                                                  |
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                            |
| valueQuantity                                    | `ObservationStatement / value` where `ObservationStatement / value [@type]` is `"PQ"` or `"IVLPQ"`                                                       |
| valueQuantity.unit                               | `ObservationStatement / value [@unit]` or `ObservationStatement / value / (high or low)`                                                                 |
| valueQuantity.comparator                         | `ObservationStatement / value / (high or low) [@inclusive]` where `ObservationStatement / value [@type]` is `"IVLPQ"`                                    |
| valueQuantity.extension\[0].url                  | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1"` <sup>1</sup>                               |
| valueQuantity.extension\[0].value                | fixed value = `true` <sup>1</sup>                                                                                                                        |
| valueString                                      | `ObservationStatement / value` where valueQuantity is not populated                                                                                      |
| effectiveDateTime                                | `ObservationStatement / effectiveTime` or else `ObservationStatement / availibiltyTime [@value]`                                                         |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                                                   |
| performer\[0]                                    | Practitioner referenced in `ObservationStatement / paticipant` <sup>3</sup>                                                                              |
| interpretation.coding\[0].code                   | `ObservationStatement / interpretationCode [@code]`                                                                                                      |
| interpretation.coding\[0].display                | `ObservationStatement / interpretationCode [@code]`                                                                                                      |
| interpretation.coding\[0].system                 | fixed value = `"http://hl7.org/fhir/v2/0078"`                                                                                                            |
| interpretation.text                              | `ObservationStatement / interpretationCode / originalText` or else `ObservationStatement / interpretationCode [@displayName]`                            |                                         
| comment                                          | Concatenated from `ObservationStatement / subject / personalRelationship / code` and `ObservationStatement / pertinentInformation / pertinentAnnotation` |
| referenceRange\[index].text                      | `ObservationStatement / referenceRange[index] / referenceInterpretationRange / text`                                                                     |
| referenceRange\[index].high                      | `ObservationStatement / referenceRange\[index] / referenceInterpretationRange / value / high`                                                            |
| referenceRange\[index].low                       | `ObservationStatement / referenceRange\[index] / referenceInterpretationRange / value / low`                                                             |

<details>
<summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Observation",
    "id": "CF0BAFD7-9E92-4DB5-B7EE-B37DBD30AD93",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "CF0BAFD7-9E92-4DB5-B7EE-B37DBD30AD93"
      }
    ],
    "status": "final",
    "code": {
      "coding": [
        {
          "extension": [
            {
              "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
              "extension": [
                {
                  "url": "descriptionId",
                  "valueId": "299757012"
                },
                {
                  "url": "descriptionDisplay",
                  "valueString": "Angina pectoris"
                }
              ]
            }
          ],
          "system": "http://snomed.info/sct",
          "code": "194828000",
          "display": "Angina pectoris"
        }
      ],
      "text": "Angina pectoris"
    },
    "subject": {
      "reference": "Patient/bbe0c375-4675-4976-8586-597056863ac7"
    },
    "context": {
      "reference": "Encounter/E92099A9-F7E9-4684-91EB-D6427F022041"
    },
    "effectiveDateTime": "2010-01-14T13:08:00+00:00",
    "issued": "2010-02-06T13:07:44.000+00:00",
    "performer": [
      {
        "reference": "Practitioner/C5DEFBF3-0174-BC6F-182C-B777B9C6FF43"
      }
    ]
  }
}
```

</details>

1. If `ObservationStatement / uncertaintyCode` is present.
2. Where the `ehrComposition` is the parent of mapped statement.
3. Where `paticipant [@typecode]` is `"PPRF"` (Primary performer) or `"PRF"` (Performer).

## Blood pressure

Blood pressure observations are mapped from a HL7 `CompoundStatement` containing two `ObservationStatement` components,
known as a "blood pressure triple", where the SNOMED codes identify them as a blood pressure. The SNOMED codes used to identify a
"blood pressure triple" are listed in the
[Gp Connect documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html#representing-blood-pressure-readings-from-gp-systems)
.

| Mapped to (JSON FHIR Observation resource field)    | Mapped from (XML HL7 / other)                                                                                                                          |
|-----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                  | `CompoundStatement / id [@root]`                                                                                                                       |
| meta / profile\[0]                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                           |
| identifier\[0].system                               | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                        |
| identifier\[0].value                                | `CompoundStatement / id \[@root]`                                                                                                                      |
| status                                              | fixed value = `"final"`                                                                                                                                |
| code                                                | `CompoundStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                               | 
| subject                                             | reference to the mapped [Patient](../patient/README.md)                                                                                                |
| context                                             | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                          |
| effectiveDateTime                                   | `CompoundStatement / effectiveTime` or else `CompoundStatement / availibiltyTime [@value]`                                                             |
| issued                                              | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                                                 | 
| performer\[0]                                       | Practitioner referenced in `CompoundStatement / paticipant` <sup>3</sup>                                                                               |
| component\[index].code                              | `ObservationStatement.code` <sup>4</sup>   as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)               |
| component\[index].valueQuantity                     | `ObservationStatement / value` where `ObservationStatement / value [@type]` is `"PQ"` or `"IVLPQ"` <sup>4</sup>                                        |
| component\[index].valueQuantity.extension\[0].url   | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1"` <sup>1</sup>                             |
| component\[index].valueQuantity.extension\[0].value | fixed value = `true` <sup>1</sup>                                                                                                                      |
| comment                                             | concatenated from `ObservationStatement / pertinentInformation / pertinentAnnotation / text` <sup>5</sup> and `NarrativeStatement / text` <sup>6</sup> |

<details>
<summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Observation",
    "id": "F25C1328-B6D2-412F-9C56-A8F21182F100",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "F25C1328-B6D2-412F-9C56-A8F21182F100"
      }
    ],
    "status": "final",
    "code": {
      "coding": [
        {
          "extension": [
            {
              "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
              "extension": [
                {
                  "url": "descriptionId",
                  "valueId": "254063019"
                },
                {
                  "url": "descriptionDisplay",
                  "valueString": "O/E - blood pressure reading"
                }
              ]
            }
          ],
          "system": "http://snomed.info/sct",
          "code": "163020007",
          "display": "O/E - blood pressure reading"
        }
      ],
      "text": "O/E - blood pressure reading"
    },
    "subject": {
      "reference": "Patient/bbe0c375-4675-4976-8586-597056863ac7"
    },
    "context": {
      "reference": "Encounter/81A3B881-FE23-4346-A7ED-48ED539E9054"
    },
    "effectiveDateTime": "2010-02-06T12:41:00+00:00",
    "issued": "2010-02-06T12:44:53.000+00:00",
    "performer": [
      {
        "reference": "Practitioner/C5DEFBF3-0174-BC6F-182C-B777B9C6FF43"
      }
    ],
    "component": [
      {
        "code": {
          "coding": [
            {
              "extension": [
                {
                  "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                  "extension": [
                    {
                      "url": "descriptionDisplay",
                      "valueString": "O/E - Systolic BP reading"
                    }
                  ]
                }
              ],
              "system": "http://snomed.info/sct",
              "code": "72313002",
              "display": "Systolic arterial pressure"
            }
          ],
          "text": "Systolic blood pressure"
        },
        "valueQuantity": {
          "value": 170,
          "unit": "mmHg"
        }
      },
      {
        "code": {
          "coding": [
            {
              "system": "http://snomed.info/sct",
              "code": "1091811000000102",
              "display": "O/E - Diastolic BP reading"
            }
          ],
          "text": "Diastolic blood pressure"
        },
        "valueQuantity": {
          "value": 130,
          "unit": "mmHg"
        }
      }
    ]
  }
}
```

</details>

4. Mapped for the both `ObservationStatement` elements that form part of the blood pressure triple.
5. Text from a `pertinentAnnotation` is prepended with `"Systolic Note: "` or `"Diastolic Note: "`,
   determined by the SNOMED code of the `OberservationStatement`.
6. Where the `NarrativeStatement` is a child of the blood pressure triple's `CompoundStatement`. Will be prepended
   with `"BP Note: "` when mapped.

## Self referral

If a referral is deemed to be a self referral, the HL7 `RequestStatement` is mapped to an FHIR `Observation`. Self referrals 
are identified by `RequestStatement / code / qualifier /value [@code]`, where it is equal to `"SelfReferral`.

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                           |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------|
| id                                               | `RequestStatement / id [@root]`                                                                                         |
| meta / profile\[0]                               | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                            |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice         |
| identifier\[0].value                             | `RequestStatement / id \[@root]`                                                                                        |
| status                                           | fixed value = `"final"`                                                                                                 |
| code                                             | `RequestStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md) |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                  |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                 |  
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                           |
| effectiveDateTime                                | `RequestStatement / effectiveTime` or else `RequestStatement / availibiltyTime [@value]`                                |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                  |                                                   
| performer\[0]                                    | [Practitioner](../practioners/README.md) referenced in `RequestStatement / paticipant` <sup>7</sup>                     |      
| comment                                          | fixed value = `"SelfRerreral"`                                                                                          |
| component\[0].code.text                          | fixed value = `"Urgency"`                                                                                               |
| component\[0].valueString                        | `RequestStatement / priorityCode / originalText`                                                                        |
| component\[1].code.text                          | fixed value = `"Text"`                                                                                                  |
| component\[1].valueString                        | `RequestStatement / text`                                                                                               |

<details>
<summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Observation",
    "id": "8D61D723-D51D-44FB-B814-0EB69DB1D6F4",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "8D61D723-D51D-44FB-B814-0EB69DB1D6F4"
      }
    ],
    "status": "final",
    "code": {
      "coding": [
        {
          "extension": [
            {
              "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
              "extension": [
                {
                  "url": "descriptionId",
                  "valueId": "283677016"
                },
                {
                  "url": "descriptionDisplay",
                  "valueString": "Referral to G.P."
                }
              ]
            }
          ],
          "system": "http://snomed.info/sct",
          "code": "183561008",
          "display": "Referral to GP"
        }
      ],
      "text": "Referral to G.P."
    },
    "subject": {
      "reference": "Patient/bed1e27e-0e28-4dea-9216-bc60fa3dc17b"
    },
    "context": {
      "reference": "Encounter/B88EECC1-8DA9-48F9-981F-8D31C517A693"
    },
    "effectiveDateTime": "2010-01-19",
    "issued": "2010-01-19T11:39:57.000+00:00",
    "performer": [
      {
        "reference": "Practitioner/C5DEFBF3-0174-BC6F-182C-B777B9C6FF43"
      }
    ],
    "comment": "SelfReferral",
    "component": [
      {
        "code": {
          "text": "Urgency"
        },
        "valueString": "Routine"
      },
      {
        "code": {
          "text": "Text"
        },
        "valueString": "Dr David McAvenue, Purpose: SelfReferral, Reason: Dry eyes"
      }
    ]
  }
}
```

</details>

7. Where `ObservationStatement / paticipant [@typecode]` is `"PPRF"` (Primary performer) or `"PRF"` (Performer).

## Investigations

For more information on how Investigations are represented in GP Connect see [Investigations Guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_pathology_guidance.html).


### Test Group Header

Test group headers are mapped from a nested `CompoundStatement` with a class code of `BATTERY` where
the parent compound statements are deemed to from a [Diagnostic Report](../diagnostic%20reports/README.md).

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                     |
|--------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `CompoundStatement / id [@root]`                                                                                                  |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                      |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                   |
| identifier\[0].value                             | `CompoundStatement / id \[@root]`                                                                                                 |
| status                                           | fixed value = `"final"`                                                                                                           |
| category\[0].coding\[0].code                     | fixed value = `"laboratory"`                                                                                                      |
| category\[0].coding\[0].system                   | fixed value = `"http://hl7.org/fhir/observation-category"`                                                                        |
| category\[0].coding\[0].display                  | fixed value = `"Laboratory"`                                                                                                      |
| code                                             | `CompoundStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)          |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                           |  
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                                     |
| effectiveDateTime                                | `CompoundStatement / effectiveTime` or else `CompoundStatement / availibiltyTime [@value]`                                        |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                            |
| performer\[0]                                    | [Practitioner](../practioners/README.md) referenced in `CompoundStatement / paticipant` or else `CompoundStatement / paticipant2` |
| comment                                          | concatenated with newlines from child `NarrativeStatement / text` where the EDIFACT comment type is not `USER COMMENT`            |
| specimen                                         | reference to the [Specimen](../diagnostic%20reports/README.md)                                                                    |
| related\[index].type                             | fixed value = `"has-member"`                                                                                                      |
| related\[index].target                           | reference to the child Observation - Test result (below)                                                                          |

<details>
<summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Observation",
    "id": "2418B6B6-C4C0-46CB-9030-5B7DD39C80FC",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "2418B6B6-C4C0-46CB-9030-5B7DD39C80FC"
      }
    ],
    "status": "final",
    "category": [
      {
        "coding": [
          {
            "system": "http://hl7.org/fhir/observation-category",
            "code": "laboratory",
            "display": "Laboratory"
          }
        ]
      }
    ],
    "code": {
      "text": "CHOL/HDL RATIO"
    },
    "subject": {
      "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
    },
    "context": {
      "reference": "Encounter/1449860E-3953-4D71-A867-3E1E79D2E11B"
    },
    "effectiveDateTime": "2010-01-20T10:46:22+00:00",
    "issued": "2010-03-26T13:49:48.000+00:00",
    "performer": [
      {
        "reference": "Practitioner/1E473786-E7FA-785E-C911-A8D38FB56F20"
      }
    ],
    "comment": "See FATS/Healthy Hearts guidelines for interpretation of lipids",
    "specimen": {
      "reference": "Specimen/92BB4158-8984-4898-8C4D-EBFD27514947"
    },
    "related": [
      {
        "type": "has-member",
        "target": {
          "reference": "Observation/C737A049-F93E-4C52-AFDF-21B0D1C7298C"
        }
      },
      {
        "type": "has-member",
        "target": {
          "reference": "Observation/E47B3A50-EEBE-4336-AA48-5932A01BC1B5"
        }
      },
      {
        "type": "has-member",
        "target": {
          "reference": "Observation/8673E805-9884-4040-993A-D72AECF4D363"
        }
      }
    ]
  }
}
```

</details>

### Test Result

With the exception of the related field, Test Result observations are mapped identically to Uncategorised Data (above). 

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                      |
|--------------------------------------------------|----------------------------------------------------|
| related\[index].type                             | fixed value = `"derived-from"`                     |
| related\[index].target                           | reference to the parent Test Group Header (above)  |

<details>
<summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Observation",
    "id": "C737A049-F93E-4C52-AFDF-21B0D1C7298C",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "C737A049-F93E-4C52-AFDF-21B0D1C7298C"
      }
    ],
    "status": "final",
    "category": [
      {
        "coding": [
          {
            "system": "http://hl7.org/fhir/observation-category",
            "code": "laboratory",
            "display": "Laboratory"
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
                  "url": "descriptionDisplay",
                  "valueString": "Serum cholesterol"
                }
              ]
            }
          ],
          "system": "http://snomed.info/sct",
          "code": "1005671000000105",
          "display": "Serum cholesterol level"
        }
      ],
      "text": "Serum cholesterol"
    },
    "subject": {
      "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
    },
    "context": {
      "reference": "Encounter/1449860E-3953-4D71-A867-3E1E79D2E11B"
    },
    "effectiveDateTime": "2010-01-20T10:46:22+00:00",
    "issued": "2010-03-26T13:49:48.000+00:00",
    "performer": [
      {
        "reference": "Practitioner/1E473786-E7FA-785E-C911-A8D38FB56F20"
      }
    ],
    "valueQuantity": {
      "value": 6.3,
      "unit": "millimole per liter",
      "code": "mmol/L"
    },
    "specimen": {
      "reference": "Specimen/92BB4158-8984-4898-8C4D-EBFD27514947"
    },
    "related": [
      {
        "type": "derived-from",
        "target": {
          "reference": "Observation/2418B6B6-C4C0-46CB-9030-5B7DD39C80FC"
        }
      }
    ]
  }
}
```

</details>

### Filing Comment

Filing Comment observations are mapped from `NarrativeStatements` where the EDIFACT comment type is `USER COMMENT`.

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                    |
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `NarrativeStatement / id [@root]`                                                                                                |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                     |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                  |
| identifier\[0].value                             | `NarrativeStatement / id \[@root]`                                                                                               |
| status                                           | fixed value = `"final"`                                                                                                          |
| code.coding\[0].system                           | fixed value = `"http://snomed.info/sct"`                                                                                         |
| code.coding\[0].code                             | fixed value = `"37331000000100"`                                                                                                 |
| code.coding\[0].display                          | fixed value = `"Comment note"`                                                                                                   |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                          |
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                                    |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                           |
| performer\[0]                                    | Practitioner referenced in `CompoundStatement / paticipant` <sup>3</sup>                                                         |
| comment                                          | `NarrativeStatement / text`                                                                                                      |
| related\[index].type                             | fixed value = `"derived-from"`                                                                                                   |
| related\[index].target                           | reference to the parent [Diagnostic Report](../diagnostic%20reports/README.md), Test Group Header (above) or Test Result (above) |                                          |


<details>
<summary>Example JSON</summary>

```JSON
{
  "resource": {
    "resourceType": "Observation",
    "id": "21DB2A74-676A-4C76-9143-C149352E9FAF",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "21DB2A74-676A-4C76-9143-C149352E9FAF"
      }
    ],
    "status": "final",
    "code": {
      "coding": [
        {
          "system": "http://snomed.info/sct",
          "code": "37331000000100",
          "display": "Comment note"
        }
      ]
    },
    "subject": {
      "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
    },
    "context": {
      "reference": "Encounter/B0696913-F11D-4EAA-8BB7-41350B296F3F"
    },
    "issued": "2010-02-01T09:33:13.000+00:00",
    "performer": [
      {
        "reference": "Practitioner/1E473786-E7FA-785E-C911-A8D38FB56F20"
      }
    ],
    "comment": "(EMISTest) - Normal - No Action",
    "related": [
      {
        "type": "derived-from",
        "target": {
          "reference": "Observation/69832EFE-727E-4270-BDF5-179851BDF295"
        }
      }
    ]
  }
}
```

</details>

## Componentised Observations

When a `CompoundStatement` element has a class code of `BATTERY` or `CLUSTER` and does not form part of a 
[Diagnostic Report](../diagnostic%20reports/README.md) or a blood pressure, it is mapped to an `observation` resource. 
Additionally, any `ObservationStatement` component will be mapped to a separate `Observation` and referenced using the 
`"derived-from"` / `"has-member"` relationship.

### Componentised observation header

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                             |
|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| id                                               | `CompoundStatement / id [@root]`                                                                                          |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                              |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice           |
| identifier\[0].value                             | `CompoundStatement / id \[@root]`                                                                                         |
| status                                           | fixed value = `"final"`                                                                                                   |
| code                                             | `CompoundStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)  |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                   |
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                             |
| effectiveDateTime                                | `CompoundStatement / effectiveTime` or else `CompoundStatement / availibiltyTime [@value]`                                |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>2</sup> or else `EhrExtract / availibilityTime [@value]`                    |
| performer\[0]                                    | Practitioner referenced in `ObservationStatement / paticipant` <sup>3</sup>                                               |
| related\[index].type                             | fixed value = `"has-member"`                                                                                              |
| related\[index].target                           | reference to the child Observation - Test result (below)                                                                  |

<details>
<summary>Example JSON</summary>

```JSON

 {
            "resource": {
                "resourceType": "Observation",
                "id": "31465FC9-15B2-4391-A5EF-8F70C2154AAF",
                "meta": {
                    "profile": [
                        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
                    ]
                },
                "identifier": [
                    {
                        "system": "https://PSSAdaptor/D5445",
                        "value": "31465FC9-15B2-4391-A5EF-8F70C2154AAF"
                    }
                ],
                "status": "final",
                "code": {
                    "coding": [
                        {
                            "extension": [
                                {
                                    "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                                    "extension": [
                                        {
                                            "url": "descriptionDisplay",
                                            "valueString": "Serum lipids"
                                        }
                                    ]
                                }
                            ],
                            "system": "http://snomed.info/sct",
                            "code": "1005661000000103",
                            "display": "Serum lipids level"
                        }
                    ],
                    "text": "Serum lipids"
                },
                "subject": {
                    "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
                },
                "effectiveDateTime": "2001-03-30",
                "issued": "2010-02-09T12:31:51.000+00:00",
                "performer": [
                    {
                        "reference": "Practitioner/C5DEFBF3-0174-BC6F-182C-B777B9C6FF43"
                    }
                ],
                "related": [
                    {
                        "type": "has-member",
                        "target": {
                            "reference": "Observation/C45E3DA5-D7BC-4FDF-B4E0-6CCECCE60D24"
                        }
                    },
                    {
                        "type": "has-member",
                        "target": {
                            "reference": "Observation/3BC7ABC9-ABCB-4C01-A519-35CE664C3543"
                        }
                    }
                ]
            }
        }
```

</details>

### Componentised Observation

With the exception of the related field, Componentised Observations are mapped identically to Uncategorised Data (above).

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                        |
|--------------------------------------------------|------------------------------------------------------|
| related\[index].type                             | fixed value = `"derived-from"`                       |
| related\[index].target                           | reference to the parent Test Group Header (above)    |

<details>
<summary>Example JSON</summary>

```JSON

{
    "resource": {
        "resourceType": "Observation",
        "id": "C45E3DA5-D7BC-4FDF-B4E0-6CCECCE60D24",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"
            ]
        },
        "identifier": [
            {
                "system": "https://PSSAdaptor/D5445",
                "value": "C45E3DA5-D7BC-4FDF-B4E0-6CCECCE60D24"
            }
        ],
        "status": "final",
        "code": {
            "coding": [
                {
                    "extension": [
                        {
                            "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                            "extension": [
                                {
                                    "url": "descriptionDisplay",
                                    "valueString": "Serum cholesterol"
                                }
                            ]
                        }
                    ],
                    "system": "http://snomed.info/sct",
                    "code": "1005671000000105",
                    "display": "Serum cholesterol level"
                }
            ],
            "text": "Serum cholesterol"
        },
        "subject": {
            "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
        },
        "effectiveDateTime": "2001-03-30",
        "issued": "2010-02-09T12:31:51.000+00:00",
        "performer": [
            {
                "reference": "Practitioner/C5DEFBF3-0174-BC6F-182C-B777B9C6FF43"
            }
        ],
        "valueQuantity": {
            "value": 6.1,
            "unit": "millimole per liter",
            "code": "mmol/L"
        },
        "related": [
            {
                "type": "derived-from",
                "target": {
                    "reference": "Observation/31465FC9-15B2-4391-A5EF-8F70C2154AAF"
                }
            }
        ]
    }
}
```

</details>

# JSON FHIR > XML HL7

FHIR Observations can be mapped to the following HL7 components. The mapping for each GP Connect 
observation type (in **bold**) is described in the sections below:

- ObservationStatement (**Uncategorised Data** and **Test Results**)
- CompoundStatement (**Blood Pressure**, **Test Group Header**)
- NarrativeStatement (**Comment note** / **Filing comments**)

## Uncategorised Data

Uncategorised data is mapped from an FHIR `Observation` to a HL7 `ObservationStatement` where it is not deemed to be a 
Blood Pressure or Comment Note. 

| Mapped to (XML HL7 ObservationStatement)             | Mapped from (JSON FHIR / other source )                                                                                                                                         |
|------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                                          | Unique ID generated by the adaptor                                                                                                                                              |
| code                                                 | `Observation.code.coding` as described in the FHIR > XML section of [Codeable Concept](../codeable%20concept/README.md)                                                         |
| statusCode                                           | fixed value = `"COMPLETE"`                                                                                                                                                      |
| effectiveTime                                        | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                                                           |
| availabilityTime                                     | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                                                     | 
| value                                                | `Observation.valueQuantity` or `Observation.valueString`                                                                                                                        |
| pertinentInformation / sequenceNumber \[@value]      | fixed value = `"+1"`                                                                                                                                                            |
| pertinentInformation / pertinentAnnotation / text    | `Observation.bodySite.text` or else `Observation.bodySite.coding.extension[index].value` where `Observation.bodySite.coding.extension[index].url` equals `"descriptionDisplay"` |
| interpretationCode \[@code]                          | `Observation.interpretation.coding.code`                                                                                                                                        | 
| interpretationCode \[@codeSystem]                    | fixed value = `"2.16.840.1.113883.2.1.6.5"`                                                                                                                                     |
| interpretationCode \[@displayName]                   | description of `Observation.interpretation.coding.code` e.g. if code = `"HI"` then displayName = `"Above high reference limit"`                                                 |
| interpretationCode / originalText                    | `Observation.interpretation.coding.display`                                                                                                                                     |
| referenceRange / referenceInterpretationRange / text | `Observation.referenceRange.text`                                                                                                                                               |
| referenceRange / referenceInterpretationRang / value | `Observation.referenceRange.value`                                                                                                                                              |
| Participant / agentRef / id \[@root]                 | `Observation.performer`                                                                                                                                                         | 


<details>
   <summary>Example XML</summary>

```XML

<ObservationStatement classCode="OBS" moodCode="EVN">
   <id root="1814312A-9F16-4311-AB4F-7E31E7B57E1F"/>
   <code code="478551000006117" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="ALT/SGPT serum level"/>
   <statusCode code="COMPLETE"/>
   <effectiveTime>
      <center value="20100323133700"/>
   </effectiveTime>
   <availabilityTime value="20100323133700"/>
   <value unit="1" value="12.000" xsi:type="PQ">
      <translation value="12.000">
         <originalText>U/L</originalText>
      </translation>
   </value>
   <pertinentInformation typeCode="PERT">
      <sequenceNumber value="+1"/>
      <pertinentAnnotation classCode="OBS" moodCode="EVN">
         <text>BodySite: Example body site comment</text>
      </pertinentAnnotation>
   </pertinentInformation>
   <Participant contextControlCode="OP" typeCode="PRF">
      <agentRef classCode="AGNT">
         <id root="910543AF-6E56-47B9-970F-6724483D808C"/>
      </agentRef>
   </Participant>
</ObservationStatement>
```
</details>

## Blood Pressure

An Observation is identified as a Blood Pressure if `Observation.code`, `Observation.component[0].code` and 
`Observation.component[1].code` contain a valid blood pressure triple, as outlined in the 
[GP Connect Documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html#representing-blood-pressure-readings-from-gp-systems).
Blood Pressure Observations are mapped to a `CompoundStatement` with `ObservationStatement` components for the systolic 
and diastolic readings and a optional `NarrativeStatement` for additional information.

### Parent Compound Statement

| Mapped to (XML HL7 CompoundStatement) | Mapped from (JSON FHIR / other source )                                                                                                                      |
|---------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                    | Unique ID generated by the adaptor                                                                                                                           |
| code \[@code]                         | `Observation.code.coding[0].code`                                                                                                                            |
| code \[@displayName]                  | `Observation.code.coding[0].display`                                                                                                                         |
| code \[@codeSystem]                   | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                             |
| code / originalText                   | `coding.text` <br/> or else `coding.display` <br/> or else `coding.extension[index].value` where `coding.extension[index].url` equals `"descriptionDisplay"` |
| statusCode                            | fixed value = `"COMPLETE"`                                                                                                                                   |
| effectiveTime                         | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                                        |
| availabilityTime \[@value]            | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                                  |
| Participant \[@typecode]              | fixed value = `"PRF"`                                                                                                                                        |
| Participant / agentRef / id \[@root]  | `Observation.performer[0]`                                                                                                                                   | 

### Observation components

The blood pressure should have two `ObservationStatement` components, one with a systolic and one with a diastolic 
SNOMED code.  

| Mapped to (XML HL7 CompoundStatement / component / ObservationStatement) | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                |
|--------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                                       | Unique ID generated by the adaptor                                                                                                                                                                                                                     |
| code \[@code]                                                            | `Observation.component[index].code.coding[0].code`                                                                                                                                                                                                     |
| code \[@displayName]                                                     | `Observation.component[index].code.coding[0].display`                                                                                                                                                                                                  |
| code \[@codeSystem]                                                      | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                                                                                                                       |
| code / originalText                                                      | `component[index].code.coding.text` <br/> or else `component[index].code.coding.display` <br/> or else `component[index].code.coding.extension[index2].value` where `component[index].code.coding.extension[index2].url` equals `"descriptionDisplay"` |
| statusCode                                                               | fixed value = `"COMPLETE"`                                                                                                                                                                                                                             |
| effectiveTime                                                            | same as the Parent Compound Statement's `effectiveTime`                                                                                                                                                                                                |
| availabilityTime \[@value]                                               | same as the Parent Compound Statement's `availatilityTime`                                                                                                                                                                                             |
| value                                                                    | `Observation.component[index].valueQuantity`                                                                                                                                                                                                           |
| referenceRange / referenceInterpretationRange / text                     | `Observation.component[index].referenceRange.text`                                                                                                                                                                                                     |
| referenceRange / referenceInterpretationRang / value                     | `Observation.component[index].referenceRange.value`                                                                                                                                                                                                    |

### Narrative Statement components

A blood pressure may have a `NarrativeStatement` component if the parent observation has a comment or if `Observation.component.dataAbsentReason`, 
`Observation.bodySite` or `Observation.component.interpretation` are populated.   

| Mapped to (XML HL7 CompoundStatement / component / NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                                                              |
|------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                                     | Unique ID generated by the adaptor                                                                                                                   |
| text                                                                   | Concatenated from `Observation.comment`, `Observation.bodySite`, `Observation.component.dataAbsentReason` and `Observation.component.interpretation` |
| statusCode                                                             | fixed value = `"COMPLETE"`                                                                                                                           |
| availabilityTime \[@value]                                             | same as the Parent Compound Statement's `availatilityTime`                                                                                           |

<details>
   <summary>Example XML</summary>

```XML

<CompoundStatement classCode="BATTERY" moodCode="EVN">
   <id root="7D78F696-D3C7-43A9-B97E-983D9C55E1C8"/>
   <code code="163020007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="O/E - blood pressure reading">
      <originalText>O/E - blood pressure reading</originalText>
   </code>
   <statusCode code="COMPLETE"/>
   <effectiveTime>
      <center value="20100206124100"/>
   </effectiveTime>
   <availabilityTime value="20100206124100"/>
   <component typeCode="COMP" contextConductionInd="true">
      <ObservationStatement classCode="OBS" moodCode="EVN">
         <id root="6C2E2D92-39C1-44B2-BCEC-57BAFC951FBB"/>
         <code code="72313002" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Systolic arterial pressure">
            <originalText>Systolic blood pressure</originalText>
         </code>
         <statusCode code="COMPLETE"/>
         <effectiveTime>
            <center value="20100206124100"/>
         </effectiveTime>
         <availabilityTime value="20100206124100"/>
         <value xsi:type="PQ" value="170.000" unit="1">
            <translation value="170.000">
               <originalText>mm[Hg]</originalText>
            </translation>
         </value>
      </ObservationStatement>
   </component>
   <component typeCode="COMP" contextConductionInd="true">
      <ObservationStatement classCode="OBS" moodCode="EVN">
         <id root="67C2CFED-2CAE-4E44-8C65-8421994C8B9D"/>
         <code code="1091811000000102" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
               displayName="Diastolic arterial pressure">
            <originalText>Diastolic blood pressure</originalText>
         </code>
         <statusCode code="COMPLETE"/>
         <effectiveTime>
            <center value="20100206124100"/>
         </effectiveTime>
         <availabilityTime value="20100206124100"/>
         <value xsi:type="PQ" value="130.000" unit="1">
            <translation value="130.000">
               <originalText>mm[Hg]</originalText>
            </translation>
         </value>
      </ObservationStatement>
   </component>
   <Participant typeCode="PRF" contextControlCode="OP">
      <agentRef classCode="AGNT">
         <id root="6D6BF46C-476B-4955-AE07-CB53C1D9CC40"/>
      </agentRef>
   </Participant>
</CompoundStatement>
```
</details>

## Comment Note

Where a FHIR `Observation` resource is coded as a Comment Note (37331000000100) and it is not part of an investigation,
it is mapped to a HL7 `NarrativeStatement` as follows.

| Mapped to (XML HL7 NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                   |
|----------------------------------------|-----------------------------------------------------------------------------------------------------------|
| id                                     | Unique ID generated by the adaptor                                                                        |
| text                                   | `Observation.comment`                                                                                     |
| statusCode                             | fixed value = `"COMPLETE"`                                                                                |
| availabilityTime \[@value]             | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start` or else `Observation.issued`  |
| Participant \[@typecode]               | fixed value = `"PRF"`                                                                                     |
| Participant / agentRef / id \[@root]   | `Observation.performer[0]`                                                                                | 



<details>
   <summary>Example XML</summary>

```XML

<NarrativeStatement classCode="OBS" moodCode="EVN">
   <id root="D2FD6804-2CB9-48D7-950E-E616661C887C"/>
   <text>This is a free text note under history</text>
   <statusCode code="COMPLETE"/>
   <availabilityTime value="20100206124100"/>
   <Participant typeCode="PRF" contextControlCode="OP">
      <agentRef classCode="AGNT">
         <id root="6D6BF46C-476B-4955-AE07-CB53C1D9CC40"/>
      </agentRef>
   </Participant>
</NarrativeStatement>
```
</details>

## Investigations

The Test Group / Result structure is itself nested as a component of the Specimen `compoundStatement` (not shown here), 
which forms part if the [Diagnostic Report](../diagnostic%20reports/README.md) structure.

### Narrative statement mapping for Test Group Header and Test Result

Where stated in the Test Group Header and Test Result mapping, a collection of `NarrativeStatement` components may be 
used to preserve data. Where this takes place, they will be mapped from the appropriate `ObservationStatement` in the following way:

| Mapped to (XML HL7 NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                        |
|----------------------------------------|----------------------------------------------------------------------------------------------------------------|
| id                                     | Unique ID generated by the adaptor                                                                             |
| text \[@mediaType]                     | fixed value = `"text/x-h7uk-pmip"`                                                                             |
| text                                   | An EDIFACT comment type of `AGGREGATE COMMENT SET` where the content is mapped from one of fields stated below |
| statusCode                             | fixed value = `"COMPLETE"`                                                                                     |
| availabilityTime                       | Same as parent `CompoundStatement`                                                                             |

Note - this is in addition to filing Comments which also mapped to narrative statements and described below


#### Observation Fields mapped to Narrative Statement

Where multiple fields are stated they are concatenated into a single EDIFACT comment

- `dataAbsentReason` (prefixed with *"Data Absent: "*)
- `interpretation` (prefixed with *"Interpretation: "*), `comment`, `valueString` (prefixed with *"Value: "*), 
`referenceRange[0].text` (prefixed with *"Range Text: "*) and `referenceRange[0].high.unit` or `referenceRange[0].low.unit` 
(prefixed with *"Range Units: "*)   
- `bodySite` (prefixed with *"Site: "*)
- `method` (prefixed with *"Method: "*)


### Test Group Header

A Test Group Header is mapped to a `CompoundStatement` with a `classcode` of `"BATTERY"`, Test Results (below) can be 
nested as components of the header.  

| Mapped to (XML HL7 CompoundStatement)     | Mapped from (JSON FHIR / other source )                                                                                                                                     |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                        | Unique ID generated by the adaptor                                                                                                                                          |
| code                                      | `Observation.code.coding` as described in [Codeable Concept](../codeable%20concept/README.md)                                                                               |
| statusCode                                | fixed value = `"COMPLETE"`                                                                                                                                                  |
| effectiveTime                             | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                                                       |
| availabilityTime                          | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                                                 |
| component\[index] / ObservationStatement  | Mapped Test Results (see below)                                                                                                                                             |  
| component\[index] / CompoundStatement     | Mapped Test Results (see below)                                                                                                                                             |
| component\[index] / NarrativeStatement    | Mapped from `Observation` as outlined in **Narrative statement mapping for Test Group Header and Test Result** (above) <sup>8</sup> and filing Comments, as described below |

8. `NarrativeStatement` components are created for each of the fields stated in **Observation Fields mapped to Narrative Statement**. If none of 
these fields are present, and no filing comments are present (see below) the `NarrativeStatement` will be omitted.

<details>
   <summary>Example XML</summary>

```XML

<CompoundStatement classCode="BATTERY" moodCode="EVN">
   <id root="1022BBB9-5DD2-439A-BB78-9227B36D8BC6"/>
   <code code="196411000000103" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
         displayName="Transfer-degraded record entry">
      <originalText>CHOL/HDL RATIO</originalText>
   </code>
   <statusCode code="COMPLETE"/>
   <effectiveTime>
      <center value="20050315"/>
   </effectiveTime>
   <availabilityTime value="20050315"/>
   <component contextConductionInd="true" typeCode="COMP">
      <NarrativeStatement classCode="OBS" moodCode="EVN">
         <id root="51962DA9-87DB-4054-A301-A0674BF62FA4"/>
         <text mediaType="text/x-h7uk-pmip">CommentType:AGGREGATE COMMENT SET
CommentDate:20050315

See FATS/Healthy Hearts guidelines for interpretation of lipids</text>
         <statusCode code="COMPLETE"/>
         <availabilityTime value="20050315"/>
      </NarrativeStatement>
   </component>
   <component contextConductionInd="true" typeCode="COMP">
 
      ...
      
   </component>
   <component contextConductionInd="true" typeCode="COMP">

      ...

   </component>

</CompoundStatement>

```
* note - the Transfer-degraded record entry was present in the FHIR, it has not been added by the Adaptor.
</details>

### Test result

A Test Result `Observation` is mapped to an `ObservationStatement`. 

Where a `NarrativeStatement` is used to preserve data from an `Observation`, the `ObservationStatement` and 
`NarrativeStatement` will be wrapped in a `CompoundStatement` with a class code of `CLUSTER`. 

#### Observation statement
| Mapped to (XML HL7 ObservationStatement)             | Mapped from (JSON FHIR / other source )                                                                                         |
|------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                                          | Unique ID generated by the adaptor                                                                                              |
| code                                                 | `Observation.code.coding` as described in [Codeable Concept](../codeable%20concept/README.md)                                   |
| statusCode                                           | fixed value = `"COMPLETE"`                                                                                                      |
| effectiveTime                                        | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                           |
| availabilityTime                                     | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                     |
| value                                                | `Observation.valueQuantity` or `Observation.valueString`                                                                        |
| interpretationCode \[@code]                          | `Observation.interpretation.coding.code`                                                                                        | 
| interpretationCode \[@codeSystem]                    | fixed value = `"2.16.840.1.113883.2.1.6.5"`                                                                                     |
| interpretationCode \[@displayName]                   | description of `Observation.interpretation.coding.code` e.g. if code = `"HI"` then displayName = `"Above high reference limit"` |
| interpretationCode / originalText                    | `Observation.interpretation.coding.display`                                                                                     |
| referenceRange / referenceInterpretationRange / text | `Observation.referenceRange.text`                                                                                               |
| referenceRange / referenceInterpretationRang / value | `Observation.referenceRange.value`                                                                                              |

#### Compound Statement Wrapper
| Mapped to (XML HL7 CompoundStatement) | Mapped from (JSON FHIR / other source )                                                                                                                                     |
|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                           | Unique ID generated by the adaptor                                                                                                                                          |
| code                                  | `Observation.code.coding` as described in [Codeable Concept](../codeable%20concept/README.md)                                                                               |
| statusCode                            | fixed value = `"COMPLETE"`                                                                                                                                                  |
| effectiveTime                         | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                                                       |
| availabilityTime                      | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                                                 |
| component / ObservationStatement      | The Observation Statement as mapped above                                                                                                                                   |
| component / NarrativeStatement        | Mapped from `Observation` as outlined in **Narrative statement mapping for Test Group Header and Test Result** (above) <sup>8</sup> and filing Comments, as described below |

<details>
<summary>Example XML</summary>

```XML

<CompoundStatement classCode="CLUSTER" moodCode="EVN">
   <id root="5F2D05CF-8FF5-42EC-9130-1E707921257A"/>
   <code code="2551271000000118" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Serum cholesterol level"/>
   <statusCode code="COMPLETE"/>
   <effectiveTime>
      <center value="20050315"/>
   </effectiveTime>
   <availabilityTime value="20050315"/>
   <component contextConductionInd="true" typeCode="COMP">
      <ObservationStatement classCode="OBS" moodCode="EVN">
         <id root="237E66DB-5FA8-4057-B295-7A0DFFF5173E"/>
         <code code="2551271000000118" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
               displayName="Serum cholesterol level"/>
         <statusCode code="COMPLETE"/>
         <effectiveTime>
            <center value="20050315"/>
         </effectiveTime>
         <availabilityTime value="20050315"/>
         <value unit="1" value="6.300" xsi:type="PQ">
            <translation value="6.300">
               <originalText>mmol/L</originalText>
            </translation>
         </value>
      </ObservationStatement>
   </component>
   <component contextConductionInd="true" typeCode="COMP">
      <NarrativeStatement classCode="OBS" moodCode="EVN">
         <id root="90A3BE12-0D75-49C8-9387-FEF7FF027378"/>
         <text mediaType="text/x-h7uk-pmip">CommentType:AGGREGATE COMMENT SET
CommentDate:20050315

ReportStatus: Supplementary result
         </text>
         <statusCode code="COMPLETE"/>
         <availabilityTime value="20050315"/>
      </NarrativeStatement>
   </component>
</CompoundStatement>
```
</details>

### Filing comments

Where Test Result Headers or Test Results have Observations coded as `Comment Note` referenced in `Observation.related` 
and `Observation.related.type` equals `has-member`, the referenced `Observation` is a Filing Comment. Filing Comments are mapped
to `NarrativeStatements` and inserted as components of the Test Group Header / Test Result `CompoundStatement`.

| Mapped to (XML HL7 NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                          |
|----------------------------------------|------------------------------------------------------------------------------------------------------------------|
| id \[@root]                            | Unique ID generated by the adaptor                                                                               |
| text \[@mediaType]                     | fixed value = `"text/x-h7uk-pmip"`                                                                               | 
| text                                   | EDIFACT comment type of USER COMMENT where the comment body is mapped from `Observation.comment` <sup>9</sup>    |
| availabilityTime                       | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                      |

9. If there is more than one filing comment for a given Observation, the comments are seperated by newlines and appended to the EDIFACT comment body.  

<details>

```XML

<NarrativeStatement classCode="OBS" moodCode="EVN">
   <id root="9E25E6E1-6799-454B-89E0-57C1952828D4"/>
   <text mediaType="text/x-h7uk-pmip">CommentType:USER COMMENT
CommentDate:20100326134545

(EMISTest) - Normal - No Action</text>
   <statusCode code="COMPLETE"/>
   <availabilityTime value="20100326134545"/>
</NarrativeStatement>
```
</details>

## Further documentation

### GP Connect Uncategorised data

[GP Connect uncategorised data guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html)

[GP Connect Observation - uncategorised data](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_uncategorisedData.html)

[GP Connect Observation - blood pressure](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_bloodPressure.html)

### GP Connect Investigations

[GP Connect Investigations guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_pathology_guidance.html)

[GP Connect Observation - test group header](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_testGroup.html)

[GP Connect Observation - test result](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_testResult.html)

[GP Connect Observation - filing comment](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_filingComments.html)

### HL7v3 MIM

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)