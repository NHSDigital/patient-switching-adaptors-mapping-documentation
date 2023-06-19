# Medication Statement Mapping

## XML HL7 > JSON FHIR

A FHIR `MedicationStatement` is mapped from a XML HL7 `MedicationStatement` containing an `EhrSupplyAuthorise` component.</br>
When `ehrSupplyAuthorise` is referenced, it refers `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise`</br>
When `ehrSupplyDiscontinue` is referenced, it refers to find the first matching `ehrSupplyDiscontinue` where `ehrSupplyDiscontinue / reversalOf / priorMedicationRef / id [@root]` matching `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise / id [@root]` in the 'ehrExtract' 
When `ehrSupplyPrescribe` is referenced `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyPrescribe`
When `EhrExtract` is referenced, it refers to the parent `EhrExtract` in the XML.</br>
When `PracticeCode` is referenced, it refers to the losing practice ODS code.

| Mapped to (JSON FHIR Medication Statement field) | Mapped from (XML HL7 / other source)                                                                                                                                       |
|--------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                               | `ehrSupplyAuthorise / id [@root]` prefixed with `-MS`                                                                                                                      |
| meta.profile\[0]                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-MedicationStatement-1"`                                                                       |
| identifier.system                                | `https:\\PSSAdaptor\{{practiceCode}}` where {{practiceCode}} is losing practice ODS Code.                                                                                  |
| identifier.value                                 | `ehrSupplyAuthorise / id [@root]` prefixed with `-MS`                                                                                                                      |
| taken                                            | fixed value = `"unk"` <sup>1</sup>                                                                                                                                         |
| basedOn.reference                                | a reference to a [MedicationRequest](../medication statements/README.md) using `ehrSupplyAuthorise / id [@root]` as the reference id                                       |
| extension[0].url                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescribingAgency-1"`                                                               |
| extension[0].valueCodeableConcept.coding.system  | fixed value = `"https://fhir.nhs.uk/STU3/CodeSystem/CareConnect-PrescribingAgency-1"`                                                                                      |
| extension[0].valueCodeableConcept.coding.code    | fixed value = `"prescribed-at-gp-practice"`                                                                                                                                |
| extension[0].valueCodeableConcept.coding.display | fixed value = `"Prescribed at GP practice"`                                                                                                                                |
| dosage[index].text                               | `MedicationStatement / pertinentInformation / pertinentMedicationDosage / text` <sup>2</sup>                                                                               |
| extension[1].url                                 | fixed value = `""https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationStatementLastIssueDate-1"`                                               |  
| extension[1].valueDateTime                       | `ehrSupplyPrescribe / availabilityTime [@value]` <sup>3</sup>                                                                                                              |
| medicationReference.reference                    | reference to a `Medication` with a system generated UUID for this medication <sup>4</sup><sup>5</sup>                                                                      |
| status                                           | `"STOPPED"`, `"COMPLETED"` or `"ACTIVE"` <sup>6</sup><sup>7</sup>                                                                                                          |
| effectivePeriod.end                              | `ehrSupplyDiscontinue / availabilityTime [@value]` <sup>8</sup>                                                                                                            |
| effectivePeriod.start                            | `ehrSupplyAuthorise / effectiveTime / center [@value]`, `ehrSupplyAuthorise / effectiveTime / low [@value]`, `ehrSupplyAuthorise / availabilityTime [@value]` <sup>9</sup> |
| dateAsserted                                     | `ehrComposition / author / time [@value]`, `ehrExtract / availabilityTime [@value]`                                                                                        |
| subject                                          | A reference to the [Patient](../patient/README.md) from `ehrExtract / recordTarget / patient`                                                                              |
| context                                          | A reference to the [Encounter](../encounters/README.md) which this Medication Statement refers to.                                                                         |

1. Taken is included as both a mapped field and field not in use.  The reason for this being that the item is mandatory in the base FHIR profile but GP systems do not record this detail. For this reason a default value of 'unk' is picked but should **not** be used</br>
2. If there is no `pertinentInformation` which has a `pertinentInformationDosage / text`, then a default value of `"No Information available"` is used
3. This uses the most recent `availabilityTime` of an `ehrSupplyPrescribe` within any medication statement in the `ehrExtract` where there is an `inFulfilmentOf` which has a `priorMedication / id [@root]` matching the id of the current `MedicationStatement`<br/> If these are no matches then this extension is not added
4. This uses the values of `[@code]`, `[@displayName]` and `originalText` from `MedicationStatement / consumable / manufacturedProduct / manufacturedMaterial / code[0]` to create a unique key with a UUID value.  If this key has been used before then a `Medication` reference with the previously created UUID is used, otherwise a new UUID is generated. 
5. If there are no `consumable` with a `manufacturedProduct` with a `manufacturedMaterial` then this value is not set
6. The `ehrExtract` is parsed . If this has `ehrSupplyDiscontinue / availabilityTime / value` then value is set to `"STOPPED"` or `"COMPLETED"`
7. If there is no matching `ehrSupplyDiscontinue` (see <sup>6</sup>) then if `ehrSupplyAuthorise / statusCode / code` = `"COMPLETE"` then `"COMPLETED"` is used, else `"ACTIVE"` is used
8. When `ehrSupplyDiscontinue / availabilityTime [@value]` does not exist and `status` = `"ACTIVE"` then this is set to the same value as `effectivePeriod.start`
9. When `ehrSupplyDiscontinue / availabilityTime [@value]` exists. If this exists but the referenced values do not then the availability time from either the `MedicationStatement`, `EhrExract` or `EhrComposition` is used. If none of these values exist then the value assigned to `effectivePeriod.end` is used instead

