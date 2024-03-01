# Diagnostic Report Mapping

## XML HL7 > JSON FHIR

### Diagnostic Report (XML > JSON)

FHIR Diagnostic Reports are mapped from HL7 Compound Statements where `CompoundStatement [@classCode]` equals `"CLUSTER"`
and `CompoundStatement / code [@code]` equals `"16488004"` (laboratory reporting).  

| Mapped to (JSON FHIR DiagnosticReport field) | Mapped from (XML HL7 / other source)                                                                                                                                                                                                                                                                           |
|----------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                           | `CompoundStatement / id[0] [@root]`                                                                                                                                                                                                                                                                            |
| meta.profile\[0]                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-DiagnosticReport-1"`                                                                                                                                                                                                              |
| identifier\[0].system                        | `"https://PSSAdaptor/{{odsCode}}` where `{{odsCode}}` is the ODS Code of the losing practice                                                                                                                                                                                                                   |
| identifier\[0].value                         | `CompoundStatement / id[0] [@root]`                                                                                                                                                                                                                                                                            |
| identifier\[1].system                        | fixed value = `"2.16.840.1.113883.2.1.4.5.5"` <sup>1</sup>                                                                                                                                                                                                                                                     |
| identifier\[1].value                         | `CompoundStatement / id[1] [@extension]` <sup>1</sup>                                                                                                                                                                                                                                                          |
| status                                       | fixed value = `"unknown"`                                                                                                                                                                                                                                                                                      |
| code.coding\[0].code                         | fixed value =  `"721981007"`                                                                                                                                                                                                                                                                                   |
| code.coding\[0].system                       | fixed value = `"http://snomed.info/sct"`                                                                                                                                                                                                                                                                       |
| code.coding\[0].display                      | fixed value = `"Diagnostic studies report"`                                                                                                                                                                                                                                                                    |
| subject.reference                            | reference to the mapped [Patient](../patient/README.md)                                                                                                                                                                                                                                                        | 
| context.reference                            | reference to the associated [Encounter](../encounters/README.md)                                                                                                                                                                                                                                               |
| issued                                       | `CompoundStatement /availabilityTime [@value]` or else `ehrComposition / author / time [@value]`                                                                                                                                                                                                               |
| specimen.reference                           | reference to the [Specimen(s)](#Specimen-(XML->-JSON)) the results are based on                                                                                                                                                                                                                                |
| result\[index].reference                     | reference to the associated Observations, either [Test Result Header](../observations/README.md#Test-Group-Header-(XML-HL7->-JSON-FHIR)), [Test Result](../observations/README.md#Test-Result-(XML-HL7->-JSON-FHIR)) or [Test Report Filing](../observations/README.md#Filing-Comment-(XML-HL7->-JSON-FHIR))   |
| conclusion                                   | `CompoundStatement / component / NarrativeStatement / text` <sup>2</sup>                                                                                                                                                                                                                                       |


<details>
    <summary>Example JSON</summary>

```JSON

{
 "resource": {
  "resourceType": "DiagnosticReport",
  "id": "5A8B9936-B771-488E-9103-3331629690C4",
  "meta": {
   "profile": [
    "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-DiagnosticReport-1"
   ]
  },
  "identifier": [
   {
    "system": "https://PSSAdaptor/D5445",
    "value": "5A8B9936-B771-488E-9103-3331629690C4"
   },
   {
    "system": "2.16.840.1.113883.2.1.4.5.5",
    "value": "1013/HA2101109A/200203301621"
   }
  ],
  "status": "unknown",
  "code": {
   "coding": [
    {
     "system": "http://snomed.info/sct",
     "code": "721981007",
     "display": "Diagnostic studies report"
    }
   ]
  },
  "subject": {
   "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
  },
  "context": {
   "reference": "Encounter/C5323AB7-2CA9-4E85-85E4-8FA896E45628"
  },
  "issued": "2010-06-24T10:34:01.000+00:00",
  "specimen": [
   {
    "reference": "Specimen/73A3DD99-861F-45E3-B7BB-30F71A74AE85"
   }
  ],
  "result": [
   {
    "reference": "Observation/F022B5F4-66E8-4DDD-ABAA-10C98713E7EE"
   },
   {
    "reference": "Observation/92DA878D-2346-488E-8742-01ECDE95E31C"
   }
  ]
 }
}
```
</details>

1. Only where `CompoundStatement / id[1] [@root]` equals `"2.16.840.1.113883.2.1.4.5.5"`, otherwise not mapped. 
2. Only if the EDIFACT comment type is `LABORATORY RESULT COMMENT(E141)`, otherwise not mapped.

#### Unmapped fields
The adaptor is not currently mapping the following Diagnostic Report fields: 

- basedOn
- category
- performer
- codedDiagnosis

### Specimen (XML > JSON)

A Specimen is mapped from HL7 `CompoundStatement` with code `123038009` (specimen (specimen)) where it is a 
**child component** of a `CompoundStatement`with code `16488004` (laboratory reporting).    

| Mapped to (JSON FHIR Specimen field) | Mapped from (XML HL7 / other source)                                                          |
|--------------------------------------|-----------------------------------------------------------------------------------------------|
| id                                   | `CompoundStatement / id [@root]`                                                              |
| meta.profile[0]                      | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Specimen-1"`     |
| identifier[0].system                 | `"https://PSSAdaptor/{{odsCode}}"` - where {{odsCode}} is the ODS code of the losing practice |
| identifier[0].value                  | `CompoundStatement / id [@root]`                                                              |
| accessionIdentifier.value            | `CompoundStatement / specimen[0] / specimenRole / id[1] [@extension]`                         |
| type.text                            | `CompoundStatement / specimen[0] / specimenRole / specimenSpecimenMaterial / desc`            | 
| subject.reference                    | reference to the mapped [Patient](../patient/README.md)                                       |
| collection.collectedDateTime         | `CompoundStatement / specimen[0] / specimenRole / effectiveTime / center [@value]`            |
| note[0].text                         | `CompoundStatement / component / NarrativeStatement / text` <sup>3</sup>                      |


<details>
    <summary>Example JSON</summary>

```JSON

{
  "resource": {
    "resourceType": "Specimen",
    "id": "73A3DD99-861F-45E3-B7BB-30F71A74AE85",
    "meta": {
      "profile": [
        "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Specimen-1"
      ]
    },
    "identifier": [
      {
        "system": "https://PSSAdaptor/D5445",
        "value": "73A3DD99-861F-45E3-B7BB-30F71A74AE85"
      }
    ],
    "accessionIdentifier": {
      "value": "HA2101109A"
    },
    "type": {
      "text": "VENOUS BLOOD"
    },
    "subject": {
      "reference": "Patient/80bd1375-a1e3-4137-95ff-fb3316cf3496"
    },
    "collection": {
      "collectedDateTime": "2003-01-09"
    },
    "note": [
      {
        "text": "Some Test Specimen Comment"
      }
    ]
  }
}
```
</details>

3. Where there is more than one Narrative Statement, the text will be concatenated into one note, seperated by newlines.

#### Unmapped fields
The adaptor is not currently mapping the following Specimen fields:

- status
- receivedTime
- collection.extension
- collection.collector
- collection.quantity
- collection.bodysite

## JSON FHIR > XML HL7

### Diagnostic Report (JSON > HL7)
The FHIR Diagnostic Report resource is mapped to a HL7 Compound Statement with a code of `16488004` (laboratory reporting)

| Mapped to (XML HL7 CompoundStatement element) | Mapped from (JSON FHIR / other source )                                                                                                               |
|-----------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------|
| id\[0] [@root]                                | Unique ID generated by the adaptor                                                                                                                    |
| id\[1] [@extension]                           | `DiagnosticReport.identifier[index].value` where `DiagnosticReport.identifier[index].system` equals `"2.16.840.1.113883.2.1.4.5.5"`                   |
| id\[1] [@root]                                | fixed value = `"2.16.840.1.113883.2.1.4.5.5"`                                                                                                         |
| code [@code]                                  | fixed value = `"16488004"`                                                                                                                            |
| code [@codeSystem]                            | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                      |
| code [@displayName]                           | fixed value = `"laboratory reporting"`                                                                                                                |
| code / originalText                           | fixed value = `"Filed Report"`                                                                                                                        |
| statusCode [@code]                            | fixed value = `"COMPLETE"`                                                                                                                            |                                                                                                          
| effectiveTime / center [@nullFlavor]          | fixed value = `"NI"`                                                                                                                                  |
| availabilityTime [@value]                     | `DiagnosticReport.issued`                                                                                                                             |
| component / NarrativeStatement                | the mapped [diagnostic report level Narrative Statements](#Diagnostic-Report-Level-Narrative-Statements)                                              |
| component / CompoundStatement                 | the mapped [Specimens](#Specimen-(JSON->-HL7)) referenced by the `DiagnosticReport.specimen` array <sup>4</sup>                                       |
| Participant [@typeCode]                       | fixed value = `"AUT"`                                                                                                                                 |
| Participant / agentRef / id [@root]           | The mapped [Practitioner](../practioners/README.md) or [Organisation](../organisations/README.md) referenced by `DiagnosticReport.performer[0].actor` |

<details>
    <summary>Example XML</summary>

```XML

<CompoundStatement classCode="CLUSTER" moodCode="EVN">
 <id root="74F2D322-9032-4ED2-999D-0AACA87B3BC8"/>
 <id extension="1013/HA2101105W/200203301621" root="2.16.840.1.113883.2.1.4.5.5"/>
 <code code="16488004" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="laboratory reporting">
  <originalText>Filed Report</originalText>
 </code>
 <statusCode code="COMPLETE"/>
 <effectiveTime>
  <center nullFlavor="NI"/>
 </effectiveTime>
 <availabilityTime value="20020330162100"/>
 <component contextConductionInd="true" typeCode="COMP">
  <NarrativeStatement classCode="OBS" moodCode="EVN">
   <id root="E8F624FF-9364-42BB-9E01-8D3512BBE732"/>
   <text mediaType="text/x-h7uk-pmip">CommentType:LABORATORY RESULT COMMENT(E141)
CommentDate:20020330162100

Interpretation: ON AZATHIOPRINE
   </text>
   <statusCode code="COMPLETE"/>
   <availabilityTime value="20020330162100"/>
  </NarrativeStatement>
 </component>
 <component contextConductionInd="true" typeCode="COMP">
  <CompoundStatement classCode="CLUSTER" moodCode="EVN">

   ...

  </CompoundStatement>
 </component>
</CompoundStatement>
```
</details>

4. A `component / CompoundStatement` is created for each `Specimen` referenced in the array 

### Diagnostic Report Level Narrative Statements

Narrative Statements are created to preserve data from FHIR fields that have no direct mapping.

| Mapped to (XML HL7 NarrativeStatement element) | Mapped from (JSON FHIR / other source ) |
|------------------------------------------------|-----------------------------------------|
| id [@root]                                     | Unique ID generated by the adaptor      |
| text [@mediaType]                              | fixed value = "text/x-h7uk-pmip"        |
| text                                           | EDIFACT comment, as described below     |
| statusCode                                     | fixed value = `"COMPLETE"`              |
| availabilityTime [@value]                      | `DiagnosticReport.issued`               |

#### Text Element Mapping

 Narrative Statements are created in relation to each of the FHIR Fields / Logical Reasons below. The value of the field 
 is mapped to the Narrative Statement's `text` element as EDIFACT comment (see example in the XML above) as the comment 
 body. The commentDate is mapped from `DiagnosticReport.issued` and the commentType is dependent in the field / reason
 shown in the table:

| FHIR field / logical reason                                                                                                                                                                                 | EDIFACT Comment Type                                    | body prepended with |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------|---------------------|
| `diagnosticReport.conclusion`                                                                                                                                                                               | `LABORATORY RESULT COMMENT(E141)`                       | `Interpretation: `  |
| concatenated from `dianosticReport.codedDiagnosis`                                                                                                                                                          | `LABORATORY RESULT COMMENT(E141)`                       | `Lab Diagnosis: `   |
| `dianosticReport.status`                                                                                                                                                                                    | `LABORATORY RESULT COMMENT(E141)`                       | `Status: `          |
| none of the fields above are populated and `dianosticReport.result` is missing                                                                                                                              | `AGGREGATE COMMENT SET` and fixed body = `EMPTY REPORT` |                     |
| `Observation.EffectiveDateTime` or `Observation.EffectivePeriod` where the Observation <br/> is a referenced in `dianosticReport.result` and the first Observation coded as `37331000000100` (Comment note) | `AGGREGATE COMMENT SET`                                 | `Filing Date: `     |
| Concatenated from `diagnosticReport.performer[index].name`                                                                                                                                                  | `AGGREGATE COMMENT SET`                                 | `Participants: `    |
| Concatenated from each `Observation.comment` where the Observation is referenced in <br/> `dianosticReport.result` and coded as `37331000000100` (comment note)                                             | `USER COMMENT`                                          |                     |

### Specimen (JSON > HL7)

The FHIR Specimen resource is mapped to a HL7 Compound Statement with a code of `123038009` (specimen (specimen))  

| Mapped to (XML HL7 CompoundStatement element)             | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                                                          |
|-----------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id\[0] [@root]                                            | Unique ID generated by the adaptor                                                                                                                                                                                                                                                               |
| code [@code]                                              | fixed value = `"123038009"`                                                                                                                                                                                                                                                                      |
| code [@codeSystem]                                        | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                                                                                                                                                                                                 |
| code [@displayName]                                       | fixed value = `"specimen (specimen)"`                                                                                                                                                                                                                                                            |
| statusCode [@code]                                        | fixed value = `"COMPLETE"`                                                                                                                                                                                                                                                                       |
| effectiveTime / center [@nullFlavor]                      | fixed value = `"NI"`                                                                                                                                                                                                                                                                             |
| availabilityTime [@value]                                 | `DiagnosticReport.issued`                                                                                                                                                                                                                                                                        |
| specimen / specimenRole / id\[0] [@root]                  | Unique ID generated by the adaptor                                                                                                                                                                                                                                                               |
| specimen / specimenRole / id\[1] [@root]                  | fixed value = `"2.16.840.1.113883.2.1.4.5.2"` if `Specimen.accessionIdentifier` is present                                                                                                                                                                                                       |
| specimen / specimenRole / id\[1] [@extension]             | `Specimen.accessionIdentifier.value`                                                                                                                                                                                                                                                             |
| effectiveTime / center [@value]                           | `Specimen.collection.collectedDateTime` or else `Specimen.collection.collectePeriod.start` or else `Specimen.recievedTime`                                                                                                                                                                       | 
| specimen / specimenRole / specimenSpecimenMaterial / desc | `Specimen.type.text` or else `Specimen.type.coding[0].extention[index].extension[index2].value` <sup>5</sup>                                                                                                                                                                                     |
| component / NarrativeStatement                            | the mapped [specimen level narrative statement](#Specimen-Level-Narrative-Statements)                                                                                                                                                                                                            |
| component / CompoundStatement                             | mapped from the [Test Group Header](../observations/README.md#Test-Group-Header-(JSON-FHIR->-XML-HL7)) or [Test Result](../observations/README.md#Test-Result-(JSON-FHIR->-XML-HL7)) Observations referenced in `DiagnosticReport.result` where `Observation.specimen` references the `Specimen` |

<details>
 <summary>Example XML</summary>

```XML

<CompoundStatement classCode="CLUSTER" moodCode="EVN">
 <id root="D6930FE2-0417-448B-85FA-0ACA7284CBD0"/>
 <code code="123038009" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="specimen (specimen)"/>
 <statusCode code="COMPLETE"/>
 <effectiveTime>
  <center nullFlavor="NI"/>
 </effectiveTime>
 <availabilityTime value="20030628112300"/>
 <specimen typeCode="SPC">
  <specimenRole classCode="SPEC">
   <id root="DB2E5F7D-B0D0-488A-825A-BE58D09953CF"/>
   <id extension="C3179539H" root="2.16.840.1.113883.2.1.4.5.2"/>
   <effectiveTime>
    <center value="20030627000000"/>
   </effectiveTime>
   <specimenSpecimenMaterial classCode="MAT" determinerCode="INSTANCE">
    <desc>Plasma</desc>
   </specimenSpecimenMaterial>
  </specimenRole>
 </specimen>
 <component contextConductionInd="true" typeCode="COMP">
  <NarrativeStatement classCode="OBS" moodCode="EVN">
   <id root="DF0FEFF0-32C1-4BB9-8DDD-76E2D13B5D0F"/>
   <text mediaType="text/x-h7uk-pmip">CommentType:LAB SPECIMEN COMMENT(E271)
CommentDate:20030627000000

Quantity: 1750.000 mL
Received Date: 2003-06-27 18:24
   </text>
   <statusCode code="COMPLETE"/>
   <availabilityTime value="20030628112300"/>
  </NarrativeStatement>
 </component>
 <component contextConductionInd="true" typeCode="COMP">
  <CompoundStatement classCode="BATTERY" moodCode="EVN">
   
   ...
   
  </CompoundStatement>
 </component>
</CompoundStatement>
```
</details>

5. Where `extension[index].url` equals `"https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid"`, 
`extension[index2].url` equals `"descriptionDisplay"` and `extension[index2].value` does not equal `Specimen.type.coding[0].display` 

### Specimen Level Narrative Statements

A Narrative Statement is created to preserve data from FHIR fields that have no direct mapping.

| Mapped to (XML HL7 NarrativeStatement element) | Mapped from (JSON FHIR / other source ) |
|------------------------------------------------|-----------------------------------------|
| id [@root]                                     | Unique ID generated by the adaptor      |
| text [@mediaType]                              | fixed value = `"text/x-h7uk-pmip"`      |
| text                                           | EDIFACT comment, as described below     |
| statusCode                                     | fixedValue = `"COMPLETE"`               |
| availabilityTime [@value]                      | `DiagnosticReport.issued`               |

#### Text Element Mapping

Unlike a diagnostic report `CompoundStatement`, which can have multiple `NarrativeStatement` components, the Specimen 
`CompoundStatement` will only have a single `NarrativeStatement` if Specimen fields without a direct mapping are 
populated. The fields are concatenated and seperated by newlines, before being inserted into the `text` element of the 
`NarrativeStatement` as an EDIFACT comment(example in the XML above). The comment type of the EDIFACT comment is fixed 
to `LAB SPECIMEN COMMENT(E271)`.

The fields / values mapped to the comment are as follows:

- fixed value = `"EMPTY SPECIMEN"` if no Observations referenced in `DiagnosticReport.result` reference the `Specimen`.  
- `Specimen.receivedTime` prepended with `"Received Date:"`
- `Specimen.collection.extension[index].valueCodeableConcept` prepended with `"Fasting Status:"`. Where 
`Specimen.collection.extension[index].url` equals `"https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-FastingStatus-1"`
- `Specimen.collection.extension[index].valueDuration` prepended with  `"Fasting Duration:"`
- `Specimen.collection.quantity` prepended with `"Quantity:"`
- `Specimen.collection.bodySite` prepended with `"Collection Site:"`
-  Concatenates `Practitioner.name.prefix`, `Practitioner.name.given`, `Practitioner.name.family` where the `Practitioner` 
is referenced by `Specimen.collection.collector`, prepended with `"Collected By:"`
- Each `Specimen.note[index]` 

## Further documentation

- [GP Connect Diagnostic report documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_DiagnosticReport.html)
- [GP Connect Specimen documentation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_specimen.html)
- [GP Connect Investigations Guidance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_pathology_guidance.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 