# Observation Mapping

Main section links:
- **[XML HL7 > JSON FHIR](#xml-hl7--json-fhir)**
- **[JSON FHIR > XML HL7](#json-fhir--xml-hl7)**
- **[Further documentation](#further-documentation)**

# XML HL7 > JSON FHIR

FHIR Observations are used throughout the Structured Record. As a result they are mapped from multiple HL7 components 
as stated below, where the items in **bold** are the FHIR `Observation` resource types, the mapping of which is 
described in the following sections:

- Observation statements  map to **[Uncategorised Data](#uncategorised-data-xml-hl7--json-fhir)**, **[Test Result](#test-result-xml-hl7--json-fhir)** and **[Componentised Observation](#componentised-observation-xml-hl7--json-fhir)**
- Request statements map to **[Self Referral](#self-referral-xml-hl7--json-fhir)**
- Compound statements map to **[Test Group Header](#test-group-header-xml-hl7--json-fhir)**, **[Blood Pressure](#blood-pressure-xml-hl7--json-fhir)** and **[Componentised Observation Header](#componentised-observation-header-xml-hl7--json-fhir)**
- Narrative statements map to **[Filing Comment](#filing-comment-xml-hl7--json-fhir)**

## Uncategorised Data (XML HL7 > JSON FHIR)

Uncategorised data is mapped from an `ObservationStatement` to to an `Observation` in the following way, if it is not
deemed to be an
[Allergy Intolerance](../allergy%20intolerances/README.md), [Blood Pressure](#blood-pressure-xml-hl7--json-fhir)
or [Immunization](../immunisations/README.md).

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                                                                                                                                                 |
|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `ObservationStatement / id \[@root]`                                                                                                                                                                                                                          |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                                                                                                                                  |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                                                                               |
| identifier\[0].value                             | `ObservationStatement / id \[@root]`                                                                                                                                                                                                                          |
| status                                           | fixed value = `"final"`                                                                                                                                                                                                                                       |
| code                                             | `ObservationStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                                                                                                                                   |
| subject.reference                                | reference to the mapped [Patient](../patient/README.md)                                                                                                                                                                                                       |
| context.reference                                | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                                                                                                                                 |
| valueQuantity                                    | `ObservationStatement / value` where `ObservationStatement / value [@type]` is `"PQ"` or `"IVLPQ"`                                                                                                                                                            |
| valueQuantity.unit                               | `ObservationStatement / value [@unit]` or `ObservationStatement / value / (high or low)`                                                                                                                                                                      |
| valueQuantity.comparator                         | `ObservationStatement / value / (high or low) [@inclusive]` where `ObservationStatement / value [@type]` is `"IVLPQ"`                                                                                                                                         |
| valueQuantity.extension\[0].url                  | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1"` <sup>1</sup>                                                                                                                                    |
| valueQuantity.extension\[0].value                | fixed value = `true` <sup>1</sup>                                                                                                                                                                                                                             |
| valueString                                      | `ObservationStatement / value` where valueQuantity is not populated                                                                                                                                                                                           |
| effective(x) <sup>2</sup>                        | `ObservationStatement / effectiveTime`<sup>2</sup> or else `ObservationStatement / availibiltyTime [@value]` <sup>2</sup>                                                                                                                                     |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup>                                                                                                                                                                                                         |
| performer\[0].reference                          | Practitioner referenced in `ObservationStatement / participant` <sup>4</sup>, or else `EhrComposition / Participant2`                                                                                                                                         |
| interpretation.coding\[0].code                   | `ObservationStatement / interpretationCode [@code]`                                                                                                                                                                                                           |
| interpretation.coding\[0].display                | `ObservationStatement / interpretationCode [@code]` - human readable translation i.e. `"High"`, `"Low"`, `"Abnormal"`                                                                                                                                         |
| interpretation.coding\[0].system                 | fixed value = `"http://hl7.org/fhir/v2/0078"`                                                                                                                                                                                                                 |
| interpretation.text                              | `ObservationStatement / interpretationCode / originalText` or else `ObservationStatement / interpretationCode [@displayName]`                                                                                                                                 |                                         
| comment                                          | Concatenated from `ObservationStatement / subject / personalRelationship / code` <br/> and each `ObservationStatement / pertinentInformation / pertinentAnnotation` in the sequence defined by `ObservationStatement / pertinentInformation / sequenceNumber` |
| referenceRange\[index].text                      | `ObservationStatement / referenceRange[index] / referenceInterpretationRange / text`                                                                                                                                                                          |
| referenceRange\[index].high.value                | `ObservationStatement / referenceRange\[index] / referenceInterpretationRange / value / high`                                                                                                                                                                 |
| referenceRange\[index].low.value                 | `ObservationStatement / referenceRange\[index] / referenceInterpretationRange / value / low`                                                                                                                                                                  |

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
                           "valueId": "2564971000000119"
                        },
                        {
                           "url": "descriptionDisplay",
                           "valueString": "Plasma triglyceride level"
                        }
                     ]
                  }
               ],
               "system": "http://snomed.info/sct",
               "code": "1010601000000105",
               "display": "Plasma triglyceride level"
            }
         ],
         "text": "Plasma triglyceride level"
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
      ],
      "valueQuantity": {
         "value": 10,
         "unit": "millimole per liter",
         "code": "mmol/L"
      },
      "interpretation": {
         "coding": [
            {
               "system": "http://hl7.org/fhir/v2/0078"
            }
         ],
         "text": "Potentially abnormal"
      },
      "comment": "Less than or equal to 5 and abnormal",
      "referenceRange": [
         {
            "low": {
               "value": 5
            }
         }
      ]
   }
}
```

</details>

1. If `ObservationStatement / uncertaintyCode` is present.
2. Where `ObservationStatement / effectiveTime` does not contain the `center` element but does contain either `low` or `high` the values are mapped to **effectivePeriod.start** and **effectivePeriod.end** respectfully. Otherwise, `ObservationStatement / effectiveTime / center` or else `ObservationStatement / availibiltyTime [@value]` are mapped to **effectiveDateTime**.       
3. Where the `ehrComposition` is the parent of mapped statement.
4. Where `participant [@typecode]` is `"PPRF"` (Primary performer) or `"PRF"` (Performer).

## Blood Pressure (XML HL7 > JSON FHIR)

Blood pressure observations are mapped from a HL7 `CompoundStatement` containing two `ObservationStatement` components,
known as a "blood pressure triple", where the SNOMED codes identify them as a blood pressure. The SNOMED codes used to identify a
"blood pressure triple" are listed in the
[GP Connect documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html#representing-blood-pressure-readings-from-gp-systems)
.

| Mapped to (JSON FHIR Observation resource field)    | Mapped from (XML HL7 / other)                                                                                                                                                                                                                                                                                        |
|-----------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                  | `CompoundStatement / id [@root]`                                                                                                                                                                                                                                                                                     |
| meta.profile\[0]                                    | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                                                                                                                                                                                         |
| meta.security                                       | When `CompoundStatement / confidentialityCode [@code]`, `EhrComposition / confidentialityCode [@code]`, or any `CompoundStatement / ObservationStatement[] / confidentialityCode [@code]` is present and has a value of `NOPAT`. See [Confidentiality Codes](../confidentiality code/README.md) for mapping details. |
| identifier\[0].system                               | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                                                                                                                                      |
| identifier\[0].value                                | `CompoundStatement / id \[@root]`                                                                                                                                                                                                                                                                                    |
| status                                              | fixed value = `"final"`                                                                                                                                                                                                                                                                                              |
| code                                                | `CompoundStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                                                                                                                                                                                             | 
| subject.reference                                   | reference to the mapped [Patient](../patient/README.md)                                                                                                                                                                                                                                                              |
| context.reference                                   | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                                                                                                                                                                                        |
| effective(x) <sup>2</sup>                           | `CompoundStatement / effectiveTime` <sup>2</sup> or else `CompoundStatement / availibiltyTime [@value]` <sup>2</sup>                                                                                                                                                                                                 |
| issued                                              | `ehrCompostion / author / time [@value]` <sup>3</sup>                                                                                                                                                                                                                                                                | 
| performer\[0].reference                             | Practitioner referenced in `CompoundStatement / participant` <sup>4</sup>, or else `EhrComposition / Participant2`                                                                                                                                                                                                   |
| component\[index].code                              | `ObservationStatement.code` <sup>5</sup>   as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                                                                                                                                                                             |
| component\[index].valueQuantity                     | `ObservationStatement / value` where `ObservationStatement / value [@type]` is `"PQ"` or `"IVLPQ"` <sup>5</sup>                                                                                                                                                                                                      |
| component\[index].valueQuantity.extension\[0].url   | fixed value = `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1"` <sup>1</sup>                                                                                                                                                                                           |
| component\[index].valueQuantity.extension\[0].value | fixed value = `true` <sup>1</sup>                                                                                                                                                                                                                                                                                    |
| component\[index].interpretation.coding\[0].code    | `ObservationStatement / interpretationCode [@code]`                                                                                                                                                                                                                                                                  |
| component\[index].interpretation.coding\[0].display | `ObservationStatement / interpretationCode [@code]` - human readable translation i.e. `"High"`, `"Low"`, `"Abnormal"`                                                                                                                                                                                                |
| component\[index].interpretation.coding\[0].system  | fixed value = `"http://hl7.org/fhir/v2/0078"`                                                                                                                                                                                                                                                                        |
| component\[index].referenceRange\[index].text       | `ObservationStatement / referenceRange[index] / referenceInterpretationRange / text`                                                                                                                                                                                                                                 |
| component\[index].referenceRange\[index].high.value | `ObservationStatement / referenceRange\[index] / referenceInterpretationRange / value / high`                                                                                                                                                                                                                        |
| component\[index].referenceRange\[index].low.value  | `ObservationStatement / referenceRange\[index] / referenceInterpretationRange / value / low`                                                                                                                                                                                                                         |
| comment                                             | concatenated from each `ObservationStatement / pertinentInformation / pertinentAnnotation / text` <sup>6</sup> and `NarrativeStatement / text` <sup>7</sup><br>Qualifiers such as episodicity (SNOMED 288526004) may be added, wrapped in curly brackets.                                                            |

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
      ],
      "security": [
        {
          "system":"http://hl7.org/fhir/v3/ActCode",
          "code":"NOPAT",
          "display":"no disclosure to patient, family or caregivers without attending provider's authorization"
        }
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
    ],
    "comment": "{Episodicity : code=255217005, displayName=First}"
  }
}
```

</details>

5. Mapped for the both `ObservationStatement` elements that form part of the blood pressure triple.
6. Text from a `pertinentAnnotation` is prepended with `"Systolic Note: "` or `"Diastolic Note: "`,
   determined by the SNOMED code of the `OberservationStatement`. If an appropriate code isn't present, it is not mapped.
7. Where the `NarrativeStatement` is a child of the blood pressure triple's `CompoundStatement`. Prepended
   with `"BP Note: "` when mapped.

## Self Referral (XML HL7 > JSON FHIR)

If a referral is deemed to be a self referral, the HL7 `RequestStatement` is mapped to an FHIR `Observation`. Self referrals 
are identified by `RequestStatement / code / qualifier /value [@code]`, where it is equal to `"SelfReferral`.

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                                   |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `RequestStatement / id [@root]`                                                                                                                 |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                    |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                 |
| identifier\[0].value                             | `RequestStatement / id \[@root]`                                                                                                                |
| status                                           | fixed value = `"final"`                                                                                                                         |
| code                                             | `RequestStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                         |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup>                                                                                           |
| subject.reference                                | reference to the mapped [Patient](../patient/README.md)                                                                                         |  
| context.reference                                | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                   |
| effective(x) <sup>2</sup>                        | `RequestStatement / effectiveTime` <sup>2</sup> or else `RequestStatement / availibiltyTime [@value]` <sup>2</sup>                              | 
| performer\[0].reference                          | [Practitioner](../practitioners/README.md) referenced in `RequestStatement / participant` <sup>8</sup>, or else `EhrComposition / Participant2` |      
| comment                                          | fixed value = `"SelfRerreral"`                                                                                                                  |
| component\[0].code.text                          | fixed value = `"Urgency"`                                                                                                                       |
| component\[0].valueString                        | `RequestStatement / priorityCode / originalText`                                                                                                |
| component\[1].code.text                          | fixed value = `"Text"`                                                                                                                          |
| component\[1].valueString                        | `RequestStatement / text`                                                                                                                       |

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

8. Where `ObservationStatement / participant [@typecode]` is `"PPRF"` (Primary performer) or `"PRF"` (Performer).

## Investigations

For more information on how Investigations are represented in GP Connect see [Investigations Guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_pathology_guidance.html).


### Test Group Header (XML HL7 > JSON FHIR)

Test group headers are mapped from a nested `CompoundStatement` with a class code of `BATTERY` where
the parent compound statements are deemed to from a [Diagnostic Report](../diagnostic%20reports/README.md).

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                                                                                |
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `CompoundStatement / id [@root]`                                                                                                                                                             |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                                                                 |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                              |
| identifier\[0].value                             | `CompoundStatement / id \[@root]`                                                                                                                                                            |
| status                                           | fixed value = `"final"`                                                                                                                                                                      |
| category\[0].coding\[0].code                     | fixed value = `"laboratory"`                                                                                                                                                                 |
| category\[0].coding\[0].system                   | fixed value = `"http://hl7.org/fhir/observation-category"`                                                                                                                                   |
| category\[0].coding\[0].display                  | fixed value = `"Laboratory"`                                                                                                                                                                 |
| code                                             | `CompoundStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                                                                     |
| subject.reference                                | reference to the mapped [Patient](../patient/README.md)                                                                                                                                      |  
| context.reference                                | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                                                                |
| effective(x) <sup>2</sup>                        | `CompoundStatement / effectiveTime` <sup>2</sup> or else `CompoundStatement / availibiltyTime [@value]` <sup>2</sup>                                                                         |
| issued                                           | `ObservationStatement / availabilityTime [@value]`, or else `CompoundStatement / availibiltyTime [@value]` for `Filed Report`, or else `ehrCompostion / author / time [@value]` <sup>3</sup> |
| performer\[0].reference                          | [Practitioner](../practitioners/README.md) referenced in `CompoundStatement / participant` or else `EhrComposition / Participant2`                                                           |
| comment                                          | concatenated with newlines from child `NarrativeStatement / text` where the EDIFACT comment type is not `USER COMMENT`                                                                       |
| specimen.reference                               | reference to the [Specimen](../diagnostic%20reports/README.md)                                                                                                                               |
| related\[index].type                             | fixed value = `"has-member"`                                                                                                                                                                 |
| related\[index].target.reference                 | reference to component [Test Result](#test-result-xml-hl7--json-fhir)                                                                                                                        |

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

### Test Result (XML HL7 > JSON FHIR)

Except for the fields outlined below, Test Result observations are mapped identically to [Uncategorised Data](#uncategorised-data-xml-hl7--json-fhir). 

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                                    |
|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------|
| comment                                          | concatenated with newlines from child `NarrativeStatement / text` where the EDIFACT comment type is not `USER COMMENT`                           |
| issued                                           | `ObservationStatement / availabilityTime [@value]` if populated, otherwise the default Uncategorised Data issued mapping                         |
| related\[index].type                             | fixed value = `"derived-from"` or fixed value = `"has-member"`                                                                                   |
| related\[index].target.reference                 | reference to the parent [Test Group Header](#test-group-header-xml-hl7--json-fhir)                                                               |
| specimen.reference                               | reference to the [Specimen](../diagnostic%20reports/README.md)                                                                                   |
| category\[0].coding\[0].code                     | fixed value = `"laboratory"`                                                                                                                     |
| category\[0].coding\[0].system                   | fixed value = `"http://hl7.org/fhir/observation-category"`                                                                                       |
| category\[0].coding\[0].display                  | fixed value = `"Laboratory"`                                                                                                                     |

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

### Filing Comment (XML HL7 > JSON FHIR)

Filing Comment observations are mapped from `NarrativeStatements` where they are components of a 
Test Group Header / Test Result `CompoundStatement` and the EDIFACT comment type is `USER COMMENT`.
When the `NarrativeStatement` is contained with a [Blood Pressure](#blood-pressure-json-fhir--xml-hl7) an additional 
Filing Comment is not created.

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                                                                                                                                                                                                             |
|--------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `NarrativeStatement / id [@root]`                                                                                                                                                                                                                                                                         |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                                                                                                                                                                                                              |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                                                                                                                           |
| identifier\[0].value                             | `NarrativeStatement / id \[@root]`                                                                                                                                                                                                                                                                        |
| status                                           | fixed value = `"unknown"`                                                                                                                                                                                                                                                                                 |
| code.coding\[0].system                           | fixed value = `"http://snomed.info/sct"`                                                                                                                                                                                                                                                                  |
| code.coding\[0].code                             | fixed value = `"37331000000100"`                                                                                                                                                                                                                                                                          |
| code.coding\[0].display                          | fixed value = `"Comment note"`                                                                                                                                                                                                                                                                            |
| subject.reference                                | reference to the mapped [Patient](../patient/README.md)                                                                                                                                                                                                                                                   |
| context.reference                                | reference to the associated [Encounter](../encounters/README.md) (if present)                                                                                                                                                                                                                             |
| issued                                           | `ObservationStatement / availabilityTime [@value]`, or else `ehrComposition / author / time [@value]` <sup>3</sup>                                                                                                                                                                                        |
| performer\[0].reference                          | Practitioner referenced in `ehrComposition / author / agentRef`                                                                                                                                                                                                                                           |
| effectiveTime                                    | `ehrComposition / author / time [@value]` <sup>3</sup>                                                                                                                                                                                                                                                    |
| related\[0].type                                 | fixed value = `"derived-from"`                                                                                                                                                                                                                                                                            |
| related\[0].target.reference                     | reference to the parent [Test Group Header](#test-group-header-xml-hl7--json-fhir) or [Test Result](#test-result-xml-hl7--json-fhir). Where the filing comments relate to the `test report`, the reference is made from the `test report` to the filing comment only and `related` will not be presented. |


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

### Componentised observation header (XML HL7 > JSON FHIR)

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                            |
|--------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------|
| id                                               | `CompoundStatement / id [@root]`                                                                                         |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Observation-1"`                             |
| identifier\[0].system                            | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice          |
| identifier\[0].value                             | `CompoundStatement / id \[@root]`                                                                                        |
| status                                           | fixed value = `"final"`                                                                                                  |
| code                                             | `CompoundStatement / code` as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md) |
| subject.reference                                | reference to the mapped [Patient](../patient/README.md)                                                                  |
| context.reference                                | reference to the associated [Encounter](../encounters/README.md) (if present)                                            |
| effective(x) <sup>2</sup>                        | `CompoundStatement / effectiveTime` <sup>2</sup> or else `CompoundStatement / availibiltyTime [@value]` <sup>2</sup>     |
| issued                                           | `ehrCompostion / author / time [@value]` <sup>3</sup>                                                                    |
| performer\[0].reference                          | Practitioner referenced in `ObservationStatement / participant` <sup>4</sup>, or else `EhrComposition / Participant2`    |
| related\[index].type                             | fixed value = `"has-member"`                                                                                             |
| related\[index].target                           | reference to the child Observation - [Test Result](#test-result-xml-hl7--json-fhir)                                      |

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

### Componentised Observation (XML HL7 > JSON FHIR)

With the exception of the related field, Componentised Observations are mapped identically to [Uncategorised Data](#uncategorised-data-xml-hl7--json-fhir).

| Mapped to (JSON FHIR Observation resource field) | Mapped from (XML HL7 / other)                                                                                     |
|--------------------------------------------------|-------------------------------------------------------------------------------------------------------------------|
| related\[index].type                             | fixed value = `"derived-from"`                                                                                    |
| related\[index].target                           | reference to the parent [Componentised Observation Header](#componentised-observation-header-xml-hl7--json-fhir)) |

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

## Unmapped fields

The following Observation fields are not currently populated by the adaptor:

- implicitRules
- language
- text
- contained
- modifierExtension
- identifier.type
- identifier.use
- identifier.period
- identifier.assigner
- basedOn
- category.id
- category.coding.id
- category.coding.version
- category.coding.userSelected
- category.coding.extension
- category.text
- subject.id
- subject.identifier
- subject.display
- context.id
- context.identifier
- context.display
- performer.id
- performer.identifier
- performer.display
- dataAbsentReason
- interpretation.coding.id
- interpretation.coding.version
- interpretation.coding.userSelected
- bodySite
- method
- specimen.id
- specimen.identifier
- specimen.display
- device
- referenceRange.id
- referenceRange.modifierExtension
- referenceRange.low.id
- referenceRange.low.unit
- referenceRange.low.system
- referenceRange.low.code
- referenceRange.high.id
- referenceRange.high.unit
- referenceRange.high.system
- referenceRange.high.code
- referenceRange.type
- referenceRange.appliesTo
- referenceRange.age
- related.id
- related.modifierExtension
- related.target.id
- related.target.identifier
- related.target.display
- component.id
- component.modifierExtension
- component.code.id
- component.dataAbsentReason
- component.interpretation.id
- component.interpretation.coding.id
- component.interpretation.coding.version
- component.interpretation.coding.userSelected
- component.interpretation.text

# JSON FHIR > XML HL7

FHIR Observations can be mapped to the following HL7 components. The mapping for each GP Connect 
observation type (in **bold**) is described in the sections below:

- ObservationStatement (**[Uncategorised Data](#uncategorised-data-json-fhir--xml-hl7)** and **[Test Result](#test-result-json-fhir--xml-hl7)**)
- CompoundStatement (**[Blood Pressure](#blood-pressure-json-fhir--xml-hl7)**, **[Test Group Header](#test-group-header-json-fhir--xml-hl7)**)
- NarrativeStatement (**[Comment Note](#comment-note-json-fhir--xml-hl7)** / **[Filing Comment](#filing-comment-json-fhir--xml-hl7)**)

## Uncategorised Data (JSON FHIR > XML HL7)

Uncategorised data is mapped from an FHIR `Observation` to a HL7 `ObservationStatement` where it is not deemed to be a 
[Blood Pressure](#blood-pressure-json-fhir--xml-hl7) or [Comment Note](#comment-note-json-fhir--xml-hl7). 

| Mapped to (XML HL7 ObservationStatement)              | Mapped from (JSON FHIR / other source )                                                                                                         |
|-------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                                           | Unique ID generated by the adaptor                                                                                                              |
| code                                                  | `Observation.code.coding` as described in the FHIR > XML section of [Codeable Concept](../codeable%20concept/README.md)                         |
| statusCode                                            | fixed value = `"COMPLETE"`                                                                                                                      |
| effectiveTime                                         | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                           |
| availabilityTime                                      | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                     | 
| value                                                 | `Observation.valueQuantity` or `Observation.valueString`                                                                                        |
| pertinentInformation / sequenceNumber \[@value]       | fixed value = `"+1"`                                                                                                                            |
| pertinentInformation / pertinentAnnotation / text     | mapped as described in [Mapping for Uncategorised Data's Pertinent Information](#mapping-for-uncategorised-datas-pertinent-information) (below) |
| interpretationCode \[@code]                           | `Observation.interpretation.coding.code`                                                                                                        | 
| interpretationCode \[@codeSystem]                     | fixed value = `"2.16.840.1.113883.2.1.6.5"`                                                                                                     |
| interpretationCode \[@displayName]                    | description of `Observation.interpretation.coding.code` e.g. if code = `"HI"` then displayName = `"Above high reference limit"`                 |
| interpretationCode / originalText                     | `Observation.interpretation.coding.display`                                                                                                     |
| referenceRange / referenceInterpretationRange / text  | `Observation.referenceRange.text`                                                                                                               |
| referenceRange / referenceInterpretationRange / value | `Observation.referenceRange.value` where `Observation.refenceRange` and `Observation.valueQuantity` are populated                               |
| Participant / agentRef / id \[@root]                  | `Observation.performer`                                                                                                                         | 


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
   <value xsi:type="PQ" unit="1" value="12.000">
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
   <referenceRange typeCode="REFV">
      <referenceInterpretationRange classCode="OBS" moodCode="EVN.CRT">
         <value>
            <low value="0.000"/>
            <high value="0.300"/>
         </value>
      </referenceInterpretationRange>
   </referenceRange>
   <Participant contextControlCode="OP" typeCode="PRF">
      <agentRef classCode="AGNT">
         <id root="910543AF-6E56-47B9-970F-6724483D808C"/>
      </agentRef>
   </Participant>
</ObservationStatement>
```
</details>

#### Mapping for Uncategorised Data's Value

The `value` is mapped from either  `Observation.valueQuantity` or `Observation.valueString`

In the case of a `valueString` the resulting `observationStatement / value` will be produced with 
`<value xsi:type="ST">{{valueString}}</value>`

In the case of a `valueQuantity`, the resulting `observationStatement / value` will be produced with an `xsi:type` set 
to either a `PQ` (physical quantity) or an `IVL_PQ` (interval physical quantity), both of which are detailed below.

In the event that a `valueQuantity` contains an extension with a `valueQuantity.extension[].url` set to
"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ValueApproximation-1", then an 
`observationStatement.uncertaintyCode` will be produced with static values as below:
```xml
<uncertaintyCode code="U" codeSystem="2.16.840.1.113883.5.1053" displayName="Recorded as uncertain" />
```

###### Physical Quantity

When a `valueQuantity.comparator` is not provided then the result will be a value with `xsi:type='PQ"`.

When `valueQuantity.system` is provided with a value of `http://unitsofmeasure.org` then the result value will be
```xml
<value xsi:type="PQ" value="{valueQuantity.value}" unit="{valueQuantity.code}" />
```
When `valueQuantity.system` is provided with a value of `http://unitsofmeasure.org` but no `valueQuantity.code` is 
provided then a nested translation, with a nested `originalText` element will be produced
```xml
<value xsi:type="PQ" value="{valueQuantity.value}" unit="1">
                <translation value="{valueQuantity.value}">
                    <originalText>{valueQuantity.unit}</originalText>
                </translation>
</value>
```
If `valueQuantity.system` is other than `http://unitsofmeasure.org` then a nested `translation` is produced.
```xml
<translation value="{valueQuantity.value}" 
             code="{valueQuantity.code}" 
             codeSystem="{valueQuantity.system}" 
             displayName="{valueQuantity.unit}" />
```
If `valueQuantity.system` is not present then `translation / displayName` will not be produced.
In the event that `valueQuantity.system` is not present or is other than `http://unitsofmeasure.org`
and `valueSystem.code` is not present then an additional `value / translation / originalText` will be added with the
content set to `valueQuantity.unit` and `value \[@unit]` will be set to `1`.

In all other cases where `valueQuantity.code` and `valueQuantity.unit` are not present then the following will be
produced:
```xml
<value xsi:type="PQ" value="{valueQuantity.value}" unit="1" />
```

If `valueQuantity.value` is not provided an `observationStatement / value` will not be produced.

###### Interval Physical Quantity

When a `valueQuantity.comparator` is not provided then the result value will be an interval PQ with `xsi:type='IVL_PQ"`.

The same rules apply to intervals as to standard physical quantities, but with an additional element named either `high`
or `low` wrapped in the original value element with a property of `inclusive`.

The additional element will be `high` when `valueQuantity.comparator` is either `<` or `<=` and conversely will be `low`
when `valueQuantity.comparitor` is either `>` or `>=`.

The `inclusive` property or of either a `value / high` or `value / low` is set with `true` if `valueQuantity.compartor`
is either `<=` or `>=` indicating that the value should be included in comparisons.

An example is provided below:
```xml
<value xsi:type="IVL_PQ">
    <high value="37.1" unit="1" inclusive="true">
        <translation value="37.1">
            <originalText>C</originalText>
        </translation>
    </high>
</value>
```


#### Mapping for Uncategorised Data's Pertinent Information

The `pertinentInformation / pertinentAnnotation / text`  element is used to map fields that have no direct mapping in 
the HL7 or add contextual information. Where each field is mapped to a comment it will be prepended as follows:

- `Observation.referenceRange[0].high.unit` or `low.unit` (prepended with `Range Units: `)
- `Observation.referenceRange[0]` where `Observation.valueQuantitiy` is not populated (prepended with `Range: `)
- `Observation.interpretation` where the interpretation code is unrecognised (prepended with `"Interpretation: "`)
- `Observation.comment`
- `Observation.bodysite` (prepended with `BodySite: `)

Where a [Condition](../conditions/README.md) references the mapped `Observation` in the actual problem extension 
(`https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-ActualProblem-1`), the following comments can 
be mapped (prepended with `Problem Info: `): 

- `Condition.code` (prepended with `Transformed Observation problem header Originally coded: `) 
- `Condition.notes` (prepended with `Problem Notes: ` and seperated with semicolons)
- `Condition.extension[index].value` where `extension[index].url` equals 
`"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-RelatedProblemHeader-1"` 
(prepended with `Related Problem: `)

For `Observation.value[x]`, where it is not `valueQuantity` or `valueString` one of the following will be mapped

- `Observation.valueCodeableConcept.coding.code` and `.display` (prepended with `Code Value: `)
- `Observation.valueBoolean` (prepended with `Boolean Value: `)
- `Observation.valueRange` (prepended with `Range Value: `)
- `Observation.valueRatio.numerator` and `.denominator` (prepended with `Ratio Value: `)
- `Observation.valueTimeType` (prepended with `Time Value: `)
- `Observation.valueDateTimeType` (prepended with `DateTime Value: `)
- `Observation.valuePeriod.start` and `.end` (formatted as `Period Value: Start {{start}} End {{end}}`)

The following values will be grouped and appended with `"Component(s): ` and enclosed in square brackets  

- `Observation.component.valueQuantity` (prepended with `Quantity Value: `)
- `Observation.component.valueString` (prepended with `String Value: `)
- `Observation.component.valueCodeableConcept.coding.code` and `.display` (prepended with `Code Value: `)
- `Observation.component.valueBoolean` (prepended with `Boolean Value: `)
- `Observation.component.valueRange` (prepended with `Range Value: `)
- `Observation.component.valueRatio.numerator` and `.denominator` (prepended with `Ratio Value: `)
- `Observation.component.valueTimeType` (prepended with `Time Value: `)
- `Observation.component.valueDateTimeType` (prepended with `DateTime Value: `)
- `Observation.component.valuePeriod.start` and `.end` (formatted as `Period Value: Start {{start}} End {{end}}`)
- `Observation.component.code` (prepended with `code: `)
- `Observation.component.interpretation.coding[index].code` (prepended with `Interpretation Code: `), where 
`coding[index].userSelected` is preferred otherwise the first repetition is used   
- `Observation.component.interpretation.text` (prepended with `Interpretation Text: `)
- `Observation.component.referenceRange` (prepended with `Range: `)

## Blood Pressure (JSON FHIR > XML HL7)

An Observation is identified as a Blood Pressure if `Observation.code`, `Observation.component[0].code` and 
`Observation.component[1].code` contain a valid blood pressure triple, as outlined in the 
[GP Connect Documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html#representing-blood-pressure-readings-from-gp-systems).
Blood Pressure triples are mapped to a `CompoundStatement` with `ObservationStatement` components for the systolic 
and diastolic readings and a optional `NarrativeStatement` for additional information.

If Blood pressure observations do not form a valid triple they will be mapped as [Unategorised Observations](#uncategorised-data-json-fhir--xml-hl7)

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

The blood pressure will have two `ObservationStatement` components for the systolic and diastolic readings or to complete 
the GP Connect triple, as outlined in the [GP Connect Documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html#representing-blood-pressure-readings-from-gp-systems).  

| Mapped to (XML HL7 CompoundStatement / component / ObservationStatement) | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                |
|--------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                                       | Unique ID generated by the adaptor                                                                                                                                                                                                                     |
| code \[@code]                                                            | `Observation.component[index].code.coding[0].code` where the code is a valid systolic or diastolic blood pressure SNOMED code                                                                                                                          |
| code \[@displayName]                                                     | `Observation.component[index].code.coding[0].display`                                                                                                                                                                                                  |
| code \[@codeSystem]                                                      | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                                                                                                                       |
| code / originalText                                                      | `component[index].code.coding.text` <br/> or else `component[index].code.coding.display` <br/> or else `component[index].code.coding.extension[index2].value` where `component[index].code.coding.extension[index2].url` equals `"descriptionDisplay"` |
| statusCode                                                               | fixed value = `"COMPLETE"`                                                                                                                                                                                                                             |
| effectiveTime                                                            | same as the Parent Compound Statement's `effectiveTime`                                                                                                                                                                                                |
| availabilityTime \[@value]                                               | same as the Parent Compound Statement's `availabilityTime`                                                                                                                                                                                             |
| value                                                                    | `Observation.component[index].valueQuantity`                                                                                                                                                                                                           |
| referenceRange / referenceInterpretationRange / text                     | `Observation.component[index].referenceRange[0].text`                                                                                                                                                                                                  |
| referenceRange / referenceInterpretationRang / value                     | `Observation.component[index].referenceRange[0].value`                                                                                                                                                                                                 |

### Narrative Statement components

A blood pressure may have a `NarrativeStatement` component if the parent observation has a comment or if `Observation.component.dataAbsentReason`, 
`Observation.bodySite` or `Observation.component.interpretation` are populated.   

| Mapped to (XML HL7 CompoundStatement / component / NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                                                              |
|------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                                     | Unique ID generated by the adaptor                                                                                                                   |
| text                                                                   | Concatenated from `Observation.comment`, `Observation.bodySite`, `Observation.component.dataAbsentReason` and `Observation.component.interpretation` |
| statusCode                                                             | fixed value = `"COMPLETE"`                                                                                                                           |
| availabilityTime \[@value]                                             | same as the Parent Compound Statement's `availabilityTime`                                                                                           |

<details>
   <summary>Example XML</summary>

```XML

<CompoundStatement classCode="BATTERY" moodCode="EVN">
   <id root="CC276BF8-4B61-4388-9C73-38A22A52852F"/>
   <code code="163020007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="O/E - blood pressure reading">
      <originalText>O/E - blood pressure reading</originalText>
   </code>
   <statusCode code="COMPLETE"/>
   <effectiveTime>
      <center value="20100206124100"/>
   </effectiveTime>
   <availabilityTime value="20100206124100"/>
   <component contextConductionInd="true" typeCode="COMP">
      <ObservationStatement classCode="OBS" moodCode="EVN">
         <id root="2C0CE794-06A6-47FA-85A1-CC8DA5D07E32"/>
         <code code="72313002" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Systolic arterial pressure">
            <originalText>Systolic blood pressure</originalText>
         </code>
         <statusCode code="COMPLETE"/>
         <effectiveTime>
            <center value="20100206124100"/>
         </effectiveTime>
         <availabilityTime value="20100206124100"/>
         <value unit="1" value="170.000" xsi:type="PQ">
            <translation value="170.000">
               <originalText>mm[Hg]</originalText>
            </translation>
         </value>
      </ObservationStatement>
   </component>
   <component contextConductionInd="true" typeCode="COMP">
      <ObservationStatement classCode="OBS" moodCode="EVN">
         <id root="CF04EB19-F04C-42DD-A450-67B30F7E8A16"/>
         <code code="1091811000000102" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Diastolic arterial pressure">
            <originalText>Diastolic blood pressure</originalText>
         </code>
         <statusCode code="COMPLETE"/>
         <effectiveTime>
            <center value="20100206124100"/>
         </effectiveTime>
         <availabilityTime value="20100206124100"/>
         <value unit="1" value="130.000" xsi:type="PQ">
            <translation value="130.000">
               <originalText>mm[Hg]</originalText>
            </translation>
         </value>
      </ObservationStatement>
   </component>
   <component contextConductionInd="true" typeCode="COMP">
      <NarrativeStatement classCode="OBS" moodCode="EVN">
         <id root="36A43E4C-9EB5-414C-B549-30B1C728E330"/>
         <text>Measurement Site: test body site</text>
         <statusCode code="COMPLETE"/>
         <availabilityTime value="20100206124100"/>
      </NarrativeStatement>
   </component>
   <Participant contextControlCode="OP" typeCode="PRF">
      <agentRef classCode="AGNT">
         <id root="D2CABB3B-43CB-4461-AC2C-1844AFB8354A"/>
      </agentRef>
   </Participant>
</CompoundStatement>
```
</details>

## Comment Note (JSON FHIR > XML HL7)

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
which forms part of the [Diagnostic Report](../diagnostic%20reports/README.md) structure.

### Test Group Header (JSON FHIR > XML HL7)

A Test Group Header is mapped to a `CompoundStatement` with a `classcode` of `"BATTERY"`, [Test Results](#test-result-json-fhir--xml-hl7) can be 
nested as components of the header.  

| Mapped to (XML HL7 CompoundStatement)     | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                   |
|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                        | Unique ID generated by the adaptor                                                                                                                                                                                                                        |
| code                                      | `Observation.code.coding` as described in [Codeable Concept](../codeable%20concept/README.md)                                                                                                                                                             |
| statusCode                                | fixed value = `"COMPLETE"`                                                                                                                                                                                                                                |
| effectiveTime                             | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                                                                                                                                     |
| availabilityTime                          | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                                                                                                                               |
| component\[index] / ObservationStatement  | Mapped [Test Results](#test-result-json-fhir--xml-hl7)                                                                                                                                                                                                    |  
| component\[index] / CompoundStatement     | Mapped [Test Results](#test-result-json-fhir--xml-hl7)                                                                                                                                                                                                    |
| component\[index] / NarrativeStatement    | Mapped from `Observation` as outlined in [Narrative statement mapping for Test Group Header and Test Result](#narrative-statement-mapping-for-test-group-header-and-test-result) <sup>9</sup> and [Filing comments](#filing-comment-json-fhir--xml-hl7)   |


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

9. `NarrativeStatement` components are created for each of the fields stated in [Observation Fields mapped to Narrative Statement](#observation-fields-mapped-to-narrative-statement) above. If none of
   these fields are present, and no [Filing Comments](#filing-comment-json-fhir--xml-hl7) are present the `NarrativeStatement` will be omitted.

### Narrative statement mapping for Test Group Header and Test Result

Where stated in the [Test Group Header](#test-group-header-json-fhir--xml-hl7) and [Test Result](#test-result-json-fhir--xml-hl7) mapping, a collection of `NarrativeStatement` components may be
used to preserve data. Where this takes place, they will be mapped from the appropriate `Observation` in the following way:

| Mapped to (XML HL7 NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                        |
|----------------------------------------|----------------------------------------------------------------------------------------------------------------|
| id                                     | Unique ID generated by the adaptor                                                                             |
| text \[@mediaType]                     | fixed value = `"text/x-h7uk-pmip"`                                                                             |
| text                                   | An EDIFACT comment type of `AGGREGATE COMMENT SET` where the content is mapped from one of fields stated below |
| statusCode                             | fixed value = `"COMPLETE"`                                                                                     |
| availabilityTime                       | Same as parent `CompoundStatement`                                                                             |

Note - this is in addition to filing Comments which also mapped to narrative statements and described below

#### Observation fields mapped to Narrative Statement

Where multiple fields are stated they are concatenated into a single EDIFACT comment

- `Observation.dataAbsentReason` (prefixed with *"Data Absent: "*)
- `Observation.interpretation` (prefixed with *"Interpretation: "*), `comment`, `valueString` (prefixed with *"Value: "*),
  `Observation.referenceRange[0].text` (prefixed with *"Range Text: "*) and `referenceRange[0].high.unit` or `referenceRange[0].low.unit`
  (prefixed with *"Range Units: "*)
- `Observation.bodySite` (prefixed with *"Site: "*)
- `Observation.method` (prefixed with *"Method: "*)

### Test Result (JSON FHIR > XML HL7)

A Test Result `Observation` is mapped to an `ObservationStatement`. 

Where a `NarrativeStatement` is used to preserve data from an `Observation`, the `ObservationStatement` and 
`NarrativeStatement` will be wrapped in a `CompoundStatement` with a class code of `CLUSTER`. Mapping for the narrative statement
is described in [Narrative statement mapping for Test Group Header and Test Result](#narrative-statement-mapping-for-test-group-header-and-test-result) 

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
| Mapped to (XML HL7 CompoundStatement) | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                   |
|---------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                           | Unique ID generated by the adaptor                                                                                                                                                                                                                        |
| code                                  | `Observation.code.coding` as described in [Codeable Concept](../codeable%20concept/README.md)                                                                                                                                                             |
| statusCode                            | fixed value = `"COMPLETE"`                                                                                                                                                                                                                                |
| effectiveTime                         | `Observation.effectiveDateTime` or else `Observation.effectivePeriod`                                                                                                                                                                                     |
| availabilityTime                      | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                                                                                                                                                               |
| component / ObservationStatement      | The Observation Statement as mapped above                                                                                                                                                                                                                 |
| component / NarrativeStatement        | Mapped from `Observation` as outlined in [Narrative statement mapping for Test Group Header and Test Result](#narrative-statement-mapping-for-test-group-header-and-test-result) <sup>9</sup> and [Filing Comments](#filing-comment-json-fhir--xml-hl7)   |

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

### Filing Comment (JSON FHIR > XML HL7)

Where Test Result Headers or Test Results have Observations coded as `Comment Note`, the referenced `Observation` is a Filing Comment. Filing Comments are mapped
to `NarrativeStatements` and inserted as components of the Test Group Header / Test Result `CompoundStatement`.

| Mapped to (XML HL7 NarrativeStatement) | Mapped from (JSON FHIR / other source )                                                                          |
|----------------------------------------|------------------------------------------------------------------------------------------------------------------|
| id \[@root]                            | Unique ID generated by the adaptor                                                                               |
| text \[@mediaType]                     | fixed value = `"text/x-h7uk-pmip"`                                                                               | 
| text                                   | EDIFACT comment type of USER COMMENT where the comment body is mapped from `Observation.comment` <sup>10</sup>    |
| availabilityTime                       | `Observation.effectiveDateTime` or else `Observation.effectivePeriod.start`                                      |

<details>
   <summary>Example XML</summary>

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

10. If there is more than one filing comment for a given Observation, the comments are seperated by newlines and appended to the EDIFACT comment body.

## Further documentation

### GP Connect Uncategorised data

- [GP Connect uncategorised data guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_uncategorisedData_guidance.html)
- [GP Connect Observation - uncategorised data](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_uncategorisedData.html)
- [GP Connect Observation - blood pressure](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_bloodPressure.html)

### GP Connect Investigations

- [GP Connect Investigations guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_pathology_guidance.html)
- [GP Connect Observation - test group header](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_testGroup.html)
- [GP Connect Observation - test result](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_testResult.html)
- [GP Connect Observation - filing comment](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_observation_filingComments.html)

### HL7v3 MIM

- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
