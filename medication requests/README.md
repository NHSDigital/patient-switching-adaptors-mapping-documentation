# Medication Request Mapping

## XML HL7 > JSON FHIR

One or more `MedicationRequest`(s) are mapped from a `MedicationStatement`</br>

When `ehrSupplyAuthorise` is referenced, it refers to `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise`.

When `ehrSupplyDiscontinue` is referenced, it refers to the first matching `ehrSupplyDiscontinue` where `ehrSupplyDiscontinue / reversalOf / priorMedicationRef / id [@root]` is equal to `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise / id [@root]` in the `EhrExtract`.

When `ehrSupplyPrescribe` is referenced, it refers to `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyPrescribe`.

When `EhrExtract` is referenced, it refers to the parent `EhrExtract` in the XML.</br>

When `ehrComposition` is referenced, it refers to the parent `ehrComposition` in the XML.</br></br>

A new `MedicationRequest` is created for each `ehrSupplyPrescribe` and `ehrSupplyAuthorise` in the `MedicationStatement` and the fields mapped are added according to if medication request is a [PLAN](#plan)(`ehrSupplyAuthorise`) or an [ORDER](#order)(`ehrSupplyPrescribe`)

For each of the `MedicationRequest`s mapped from a single `MedicationStatement` the following fields are added:

| Mapped to (JSON FHIR MedicationRequest field) | Mapped from (XML HL7 / other source)                                                                                                                                                                                                                                                                                                                 |
|-----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| meta.profile                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-MedicationRequest-1"`                                                                                                                                                                                                                                                   |                                                                                       |
| requester.agent.reference                     | contains a reference to a [Practitioner](../practioners/README.md) where the reference id is the first `MedicationStatement / participant / agentRef / id \[@root]` where `MedicationStatement / participant / nullFlavor` does not exist and `Medication Statement / participant / typeCode` is either `"PPRF"` or `"PRF"`<sup>1</sup> <sup>2</sup> |
| recorder.reference                            | contains a reference to a [Practitioner](../practioners/README.md) where the reference id is the first `MedicationStatement / participant / agentRef / id \[@root]` where `MedicationStatement / participant / nullFlavor` does not exist and `Medication Statement / participant / typeCode` is either `"PPRF"` or `"PRF"`<sup>1</sup> <sup>2</sup> |
| subject                                       | contains a reference to a mapped [Patient](../patient/README.md) where the reference id is `Patient.id ` (previously generated when mapping `Patient` from `EhrExtract / recordTarget / patient`)                                                                                                                                                    |
| context                                       | contains a reference to a mapped [Encounter](../encounters/README.md) where the reference id is `Encounter.id` (previously set when mapping `encounter`s from `ehrComposition`s)                                                                                                                                                                     |
| authoredOn                                    | `MedicationStatement / availabilityTime` or `ehrComposition / availabilityTime [@value]` or `EhrExtract / author/ time [@value]`<sup>3</sup>                                                                                                                                                                                                         |

1. If this value is not present then the reference id is the first `ehrComposition / participant2 / agentRef / id [@root]` where `ehrComposition / participant2 / nullFlavor` does not exist
2. If neither of these conditions match then this field is omitted
3. If neither of the values are present then this field is omitted

The following MedicationRequest fields are not currently populated by the adaptor:
- definition
- category
- priority
- supportingInformation
- reasonCode
- reasonReference
- substitution
- detectedIssue
- eventHistory

### Order

When `ehrSupplyPrescribe` is present this is considered an `ORDER` and the following fields are mapped:

| Mapped to (JSON FHIR Medication Request field) | Mapped from (XML HL7 / other source)                                                                                                                                                             |
|------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                             | `ehrSupplyPrescribe / id [@root]`                                                                                                                                                                |
| identifier.system                              | `https:\\PSSAdaptor\{{practiceCode}}` where `{{practiceCode}}` is losing practice ODS Code.                                                                                                      |
| identifier.value                               | `ehrSupplyPrescribe / id [@root]`                                                                                                                                                                |
| status                                         | fixed value = `"COMPLETED"`                                                                                                                                                                      |
| intent                                         | fixed value = `"ORDER"`                                                                                                                                                                          |
| dosageInstruction.text                         | `MedicationStatement / pertinentInformation / pertinentMedicationDosage / text`<sup>4</sup>                                                                                                      |
| dispenseRequest.quantity.value                 | `ehrSupplyPrescribe / quantity [@value]` when exists<sup>5<sup>                                                                                                                                  |
| dispenseRequest.quantity.unit                  | `ehrSupplyPrescribe / quantity / translation / originalText` when exists<sup>5</sup>                                                                                                             |
| dispenseRequest.validityPeriod.start           | `ehrSupplyPrescribe / availabilityTime [@value]` when exists otherwise omitted                                                                                                                   |
| note.text                                      | a list of all values of `ehrSupplyPrescribe / pertinentInformation / pertinentSupplyAnnotation / text` suffixed by a new line<sup>6</sup>                                                        |
| medication.reference                           | contains a reference to a [Medication](../medications/README.md) using an id of `MedicationStatement / consumable / manufacturedMaterial [@code]` when `medicationStatement / consumable` exists |

4. If there is no pertinentInformation which has a pertinentInformationDosage / text, then a default value of "No Information available" is used
5. If `ehrSupplyPrescribe / quantity` does not exist then `dispenseRequest.quantity` is omitted
6. If `ehrSupplyPrescribe / code \[@code] \[@displayName]` both exist and `\[@displayName]` is not equal to `"NHS prescription"` then an additional `note.text` is added with the value of `\[@displayName]` prefixed by `"Prescription type: "`

The following fields are also added if `ehrSupplyPrescribe / inFulfillmentOf / priorMedicationRef / id \[@root]` exists

| Mapped to (JSON FHIR Medication Request field) | Mapped from (XML HL7 / other source)                                                                                         |
|------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------|
| extension.url                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescriptionType-1"`                  |
| extension.valueCodeableConcept.coding.system   | fixed value = `"https://fhir.nhs.uk/STU3/CodeSystem/CareConnect-PrescriptionType-1"`                                         |
| extension.valueCodeableConcept.coding.code     | `"acute"`<sup>7</sup> else `"repeat"`                                                                                        |
| extension.valueCodeableConcept.coding.display  | `"Acute"`<sup>7</sup> else `"Repeat"`                                                                                        |
| basedOn.reference                              | a reference to a `MedicationRequest` with an id of `ehrSupplyPrescribe / inFulfillmentOf / priorMedicationRef / id \[@root]` |

7. When there is an `ehrSupplyAuthorise / id \[@root]` in the `EhrExtract` which equals `ehrSupplyPrescribe / inFulfillmentOf / priorMedicationRef / id \[@root]` and when that `EhrSupplyAuthorise \ repeatNumber /[@value]` = `0`

<details>
    <summary>Example JSON for Order</summary>

```
{
    "resource": {
        "resourceType": "MedicationRequest",
        "id": "ADA08729-435C-4F4A-B177-DA4DE0253BDC",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-MedicationRequest-1"
            ]
        },
        "extension": [
            {
                "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescriptionType-1",
                "valueCodeableConcept": {
                    "coding": [
                        {
                            "system": "https://fhir.nhs.uk/STU3/CodeSystem/CareConnect-PrescriptionType-1",
                            "code": "repeat",
                            "display": "Repeat"
                        }
                    ]
                }
            }
        ],
        "identifier": [
            {
                "system": "https://PSSAdaptor/D5445",
                "value": "ADA08729-435C-4F4A-B177-DA4DE0253BDC"
            }
        ],
        "basedOn": [
            {
                "reference": "MedicationRequest/EC938F8C-D27A-4E82-A1EA-55FA77CAA01E"
            }
        ],
        "status": "completed",
        "intent": "order",
        "medicationReference": {
            "reference": "Medication/8beaa363-8e19-469b-b902-f07178e9f4c0"
        },
        "subject": {
            "reference": "Patient/7f3c8002-c7b5-4e1e-ac87-292c9a382a4e"
        },
        "authoredOn": "2016-09-01",
        "requester": {
            "agent": {
                "reference": "Practitioner/B06AD8D8-D6FB-8F07-39C7-6BF81CA317C4"
            }
        },
        "recorder": {
            "reference": "Practitioner/B06AD8D8-D6FB-8F07-39C7-6BF81CA317C4"
        },
        "dosageInstruction": [
            {
                "text": "Two To Be Taken Immediately Then One To Be Taken After Each Loose Motion"
            }
        ],
        "dispenseRequest": {
            "validityPeriod": {
                "start": "2016-09-01"
            },
            "quantity": {
                "value": 30.0,
                "unit": "capsule"
            }
        }
    }
}
```
</details>


### Plan

When `ehrSupplyAuthorise` is present, this is considered a `PLAN` and the following fields are added:

| Mapped to (JSON FHIR Medication Request field)    | Mapped from (XML HL7 / other source)                                                                                                                                                                         |
|---------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                | `ehrSupplyAuthorise / id [@root]`                                                                                                                                                                            |
| identifier.system                                 | `https:\\PSSAdaptor\{{practiceCode}}` where `{{practiceCode}}` is losing practice ODS Code.                                                                                                                  |
| identifier.value                                  | `ehrSupplyAuthorise / id [@root]`                                                                                                                                                                            |
| intent                                            | fixed value = `"PLAN"`                                                                                                                                                                                       |
| dosageInstruction.text                            | `MedicationStatement / pertinentInformation / pertinentMedicationDosage / text`<sup>8</sup>                                                                                                                  |
| dispenseRequest.quantity.value                    | `ehrSupplyAuthorise / quantity [@value]` when exists<sup>9</sup>                                                                                                                                             |
| dispenseRequest.quantity.unit                     | `ehrSupplyAuthorise / quantity / translation / originalText` when exists<sup>9</sup>                                                                                                                         |
| dispenseRequest.validityPeriod.start              | `ehrSupplyAuthorise / effectiveTime / center [@value]`, `ehrSupplyAuthorise / effectiveTime / low [@value]` or `ehrSupplyAuthorise / availabilityTime [@value]`<sup>3</sup>                                  |
| dispenseRequest.validityPeriod.end                | `ehrSupplyAuthorise / effectiveTime / high [@value]`, `MedicationStatement.effectiveTime / high [@value]`                                                                                                    |
| extension\[repeatInformationExtensions].url       | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationRepeatInformation-1"`                                                                                       |
| extension\[repeatInformationExtensions].Extension | a list of extensions related to `Repeat Information`. Details below in [Repeat Information Extensions](#repeat-information-extensions)                                                                       |
| extension\[statusChangeExtensions].url            | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationStatusReason-1"`                                                                                            |
| extension\[statusChangeExtensions].Extension      | a list of extensions related to `Status Change`. Details below in [Status Change Extensions](#status-change-extensions)                                                                                      |
| status                                            | `"COMPLETED"`<sup>10</sup>, `"STOPPED"`<sup>11</sup> or `"ACTIVE"`<sup>12</sup>                                                                                                                              |
| note[].text                                       | a list of all values of `ehrSupplyAuthorise / pertinentInformation / pertinentSupplyAnnotation / text` suffixed by a new line<sup>13</sup>                                                                   |
| priorPrescription.reference                       | contains a reference to a `MedicationRequest` with an id of `ehrSupplyAuthorise / predecessor[0] / priorMedicationRef / id [@root`<sup>14</sup>                                                              |
| medication.reference                              | contains a reference to a mapped [Medication](../medications/README.md) using an id of `MedicationStatement / consumable / manufacturedMaterial \[@code]` when `medicationStatement / consumable` is present |

8. If there is no `pertinentInformation` which has a `pertinentInformationDosage / text`, then a default value of `"No Information available"` is used
9. If `ehrSupplyAuthorise / quantity` is not present then `dispenseRequest.quantity` is omitted
10. When `ehrSupplyDiscontinue` is present and no [Status Change Extensions](#status-change-extensions) are present, or when `ehrSupplyDiscontinue` is not present and `ehrSupplyAuthorise / statusCode / code` equals `"COMPLETE"`
11. When `ehrSupplyDiscontinue` is present and any [Status Change Extensions](#status-change-extensions) are present
12. When `ehrSupplyDiscontinue` is not present and `ehrSupplyAuthorise / statusCode / code` is not equal to `"COMPLETE"`
13. If `ehrSupplyAuthorise / code [@displayName]` both exist and `[@displayName]` is not equal to `"NHS prescription"` then an additional `note.text` is added with the value of `[@displayName]` prefixed by `"Prescription type: "`
14. If `ehrSupplyAuthorise / predecessor[0] / priorMedicationRef / id [@root]` is not present then this is omitted 


#### Repeat Information Extensions

This is a list of extensions related to repeat information which are included when mapping an `PLAN`</br>
For each of the `extension`, if the value required to set `extension.value` is not present then that extension is not added

| Mapped to (JSON FHIR MedicationRequest.extension.Extension field) | Mapped from (XML HL7 / other source)                                                                                                                                                                                                                     |
|-------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| extension\[0].url                                                 | fixed value = `"numberOfRepeatPrescriptionsAllowed"`<sup>15</sup>                                                                                                                                                                                        |
| extension\[0].valueUnsignedInt                                    | `ehrSupplyAuthorise / repeatNumber [@value]`<sup>15</sup>                                                                                                                                                                                                |
| extension\[1].url                                                 | fixed value = `"numberOfRepeatPrescriptionsIssued"`<sup>16</sup>                                                                                                                                                                                         |
| extension\[1].valueUnsignedInt                                    | the count of the number of `MedicationStatement`s in the `EhrExtract` where `MedicationStatement / ehrSupplyPrescribe / inFulfillmentOf / priorMedicationRef / id [@root]` exists and `id [@root]` equals `ehrSupplyAuthorise / id [@root]`<sup>16</sup> |
| extension\[2].url                                                 | fixed value = `"authorisationExpiryDate"`                                                                                                                                                                                                                |
| extension\[2].valueDateTime                                       | `ehrSupplyAuthorise / effectiveTime / high [@value]`                                                                                                                                                                                                     |

15. When `ehrSupplyAuthorise / repeatNumber \[@value]` is present and `[@value]` is not equal to `0`
16. When `ehrSupplyAuthorise / repeatNumber \[@value]` is present and `[@value]` is not equal to `0` or when `ehrSupplyAuthorise / repeatNumber` is not present

#### Status Change Extensions

This is a list of extensions related to status changes which are included when mapping a `PLAN`</br>
If `EhrSupplyDiscontinue` exists and `EhrSupplyDiscontinue / availabilityTime [@value]` not present then these extensions are not added

| Mapped to (JSON FHIR MedicationRequest.extension.Extension field) | Mapped from (XML HL7 / other source)                                                                                                                                                                                    |
|-------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| extension\[0].url                                                 | fixed value = `"statusChangeDate"`                                                                                                                                                                                      |
| extension\[0].valueDateTime                                       | `EhrSupplyDiscontinue / availabilityTime [@value]`                                                                                                                                                                      |
| extension\[1].url                                                 | fixed value = `"statusReason"`                                                                                                                                                                                          |
| extension\[1].valueCodeableConcept.text                           | `"({{text}})"` where text = `ehrSupplyDiscontinue / code / originalText` when present, and all values `ehrSupplyDiscontinue / pertinentInformation / pertinentSupplyAnnotation / text` separated by `", "`<sup>17</sup> |

17. When there are no `ehrSupplyDiscontinue / pertinentInformation / pertinentSupplyAnnotation / text` present then a value of `"No information available"` is added instead 

<details>
    <summary>Example JSON for Plan</summary>

```
{
    "resource": {
        "resourceType": "MedicationRequest",
        "id": "70FA5735-11CB-4E5C-825A-E5688FDC888C",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-MedicationRequest-1"
            ]
        },
        "extension": [
            {
                "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationRepeatInformation-1",
                "extension": [
                    {
                        "url": "authorisationExpiryDate",
                        "valueDateTime": "2010-01-14"
                    }
                ]
            },
            {
                "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescriptionType-1",
                "valueCodeableConcept": {
                    "coding": [
                        {
                            "system": "https://fhir.nhs.uk/STU3/CodeSystem/CareConnect-PrescriptionType-1",
                            "code": "acute",
                            "display": "Acute"
                        }
                    ]
                }
            }
        ],
        "identifier": [
            {
                "system": "https://PSSAdaptor/D5445",
                "value": "70FA5735-11CB-4E5C-825A-E5688FDC888C"
            }
        ],
        "status": "completed",
        "intent": "plan",
        "medicationReference": {
            "reference": "Medication/9314aec6-5cad-47fe-b782-6e013b638a76"
        },
        "subject": {
            "reference": "Patient/e2cf4cc1-08da-4971-845f-35b8447cabdd"
        },
        "authoredOn": "2010-01-14",
        "requester": {
            "agent": {
                "reference": "Practitioner/5F8BBC0E-0FB7-4828-84AA-817F5243A12E"
            }
        },
        "recorder": {
            "reference": "Practitioner/5F8BBC0E-0FB7-4828-84AA-817F5243A12E"
        },
        "note": [
            {
                "text": "Expected Supply Duration: 28 day"
            }
        ],
        "dosageInstruction": [
            {
                "text": "One To Be Taken Each Day"
            }
        ],
        "dispenseRequest": {
            "validityPeriod": {
                "start": "2010-01-14",
                "end": "2010-01-14"
            },
            "quantity": {
                "value": 28.0,
                "unit": "tablet"
            }
        }
    }
}
```
</details>

## JSON FHIR > XML HL7

For details of mapping for both `PLAN` and `ORDER`, see the JSON FHIR > XML HL7 mapping for [MedicationStatement](../medication%20statements/README.md) 

## Further documentation

- [GP Connect MedicationRequest](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_medicationrequest.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
