# Immunization Mapping

## XML HL7 > JSON FHIR

An Immunization is mapped from a ObservationStatement.

| Mapped to (JSON FHIR Immunization field)  | Mapped from (XML HL7 / other source)                                                                                                                 |
|-------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                        | `ObservationStatement / id / [root]`                                                                                                                 |
| meta.profile                              | fixed value = `https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Immunization-1`                                                          |
| extension\[0].url                         | fixed value = `https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-VaccinationProcedure-1`                                        |
| extension\[0].valueCodeableConcept.text   | `ObservationStatement / code / displayname`                                                                                                          |
| extension\[1].url                         | fixed value = `https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-DateRecorded-1`                                                |
| extension\[1].valueDateTime               | `ehrComposition / author / time [@value]` or `ehrComposition / availabilityTime [@value]`                                                            |
| identifier\[0].system                     | "https://PSSAdaptor/{{losingOdsCode}}" - where the {{losingOdsCode}} is the ODS code of the losing practice                                          |
| identifier\[0].value \[@root]             | `ObservationStatement / id`                                                                                                                          |
| status                                    | fixed value = `COMPLETED`                                                                                                                            |
| notGiven                                  | fixed boolean value = `false`                                                                                                                        |
| patient.reference                         | reference to the mapped [patient](../patient/README.md)                                                                                              |
| encounter.reference                       | reference to the associated [encounter](../encounters/README.md)                                                                                     |
| date                                      | `ObservationStatement / effectiveTime / center` or else `ObservationStatement / effectiveTime / low` or else `ObservationStatement.availabilityTime` |
| primarySource                             | fixed boolean value = `false`                                                                                                                        |
| practitioner.actor.reference              | `ObservationStatement / Participant / typeCode / agentRef / id [@root]`                                                                              |
| practitioner.role                         | `EP` type code or null                                                                                                                               |
| note\[0].text                             | `ObservationStatement / pertiinentInformation / pertiantAnnotation / text` - Built from multiple                                                     |
| note\[1].text\[1]                         | `ObservationStatement / effectiveTime / high` prepended with End Date:                                                                               |

<details>
    <summary>Example JSON</summary>

```JSON
{
     "resource": {
         "resourceType": "Immunization",
         "id": "immunization-id",
         "meta": {
             "profile": [
                 "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Immunization-1"
             ]
         },
         "extension": [
             {
                 "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-VaccinationProcedure-1", 
                 "valueCodeableConcept": {
                     "text": "Haemophilus influenzae type B and meningitis C vaccination"
                 }                 
             },
             {
                 "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-DateRecorded-1",
                 "valueDateTime": "2010-01-13T15:13:32+00:00.00"
             }
         ],
         "identifier": [
             {
                 "system": "https://PSSAdaptor/2167888433",
                 "value": "immunization-id"
             }
         ],
         "status": "completed",
         "notGiven": false,
         "patient": {
             "reference": "Patient/c2e046b3-6d29-423a-96af-d58640d65e7e"
         },
         "encounter": {
             "reference": "Encounter/2485BC20-90B4-11EC-B1E5-0800200C9A66"
         },
         "date": "2010-01-18T11:41:00+00:00",
         "primarySource": false,
         "practitioner": [
             {
               "role": {
                 "text": "EP"
               },
               "actor": {
                     "reference": "Practitioner/9C1610C2-5E48-4ED5-882B-5A4A172AFA35"
                 }
             }
         ],
         "note": [
             {
                 "text": "Primary Source: true Location: EMIS Test Practice Location Manufacturer:\n another company Batch: past2003 Expiration: 2003-01-17 Site: Right arm GMS : Not\nGMS\n"               
             },
             {
                 "text": "End Date: 2010-01-18T11:41:00+00:00"
             }
         ]
     }
 }
```
</details>

## Unmapped Fields

The following Immunization fields are not currently populated by the adaptor:

- vaccineCode
- reportOrigin
- location
- implicitRules
- language
- valueCodeableConcept
- modifierExtension
- manufacturer
- lotNumber
- expirationDate
- site
- reasonNotGiven
- doseQuantity
- practitioner.role
- practitioner.role
- explanation.reason
- vaccinationProtocol
- reaction
- reported
- doseStatusReason
- route


## XML HL7 > JSON FHIR

An Immunization is primarily mapped from an Observation Statement.

| Mapped to (XML HL7)                               | Mapped from (JSON FHIR / other source ) |
|---------------------------------------------------|-----------------------------------------|
| id [@root]                                        | `id`                                    |
| code / display                                    | `extension.valueCodeableConcept.text`   |
| Participant / agentRef / id [@root]               | practitioner.role = `AP`                |
| statusCode                                        | fixed value = `"COMPLETE"`              |
| effectiveTime                                     | `date`                                  |
| availabilityTime                                  | `date`                                  |
| pertiinentInformation / pertiantAnnotation / text | `note.text`                             |
| Participant [@typecode] / agentRef / id [@root]   | fixed value = `PRF`                     |


<details><summary>Example XML</summary>

```XML

<ObservationStatement classCode="OBS" moodCode="EVN">
    <id root="9B45E4E6-9522-4C7E-A0CC-9632CF84B0C2"/>
    <code code="65004017" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Measles-mumps-rubella vaccination"/>
    <statusCode code="COMPLETE"/>
    <effectiveTime>
        <center value="20100630055900"/>
    </effectiveTime>
    <availabilityTime value="20100630055900"/>
    <pertinentInformation typeCode="PERT">
        <sequenceNumber value="+1"/>
        <pertinentAnnotation classCode="OBS" moodCode="EVN">
            <text>Primary Source: true Location: EMIS Test Practice Location Manufacturer: Pete Batch: 123456 Expiration: 2011-06-21
                Site: Left arm GMS : GMS test text.
            </text>
        </pertinentAnnotation>
    </pertinentInformation>
    <Participant contextControlCode="OP" typeCode="PRF">
        <agentRef classCode="AGNT">
            <id root="63992CB8-1168-4DCC-8344-F5A9946BB6D1"/>
        </agentRef>
    </Participant>
</ObservationStatement>
```

</details>

## Further documentation

- [GP Connect Immunisation](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_immunization.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)