### Unmapped fields

The following Medication Statement fields are not currently populated by the adaptor:
- meta.versionId
- meta.lastUpdated
- extension(ChangeSummary)
- partOf
- category
- derivedFrom
- reasonNotTaken
- reasonCode
- reasonReference
- taken <sup>1</sup>


<details>
    <summary>Example JSON</summary>

```
{
  "resource": {
    "resourceType": "MedicationStatement",
    "id": "AEDDB53B-A30B-4EC9-A20F-A059C29A1C3E-MS",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-MedicationStatement-1"
      ]
    },
    "extension": [
      {
        "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescribingAgency-1",
        "valueCodeableConcept": {
          "coding": [
            {
              "system": "https://fhir.nhs.uk/STU3/CodeSystem/CareConnect-PrescribingAgency-1",
              "code": "prescribed-at-gp-practice",
              "display": "Prescribed at GP practice"
            }
          ]
        }
      }
    ],
    "identifier": [
      {
        "system": "https://PSSAdaptor/B83002",
        "value": "AEDDB53B-A30B-4EC9-A20F-A059C29A1C3E-MS"
      }
    ],
    "basedOn": [
      {
        "reference": "MedicationRequest/AEDDB53B-A30B-4EC9-A20F-A059C29A1C3E"
      }
    ],
    "status": "completed",
    "medicationReference": {
      "reference": "Medication/9fa0d37d-912f-44c2-855b-06d1e8623c97"
    },
    "effectivePeriod": {
      "start": "2010-02-10",
      "end": "2010-02-10"
    },
    "dateAsserted": "2010-02-10T08:31:19+00:00",
    "subject": {
      "reference": "Patient/c782c5ef-8235-4acf-945b-268a1bbe9d64"
    },
    "taken": "unk",
    "dosage": [
      {
        "text": "To Be Used As Directed"
      }
    ]
  }
}
```
</details>

## JSON FHIR > XML HL7

A XML HL7 `MedicationStatement` is mapped from a FHIR `medicationRequest`

| Mapped to (XML HL7)                                 | Mapped from (JSON FHIR / other source )                                           |
|-----------------------------------------------------|-----------------------------------------------------------------------------------|
| MedicationStatement / id \[@root]                   | CompoundStatement ID (UUID) generated by the Adaptor                              |
| MedicationStatement / statusCode                    | `medicationRequest.status` when `"active"` = `"ACTIVE"` otherwise is `"COMPLETE"` |
| MedicationStatement / effectiveTime / low [@value]  | `medicationRequest.dispenseRequest.validityPeriod.start`                          |
| MedicationStatement / effectiveTime / high [@value] | `medicationRequest.dispenseRequest.validityPeriod.end` <sup>1</sup>               |
| MedicationStatement / availabilityTime  [@value]    | `medicationRequest.dispenseRequest.validityPeriod.start`                          |
| MedicationStatement /                               | |


1. If `medicationRequest.dispenseRequest.validityPeriod.end` does not exist then this value is not populated 
2. 

<details><summary>Example XML</summary>

