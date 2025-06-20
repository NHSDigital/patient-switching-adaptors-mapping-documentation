# Referral Request Mapping

## XML HL7 > JSON FHIR

A Referral Request is mapped from a `RequestStatement`

| Mapped to (JSON FHIR Referral Request field) | Mapped from (XML HL7 / other source)                                                                                                                                                                                               |
|----------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| id                                           | `RequestStatement / id [@root]`                                                                                                                                                                                                    |
| meta.profile\[0]                             | fixed value = `"https://fhir.nhs.uk/STU3/StructureDefinition/CareConnect-GPC-ReferralRequest-1"`                                                                                                                                   |
| meta.security                                | When `RequestStatement / confidentialityCode [@code]` or `EhrComposition / confidentialityCode [@code]` is present and has a value of `NOPAT`. See [Confidentiality Codes](../confidentiality code/README.md) for mapping details. |
| identifier                                   | `"https://PSSAdaptor/{{losingOdsCode}}"` - where the `{{losingOdsCode}}` is the ODS code of the losing practice                                                                                                                    |
| status                                       | fixed value = `unknown`                                                                                                                                                                                                            |
| intent                                       | fixed value = `order`                                                                                                                                                                                                              |
| priority                                     | `RequestStatement / priorityCode [@code]` and mapped to string values of `Routine`, `Urgent`, or `ASAP`. If not present then it is omitted.                                                                                        |
| subject                                      | reference to the mapped [Patient](../patient/README.md)                                                                                                                                                                            |
| context                                      | reference to the mapped [Encounter](../encounters/README.md)                                                                                                                                                                       |
| authoredOn                                   | `RequestStatement / availabilityTime [@value]`                                                                                                                                                                                     |
| requester.agent                              | reference to mapped [Practitioner](../practitioners/README.md) from `RequestStatement / Participent / agentRef` or `Composition / Participent / agentRef`                                                                          |
| recipient                                    | reference to mapped [Practitioner](../practitioners/README.md) or [Organization](../organizations/README.md) from `RequestStatement / responsibleParty / agentRef`                                                                 |
| reasonCode                                   | mapped CodeableConcept from `RequestStatement / code` <sup>1</sup> as described in the XML > FHIR section of [Codeable Concept](../codeable%20concept/README.md)                                                                   |
| note                                         | mapped Annotation from `RequestStatement / text` & `RequestStatement / priorityCode` & `RequestStatement / priorityCode`                                                                                                           |
| supportingInfo                               | one or more `DocumentReferences` mapped a `LinkSet / component / statementRef / id [@root]` where `LinkSet / conditionNamed / namedStatementRef / id [@root]` references `RequestStatement / id [@root]`<sup>2</sup>               |

1.  If the SNOMED code is not found then a `Transfer-degraded referral` code is inserted instead (96431000000106)
2.  Only populated when the `RequestStatement` is referenced by a `LinkSet` which is a linkage between a 
    `ReferralRequest` and one or more `DocumentReferences`.

The following Allergy Intolerance fields are not currently populated by the adaptor:
- basedOn
- serviceRequested
- requester.onBehalfOf
- speciality
- description


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
    "supportingInfo": [
        {
            "reference": "DocumentReference/BFBF038A-F142-4C67-B05B-D155E2C89990"
        },
        {
            "reference": "DocumentReference/6DC83A17-4DFD-4C1C-A452-45F8F8A8FBA1"
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

A ReferralRequest is mapped to a RequestStatement

| Mapped to (XML HL7 RequestStatement)                                                                                         | Mapped from (JSON FHIR / other source )                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|:-----------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `RequestStatement / id [@root]`                                                                                              | Fetched from resource ID or, if not valid UUID, generated by the Adaptor                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `RequestStatement / code [@code] [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15']`                                             | `ReferralRequest.reasonCode` <sup>1</sup>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |
| `RequestStatement / confidentialityCode`                                                                                     | `ReferralRequest.meta.security[@code]` is present and has a value of `NOPAT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `RequestStatement / text`                                                                                                    | `ReferralRequest.description` & `ReferralRequest.note` & `ReferralRequest.speciality` & `referralRequest.serviceRequested` & `referralRequest.supportingInfo`.  <br/>`referralRequest.reasonCode` or `referralRequest.Recipient.name` when more than one of each.  <br/>`referralRequest.Priority` when `referralRequest.priority` is `ASAP`.  <br/>`referralRequest.identifier.value` when `referralRequest.identifer` contains a `system` of `https://fhir.nhs.uk/Id/ubr-number` or `https://fhir.nhs.uk/Id/cross-care-setting-identifier` <br/><br/>when `referralRequest.requester` is present will include details from this type <sup>2</sup> |
| `RequestStatement / statusCode [@code]`                                                                                      | fixed value = `Complete`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `RequestStatement / effectiveTime`                                                                                           | fixed value = `<center nullFlavor="NI">`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            |
| `RequestStatement / availabilityTime`                                                                                        | `ReferralRequest.authoredOn`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
| `RequestStatement / priorityCode [@code='394848005'] [@displayName='Normal'] [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15']` | `ReferralRequest.priority` when `priority` = `ROUTINE`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              |
| `RequestStatement / priorityCode [@code='394849002'] [@displayName='High'] [@codeSystem='2.16.840.1.113883.2.1.3.2.4.15']`   | `ReferralRequest.priority` when `priority` = `URGENT`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `RequestStatement / responsibleParty [@typeCode="RESP"] / agentRef [@classCode="AGNT"] / id [@root]`                         | `ReferralRequest.recipient.agentref.id`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
| `RequestStatement / Participant [@typeCode='AGNT'] / agentRef `                                                              | `ReferralRequest.requester.agent.reference` & `ReferralRequest.onBehalfOf`  when agent is `practictioner` and `onBehalfOf` is reference to [Organization](../organizations/README.md)                                                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `RequestStatement / Participant [@typeCode='AGNT'] / agentRef / id [@root]`                                                  | `ReferralRequest.requester.agent.reference` when agent is `practictioner`                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           |

1. When `referralRequest.reasonCode` is missing then a default SNOMED code of `357005` (patient referral) is added
2. if referencing agent is:
* `Device` then includes `Requester Device:` + device type
* `Organization` then includes `Requester Org:` + organization name
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
        <confidentialityCode code="NOPAT" 
            codeSystem="2.16.840.1.113883.4.642.3.47" 
            displayName="no disclosure to patient, family or caregivers without attending provider's authorization" />
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

- [GP Connect ReferralRequest](https://developer.nhs.uk/apis/gpconnect-1-6-0/accessrecord_structured_development_referralrequest.html)
- [MIM 4.2.00](https://data.developer.nhs.uk/dms/mim/4.2.00/Index.htm)