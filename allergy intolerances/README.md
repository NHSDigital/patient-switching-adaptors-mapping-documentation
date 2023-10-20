# Allergy Intolerance Mapping

## XML HL7 > JSON FHIR

An `AllergyIntolerance` is mapped from a `CompoundStatement` with an `ObservationStatement` component as a child. Where 
the `compoundStatement` has a code of `SN53.00` or `14L..00` from Read Codes version 2 (`2.16.840.1.113883.2.1.6.2`).   

| Mapped to (JSON FHIR Allergy Intolerance field) | Mapped from (XML HL7 / other source)                                                                                                                                                                 |
|-------------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                              | `ObservationStatement / id [@root] `                                                                                                                                                                 |
| meta.profile\[0]                                | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-AllergyIntolerance-1"`                                                                                                  |
| extension[0].url                                | fixed value = `http://hl7.org/fhir/StructureDefinition/encounter-associatedEncounter`                                                                                                                |
| extension[0].valueReference.reference           | reference to the associated [Encounter](../encounters/README.md)                                                                                                                                     |
| identifier\[0].system                           | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                      |
| identifier\[0].value                            | `ObservationStatement / id [@root]`                                                                                                                                                                  |
| clinicalStatus                                  | fixed value = `active`                                                                                                                                                                               |
| verificationStatus                              | fixed value = `unconfirmed`                                                                                                                                                                          |
| category\[0]                                    | fixed value = `medication` if `CompoundStatement / code [@code] == 14L..00` otherwise fixed value = `environment`                                                                                    |
| code                                            | Mapped from `ObservationStatement / value` or `ObservationStatement / code` <sup>1</sup> as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md). <sup>2</sup> |
| patient                                         | reference to the mapped [Patient](../patient/README.md)                                                                                                                                              |
| onsetDateTime                                   | `CompoundStatement / effectiveTime / low [@value]`                                                                                                                                                   |
| assertedDate                                    | `CompoundStatement / availabilityTime [@value]`                                                                                                                                                      |
| recorder                                        | reference to the mapped [Practitioner](../practitioners/README.md)                                                                                                                                   |
| asserter                                        | Mapped from 'ObservationStatement / Author' field otherwise (if 'Author' value is not provided) take it from Participant field, reference to the mapped [Practitioner](../practitioners/README.md)   |
| note\[0].text                                   | `ObservationStatement / pertinentInformation / pertinentAnnotation / text`                                                                                                                           |

1. Where a valid SNOMED code isn't provided, a value of Transfer-degraded non-drug allergy (196471000000108), 
or Transfer-degraded drug allergy (196461000000101) is inserted by the adaptor instead.
2. `code.text` will be set from the `ObservationStatement / value [@displayName]` when this is present. 

### Unmapped fields

The following Allergy Intolerance fields are not currently populated by the adaptor:
- type
- criticality
- context.related
- lastOccurrence
- reaction


<details>
    <summary>Example JSON</summary>

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
            "reference": "Practitioner/9F2ABD26-1682-FDFE-1E88-19673307C67A"
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
</details>

## JSON FHIR > XML HL7

An Allergy Intolerance is mapped to a CompoundStatement with inner ObservationStatement.

| Mapped to (XML HL7)                                                          | Mapped from (JSON FHIR / other source )                                                                      |
|------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------|
| CompoundStatement / id \[@root]                                              | CompoundStatement ID (UUID) generated by the Adaptor                                                         |
| CompoundStatement / code \[@code]                                            | fixed value = `SN53.00` if `AllergyIntolerance.category[0] == environment` otherwise fixed value = `14L..00` |
| CompoundStatement / code \[@codeSystem]                                      | fixed value = `2.16.840.1.113883.2.1.6.2`                                                                    |
| CompoundStatement / effectiveTime                                            | `AllergyIntolerance.onsetDateTime`                                                                           |
| CompoundStatement / availabilityTime                                         | `AllergyIntolerance.assertedDate`                                                                            |
| ObservationStatement / id \[@root]                                           | ObservationStatement ID (UUID) generated by the Adaptor                                                      |
| ObservationStatement / code                                                  | `AllergyIntolerance.code`                                                                                    |
| ObservationStatement / effectiveTime                                         | `AllergyIntolerance.onsetDateTime`                                                                           |
| ObservationStatement / availabilityTime                                      | `AllergyIntolerance.assertedDate`                                                                            |
| ObservationStatement / pertinentInformation / pertinentAnnotation / text     | `AllergyIntolerance.note\[0].text`                                                                           |
| ObservationStatement / Participant \[@typeCode=AUT] / agentRef / id \[@root] | `AllergyIntolerance.recorder`                                                                                |
| ObservationStatement / Participant \[@typeCode=PRF] / agentRef / id \[@root] | `AllergyIntolerance.asserter` or `AllergyIntolerance.recorder`                                               |            

<details><summary>Example XML</summary>

```
<component typeCode="COMP">
	<CompoundStatement classCode="CATEGORY" moodCode="EVN">
		<id root="437D0657-7E09-4E9E-9320-49F160C19E67"/>
		<code code="14L..00" codeSystem="2.16.840.1.113883.2.1.6.2" displayName="H/O: drug allergy"/>
		<statusCode code="COMPLETE"/>
		<effectiveTime>
			<center value="20100630"/>
		</effectiveTime>
		<availabilityTime value="20100630" />
		<component typeCode="COMP" contextConductionInd="true">
			<ObservationStatement classCode="OBS" moodCode="ENV">
				<id root="966804FE-5DE8-46C4-9EA5-CEB0EBFAAD81"/>
				<code code="811091000006112" codeSystem="2.16.840.1.113883.2.1.3.2.4.15" displayName="Allergy to penicillin">
					<originalText>Allergy to penicillin</originalText>
				</code>
				<statusCode code="COMPLETE"/>
				<effectiveTime>
					<center value="20100630"/>
				</effectiveTime>
				<availabilityTime value="20100630" />
				<pertinentInformation typeCode="PERT">
					<sequenceNumber value="+1"/>
					<pertinentAnnotation classCode="OBS" moodCode="EVN">
						<text>Status: Active This is a note</text>
					</pertinentAnnotation>
				</pertinentInformation>
				<author typeCode="AUT" contextControlCode="OP">
					<time value="20231004123014" />
					<agentRef classCode="AGNT">
						<id root="E7E7B550-09EF-BE85-C20F-34598014166C" />
					</agentRef>
				</author>
				<Participant typeCode="AUT" contextControlCode="OP">
					<agentRef classCode="AGNT">
						<id root="0BA5C685-D2AA-4E82-8857-484CC3B2CCD8"/>
					</agentRef>
				</Participant>
				<Participant typeCode="PRF" contextControlCode="OP">
					<agentRef classCode="AGNT">
						<id root="0BA5C685-D2AA-4E82-8857-484CC3B2CCD8"/>
					</agentRef>
				</Participant>
			</ObservationStatement>
		</component>
	</CompoundStatement>
</component>
```
</details>

## Further documentation

- [GP Connect Allergy Intolerance](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_allergyintolerance.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 
