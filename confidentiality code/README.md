# Confidentiality Codes

## XML HL7 > JSON FHIR

Each FHIR resource has a list of HL7v3 elements - the adaptor will check for the presence of a `confidentialityCode` element. 

If a `confidentialityCode` element is found, and contains the value `NOPAT` then the `meta.security` field **SHALL** be added
otherwise the `meta.security` field **SHALL NOT** be added. 

```xml
<confidentialityCode 
        code="NOPAT" 
        codeSystem="2.16.840.1.113883.4.642.3.47" 
        displayName="no disclosure to patient, family or caregivers without attending provider's authorization" />
```

```json
{
  "meta": {
    "security": [
      {
        "system": "http://hl7.org/fhir/v3/ActCode",
        "code": "NOPAT",
        "display": "no disclosure to patient, family or caregivers without attending provider's authorization"
      }
    ]
  }
}
```

