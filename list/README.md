# Referral Request Mapping

## XML HL7 > JSON FHIR

A List is mapped from an [Encounter](../encounters/README.md)

| Mapped to (JSON FHIR Referral Request field) | Mapped from (XML HL7 / other source)                                                                                                                                                                |
|----------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| meta.profile\[0]                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-List-1"`                                                                                                               |
| extension /                                  | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                     |
| status                                       | fixed value = `unknown`                                                                                                                                                                             |
| intent                                       | fixed value = `order`                                                                                                                                                                               |
| priority                                     | `RequestStatement / priorityCode [@code]` or `CompoundStatement / priorityCode [@code]` and mapped to string values of `routine`, `urgent`, or `stat`. If not present in either location is omitted |
| subject                                      | reference to the mapped [Patient](../patient/README.md)                                                                                                                                             |
| context                                      | reference to the mapped [Encounter](../encounters/README.md)                                                                                                                                        |
| authoredOn                                   | `RequestStatement / availabilityTime [@value]`                                                                                                                                                      |
| requester.agent                              | reference to mapped [Practitioner](../practioners/README.md) from `RequestStatement / Participent / agentRef` or `Composition / Participent / agentRef`                                             |
| recipient                                    | reference to mapped [Practitioner](../practioners/README.md) from `RequestStatement / responsibleParty / agentRef`                                                                                  |
| reasonCode                                   | mapped CodeableConcept from `RequestStatement / code` <sup>1</sup>                                                                                                                                  |
| note                                         | mapped Annotation from `RequestStatement / text` & `RequestStatement / priorityCode` & `RequestStatement / priorityCode`                                                                            |

1.  If the SNOMED code is not found then a `Transfer-degraded referral` code is inserted instead (96431000000106)

The following Allergy Intolerance fields are not currently populated by the adaptor:
- basedOn
- serviceRequested
- requester.onBehalfOf
- speciality
- description
- supportingInfo


<details>
    <summary>Example JSON</summary>

```
{
    "resourceType": "ReferralRequest",
    "id": "referral-request-id",
    "meta": {
        "profile": [
            "https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ReferralRequest-1"
        ]
    },
    "identifier": [
        {
            "system": "https://PSSAdaptor/2167888433",
            "value": "referral-request-id"
        }
    ],
    "status": "unknown",
    "intent": "order",
    "priority": "routine",
    "subject": {
        "reference": "Patient/180b44bf-31d8-407b-b8ca-994a3f4a226c"
    },
    "context": {
        "reference": "Encounter/2485BC20-90B4-11EC-B1E5-0800200C9A66"
    },
    "authoredOn": "2010-01-01T12:30:00+00:00",
    "requester": {
        "agent": {
            "reference": "Practitioner/58341512-03F3-4C8E-B41C-A8FCA3886BBB"
        }
    },
    "recipient": [
        {
            "reference": "Practitioner/B8CA3710-4D1C-11E3-9E6B-010000001205"
        }
    ],
    "reasonCode": [
        {
            "coding": [
                {
                    "extension": [
                        {
                            "url": "https://fhir.nhs.uk/STU3/StructureDefinition/Extension-coding-sctdescid",
                            "extension": [
                                {
                                    "url": "descriptionDisplay",
                                    "valueString": "Reason Code 1"
                                }
                            ]
                        }
                    ],
                    "system": "http://snomed.info/sct",
                    "code": "183885007",
                    "display": "Private referral to obstetrician"
                }
            ],
            "text": "Reason Code 1"
        }
    ],
    "note": [
        {
            "text": "Priority: Routine"
        },
        {
            "text": "Action Date: 2005-04-06"
        },
        {
            "text": "Test request statement text\n                                                            New line\n                                                        "
        }
    ]
}
```
</details>

## JSON FHIR > XML HL7

An Allergy Intolerance is mapped to a RequestStatement

