# Encounter Mapping

## XML HL7 > JSON FHIR

An GP Connect FHIR `Encounter` is mapped from an GP2GP HL7v3 `EHR Composition`.

| Mapped to (JSON FHIR Encounter field)                        | Mapped from (XML HL7 / other source)                                                                                                                                                        |
|--------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                                           | `ehrComposition / id \[@root]`                                                                                                                                                              |
| meta.profile\[0]                                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Encounter-1"`                                                                                                  |
| identifier\[0].system                                        | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                             |
| identifier\[0].value                                         | `ehrComposition / id \[@root]`                                                                                                                                                              |
| status                                                       | fixed value = `"finished"`                                                                                                                                                                  |
| type\[0]                                                     | `ehrComposition / code [@code]` or `ehrCompostion / code / translation [@code]` <sup>1</sup> as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)  |
| subject                                                      | reference to the mapped [Patient](../patient/README.md)                                                                                                                                     |
| participant\[index].type\[0].coding\[0].system <sup>2</sup>  | fixed value = `"https://fhir.nhs.uk/STU3/CodeSystem/GPConnect-ParticipantType-1"`                                                                                                           |
| participant\[index].type\[0].coding\[0].code <sup>2</sup>    | fixed value = `"REC"`                                                                                                                                                                       |
| participant\[index].type\[0].coding\[0].display <sup>2</sup> | fixed value = `"recorder"`                                                                                                                                                                  |
| participant\[index].individual <sup>2</sup>                  | `ehrComposition / author / agentRef / id [@root]`                                                                                                                                           |
| participant\[index].type\[0].coding\[0].system               | fixed value = `"http://hl7.org/fhir/v3/ParticipationType"`                                                                                                                                  |
| participant\[index].type\[0].coding\[0].code                 | fixed value = `"PPRF"`                                                                                                                                                                      |
| participant\[index].type\[0].coding\[0].display              | fixed value = `"primary performer"`                                                                                                                                                         |
| participant\[index].individual                               | `ehrComposition / participant2[0] / AgentRef / id [@root] `                                                                                                                                 |
| period.start                                                 | `ehrComposition / effectiveTime / center` or else  `ehrComposition / effectiveTime / low` or else `ehrComposition / availibiltyTime`                                                        |
| period.end                                                   | `ehrComposition / effectiveTime / high`                                                                                                                                                     |           
| location                                                     | the associated [location](../locations/README.md) identified by `ehrComposition / location`                                                                                                 |

<details>
    <summary>Example JSON</summary>

```
 {
    "resource": {
        "resourceType": "Encounter",
        "id": "9FB8560B-A7FF-4F04-9E0B-CFBB4D0AF4E9",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-Encounter-1"
            ]
        },
        "identifier": [
            {
                "system": "https://PSSAdaptor/D5445",
                "value": "9FB8560B-A7FF-4F04-9E0B-CFBB4D0AF4E9"
            }
        ],
        "status": "finished",
        "type": [
            {
                "coding": [
                    {
                        "system": "http://snomed.info/sct",
                        "code": "24561000000109",
                        "display": "A+E report"
                    }
                ],
                "text": "GP Surgery"
            }
        ],
        "subject": {
            "reference": "Patient/cacf81fd-cb4c-49de-af29-d6968f4de978"
        },
        "participant": [
            {
                "type": [
                    {
                        "coding": [
                            {
                                "system": "https://fhir.nhs.uk/STU3/CodeSystem/GPConnect-ParticipantType-1",
                                "code": "REC",
                                "display": "recorder"
                            }
                        ]
                    }
                ],
                "individual": {
                    "reference": "Practitioner/2E86E940-9011-11EC-B1E5-0800200C9A66"
                }
            },
            {
                "type": [
                    {
                        "coding": [
                            {
                                "system": "http://hl7.org/fhir/v3/ParticipationType",
                                "code": "PPRF",
                                "display": "primary performer"
                            }
                        ]
                    }
                ],
                "individual": {
                    "reference": "Practitioner/70555A33-0550-405D-BB67-E9805440B38C"
                }
            }
        ],
        "period": {
            "start": "2010-01-13T15:20:00+00:00",
            "end": "2010-01-13T15:20:00+00:00"
        },
        "location": [
            {
                "location": {
                    "reference": "Location/5E54EFE1-70E8-433D-AB36-F62EC443E5C2"
                }
            }
        ]
    }
}

```
</details>

1. If a SNOMED CT code cannot be found `type[0].coding` will not be populated.
2. Where the participant is the Practitioner that recorded the consultation on the system, identified by `ehrComposition / author`.  

