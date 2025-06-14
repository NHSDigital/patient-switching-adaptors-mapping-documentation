# Medication Statement Mapping

## XML HL7 > JSON FHIR

A FHIR `MedicationStatement` is mapped from an XML HL7 `MedicationStatement` containing an `EhrSupplyAuthorise` component.

When `ehrSupplyAuthorise` is referenced, it refers to `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise` where `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise / id [@root]` is present.

When `ehrSupplyDiscontinue` is referenced, it refers to find the first matching `ehrSupplyDiscontinue` where `ehrSupplyDiscontinue / reversalOf / priorMedicationRef / id [@root]` matching `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise / id [@root]` in the `EhrExtract`.

When `ehrSupplyPrescribe` is referenced, it refers to `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyPrescribe` where `MedicationStatement / component [@typeCode="COMP"] / ehrSupplyAuthorise / id [@root]` is present.

When `EhrExtract` is referenced, it refers to the parent `EhrExtract` in the XML.

When `PracticeCode` is referenced, it refers to the losing practice ODS code.

| Mapped to (JSON FHIR Medication Statement field)  | Mapped from (XML HL7 / other source)                                                                                                                                                                                                  |
|---------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                | `ehrSupplyAuthorise / id [@root]` suffixed with `-MS`                                                                                                                                                                                 |
| meta.profile\[0]                                  | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-MedicationStatement-1"`                                                                                                                                  |
| meta.security                                     | When `MedicationStatement / confidentialityCode [@code]` or `EhrComposition / confidentialityCode [@code]` is present and has a value of `NOPAT`. See [Confidentiality Codes](../confidentiality code/README.md) for mapping details. |
| identifier.system                                 | `https:\\PSSAdaptor\{{practiceCode}}` where {{practiceCode}} is losing practice ODS Code.                                                                                                                                             |
| identifier.value                                  | `ehrSupplyAuthorise / id [@root]` suffixed with `-MS`                                                                                                                                                                                 |
| taken                                             | fixed value = `"unk"` <sup>1</sup>                                                                                                                                                                                                    |
| basedOn.reference                                 | a reference to a [MedicationRequest](../medication requests/README.md) using `ehrSupplyAuthorise / id [@root]` as the reference id                                                                                                    |
| extension\[0].url                                 | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescribingAgency-1"`                                                                                                                          |
| extension\[0].valueCodeableConcept.coding.system  | fixed value = `"https://fhir.nhs.uk/STU3/CodeSystem/CareConnect-PrescribingAgency-1"`                                                                                                                                                 |
| extension\[0].valueCodeableConcept.coding.code    | fixed value = `"prescribed-at-gp-practice"`                                                                                                                                                                                           |
| extension\[0].valueCodeableConcept.coding.display | fixed value = `"Prescribed at GP practice"`                                                                                                                                                                                           |
| dosage\[0].text                                   | `MedicationStatement / pertinentInformation / pertinentMedicationDosage / text` <sup>2</sup>                                                                                                                                          |
| extension\[1].url                                 | fixed value = `""https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationStatementLastIssueDate-1"`                                                                                                          |  
| extension\[1].valueDateTime                       | `ehrSupplyPrescribe / availabilityTime [@value]` <sup>3</sup>                                                                                                                                                                         |
| medicationReference.reference                     | reference to a `Medication` with a system generated UUID for this medication <sup>4</sup><sup>5</sup>                                                                                                                                 |
| status                                            | `"STOPPED"`, `"COMPLETED"` or `"ACTIVE"` <sup>6</sup><sup>7</sup>                                                                                                                                                                     |
| effectivePeriod.end                               | `ehrSupplyDiscontinue / availabilityTime [@value]` otherwise `ehrSupplyAuthorise / effectiveTime / high [@value]` otherwise `medicationStatement / effectiveTime [@value]`                                                            |
| effectivePeriod.start                             | `ehrSupplyAuthorise / effectiveTime / center [@value]` otherwise `ehrSupplyAuthorise / effectiveTime / low [@value]` otherwise `ehrSupplyAuthorise / availabilityTime [@value]`<sup>8</sup>                                           |
| dateAsserted                                      | `ehrComposition / author / time [@value]`, `ehrExtract / availabilityTime [@value]`                                                                                                                                                   |
| subject                                           | A reference to the [Patient](../patient/README.md) from `ehrExtract / recordTarget / patient`                                                                                                                                         |
| context                                           | A reference to the [Encounter](../encounters/README.md) which this Medication Statement refers to.                                                                                                                                    |