| Mapped to (XML HL7)                                                                                                          | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
|------------------------------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `RequestStatement / id [@root]`                                                                                              | new system generated UUID                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `RequestStatement / code [@code] [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15']`                                                                                           | `ReferralRequest.reasonCode` <sup>1</sup>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| `RequestStatement / text`                                                                                                    | `ReferralRequest.description` & `ReferralRequest.note` & `ReferralRequest.speciality` & `referralRequest.serviceRequested` & `referralRequest.supportingInfo`.  <br/>`referralRequest.reasonCode` or `referralRequest.Recipient.name` when more than one of each.  <br/>`referralRequest.Priority` when `referralRequest.priority` is `ASAP`.  <br/>`referralRequest.identifier.value` when `referralRequest.identifer` contains a `system` of `https://fhir.nhs.uk/Id/ubr-number`  <br/><br/>when `referralRequest.requester` is present will include details from this type <sup>2</sup> |
| `RequestStatement / statusCode [@code]`                                                                                      | fixed value = `Complete`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `RequestStatement / effectiveTime`                                                                                           | fixed value = `<center nullFlavor="NI">`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `RequestStatement / availabilityTime`                                                                                        | `ReferralRequest.authoredOn`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `RequestStatement / priorityCode [@code='394848005'] [@displayName='Normal'] [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15']` | `ReferralRequest.priority` when `priority` = `ROUTINE`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| `RequestStatement / priorityCode [@code='394849002'] [@displayName='High'] [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15']`   | `ReferralRequest.priority` when `priority` = `URGENT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| `RequestStatement / responsibleParty [@typeCode="RESP"] / agentRef [@classCode="AGNT"] / id [@root]`                         | `ReferralRequest.recipient.agentref.id`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| `RequestStatement / Participant [@typeCode='AGNT'] / agentRef `                                                              | `ReferralRequest.requester.agent.reference` & `ReferralRequest.onBehalfOf`  when agent is `practictioner` and `onBehalfOf` is reference to [Organisation](../organisations/README.md)                                                                                                                                                                                                                                                                                                                                                                                                                                  |
| `RequestStatement / Participant [@typeCode='AGNT'] / agentRef / id [@root]`                                                  | `ReferralRequest.requester.agent.reference` when agent is `practictioner`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

1. When `referralRequest.reasonCode` is missing then a default SNOMED code of `357005` (patient referral) is added
2. if referencing agent is:
* `Device` then includes `Requester Device:` + device type
* `Organisation` then includes `Requester Org:` + organisation name
* `Patient` then includes `Requester: Patient`
* `RelatedPerson` then includes `Requester: Relation` + name


<details><summary>Example XML</summary>

```
<component typeCode="COMP">
    <RequestStatement classCode="OBS" moodCode="RQO">
        <id root="B4303C92-4D1C-11E3-A2DD-010000000161" />
        <code code="8HV6." codeSystem="2.16.840.1.113883.2.1.3.2.4.14" displayName="Reason Code">
            <translation code="183885007" codeSystem="2.16.840.1.113883.2.1.3.2.4.15"
                displayName="Reason Code 1" />
            <translation code="8HV6.00" codeSystem="2.16.840.1.113883.2.1.6.2"
                displayName="Reason Code 2" />
        </code>
        <text>Test request statement text</text>
        <statusCode code="COMPLETE" />
        <effectiveTime>
            <center value="20050406" nullFlavor="NI" />
        </effectiveTime>
        <availabilityTime value="20100101123000" />
        <priorityCode code="394848005" displayName="Normal"
            codeSystem="2.16.840.1.113883.2.1.3.2.4.15">
            <originalText>Routine</originalText>
        </priorityCode>
        <responsibleParty typeCode="RESP">
            <agentRef classCode="AGNT">
                <id root="B8CA3710-4D1C-11E3-9E6B-010000001205" />
            </agentRef>
        </responsibleParty>
        <Participant typeCode="PPRF" contextControlCode="OP">
            <agentRef classCode="AGNT">
                <id root="58341512-03F3-4C8E-B41C-A8FCA3886BBB" />
            </agentRef>
        </Participant>
    </RequestStatement>
</component>
```
</details>

## Further documentation
[GP Connect Referral Request](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_referralrequest.html)

[MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm) 