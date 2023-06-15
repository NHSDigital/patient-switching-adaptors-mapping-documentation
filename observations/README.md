# Observation Mapping

## XML HL7 > JSON FHIR

Observations are mapped from:

* Observation statements  (Uncategorised Data and Test Results)
* Request statements (Self Referral)
* Compound statements (Test Group Header, Blood Pressure and Componentised Observation Header)
* Narrative statements (Filing Comments)

### Code Mapping

The FHIR `CodeableConcept` element is used by multiple fields, where stated in the mapping tables below, the element
will be mapped in the following way:

| Mapped to (JSON FHIR code field)                        | Mapped from (XML HL7 code element/ other)                                                              |                                                                                                                            
|---------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| code.coding\[0].code                                    | `code [@code]` or `code / translation [@code]`  <sup>1</sup>                                           |
| code.coding\[0].system                                  | fixed value = `"http://snomed.info/sct"` <sup>1</sup>                                                  | 
| code.coding\[0].display                                 | found by searching the adaptors SNOMED database for the appropriate description <sup>1</sup>           |
| code.coding\[0].extension\[0].url                       | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid"` <sup>1</sup> |
| code.coding\[0].extension\[0].extension\[0].url         | fixed value = `"decriptionId"` <sup>1</sup>                                                            |
| code.coding\[0].extension\[0].extension\[0].valueId     | found by searching the adaptors SNOMED database for the preferred description ID <sup>1</sup>          |
| code.coding\[0].extension\[0].extension\[1].url         | fixed value = `"descriptionDisplay"` <sup>1</sup>                                                      |
| code.coding\[0].extension\[0].extension\[1].valueString | found by searching the adaptors SNOMED database for the preferred description <sup>1</sup>             |
| code.text                                               | `code [@displayName]` or else `code / originalText`                                                    |

1. If a SNOMED CT code cannot be found `code.coding` will not be populated.

### Uncategorised data

### Observation - Uncategorised data

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
| code                                             | `ObservationStatement / code` as described in **Code Mapping** above                                                                                     |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                                                  |
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                            |
| valueQuantity                                    | `ObservationStatement / value` where `ObservationStatement / value [@type]` is `"PQ"` or `"IVLPQ"`                                                       |
| valueQuantity.unit                               | `ObservationStatement / value [@unit]` or `ObservationStatement / value / (high or low)`                                                                 |
| valueQuantity.comparator                         | `ObservationStatement / value / (high or low) [@inclusive]` where `ObservationStatement / value [@type]` is `"IVLPQ"`                                    |
| valueQuantity.extension\[0].url                  | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1"` <sup>2</sup>                               |
| valueQuantity.extension\[0].value                | fixed value = `true` <sup>2</sup>                                                                                                                        |
| valueString                                      | `ObservationStatement / value` where valueQuantity is not populated                                                                                      |
| effectiveDateTime                                | `ObservationStatement / effectiveTime` or else `ObservationStatement / availibiltyTime [@value]`                                                         |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`                                                   |
| performer\[0]                                    | Practitioner referenced in `ObservationStatement / paticipant` <sup>4</sup>                                                                              |
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

2. If `ObservationStatement / uncertaintyCode` is present.
3. Where the `ehrComposition` is the parent of mapped statement.
4. Where `paticipant [@typecode]` is `"PPRF"` (Primary performer) or `"PRF"` (Performer).

### Observation - Blood pressure

Blood pressure observations are mapped from a HL7v3 `CompoundStatement` containing two `ObservationStatement` elements,
where the SNOMED codes identify them as blood pressure observations. The SNOMED codes used to identify a
"blood pressure triple" are listed in the
[Gp connect documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html#representing-blood-pressure-readings-from-gp-systems)
.

| Mapped to (JSON FHIR Observation resource field)    | Mapped from (XML HL7 / other)                                                                                                                          |
|-----------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                  | `CompoundStatement / id [@root]`                                                                                                                       |
| meta / profile\[0]                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                           |
| identifier\[0].system                               | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                        |
| identifier\[0].value                                | `CompoundStatement / id \[@root]`                                                                                                                      |
| status                                              | fixed value = `"final"`                                                                                                                                |
| code                                                | `CompoundStatement / code` as described in **Code Mapping** above                                                                                      | 
| subject                                             | reference to the mapped [Patient](../patient/README.md)                                                                                                |
| context                                             | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                          |
| effectiveDateTime                                   | `CompoundStatement / effectiveTime` or else `CompoundStatement / availibiltyTime [@value]`                                                             |
| issued                                              | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`                                                 | 
| performer\[0]                                       | Practitioner referenced in `CompoundStatement / paticipant` <sup>4</sup>                                                                               |
| component\[index].code                              | `ObservationStatement.code` as described in **Code Mapping** above <sup>5</sup>                                                                        |
| component\[index].valueQuantity                     | `ObservationStatement / value` where `ObservationStatement / value [@type]` is `"PQ"` or `"IVLPQ"`                                                     |
| component\[index].valueQuantity.extension\[0].url   | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1"` <sup>2</sup>                             |
| component\[index].valueQuantity.extension\[0].value | fixed value = `true` <sup>2</sup>                                                                                                                      |
| comment                                             | concatenated from `ObservationStatement / pertinentInformation / pertinentAnnotation / text` <sup>6</sup> and `NarrativeStatement / text` <sup>7</sup> |

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