1. Taken is included as both a mapped field and field not in use.  The reason for this being that the item is mandatory in the base FHIR profile but GP systems do not record this detail. For this reason a default value of 'unk' is picked but should **not** be used</br>
2. If there is no `pertinentInformation` which has a `pertinentInformationDosage / text`, then a default value of `"No Information available"` is used
3. This uses the most recent `availabilityTime` of an `ehrSupplyPrescribe` within any medication statement in the `ehrExtract` where there is an `inFulfilmentOf` which has a `priorMedication / id [@root]` matching the id of the current `MedicationStatement`<br/> If these are no matches then this extension is not added
4. This uses the values of `[@code]`, `[@displayName]` and `originalText` from `MedicationStatement / consumable / manufacturedProduct / manufacturedMaterial / code[0]` to create a unique key with a UUID value.  If this key has been used before then a `Medication` reference with the previously created UUID is used, otherwise a new UUID is generated. 
5. If there are no `consumable` with a `manufacturedProduct` with a `manufacturedMaterial` then this value is not set
6. The `ehrExtract` is parsed . If this has `ehrSupplyDiscontinue / availabilityTime / value` then value is set to `"STOPPED"` otherwise `"COMPLETED"`
7. If there is no matching `ehrSupplyDiscontinue` (see <sup>6</sup>) then if `ehrSupplyAuthorise / statusCode / code` = `"COMPLETE"` then `"COMPLETED"` is used, else `"ACTIVE"` is used
8. When `ehrSupplyDiscontinue / availabilityTime [@value]` exists. If this exists but the referenced values do not then the availability time from either the `MedicationStatement`, `EhrExract` or `EhrComposition` is used. If none of these values exist then the value assigned to `effectivePeriod.end` is used instead

### Unmapped fields

The following Medication Statement fields are not currently populated by the adaptor:

- dosage.patientInstruction
- informationSource
- note



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
      ],
      "security": [
        {
          "system": "http://hl7.org/fhir/v3/ActCode",
          "code": "NOPAT",
          "display": "no disclosure to patient, family or caregivers without attending provider's authorization"
        }
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

An XML HL7 `MedicationStatement` is mapped from a FHIR `medicationRequest` and a FHIR `medication`.  This can be one of two types, `ORDER` or `PLAN` which is determined by value of `medicationRequest.intent`

### Order

| Mapped to (XML HL7 MedicationStatement)                                                  | Mapped from (JSON FHIR / other source)                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
|------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                                                                              | Fetched from resource ID or, if not valid UUID, generated by the Adaptor                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| statusCode \[@code]                                                                      | Mapped from `medicationRequest.status` - either `ACTIVE` or `COMPLETE`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| effectiveTime                                                                            | Mapped from `medicationRequest.dispenseRequest.validityPeriod` - will either have Start (low)  & End (high) or only Start                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| availabilityTime \[@value]                                                               | Mapped from `medicationRequest.dispenseRequest.validityPeriod.start`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| confidentialityCode                                                                      | When `medicationRequest.meta.security` is present and has a value of `NOPAT`. See [Confidentiality Codes](../confidentiality code/README.md) for mapping details.                                                                                                                                                                                                                                                                                                                                                                                                                    |
| consumable \ manufacturedProduct / manufacturedMaterial \[@determinerCode]               | fixed value = `"KIND"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| consumable / manufacturedProduct / manufacturedMaterial / code                           | Mapping for this code section detailed in the section below: [Medication Code Mapping](#medication-code-mapping)                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| component / ehrSupplyPrescribe / id \[@root]                                             | `medicationRequest.id`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | 
| component / ehrSupplyPrescribe / code \[@code]                                           | Only mapped when `medicationStatement.extension[index]` where `extension[index].url.value` equals `https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescribingAgency-1` <br/> 1. If `MedicationStatement.extension[index].valueCodeableConcept.coding[0].code` is `"prescribed-at-gp-practice"` OR `"prescribed-by-previous-practice"` then value is `394823007`.<br/>2. If `MedicationStatement.extension[index].valueCodeableConcept.coding[0].code` is `"prescribed-by-another-organisation"` then value is `394828003`.                                   |
| component / ehrSupplyPrescribe / code \[@displayName]                                    | Only mapped when `medicationStatement.extension[index]` where `extension[index].url.value` equals `https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescribingAgency-1` <br/> 1. If `MedicationStatement.extension[index].valueCodeableConcept.coding[0].code` is `"prescribed-at-gp-practice"` OR `"prescribed-by-previous-practice"` then value is `NHS Prescription`.<br/>2. If `MedicationStatement.extension[index].valueCodeableConcept.coding[0].code` is `"prescribed-by-another-organisation"` then value is `Prescription by another organisation`. |
| component / ehrSupplyPrescribe / code \[@codeSystem]                                     | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| component / ehrSupplyPrescribe / statusCode \[@code]                                     | Mapped from `medicationRequest.status` - either `ACTIVE` or `COMPLETE`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| component / ehrSupplyPrescribe / availabilityTime \[@value]                              | Mapped from `medicationRequest.dispenseRequest.validityPeriod.start`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| component / ehrSupplyPrescribe / quantity \[@value]                                      | If present then: `medicationRequest.dispenseRequest.quantity.value` otherwise defaults to `"1"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| component / ehrSupplyPrescribe / quantity \[@unit]                                       | If present then: `medicationRequest.dispenseRequest.quantity.unit`.<br/>Otherwise if present then: `medicationRequest.dispenseRequest.extension`.<br/>Otherwise if present then: `medicationRequest.dispenseRequest.quantity.extension`.<br/>If none present then default value is `"Unk UoM"` (unknown unit of measurement).                                                                                                                                                                                                                                                        |
| component / ehrSupplyPrescribe / quantity / translation \[@value]                        | If present then: `medicationRequest.dispenseRequest.quantity.value` otherwise defaults to `"1"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
| component / ehrSupplyPrescribe / quantity / translation / originalText                   | If present then: `medicationRequest.dispenseRequest.quantity.value`.<br/>Otherwise if present then: `medicationRequest.dispenseRequest.extension`.<br/>Otherwise if present then: `medicationRequest.dispenseRequest.quantity.extension`.<br/>If none present then default value is `"Unk UoM"` (unknown unit of measurement).                                                                                                                                                                                                                                                       |
| component / ehrSupplyPrescribe / inFulfillmentOf / priorMedicationRef \[@moodCode]       | fixed value = `"INT"`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| component / ehrSupplyPrescribe / inFulfillmentOf / priorMedicationRef / id \[@root]      | Built from `medicationRequest.id`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| component / ehrSupplyPrescribe / pertinentInformation \[@typeCode]                       | fixed value = `PERT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| component / ehrSupplyPrescribe / pertinentInformation / pertinentSupplyAnnotation / text | Only contains the present values and separated by whitespace (`" "`) - built from:<br/> -`medicationRequest.note[0].text`.<br/> -`medicationRequest.dosageInstruction[0].text`<br/> - `medicationRequest.dispenseRequest.expectedSupplyDuration`                                                                                                                                                                                                                                                                                                                                     || pertinentInformation \[@typeCode]                                                        | fixed value = `PERT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| pertinentInformation / pertinentMedicationDosage \[@moodCode]                            | fixed value = `RMD`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| pertinentInformation / pertinentMedicationDosage / text                                  | Built from `medicationRequest.dosageInstruction[0].text`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| participant \[@typeCode]                                                                 | fixed value = `AUT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| participant \[@contextControlCode]                                                       | fixed value = `OP`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| participant / agentRef / id \[@root]                                                     | If resourceType is `"practitioner"` OR `"practitionerRole"` build from `"practitioner.reference"`<br/> If resourceType is `"Organization"` then build from `"organization.reference"`                                                                                                                                                                                                                                                                                                                                                                                                |


<details>
    <summary>Example XML</summary>

```
<MedicationStatement classCode="SBADM" moodCode="ORD">
   <id root="0FC9E511-9CCA-42DD-986A-D58ABD8208C8"/>
   <statusCode code="COMPLETE"/>
   <effectiveTime>
      <low value="20130103"/><high value="20130602"/>
   </effectiveTime>
   <availabilityTime value="20130103"/>
   <confidentialityCode 
       code="NOPAT" 
       codeSystem="2.16.840.1.113883.4.642.3.47" 
       displayName="no disclosure to patient, family or caregivers without attending provider's authorization"
   />
   <consumable typeCode="CSM">
      <manufacturedProduct classCode="MANU">
         <manufacturedMaterial determinerCode="KIND" classCode="MMAT">
            <code code="178011000001104" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Zimovane LS 3.75mg tablets (Sanofi)">
              </code>
         </manufacturedMaterial>
      </manufacturedProduct>
   </consumable>
   <component typeCode="COMP">
      <ehrSupplyPrescribe>
         <id root="61B8E9FD-552A-42A1-8562-208B612C429B"/>
         <code code="394823007" displayName="NHS Prescription" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"/>
         <statusCode code="COMPLETE"/>
         <availabilityTime value="20130103"/>
         <quantity value="150" unit="1">
            <translation value="150">
               <originalText>tablet</originalText>
            </translation>
         </quantity>
         <inFulfillmentOf typeCode="FLFS">
            <priorMedicationRef moodCode="INT">
               <id root="E41C267A-D303-4A13-8F50-C93A1543A279"/>
            </priorMedicationRef>
         </inFulfillmentOf>
         <pertinentInformation typeCode="PERT">
            <pertinentSupplyAnnotation>
               <text>Expected Supply Duration: 150 day</text>
            </pertinentSupplyAnnotation>
         </pertinentInformation>
      </ehrSupplyPrescribe>
   </component>
   <pertinentInformation typeCode="PERT">
      <pertinentMedicationDosage classCode="SBADM" moodCode="RMD">
         <text>One To Be Taken At Night</text>
      </pertinentMedicationDosage>
   </pertinentInformation>
   <Participant typeCode="AUT" contextControlCode="OP">
      <agentRef classCode="AGNT">
         <id root="2D70F602-6BB1-47E0-B2EC-39912A59787D"/>
      </agentRef>
   </Participant>
</MedicationStatement>
```
</details>

### Plan

| Mapped to (XML HL7)                                                                                               | Mapped from (JSON FHIR / other source )                                                                                                                           |
|-------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| MedicationStatement \[@moodCode]                                                                                  | fixed value = `"INT"`                                                                                                                                             |
| MedicationStatement / id \[@root]                                                                                 | Fetched from resource ID or, if not valid UUID, generated by the Adaptor                                                                                          |
| MedicationStatement / statusCode                                                                                  | `medicationRequest.status` when `"active"` = `"ACTIVE"` otherwise is `"COMPLETE"`                                                                                 |
| MedicationStatement / effectiveTime / low \[@value]                                                               | `medicationRequest.dispenseRequest.validityPeriod.start`                                                                                                          |
| MedicationStatement / effectiveTime / high \[@value]                                                              | `medicationRequest.dispenseRequest.validityPeriod.end` <sup>1</sup>                                                                                               |
| MedicationStatement / availabilityTime  \[@value]                                                                 | `medicationRequest.dispenseRequest.validityPeriod.start`                                                                                                          |
| confidentialityCode                                                                                               | When `medicationRequest.meta.security` is present and has a value of `NOPAT`. See [Confidentiality Codes](../confidentiality code/README.md) for mapping details. |
| MedicationStatement / consumable / manufacturedProduct / manufacturedMaterial \[@determinerCode]                  | fixed value = `"KIND"`                                                                                                                                            |
| MedicationStatement / consumable / manufacturedProduct / manufacturedMaterial / code                              | Mapping for this code section detailed in the section below: [Medication Code Mapping](#medication-code-mapping)                                                  |
| MedicationStatement / component \[@typeCode="COMP"] / ehrSupplyAuthorise                                          | Mapping for `ehrSupplyAuthorise` is detailed in the section below: [EhrSupplyAuthorise Mapping](#ehrsupplyauthorise-mapping)                                      |
| MedicationStatement / component \[@typeCode="COMP"] / ehrSupplyDiscontinue \[@classCode="SPY"] \[@moodCode="PRQ"] | Mapping for `ehrSupplyDiscontinue` is detailed in the section below: [EhrSupplyDiscontinue Mapping](#ehrsupplydiscontinue-mapping)                                |
| MedicationStatement / pertinentInformation / pertinentMedicationDosage \[@moodCode]                               | fixed value `"RMD"`                                                                                                                                               |
| MedicationStatement / pertinentInformation / pertinentMedicationDosage / text                                     | `medicationRequest.dosageInstruction[0].text`                                                                                                                     |
| MedicationStatement / participant \[@typeCode] \[@contextControlCode]                                             | fixed value = `@typeCode="AUT"`, fixed value = `@tcontextControlCode="OP"`. There may be zero, one or many participants                                           |
| MedicationStatement / participant / agentRef / id \[@root]                                                        | See <sup>3</sup>                                                                                                                                                  |

1. If `medicationRequest.dispenseRequest.validityPeriod.end` does not exist then this value is not populated
2. When `medicationRequest.requester.agent.referenceElement.resourceType` = `"Practitioner"` and `medicationRequest.requester.onBehalfOf` is populated then value is set from the reference to the `medicationRequest.requester.agent` or `medicationRequest.requester.onBehalfOf`</br> Otherwise, when only `medicationRequest.requester.agent` exists then value is set from the reference to `medicationRequest.requester.agent.reference` (when `resourceType` = `Practicioner`) or is the reference to the organisation (when `resourceType` = `Organisation`)</br>  If there are no matches for the previous options and `medicationRequest.recorder.reference` exists and the reference is either a `Practitioner`,`PractitionerRole` or `Organisation` then the reference is set from the reference to `medicationRequest.recorder.reference` 


### EhrSupplyAuthorise mapping

This is part of the mapping for a `MedicationRequest` and is mandatory and only included when the mapping intent is a `"PLAN"`

| Mapped to (EhrSupplyAuthorise XML HL7)                                       | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                                                                                                                                               |
|------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ehrSupplyAuthorise / id \[@root]                                             | Fetched from resource ID or, if not valid UUID, generated by the Adaptor                                                                                                                                                                                                                                                                                                              |
| ehrSupplyAuthorise / code \[@code]                                           | when `code.coding.code` = `"prescribed-by-another-organisation"` then value of `"394828003"` is used, otherwise value of `"394823007"` is used<sup>1</sup>                                                                                                                                                                                                                            |
| ehrSupplyAuthorise / code \[@displayName]                                    | when `code.coding.code` = `"prescribed-by-another-organisation"` then value of `"Prescription by another organisation"` is used,  otherwise value of `"NHS Prescription"` is used<sup>1</sup>                                                                                                                                                                                         |
| ehrSupplyAuthorise / code \[@codeSystem]                                     | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                                                                                                                                                                                                                                                      |
| ehrSupplyAuthorise / statusCode \[@code]                                     | `medicationRequest.status` when `"active"` = `"ACTIVE"` otherwise is `"COMPLETE"`                                                                                                                                                                                                                                                                                                     |
| ehrSupplyAuthorise / effectiveTime / low \[@value]                           | `medicationRequest.dispenseRequest.validityPeriod.start`                                                                                                                                                                                                                                                                                                                              |
| ehrSupplyAuthorise / effectiveTime / high \[@value]                          | `medicationRequest.dispenseRequest.validityPeriod.end` <sup>2</sup>                                                                                                                                                                                                                                                                                                                   |
| ehrSupplyAuthorise / availabilityTime \[@value]                              | `medicationRequest.dispenseRequest.validityPeriod.start`                                                                                                                                                                                                                                                                                                                              |
| ehrSupplyAuthorise / repeatNumber \[@value]                                  | `medicationRequest.extension.value.coding[0].code` when `medicationRequest.extension.url` = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescriptionType-1"` <sup>3</sup>                                                                                                                                                                                | 
| ehrSupplyAuthorise / quantity \[@value] \[@unit=`"1"`]                       | `medicationRequest.dispenseRequest.quantity.value` when present, otherwise a default value of `"1"` is used                                                                                                                                                                                                                                                                           |
| ehrSupplyAuthorise / quantity / translation \[@value]                        | `medicationRequest.dispenseRequest.quantity.value` when present, otherwise a default value of `"1"` is used                                                                                                                                                                                                                                                                           |
| ehrSupplyAuthorise / quantity / translation / originalText                   | `medicationRequest.dispenseRequest.quantity.unit`, `medicationRequest.dispenseRequest.extension[0].value`<sup>4</sup>, `medicationRequest.dispenseRequest.quantity.extension[0].value`<sup>5</sup><sup>6</sup>                                                                                                                                                                        |
| ehrSupplyAuthorise / pertinentInformation / pertinentSupplyAnnotation / text | `medicationRequest.dosageInstruction[0].patientInstruction` prefixed by `"Patient Instruction: "` & `medicationRequest.dispenseRequest.expectedSupplyDuration.value` prefixed by `"Expected Supply Duration"` and suffixed with `medicationRequest.dispenseRequest.expectedSupplyDuration.code` displayed as a unit of time & all `medicationRequest.note.text` prefixed by `"Notes"` |

1. The `codableConcept` from `medicationStatement.extension[0].value` from the `ehrExtract` where `medicationStatement.basedOn` contains a reference to `medicationRequest.id` and `medicationStatement.extension[0].url` = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescribingAgency-1"` is used to generate the values.
2. If `medicationRequest.dispenseRequest.validityPeriod.end` does not exist then this value is not populated
3. When `medicationRequest.extension.value.coding[0].code` = `"acute"` or `"acute-handwritten"` and `medicationRequest.extension.url` = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-PrescriptionType-1"` then value used is `0`.</br> When `medicationRequest.extension.value.coding[0].code` = `"delayed-prescribing"`, `"repeat"` or `"repeat-dispensing"` then `medicationRequest.extension[0].value` when `medicationRequest.extension[0].url` = `"numberOfRepeatPrescriptionsAllowed"`.</br> If no value is found in either case then a default value of `1` is used.
4. When `medicationRequest.dispenseRequest.extension[0].url` = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationQuantityText-1"`
5. When 'medicationRequest.dispenseRequest.quantity.extension[0].url' = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationQuantityText-1"`
6. When no other values are found a default value of `"Unk UoM"` is used.


### EhrSupplyDiscontinue mapping

This is part of the mapping for a `MedicationRequest` and is mapped when one of `MedicationRequest.extension\[index].url` = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationStatusReason-1"` (Medication Status Reason Stopped URL).  If this is not present then this mapping is omitted.

When `statusReasonExtension` is used, it refers to the `medicationRequest.extension` where `medicationRequest.extension.url` = `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-CareConnect-GPC-MedicationStatusReason-1"`

| Mapped to (EhrSupplyDiscontinue XML HL7)                                                                           | Mapped from (JSON FHIR / other source )                                                                                                                                                |
|--------------------------------------------------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ehrSupplyDiscontinue / id \[@root]                                                                                 | adaptor generated UUID                                                                                                                                                                 |
| ehrSupplyDiscontinue / code                                                                                        | set from the `codableConcept` in `statusReasonExtension.extension[0].value` when `statusReasonExtension.extension[0].value.url` = `"statusReason"`                                     |
| ehrSupplyDiscontinue / statusCode \[@code]                                                                         | fixed value = `"COMPLETE"`                                                                                                                                                             |
| ehrSupplyDiscontinue / availabilityTime \[@value]                                                                  | `statusReasonExtension.extension[0].value` when `statusReasonExtension.extension[0].url` = `"statusChangeDate"`<sup>1</sup>                                                            |
| ehrSupplyDiscontinue / reversalOf \[@typeCode]                                                                     | fixed value = `"REV"`                                                                                                                                                                  |
| ehrSupplyDiscontinue / reversalOf / priorMedicationRef \[@moodCode] \[@classCode]                                  | fixed values `@moodCode="ORD"`, `@classCode="SBADM"`                                                                                                                                   |
| ehrSupplyDiscontinue / reversalOf / priorMedicationRef / id \[@root]                                               | same generated UUID as used for `ehrSupplyAuthorise / id \[@root]`                                                                                                                     |
| ehrSupplyDiscontinue / pertinentInformation / pertinentSupplyAnnotation \[@moodCode] \[@classCode]                 | fixed values `@moodCode="EVN"`, `@classCode="OBS"`                                                                                                                                     |
| ehrSupplyDiscontinue / pertinentInformation / pertinentSupplyAnnotation / text                                     | `statusReasonExtension.value.text`, `statusReasonExtension.value.coding[0].extension[0].extension[0].value`<sup>2</sup> or `statusReasonExtension.value.coding[0].display`<sup>2</sup> |
| ehrSupplyDiscontinue / pertinentInformation / pertinentSupplyAnnotation / code \[@nullFlavour="UNK" / originalText | this is omitted unless `ehrSupplyDiscontinue / pertinentInformation / pertinentSupplyAnnotation / text` cannot be set then fixed value = `"Stopped"` is used.                          |

1. When the other values do not exist then a default value of `@nullFlavor="UNK"` is used and `ehrSupplyDiscontinue / code / originalText` is added with a value of `"Stopped"`
2. When `statusReason` contains an extension where `extension.url` = `"statusReason"`

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
                            <confidentialityCode 
                                code="NOPAT" 
                                codeSystem="2.16.840.1.113883.4.642.3.47" 
                                displayName="no disclosure to patient, family or caregivers without attending provider's authorization"
                            />
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
                            <Participant typeCode="AUT" contextControlCode="OP">
                                <agentRef classCode="AGNT">
                                    <id root="2D70F602-6BB1-47E0-B2EC-39912A59787D"/>
                                </agentRef>
                            </Participant>
                        </MedicationStatement>
                    </component>
                </ehrComposition>
            </component>
        </ehrFolder>
    </component>
</EhrExtract>
```
</details>


### Medication Code mapping

This is mapped from either `Medication.code` where `Medication` is the resource referenced by 
`medicationRequest.medicationReference.reference`, or directly from `medicationRequest.medicationCodeableConcept`.

| Mapped to (MedicationRequest / Code XML HL7) | Mapped from (JSON FHIR / other source )                                                                                                        |
|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|
| code \[@nullFlavor]                          | fixed value = `"UNK"` when `code.coding.system` is not `"http://snomed.info/sct"`                                                              |
| code \[@code]                                | when `code.coding.system` is `"http://snomed.info/sct"` then `code.coding.code` if present                                                     |
| code \[@codeSystem]                          | when `code.coding.system` is `"http://snomed.info/sct"` then fixed value `[@code="2.16.840.1.113883.2.1.3.2.4.15"]`                            |
| code \[@displayName]                         | `code.coding.extension[0].value` or `Medication.code.coding.display` <sup>1</sup>                                                              |
| code / originalText                          | `code.text` or `code.coding.display` or `code.coding.extension.value` when `code.coding.extension[0].url` = `"descriptionDisplay"`<sup>2</sup> |

1. if `code.coding.extension[0].url` = `"descriptionDisplay"` then `code.coding.extension[0].value` is used. Otherwise `code.coding.display` is used
2. if no value for `originalText` can be found then this section is omitted.

## Further documentation

- [GP Connect MedicationStatement](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_medicationstatement.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
