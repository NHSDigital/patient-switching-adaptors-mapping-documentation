# Immunization Mapping

## JSON FHIR > XML HL7

An Immunization is mapped to a ObservationStatement.

| Mapped to (JSON FHIR Immunization field)                 | Mapped from (XML HL7 / other source)                                                                          |
|----------------------------------------------------------|---------------------------------------------------------------------------------------------------------------|
| id                                                       | `ObservationStatement / id / root]`                                                                           |
| meta / profile                                           | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Immunization-1"`                 |
| extension / url                                          | fixed value = `https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-VaccinationProcedure-1` |
| extension / valueExtension / url                         | fixed value = `https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid`                       |
| extension / valueExtension / valueCodeableConcept / text | `ObservationStatement / code / displayname`                                                                   |
| extension / url[1]                                       | fixed value = `https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-DateRecorded-1`         |
| extension / valueString                                  | `ObservationStatement / effectiveTime / highValue`                                                            |
| identifier / system                                      | `ObservationStatement / code / displayname`                                                                   |
| identifier / value                                       | `ObservationStatement / id`                                                                                   |
| status                                                   | `ObservationStatement / statusCode`                                                                           |
| notGiven                                                 | boolean value = `if passed through then set to true, otherwise it's false`                                    |
| patient / reference                                      | reference to the mapped [patient](../patient/README.md)                                                       |
| encounter / reference                                    | reference to the associated [encounter] ((../encounter/README.md))                                            |
| date                                                     | `ObservationStatement / effectiveTime`                                                                        |
| primarySource                                            | boolean value = `if passed through then set to true, otherwise it's false`                                    |
| practitioner / actor / reference                         | `ObservationStatement / Participant / typeCode / agentRef / id / root`                                        |
| note / text                                              | `ObservationStatement / pertiinentInformation / pertiantAnnotation / text`                                    |
| note / text[1]                                           | `ObservationStatement / availabletime`                                                                        |


<details><summary>Example JSON</summary>

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
                 "valueExtension": {
                     "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                     "valueCodeableConcept": {
                         "text": "Haemophilus influenzae type B and meningitis C vaccination"
                     }
                 }
             },
             {
                 "url": "https://fhir.hl7.org.uk/STU3/StructureDefinition/Extension-CareConnect-DateRecorded-1",
                 "valueString": "20100113151332"
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
                 "actor": {
                     "reference": "Practitioner/9C1610C2-5E48-4ED5-882B-5A4A172AFA35"
                 }
             }
         ],
         "note": [
             {
                 "text": "Primary Source: true Location: EMIS Test Practice Location Manufacturer:\n                                                                    another company Batch: past2003 Expiration: 2003-01-17 Site: Right arm GMS : Not\n                                                                    GMS\n                                                                "
             },
             {
                 "text": "End Date: 2010-01-18T11:41:00+00:00"
             }
         ]
     }
 },
```
</details>


## XML HL7 > JSON FHIR

An Immunization is primarily mapped from an Observation Statement.

| Mapped to (XML HL7)                                | Mapped from (JSON FHIR / other source )              |
|----------------------------------------------------|------------------------------------------------------|
| id / root                                          | `id`                                                 |
| code / display                                     | `extension.valueExtension.valueCodeableConcept.text` |
| Participant / typeCode / agentRef / id /root       | `practitioner.actor.reference`                       |
| statusCode                                         | `status`                                             |
| effectiveTime                                      | `date`                                               |
| availableTime                                      | `date`                                               |
| pertiinentInformation / pertiantAnnotation / text  | `note.text`                                          |
| Participant / typecode / agentRef / id / root      | `practitioner.actor.reference`                       |


<details><summary>Example XML</summary>

```XML
<ObservationStatement classCode="OBS" moodCode="EVN" xmlns="urn:hl7-org:v3">
    <id root="immunization-id"/>
    <code code="33879002" codeSystem="2.16.840.1.113883.2.1.3.2.3.15"
          displayName="Haemophilus influenzae type B and meningitis C vaccination">
    </code>
    <Participant typeCode="PRF" contextControlCode="OP">
        <agentRef classCode="AGNT">
            <id root="9C1610C2-5E48-4ED5-882B-5A4A172AFA35"/>
        </agentRef>
    </Participant>
    <statusCode code="COMPLETE"/>
    <effectiveTime>
        <high value="20100118114100"/>
    </effectiveTime>
    <availabilityTime value="20100118114100"/>
    <pertinentInformation typeCode="PERT">
        <sequenceNumber value="+1"/>
        <pertinentAnnotation classCode="OBS" moodCode="EVN">
            <text>Primary Source: true Location: EMIS Test Practice Location Manufacturer:
                another company Batch: past2003 Expiration: 2003-01-17 Site: Right arm GMS : Not
                GMS
            </text>
        </pertinentAnnotation>
    </pertinentInformation>
    <Participant typeCode="PRF" contextControlCode="OP">
        <agentRef classCode="AGNT">
            <id root="9A5D5A78-1F63-434C-9637-1D7E7843341B"/>
        </agentRef>
    </Participant>
</ObservationStatement>
```

</details>

## Further documentation

[Immunistion History - FIHR API](https://digital.nhs.uk/developer/api-catalogue/immunisation-history-fhir)
[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 