### Unmapped fields

- length
- serviceProvider

## JSON FHIR > XML HL7
An GP Connect FHIR `Encounter` is mapped to a GP2GP HL7v3 `ehrComposition`.  

| Mapped to (XML HL7 ehrComposition child element) | Mapped from (JSON FHIR / other source )                                                                                   |
|--------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------|
| id \[@root]                                      | A random UUID generated by the Adaptor                                                                                    |
| code \[@code]                                    | `Encounter.type[0].coding[0].code` <sup>3</sup>                                                                           |
| code \[@displayName]                             | `Encounter.type[0].coding[0].display` <sup>3</sup>                                                                        |
| code \[@codeSystem]                              | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                          |
| code / originalText                              | `Encounter.type[0].text` or else `Encounter.type[0].coding[0].display` <sup>4</sup>                                       |
| statusCode                                       | fixed value =`"COMPLETE"`                                                                                                 |
| effectiveTime                                    | `Enounter.period.start` and `Encounter.period.end` (if present)                                                           |
| availabilityTime \[@value]                       | `Encounter.period.start`                                                                                                  |
| author / agentRef / id \[@root]                  | `Encounter.participant[index].individual` where `Encounter.participant[index].type` contains a `coding.code` of `"REC"`   | 
| author / time \[@value]                          | `List.date` <sup>5</sup>                                                                                                  |
| location / locatedEntity / code \[@code]         | fixed value = `"394730007"`                                                                                               |
| location / locatedEntity / code \[@codeSystem]   | fixed value = `"2.16.840.1.113883.2.1.3.2.4.15"`                                                                          |
| location / locatedEntity / code \[@displayName]  | fixed value = `"Healthcare related organisation"`                                                                         |
| location / locatedEntity / locatedPlace / name   | The `name` field of the [Location](../locations/README.md) resource referenced by `Encounter.location`                    |
| Participant2 / AgentRef / id \[@root]            | `Encounter.participant[index].individual` where `Encounter.participant[index].type` contains a `coding.code` of `"PPRF"`  | 


<details>
    <summary>Example XML</summary>

```
<ehrComposition classCode=\"COMPOSITION\" moodCode=\"EVN\">
    <id root=\"4BBABD06-93E2-4E87-9345-9B1171AC576F\" />
    <code code=\"24591000000103\" displayName=\"Other report\" codeSystem=\"2.16.840.1.113883.2.1.3.2.4.15\">
        <originalText>Surgery Consultation</originalText>
    </code>
    <statusCode code=\"COMPLETE\" />
    <effectiveTime>
        <low value=\"20190328103000\"/><high value=\"20190328103800\"/>
    </effectiveTime>
    <availabilityTime value=\"20190328103000\"/>
    <author typeCode=\"AUT\" contextControlCode=\"OP\">
        <time value=\"20190328103000\" />
        <agentRef classCode=\"AGNT\">
            <id root=\"4ED3292E-EC9E-400D-84D2-758CCDEA40A4\" />
        </agentRef>
    </author>
    <location typeCode="LOC">
        <locatedEntity classCode="LOCE">
            <code code="394730007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Healthcare related organisation" />
            <locatedPlace classCode="PLC" determinerCode="INSTANCE">
                <name>Example location</name>
            </locatedPlace>
        </locatedEntity>
    </location>
    <Participant2 typeCode=\"PRF\" contextControlCode=\"OP\">
        <agentRef classCode=\"AGNT\">
            <id root=\"4ED3292E-EC9E-400D-84D2-758CCDEA40A4\"/>
        </agentRef>
    </Participant2>
    <component typeCode=\"COMP\">

    ...

    </component>
</ehrComposition>
```
</details>

3. If the code is a SNOMED code within the [EHR Composition Name Vocabulary](https://data.developer.nhs.uk/dms/mim/6.3.01/Vocabulary/EhrCompositionName.htm)
then that code and display name is used. Otherwise, the SNOMED code `24591000000103` and the display name `Other report` are inserted by the adaptor.
4. `Encounter.type[0].coding[0].display` is only used if the adaptor inserts `Other report`, as described in footnote 3.
5. Where the List is the consultation [List](../list/README.md) resource that references the Encounter.

## Further documentation

- [GP Connect Encounter](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_encounter.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)
- [EHR Composition Name Vocabulary](https://data.developer.nhs.uk/dms/mim/6.3.01/Vocabulary/EhrCompositionName.htm)