5. Mapped for the both `ObservationStatement` elements that form part of the blood pressure triple.
6. Text from a `pertinentAnnotation` is prepended with `"Systolic Note: "` or `"Diastolic Note: "`,
   determined by the SNOMED code of the `OberservationStatement`.
7. Where the `NarrativeStatement` is a child of the blood pressure triple's `CompoundStatement` and prepended
   with `"BP Note: "`.

### Observation - Self referral

A HL7v3 `RequestStatement` is mapped to an `Observations` if `RequestStatement / code / qualifier /value [@code]`
is equal to `"SelfReferral`.

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                   |
|--------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| id                                               | `RequestStatement / id [@root]`                                                                                 |
| meta / profile\[0]                               | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                    |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice |
| identifier\[0].value                             | `RequestStatement / id \[@root]`                                                                                |
| status                                           | fixed value = `"final"`                                                                                         |
| code                                             | `RequestStatement / code` as described in **Code Mapping** above                                                |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`          |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                         |  
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                   |
| effectiveDateTime                                | `RequestStatement / effectiveTime` or else `RequestStatement / availibiltyTime [@value]`                        |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`          |                                                   
| performer\[0]                                    | [Practitioner](../practioners/README.md) referenced in `RequestStatement / paticipant` <sup>5</sup>             |      
| comment                                          | fixed value = `"SelfRerreral"`                                                                                  |
| component\[0].code.text                          | fixed value = `"Urgency"`                                                                                       |
| component\[0].valueString                        | `RequestStatement / priorityCode / originalText`                                                                |
| component\[1].code.text                          | fixed value = `"Text"`                                                                                          |
| component\[1].valueString                        | `RequestStatement / text`                                                                                       |

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

5. Where `ObservationStatement / paticipant [@typecode]` is `"PPRF"` (Primary performer) or `"PRF"` (Performer).

### Investigations

### Observation - Test group header

Observation - Test group headers are mapped from a nested `CompoundStatement` with a class code of `BATTERY` where
the parent compound statements are deemed to form a [Diagnostic Report](../diagnostic%20reports/README.md).

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
| code                                             | `CompoundStatement / code` as described in **Code Mapping** above                                                                 |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                                           |  
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                                     |
| effectiveDateTime                                | `CompoundStatement / effectiveTime` or else `CompoundStatement / availibiltyTime [@value]`                                        |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`                            |
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

### Observation - Test result

Test Result observations, with the exception of the related field, are mapped identically to Uncategorised Data (above).

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

### Observation - Filing comment

Filing Comment observations are mapped from diagnostic report `NarrativeStatements` where the EDIFACT comment type is
`USER COMMENT`.

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
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`                           |
| performer\[0]                                    | Practitioner referenced in `CompoundStatement / paticipant` <sup>4</sup>                                                         |
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

When a `CompoundStatement` element has a class code of `BATTERY` or `CLUSTER` and does not form part of a pathology 
investigation or a blood pressure, it is mapped to an `observation` resource. Additionally, any child `ObservationStatement` 
will be referenced using the `"derived-from"` / `"has-member"` relationship.

### Observation - Componentised observation header

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                   |
|--------------------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| id                                               | `CompoundStatement / id [@root]`                                                                                |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                    |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice |
| identifier\[0].value                             | `CompoundStatement / id \[@root]`                                                                               |
| status                                           | fixed value = `"final"`                                                                                         |
| code                                             | `CompoundStatement / code` as described in **Code Mapping** above                                               |
| subject                                          | reference to the mapped [Patient](../patient/README.md)                                                         |
| context                                          | reference to the associated [Encounter](../encounters/README.md) (if present)                                   |
| effectiveDateTime                                | `CompoundStatement / effectiveTime` or else `CompoundStatement / availibiltyTime [@value]`                      |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup> or else `EhrExtract / availibilityTime [@value]`          |
| performer\[0]                                    | Practitioner referenced in `ObservationStatement / paticipant` <sup>4</sup>                                     |
| related\[index].type                             | fixed value = `"has-member"`                                                                                    |
| related\[index].target                           | reference to the child Observation - Test result (below)                                                        |

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

### Observation - Componentised observation

The `ObservationStatement` components nested within the header, with the exception of the related field, 
are mapped identically to Uncategorised Data (above).

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

## JSON FHIR > XML HL7

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