```
<EhrExtract xmlns="urn:hl7-org:v3" classCode="EXTRACT" moodCode="EVN">
    <availabilityTime value="20100115" />
    <component typeCode="COMP">
        <ehrFolder classCode="FOLDER" moodCode="EVN">
            <component typeCode="COMP">
                <ehrComposition classCode="COMPOSITION" moodCode="EVN">
                    <component typeCode="COMP">
                        <MedicationStatement xmlns="urn:hl7-org:v3" classCode="SBADM" moodCode="INT">
                            <id root="B4D70A6D-2EE4-41B6-B1FB-F9F0AD84C503" />
                            <statusCode code="ACTIVE" />
                            <effectiveTime>
                                <high value="20060426" />
                                <center nullFlavor="NI" />
                            </effectiveTime>
                            <availabilityTime value="20100115" />
                            <consumable typeCode="CSM">
                                <manufacturedProduct classCode="MANU">
                                    <manufacturedMaterial classCode="MMAT" determinerCode="KIND">
                                        <code code="RACA57NEMIS"
                                            codeSystem="2.16.840.1.113883.2.1.6.9"
                                            displayName="Ramipril 10mg capsules">
                                            <translation code="318906001"
                                                codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                                                displayName="Ramipril 10mg capsules" />
                                        </code>
                                        <quantity value="28" unit="1">
                                            <translation value="28">
                                                <originalText>capsule</originalText>
                                            </translation>
                                        </quantity>
                                    </manufacturedMaterial>
                                </manufacturedProduct>
                            </consumable>
                            <component typeCode="COMP">
                                <ehrSupplyAuthorise classCode="SPLY" moodCode="INT">
                                    <id root="TEST_ID" />
                                    <code code="9bG0.00" codeSystem="2.16.840.1.113883.2.1.6.2"
                                        displayName="NHS prescription" />
                                    <statusCode code="ACTIVE" />
                                    <effectiveTime>
                                        <high value="20060427" />
                                        <center nullFlavor="NI" />
                                    </effectiveTime>
                                    <availabilityTime value="20100114" />
                                    <repeatNumber value="6" />
                                    <quantity value="28" unit="1">
                                        <translation value="28">
                                            <originalText>capsule</originalText>
                                        </translation>
                                    </quantity>
                                    <predecessor typeCode="SUCC">
                                        <priorMedicationRef classCode="SBADM" moodCode="INT">
                                            <id root="TEST_ID" />
                                        </priorMedicationRef>
                                    </predecessor>
                                    <pertinentInformation typeCode="PERT">
                                        <pertinentSupplyAnnotation classCode="OBS" moodCode="EVN">
                                            <text>Pharmacy Text: Repeat Dispensing Pharmacy Note. 1</text>
                                        </pertinentSupplyAnnotation>
                                    </pertinentInformation>
                                    <pertinentInformation typeCode="PERT">
                                        <pertinentSupplyAnnotation classCode="OBS" moodCode="EVN">
                                            <text>Pharmacy Text: Repeat Dispensing Pharmacy Note. 2</text>
                                        </pertinentSupplyAnnotation>
                                    </pertinentInformation>
                                </ehrSupplyAuthorise>
                            </component>
                            <component typeCode="COMP">
                                <ehrSupplyPrescribe classCode="SPLY" moodCode="RQO">
                                    <id root="9B4B797A-D674-4362-B666-2ADC8551EEDA" />
                                    <code code="394823007" displayName="NHS Prescription"
                                        codeSystem="2.16.840.1.113883.2.1.3.2.4.15" />
                                    <statusCode code="COMPLETE" />
                                    <availabilityTime value="20060426" />
                                    <quantity value="1" unit="1">
                                        <translation value="1">
                                            <originalText>tablet(s)</originalText>
                                        </translation>
                                    </quantity>
                                    <inFulfillmentOf typeCode="FLFS">
                                        <priorMedicationRef moodCode="INT">
                                            <id root="TEST_ID" />
                                        </priorMedicationRef>
                                    </inFulfillmentOf>
                                </ehrSupplyPrescribe>
                            </component>
                            <component typeCode="COMP">
                                <ehrSupplyDiscontinue classCode="SPLY" moodCode="RQO">
                                    <id root="D0BF39CA-E656-4322-879F-83EE6E688053" />
                                    <code code="EMISDRUG_DISCONTINUATION"
                                        codeSystem="2.16.840.1.113883.2.1.6.3"
                                        displayName="Medication Course Ended">
                                        <originalText>Ended</originalText>
                                    </code>
                                    <statusCode code="COMPLETE" />
                                    <availabilityTime value="20060426" />
                                    <reversalOf typeCode="REV">
                                        <priorMedicationRef classCode="SBADM" moodCode="ORD">
                                            <id root="TEST_ID" />
                                        </priorMedicationRef>
                                    </reversalOf>
                                    <pertinentInformation typeCode="PERT">
                                        <pertinentSupplyAnnotation classCode="OBS" moodCode="EVN">
                                            <text>Patient no longer requires these</text>
                                        </pertinentSupplyAnnotation>
                                    </pertinentInformation>
                                </ehrSupplyDiscontinue>
                            </component>
                            <pertinentInformation typeCode="PERT">
                                <pertinentMedicationDosage classCode="SBADM" moodCode="RMD">
                                    <text>One To Be Taken Each Day</text>
                                </pertinentMedicationDosage>
                            </pertinentInformation>
                        </MedicationStatement>
                    </component>
                </ehrComposition>
            </component>
        </ehrFolder>
    </component>
</EhrExtract>
```
</details>

## Further documentation
[GP Connect Medication Statement](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_medicationstatement.html)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
