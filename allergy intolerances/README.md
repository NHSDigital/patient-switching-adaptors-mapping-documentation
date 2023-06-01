# Allergy Intolerance Mapping

## Example JSON
```
{
    "resource": {
        "resourceType": "AllergyIntolerance",
        "id": "allergy-observation-id",
        "meta": {
            "profile": [
                "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-AllergyIntolerance-1"
            ]
        },
        "extension": [
            {
                "url": "http://hl7.org/fhir/StructureDefinition/encounter-associatedEncounter",
                "valueReference": {
                    "reference": "Encounter/2485BC20-90B4-11EC-B1E5-0800200C9A66"
                }
            }
        ],
        "identifier": [
            {
                "system": "https://PSSAdaptor/2167888433",
                "value": "allergy-observation-id"
            }
        ],
        "clinicalStatus": "active",
        "verificationStatus": "unconfirmed",
        "category": [
            "medication"
        ],
        "code": {
            "coding": [
                {
                    "extension": [
                        {
                            "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                            "extension": [
                                {
                                    "url": "descriptionId",
                                    "valueId": "1488801013"
                                },
                                {
                                    "url": "descriptionDisplay",
                                    "valueString": "H/O: aspirin allergy"
                                }
                            ]
                        }
                    ],
                    "system": "http://snomed.info/sct",
                    "code": "395102008",
                    "display": "H/O: aspirin allergy"
                }
            ],
            "text": "H/O: aspirin allergy"
        },
        "patient": {
            "reference": "Patient/180b44bf-31d8-407b-b8ca-994a3f4a226c"
        },
        "onsetDateTime": "2010-01-13",
        "assertedDate": "2010-01-13",
        "recorder": {
            "reference": "Practitioner/3707E1F0-9011-11EC-B1E5-0800200C9A66"
        },
        "asserter": {
            "reference": "Practitioner/3707E1F0-9011-11EC-B1E5-0800200C9A66"
        },
        "note": [
            {
                "text": "Drug Allergy - Apsrin"
            }
        ]
    }
},
```

## Example XML
```
<component typeCode="COMP" contextConductionInd="true">
    <ObservationStatement classCode="OBS" moodCode="EVN">
        <id root="C6FAF730-ECA9-460E-94D3-1F0602B537DC"/>
        <code code="14LK.00" codeSystem="2.16.840.1.113883.2.1.6.2"
                displayName="H/O: aspirin allergy">
            <qualifier inverted="false">
                <name code="255217005" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                        displayName="First"/>
            </qualifier>
            <translation code="395102008" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                            displayName="H/O: aspirin allergy"/>
        </code>
        <statusCode code="COMPLETE"/>
        <effectiveTime>
            <center nullFlavor="NI"/>
        </effectiveTime>
        <availabilityTime value="20100113"/>
        <value type="CD" code="ALLERGY-READ2-14LK" codeSystem="2.16.840.1.113883.2.1.6.3"
                displayName="H/O: aspirin allergy"/>
        <pertinentInformation typeCode="PERT">
            <sequenceNumber value="+1"/>
            <pertinentAnnotation classCode="OBS" moodCode="EVN">
                <text>Drug Allergy - Apsrin</text>
            </pertinentAnnotation>
        </pertinentInformation>
    </ObservationStatement>
</component>
```

## XML HL7 > JSON FHIR

An Allergy Intolerance is primarily mapped from an Observation Statement.

| Mapped to (JSON FHIR Allergy Intolerance field) | Mapped from (XML HL7 / other source)                                                                  |
|---------------------------------------|-----------------------------------------------------------------------------------------------------------------|
| id                                    | `ObservationStatement / id [@root `                                                                             |
| meta.profile\[0]                      | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-AllergyIntolerance-1"`             |
| extension[0].url                      | fixed value = `http://hl7.org/fhir/StructureDefinition/encounter-associatedEncounter`                           |
| extension[0].valueReference.reference | reference to the associated [Encounter](../encounters/README.md)                                                |
| identifier\[0].system                 | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice |
| identifier\[0].value                  | `ObservationStatement / id [@root]`                                                                             |
| clinicalStatus                        | fixed value = `active`                                                                                          |
| verificationStatus                    | fixed value = `unconfirmed`                                                                                     |
| category[0]                           | `CompoundStatement / code [@code]`                                                                              |
| code                                  | Mapped from `ObservationStatement / value` or `ObservationStatement / code`                                     |
| patient                               | reference to the mapped [Patient](../patient/README.md)                                                         |
| onsetDateTime                         | `CompoundStatement / effectiveTime / low [@value]`                                                              |
| assertedDate                          | `CompoundStatement / availabilityTime [@value]`                                                                 |
| recorder                              | reference to the mapped [Practitioner](../practioners/README.md)                                                |
| asserter                              | reference to the mapped [Practitioner](../practioners/README.md)                                                |
| note\[0].text                         | `ObservationStatement / pertinentInformation / pertinentAnnotation / text`                                      |

### Unmapped fields

The following Allergy Intolerance fields are not currently populated by the adaptor:
- type
- criticality
- context.related
- lastOccurrence
- reaction

## JSON FHIR > XML HL7

TODO.

## Further documentation
[GP Connect Allergy Intolerance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_allergyintolerance.html)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
