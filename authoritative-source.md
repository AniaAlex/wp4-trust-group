# Verifying the authoritative source of an attestation

**Status:** gap note for discussion — revised 2026-09-18

## Summary

The mechanism for binding an issuer to an authentic source **already exists and is complete**
for public-sector attestations. It does **not** exist for QEAA or non-qualified EAA. The gap is
therefore much narrower than first stated, and lands specifically on private-sector issuers.

| Branch | Authentic-source binding | Verifiable today |
|---|---|---|
| Pub-EAA (PSBEAA) | `QcPSB` qcStatement in the signing certificate + Commission Art. 45f(3) list | **Yes** |
| QEAA | none | No |
| Non-qualified EAA | none | No |
| PID | separate regime (Commission PID provider list) | Status only |

The Catalogue of Attributes and Catalogue of Schemes (TS11) close part of the remaining gap: they
carry the legal basis of an attribute as an ELI URI, register authentic sources per Member State,
and let a scheme require that issuers chain to a named trust anchor. They do not carry the identity
of an authorised issuer. See Finding 5.

## The question

A verifier receives an attestation. The signature verifies, the issuer is registered, its access
certificate is valid, and it appears on the relevant trusted list. None of that tells the verifier
whether this issuer is the **right party** to assert this particular claim. Two examples raised in
the group: which organisation should issue a Finnish VAT-ID, and how to check that an IBAN
attestation came from the bank responsible for that account.

The framework answers **authenticity** (is this entity real, registered, operating lawfully) well.
The question is whether it answers **authoritativeness** (is this entity the source of record for
this attribute).

## Finding 1 — for public sector bodies, it is already solved

ETSI TS 119 412-6 clause 8.3 requires the PSBEAA provider's certificate to carry a `QcPSB`
qcStatement:

- **PSB-8.3-02:** "shall contain the identification for the law under which the PSBEAA is
  established responsible for the authentic source"
- **PSB-8.3-03:** "shall contain an unambiguous identification for the authentic source"

Annex A defines it normatively:

```
QcPSB ::= SEQUENCE {
  countryOfLegislation      PrintableString (SIZE (2)),  -- ISO 3166 alpha-2, or 'EU'
  authSourceIdentification  UTF8String,                  -- the authentic source
  legislationIdentification UTF8String                   -- the law
}
```

TS 119 602 Annex H reinforces this at list level: a Pub-EAA provider's `TETradeName` must carry
the reference to the Union or national law establishing it as responsible for the authentic
source, formatted as a URI (`OJ:` + `EU`/country code + law identifier), and its service carries
`http://uri.etsi.org/19602/SvcType/PubEAA/Issuance` with status `notified` or `withdrawn`.

So a verifier receiving a Pub-EAA attestation can already establish, from the certificate and the
list alone, which authentic source the issuer is responsible for and under which law.

**Consequence for the VAT-ID example:** if the Finnish Tax Administration is notified as a PSBEAA
provider and the VAT-ID is issued as a Pub-EAA, the question is answerable today. The problem is
not missing infrastructure; it is whether this route is actually used for such attributes.

## Finding 2 — no equivalent exists outside the public sector

`QcPSB` and `authSourceIdentification` appear in **only one specification** — TS 119 412-6.
There is no counterpart in the QEAA or non-qualified EAA certificate profiles, and TS 119 411-8
does not mention authentic sources or attribute authorities at all.

A QTSP issuing a QEAA is verifiable as a qualified provider and nothing more. Its certificate
carries no statement of what it is authoritative for. **The IBAN case sits here:** a bank is not a
public sector body responsible for an authentic source, so no `QcPSB` applies.

## Finding 3 — registration verifies entitlement, not authority

CIR (EU) 2025/848 does mandate verification, but of role rather than of authority:

- **Article 6(3):** the registrar shall verify, in an automated manner where possible, "the
  accuracy, validity, authenticity and integrity of the information required under Article 5", the
  type of entitlement(s), and the absence of an existing registration in another national register.
- **Article 6(4):** verification against supporting documentation or "appropriate authentic sources
  or other official electronic records".
- **Annex III** routes entitlement evidence to external sources: Article 22 national trusted lists
  for qualified providers; the Commission's PID provider list; and the Commission's Article 45f(3)
  list for public sector attestation issuers.
- **Article 5 / Annex I point 9:** the relying party self-declares its intended data requests.

The registrar therefore checks entitlements against authoritative external lists, not against
self-declarations. What it does not appear to do is validate that a declared attestation type is
one the party is *entitled to assert*. Verifying the accuracy of a declaration of intent is not the
same as establishing authority: an entity can accurately declare that it intends to issue VAT-ID
attestations without being the authority for them.

> **Primary-source caveat:** EUR-Lex could not be retrieved in preparing this note; the article
> numbers and wording above come from a secondary summary and must be confirmed against the
> official text before this note is used externally.

## Finding 4 — TS 119 475 already anticipates the distinction

`providesAttestations` is defined as "sub-entitlements of the WRP" (Annex I.13), present only for
PID/QEAA/Non-Q EAA/Pub-EAA entitlements. GEN-5.2.4-05 says the WRPRC **should** include the
registry-provided fields of Table 8 (`format`, `meta`, `claim`), with `claim` "present only if
provided by the registry". NOTE 2 states:

> "Use of Credential subfields in the provides_attestations is used for both self-declared
> attestations and ones that are referenceable in an attestation catalogue. If once the attestation
> provider has their provided credential/s listed on the Catalogue of Attestations of the EU the
> meta subfield points to URL of the credential's machine-readable scheme in the catalogue."

Two observations. First, the spec itself distinguishes self-declared from catalogue-referenced
attestations — so the weakness is acknowledged, not overlooked. Second, the asymmetry with the
relying side is stark: Table 9 requires (**shall**) the registry fields for service providers and
states the wallet uses them "to perform an over-asking validation". There is an active enforcement
mechanism constraining what relying parties may *request*, and only a **should** on the issuing
side, with no counterpart enforcement on what issuers may *assert*.

## Finding 5 — the catalogues (TS11) answer part of it

EUDI TS11, *Interfaces and formats for Catalogue of Attributes and Catalogue of Schemes*, supplies
three pieces that the earlier version of this note treated as missing.

**`legalBasis`** on an Attribute entry: "information with respect to the EU or national level law
that acts as the legal basis of the attribute. Value SHOULD be an ELI URI pointing to the EU or
national regulation text." ELI is therefore the controlled vocabulary for the law reference —
residual gap 2 is half closed. Note it is a **SHOULD**, so the field may be absent or free-form.

**`authenticSources`** on an Attribute entry: an array of `DataService` objects — "Each Member
State providing the attribute through an authentic source(s) shall register their national
evidence/attribute verification services" — with `country`, `nationalSubID`, `endpointDescription`
and `endpointURL` ("for either verifying or retrieving data from the service").

This is the decisive detail. The catalogue models the authentic source as a **service to query**,
not as an **authority whose signature to expect**. It tells a party where to go and verify an
attribute at source; it does not tell a verifier who was entitled to attest it. Those are different
roles, and the VAT-ID question needs the second.

**`trustedAuthorities`** on a Schemes entry (`SchemaMeta`): "an optional array of objects that
resolve to the applicable trust management scheme(s) (trust model) or trust anchor(s) to be used
according to the options available in OpenID4VP", with `frameworkType` (`aki`, `etsi_tl`,
`openid_federation`), `value` and `isLoTE`. This scopes issuers to a **trust anchor set** and is
enforced by the verifier at presentation through OpenID4VP. It constrains which list an issuer must
appear on, not which entity it must be.

`SchemaMeta` also carries `rulebookURI`, pointing at the human-readable Attestation Rulebook — so
the catalogue already has the hook that Recommendation 1 needs.

**No field in either catalogue carries an organisation identifier, legal person identity or the
name of a specific authorised issuer.** `contactInfo` holds contact URIs for the entity that
requested the attribute's inclusion, which is not the same thing.

## The residual gap

1. **Private-sector issuers have no authentic-source binding at all.** QEAA and non-qualified EAA
   carry nothing equivalent to `QcPSB`.
2. **`authSourceIdentification` is an unconstrained `UTF8String`.** ELI now supplies a controlled
   form for the *law* (TS11 `legalBasis`), but not for the *source*. A verifier reading a `QcPSB`
   still has no vocabulary to match `authSourceIdentification` against.
3. **Nothing maps an attestation type to an expected issuer identity.** The catalogues record where
   an attribute can be verified (`authenticSources`) and which trust anchors apply
   (`trustedAuthorities`), but not who is authorised to attest it. A verifier can therefore check
   that an issuer is on the right list, never that it is the right body.

## Action 2 result — TS 119 612 / TS 119 602 feasibility

**Feasible in both, with TS 119 602 the better target.**

TS 119 612 clause 5.5.9 provides Service information extensions, and clause 5.5.9.4
`additionalServiceInformation` is the established pattern for narrowing a service type with
further URIs — exactly how qualified-certificate services are already distinguished into
signature, seal and website authentication. Scoping a service entry by attestation type would
reuse that pattern without structural change.

TS 119 602 is the better target: it is the EUDI-specific List of Trusted Entities format, it
already carries the authentic-source legal reference for Pub-EAA providers, and it already has
attestation-oriented service type URIs. Extending the QEAA branch there to carry an equivalent of
`QcPSB` — a country, a law reference and an authentic-source identifier — would close the gap with
a mechanism the ecosystem has already accepted for the public-sector branch.

## Recommendation 1 — rulebooks should declare the expected authoritative issuer

Attestation rulebooks should state, for each attestation type, the expected authoritative source
and the basis on which a party may be considered authoritative for it. Without this there is
nothing for a registrar to check a `providesAttestations` entry against, and nothing for a verifier
to compare an `authSourceIdentification` to.

This is the cheapest of the available levers and needs no change to certificate profiles, trusted
list formats or registration procedures. It also supplies the controlled vocabulary that residual
gaps 2 and 3 require. Where the authoritative source is a public sector body, the rulebook should
point to the Pub-EAA route rather than inventing a parallel mechanism.

## Recommendation 2 — treat instance-level authority as a separate problem

The two examples are not the same problem and should not be bundled.

| | Finnish VAT-ID | IBAN |
|---|---|---|
| Authority determined by | attestation type | attribute value |
| Authoritative party | a fixed, nameable body | the bank identified within the IBAN |
| Expressible in a certificate or list? | yes | no |
| Status | solvable today via Pub-EAA | no mechanism at any layer |

Type-level authority can be published in a certificate, a trusted list or a rulebook. Instance-level
authority cannot: the authoritative party differs per attestation, so it requires resolution against
a sectoral registry at verification time. Folding it into the same recommendation would weaken the
type-level case, which is close to complete.

## Open items

1. Confirm the CIR (EU) 2025/848 article numbers and wording against the official EUR-Lex text.
2. ~~Establish whether the Catalogue of Attestations records an expected authoritative issuer.~~
   **Answered (Finding 5):** it does not. It records `authenticSources` as verification endpoints
   and `trustedAuthorities` as trust anchors. Follow-up: confirm whether `trustedAuthorities`
   scoped to a narrow LoTE is considered an acceptable way to express authoritativeness (Annex A.7).
3. Establish whether Member States are in practice routing public-register attributes (VAT, company
   registration, address) through the Pub-EAA route, or issuing them as ordinary QEAA and so
   bypassing the `QcPSB` binding. If the latter, the gap is one of practice rather than of
   specification.

## Sources

Local copies under `references/etsi/`:

- `ETSI_TS_119_412-6_V1.1.1.md` clause 8.3 and Annex A — `QcPSB`, `authSourceIdentification`
- `ETSI_TS_119_475.md` GEN-5.2.4-05, Table 8 and NOTE 2, Annex I.13 — `providesAttestations`
- `ts_119602v010101p.md` Annex H — Pub-EAA `TETradeName` law reference, `SvcType/PubEAA/Issuance`
- `ts_119612v020401p.md` clauses 5.5.9, 5.5.9.4 — Service information extensions
- `ETSI_TS_119_411-8_V1.1.1.md` — no authentic-source provision (negative result)

CIR (EU) 2025/848 — secondary summary only, pending primary confirmation (open item 1).

---

# Annex A — proposed shape of the rulebook record

Field names below are a proposal. The **value forms** are not: each reuses an identifier syntax
that already exists, so that a registrar and a verifier can compare rulebook values directly
against fields they already hold.

| Rulebook field | Reuses | Compared against |
|---|---|---|
| `countryOfLegislation` | ISO 3166-1 alpha-2, or `EU` | `QcPSB.countryOfLegislation` (TS 119 412-6 Annex A) |
| `legalBasis` | ELI URI (TS11 Attribute `legalBasis`); `OJ:` URI form in TS 119 602 Annex H | `QcPSB.legislationIdentification` |
| `authSourceIdentification` | — *(unconstrained today; see residual gap 2)* | `QcPSB.authSourceIdentification` |
| `issuer[].organizationIdentifier` | EN 319 412-1 clause 5 semantics (`NTRFI-…`, `VATFI-…`, `LEIXG-…`) | `organizationIdentifier` in the issuer certificate subject (EN 319 412-3 LEG-4.2.1-6) |
| `vct` / `doctype` + `format` | SD-JWT VC / ISO 18013-5 | `provides_attestations.format` (TS 119 475 Table 8) |
| `catalogue` | Catalogue of Attestations scheme URL | `provides_attestations.meta` (TS 119 475 NOTE 2) |

## A.1 Authority models

Every attestation type declares exactly one model. This is what separates the type-level case from
the instance-level case in the record itself, rather than in prose.

- **`designated`** — a fixed, nameable body is authoritative for the whole attestation type.
- **`instance-resolved`** — the authoritative party depends on the attribute value; the rulebook
  states how to derive it and what class the issuer must belong to.
- **`open`** — no authority constraint beyond the entitlement; any entitled provider may issue.

## A.2 Worked example — designated (Finnish VAT-ID)

```yaml
attestation:
  vct: "urn:eu.europa.ec.eudi:vat:fi:1"
  format: "dc+sd-jwt"
  catalogue: "https://ec.europa.eu/eudi/catalogue/schemes/vat-fi-1.json"

authority:
  model: designated
  route: PUB_EAA_Provider           # required entitlement (TS 119 475 A.2)
  countryOfLegislation: "FI"
  legalBasis: "http://data.europa.eu/eli/fi/<establishing act>"   # TS11 form, ELI
  authSourceIdentification: "<expected QcPSB value for the Finnish VAT register>"
  authSourceName: "Finnish VAT register"
  issuer:
    - organizationName: "Verohallinto"
      organizationIdentifier: "NTRFI-<registration number>"
```

## A.3 Worked example — instance-resolved (IBAN)

```yaml
attestation:
  vct: "urn:eu.europa.ec.eudi:iban:1"
  format: "dc+sd-jwt"
  catalogue: "https://ec.europa.eu/eudi/catalogue/schemes/iban-1.json"

authority:
  model: instance-resolved
  route: QEAA_Provider
  resolution:
    from: "iban"                    # the claim carrying the value
    method: "iban-institution-id"   # derive the institution from the value itself
    note: "Positions 5-8 of the IBAN identify the institution within the national scheme."
  qualifyingClass:
    description: "Credit institution authorised in the EEA that maintains the account identified by the IBAN."
    register: "https://euclid.eba.europa.eu/register/"
```

Note what the record does **not** claim here: it does not name an authoritative issuer, because
there is not one per type. It names the derivation and the register, and leaves the resolution to
verification time. This is the honest encoding of Recommendation 2.

## A.4 What a registrar checks

At registration, against each declared `providesAttestations` entry:

1. Resolve the rulebook from the declared `format` + `meta`.
2. `model: designated` — the applicant's `organizationIdentifier` must match an entry in
   `authority.issuer[]`, **and** its entitlements must include `authority.route`. Otherwise refuse
   the entry, or record it as unverified rather than accepting it silently.
3. `model: instance-resolved` — the applicant must be verifiable as a member of
   `qualifyingClass` via the named register.
4. `model: open` — entitlement check only, as today.

This is the concrete answer to "what must a registrar verify before accepting a
`providesAttestations` entry": steps 2 and 3, which today have nothing to check against.

## A.5 What a verifier checks

At presentation, after ordinary signature and trusted-list validation:

1. Resolve the rulebook from the credential's `vct` / `doctype`.
2. `designated` **via Pub-EAA** — compare `QcPSB.authSourceIdentification` and
   `QcPSB.countryOfLegislation` from the issuer certificate against the rulebook values.
   *This path works today.*
3. `designated` **via QEAA** — no `QcPSB` exists, so instead compare the `organizationIdentifier`
   in the issuer certificate subject against `authority.issuer[]`. **This is weaker** — it
   establishes identity, not legal mandate — but it is available now and needs no change to any
   certificate profile.
4. `instance-resolved` — derive the expected institution from the attribute value per
   `authority.resolution`, then compare against the issuer's identity.
5. `open` — no further check.

## A.6 Why this is deployable before any specification changes

Step A.5.3 is the significant one. Every legal-person certificate already carries
`organizationIdentifier` (EN 319 412-3 LEG-4.2.1-6), and its semantics are already structured
(EN 319 412-1 clause 5). A rulebook that names expected issuers by `organizationIdentifier`
therefore gives verifiers something concrete to check **today**, for QEAA as well as Pub-EAA,
without waiting for the ETSI change requested in the escalation draft.

That does not make the ETSI request unnecessary: comparing an organisation identifier proves only
that the expected organisation signed, not that it holds a legal mandate for the attribute, and it
requires every verifier to fetch and trust the rulebook. It does mean the rulebook recommendation
can proceed independently of, and ahead of, the specification work.

> **Open point for the group:** A.2 leaves `authSourceIdentification` as a placeholder because no
> controlled vocabulary exists for it (residual gap 2). The law reference is solved — use the ELI
> form that TS11 `legalBasis` already specifies. Until the source identifier is solved, `designated`
> rulebooks should carry `issuer[]` as the operative field and treat `authSourceIdentification` as
> advisory.

## A.7 The alternative: scope the scheme with `trustedAuthorities`

TS11 `SchemaMeta.trustedAuthorities` resolves to the trust anchor(s) a credential of that scheme
must chain to, and is enforced by the verifier through OpenID4VP. It is worth weighing against the
`issuer[]` approach in A.5.3, because it needs no new field anywhere and the enforcement path
already exists.

It expresses authoritativeness **only if the referenced list is narrow enough**. Pointing a Finnish
VAT-ID scheme at the general Finnish trusted list says nothing useful. Pointing it at a LoTE that
contains only the bodies authoritative for that attribute says exactly the right thing.

| | `trustedAuthorities` → narrow LoTE | `issuer[]` → `organizationIdentifier` |
|---|---|---|
| New specification needed | none | none |
| Enforcement | verifier, via OpenID4VP | verifier, by comparing the certificate subject |
| Who maintains it | the Member State, as list operator | the rulebook author |
| Cost of a change of issuer | reissue the list | revise the rulebook |
| Expresses legal mandate | no — list membership only | no — identity only |
| Works for `instance-resolved` | no | no |

Neither proves a legal mandate; both are proxies. The `trustedAuthorities` route is the stronger of
the two where a Member State is willing to operate an attribute-scoped list, because enforcement is
automatic and the list is maintained by the party that knows when authority changes hands. The
`issuer[]` route is available even where no such list exists.

**This is a question for the group:** is attribute-scoped LoTE maintenance realistic for Member
States, or does it multiply lists beyond what operators will carry? The answer determines which of
the two we recommend, and it is a question about practice rather than specification.

---

# Annex B — what "attribute-scoped" would actually mean in a LoTE

Annex A.7 asked whether attribute-scoped list maintenance is realistic. That question was
underspecified: there are two ways to scope, with very different operational cost, and the choice
interacts with how TS11 `trustedAuthorities` works.

## B.1 The two variants

**Variant A — scope the list.** One LoTE per attestation type. `LoTEType` is narrowed to the
attribute, and the list contains only the bodies authoritative for it.

**Variant B — scope the service.** Keep one LoTE per provider category, as today, and tag each
`TrustedEntityService` with the attestation types that entity is authoritative for, using the
`ServiceInformationExtensions` element (TS 119 602 clause 6.6.9).

"Maintenance" in variant A means, per attribute: a scheme operator, a signing key and its
lifecycle, a publication endpoint, `LoTESequenceNumber` discipline, a `NextUpdate` cadence to
honour, and a pointer entry in the LoTL. Twenty attributes means twenty of each. Variant B adds no
lists, no keys and no endpoints — it adds elements inside lists that already exist and are already
signed and published.

## B.2 Variant B in the current schema

`ServiceInformationExtensions` is present on `TrustedEntityService` in `1960201.xsd` (line 353),
typed `ExtensionsListType`, `minOccurs="0"`. Each `Extension` is `AnyType` with a **required**
`Critical` boolean. The extension namespace
`http://uri.etsi.org/019602/v1/ServiceInformationExtensions` already exists and currently defines a
single element, `ServiceUniqueIdentifier` — so this is a designed extension point with room in it.

None of the LoTEs currently generated under `publish-trusted-list/` emit any extension.

**What is standard and what is proposed.** The container (`ServiceInformationExtensions`, clause
6.6.9), the `Extension` wrapper and its required `Critical` attribute are defined by TS 119 602.
The whole namespace currently defines exactly one extension, `ServiceUniqueIdentifier`
(clause 6.6.9.1). **`AuthoritativeFor` and every element inside it below are a proposal of this
note and exist in no specification.** Defining them is the substance of the ETSI request.

Because `Extension` derives from `AnyType` — `mixed="true"` with
`<xsd:any processContents="lax"/>`, unbounded — arbitrary well-formed XML in any namespace is
accepted, and validated only where a schema for that namespace is available. A private extension
in a WE BUILD namespace therefore validates against the official XSD today, with no ETSI change.

Proposed element, shown inside a service entry in the format the v3 generator already produces:

```xml
<TrustedEntityService>
  <ServiceInformation>
    <ServiceTypeIdentifier>http://uri.etsi.org/19602/SvcType/PubEAA/Issuance</ServiceTypeIdentifier>
    <ServiceName>
      <Name lang="en">Finnish VAT Register Attestation Service</Name>
    </ServiceName>
    <ServiceDigitalIdentity>
      <DigitalId><X509Certificate>...</X509Certificate></DigitalId>
    </ServiceDigitalIdentity>
    <ServiceStatus>http://uri.etsi.org/19602/PubEAAProvidersList/SvcStatus/notified</ServiceStatus>
    <StatusStartingTime>2026-09-18T00:00:00Z</StatusStartingTime>

    <ServiceInformationExtensions>
      <Extension Critical="true">
        <AuthoritativeFor xmlns="http://uri.etsi.org/019602/v1/ServiceInformationExtensions">
          <AttestationType>
            <TypeIdentifier>urn:eu.europa.ec.eudi:vat:fi:1</TypeIdentifier>
            <Format>dc+sd-jwt</Format>
            <CatalogueURI>https://ec.europa.eu/eudi/catalogue/schemes/vat-fi-1.json</CatalogueURI>
            <LegalBasis>http://data.europa.eu/eli/fi/[act]</LegalBasis>
            <AuthenticSourceName xml:lang="en">Finnish VAT register</AuthenticSourceName>
          </AttestationType>
        </AuthoritativeFor>
      </Extension>
    </ServiceInformationExtensions>
  </ServiceInformation>
</TrustedEntityService>
```

`TypeIdentifier`, `Format` and `CatalogueURI` are the join keys: they match
`provides_attestations.format` / `.meta` in the registry (TS 119 475 Table 8) and the `vct` or
`doctype` a verifier reads off the credential. `LegalBasis` uses the ELI form TS11 already
specifies.

**`Critical` is the substantive choice, and it depends on the stage.**

In the standardised end state it must be `true`: a verifier that cannot process the extension must
not fall back to treating the entity as generally authoritative. Set `false` there and the
mechanism is advisory only — a legacy verifier ignores it and accepts any attestation from any
listed provider.

While the extension is private and its namespace unregistered, it must be `false`. A conforming
verifier encountering `Critical="true"` in a namespace it does not know is required to reject that
service entry, so a prototype published with `Critical="true"` would break consumers of the list
for every attestation, not just the scoped ones. Flip it to `true` only once the namespace is
registered and verifiers can process it.

JSON equivalent, for the `lote.json` distribution:

```json
"serviceInformationExtensions": [
  { "critical": true,
    "authoritativeFor": [
      { "typeIdentifier": "urn:eu.europa.ec.eudi:vat:fi:1",
        "format": "dc+sd-jwt",
        "catalogueURI": "https://ec.europa.eu/eudi/catalogue/schemes/vat-fi-1.json",
        "legalBasis": "http://data.europa.eu/eli/fi/[act]",
        "authenticSourceName": "Finnish VAT register" } ] } ]
```

## B.3 The catch — `trustedAuthorities` points at lists, not services

TS11 `trustedAuthorities` resolves to a trust anchor or list, and OpenID4VP enforces it at that
granularity. It can therefore express variant A directly: *credentials of this scheme must chain to
this list*, where the list contains only authoritative bodies.

It cannot express variant B on its own. Under variant B a verifier must fetch the LoTE, locate the
service entry matching the issuer's certificate, and read the extension — a step OpenID4VP does not
perform for it. Variant B is cheaper to operate and more expensive to consume.

| | Variant A — scope the list | Variant B — scope the service |
|---|---|---|
| New lists, keys, endpoints per attribute | yes | none |
| Expressible via TS11 `trustedAuthorities` | yes, directly | no — verifier must read the list |
| Enforced automatically by OpenID4VP | yes | no |
| Specification change needed | none | one extension element |
| Scales to many attributes | poorly | well |
| Revocation of authority | reissue that list | reissue the category list |

## B.4 Recommendation

Variant B, prototyped first as a private extension with `Critical="false"` — which needs no
specification change and no permission — and standardised afterwards, with variant A reserved for
the small number of high-value attributes where automatic OpenID4VP enforcement justifies a
dedicated list. This keeps list count proportional to provider
categories rather than to attributes, and confines the specification change to a single extension
element in a namespace that already exists.

The open question for the group is no longer "is attribute-scoped maintenance realistic" — variant
B makes it cheap. It is **whether verifiers will do the extra fetch-and-inspect step that variant B
requires**, given OpenID4VP will not do it for them. If the answer is no, the cost moves back to
list operators and variant A becomes the only workable option.

---

# Annex C — using a documented extension instead

Annex B proposed a new extension (`AuthoritativeFor`). That was premature. TS 119 602 already
defines an extension whose semantics fit, and using it reduces the ask from "define a new
extension" to "define URI values in a profile" — which the specification explicitly anticipates.

Worked mock: `examples/lote-authoritative-mock.xml`.

## C.1 The extension that fits

**`OtherAssociatedBodies`**, TS 119 602 clause 6.5.5.1, in the Trusted Entity Extensions namespace.

> "The OtherAssociatedBodies component specifies information about bodies different from the TE
> identified through the TEName component that are associated with the identified TE in a way that
> is meaningful in the context of the LoTE scheme and with respect to the listed services."
> — clause 6.5.5.1.0

> "Specific profiles making use of this extension shall define the requirements applying to the
> bodies listed through this extension."

This is exactly the Pub-EAA relationship: an attestation provider issuing *by or on behalf of* a
public sector body responsible for an authentic source (ETSI TS 119 412-6 defines the role in those
words). `AssociatedBody` carries:

| Component | Clause | Carries |
|---|---|---|
| `AssociatedBodyName` | 6.5.5.1.2 | the formal legal name of the authentic source body |
| `AssociatedBodyTradeName` | 6.5.5.1.3 | "an official registration identifier as registered in official records … that unambiguously identifies the body" |
| `AssociatedBodyInformationURI` | 6.5.5.1.5 | where to read about it |
| `AssociatedBodyTypeIdentifier` | 6.5.5.1.6 | **a URI stating the nature of the association** |
| `AssociatedBodyInformationExtensions` | 6.5.5.1.7 | open, format "left open" |

`AssociatedBodyTradeName` supplies the unambiguous organisational identifier that Annex A.5.3
needed, without inventing a field.

## C.2 What is actually missing — a profile, not an extension

Clause 6.5.5.1.6 specifies the type identifier as "an indicator expressed as a URI" and states:

> "LoTE profiles making use of this extension shall specify, when they require the usage of this
> component, the set of URI values and their semantics that can be used for the
> AssociatedBodyTypeIdentifier component."

So the specification hands the definition of these URIs to profiles by design. The EUDI LoTE
profile needs to define one value — proposed in the mock as
`http://uri.etsi.org/19602/AssociatedBodyType/AuthenticSource`, meaning *the named body is the
authentic source on the basis of which this entity issues attestations*.

That is a profile decision, not a schema change, and it is a far smaller request than Annex B's.

## C.3 What this still does not do

`OtherAssociatedBodies` sits in `TEInformationExtensions`, at **trusted entity** level, not service
level. It therefore says "this entity is associated with authentic source X" for the entity as a
whole. It cannot vary per attestation type within one entity.

Where an entity is authoritative for several attributes from different sources, the current
mechanisms give only partial separation: one `TrustedEntityService` per attestation type, each with
its own `ServiceUniqueIdentifier` (clause 6.6.9.1) and its own `ServiceName`. But
`ServiceUniqueIdentifier` is an opaque identifier — nothing binds a service entry to a `vct` or
`doctype` a verifier can match. That binding remains genuinely undefined, and it is the one thing
Annex B's proposal still adds.

**Revised recommendation:** ask for the profile URI value first (C.2), which closes the
public-sector case using documented mechanisms only. Keep the service-level attestation-type
binding as a separate, smaller follow-on request.

## C.4 Schema defects found while building the mock

`1960201_xsd_schema_tie.xsd` does not match clause 6.5.5.1 of the specification it binds. Four
discrepancies, worth reporting to ETSI:

| # | Specification (v1.1.1, 2025-11) | XSD |
|---|---|---|
| 1 | `AssociatedBodyTypeIdentifier` is a component of `AssociatedBody` (6.5.5.1.6) | **element absent entirely** |
| 2 | `AssociatedBodyAddress` is optional ("may optionally contain") | mandatory — no `minOccurs="0"` |
| 3 | `AssociatedBodyInformationURI` is optional | mandatory — no `minOccurs="0"` |
| 4 | `OtherAssociatedBodies` "shall be a sequence of AssociatedBody elements" | `maxOccurs` defaults to 1 — only one body permitted |

Defect 1 is blocking: the type identifier cannot be expressed in schema-valid XML today, so the
mock will not validate against the published XSD. Defects 2 and 3 force an address and information
URI onto every associated body even where the specification does not. Defect 4 prevents listing
more than one associated body, which the plural element name and the specification text both
contemplate.

Reporting these is worth doing independently of the authoritative-source question, and doing so
establishes standing for the profile request in C.2.

---

# Annex D — why Annex C does not help with IBAN, and what would

Annex C closes the public-sector case. It does nothing for IBAN, for two independent reasons.
Neither is about the shape of `OtherAssociatedBodies`.

## D.1 Reason one — a bank is not in that list, and there is no list for it

`EUPubEAAProvidersList` holds "public sector bodies issuing electronic attestation of attribute,
which are notified by Member States". A bank issuing an IBAN attestation is not a public sector body
responsible for an authentic source, so it does not appear there and carries no `QcPSB`.

It would be listed instead as a QTSP on an Article 22 national trusted list. Those are ETSI
TS 119 612 trusted lists, not TS 119 602 LoTEs — and **Annex H registers no QEAA providers LoTE
type at all**. The only EAA-related registered type is the public-sector one.

`OtherAssociatedBodies` is a TS 119 602 Trusted Entity Extension. It does not exist in TS 119 612.
The four extensions available there — `expiredCertsRevocationInfo`, `Qualifications`,
`TakenOverBy`, `additionalServiceInformation` — carry certificate and signature properties, and
none can express an association with an authentic source.

So the Annex C mechanism is not merely unsuitable for IBAN; it is unavailable on the list where the
issuer would be found.

## D.2 Reason two — entity-level cannot answer a per-value question

Even if the extension were available, it is static and scoped to the entity. It could record "Bank X
is associated with the account register it maintains". The verification question is *is this bank
responsible for **this** IBAN?* — which differs per attestation. No list entry can answer it,
because the answer is a function of the attribute value, not of the issuer.

This is the instance-level case from Recommendation 2, and it should stay separate.

## D.3 What would work — split the check in two

The IBAN check decomposes into two steps. Only the second is a trust-infrastructure problem.

**Step A — value to institution.** Derive the institution from the IBAN and resolve it to a legal
entity. The institution identifier sits in the BBAN, whose format is national, so this needs a
register per country or an aggregated directory. Candidates are national central bank code lists,
the EPC routing directory for SEPA participants, and the commercial BIC directory. There is no
single free, authoritative, EU-wide IBAN-to-institution register, and this step is sectoral rather
than something the trust framework should absorb.

**Step B — institution to issuer identity.** Compare that legal entity against the identity in the
attestation issuer's certificate. **This is solvable now, provided both sides use a shared
identifier.**

The Legal Entity Identifier is the natural choice: it is effectively universal among EU credit
institutions, banking registers already carry it, and the ecosystem already accepts the form —
TS 119 475 uses `LEIXG-529900T8BM49AURSDO55` as a subject identifier in its registration
certificate examples, and the `LEIXG-` prefix is the EN 319 412-1 organizationIdentifier semantic
for an LEI.

## D.4 Recommendation for instance-resolved attestations

For any attestation type whose rulebook declares `model: instance-resolved` (Annex A.1):

1. **Require the issuer's LEI in the certificate.** The signing certificate's subject
   `organizationIdentifier` shall carry the `LEIXG-` form. This makes step B a string comparison and
   needs no change to any certificate profile — only a rulebook requirement.
2. **Name the register in the rulebook.** `authority.resolution.register` shall identify the
   register used for step A, per country where the format is national.
3. **State the derivation.** `authority.resolution.method` shall say how the institution identifier
   is extracted from the attribute value.

This converts "no mechanism at any layer" into a concrete, if partly manual, verification path. What
remains genuinely unsolved is step A's fragmentation: until an authoritative EU-wide
IBAN-to-institution resolution exists, verifiers depend on per-country registers of varying quality
and access terms.

That is a banking-sector data problem, not an eIDAS one, and the trust group should say so plainly
rather than propose a trust-layer mechanism that cannot reach it.

## D.5 "Sectoral resolution service" — definition

The term is this note's, not one used in any specification.

**Definition.** A service that answers: *given this attribute value, which legal entity is
authoritative for it?* — returning an identifier comparable to what appears in a certificate
subject, such as an LEI or an EN 319 412-1 organizationIdentifier.

| | |
|---|---|
| Input | attribute value + attestation type (e.g. `FI21 1234 5600 0007 85`, IBAN) |
| Output | the authoritative legal entity, as a comparable identifier |
| Operated by | the sector's competent register, not a trust service provider |
| Consumed by | the **verifier**, at **presentation** time |

It is "sectoral" because the answer lives in a domain register — banking, company, vehicle, medical
licensing — and cannot be centralised into eIDAS infrastructure.

### What already exists, and why none of it fits

**TS11 `authenticSources`** looks closest but serves a different actor at a different moment. The
group's own catalogue analysis records requirement CAT_04: "A request to include or to modify an
attribute in the catalogue of attributes SHALL indicate how a QTSP can use the verification point
for that attribute." It is a verification point **for the QTSP, before issuing**. The
authoritativeness question arises **at the verifier, after issuing**. The catalogue was never
intended to answer it.

**OOTS** (Once-Only Technical System, Regulation (EU) 2018/1724) is the nearest real machinery: it
locates and queries authentic sources for a given evidence type in a given country. The group's
catalogue note already records that Member States "remain free to implement [their] own
verification mechanisms, including the use of OOTS". But OOTS resolves *evidence type + country →
authentic source*, and is built for administrative evidence exchange, not for a relying party
checking an attestation in real time.

**BRIS** interconnects national business registers; the **EBA register** lists authorised credit
institutions. Both are genuine sectoral registers, and either could underpin a resolution service
for company identifiers or for banks respectively.

### The common shortfall

Every one of these resolves **type + country → a service**. None resolves **value → an entity
identifier**. That single missing step is what separates the IBAN case from the VAT-ID case, and it
is why no amount of trust-list or certificate-profile work reaches it.

---

# Annex E — the two verification paths end to end

Each step is marked with what it depends on:

**[TODAY]** works with what is specified and deployed · **[RULEBOOK]** needs only a rulebook record
(Annex A) · **[PROFILE]** needs the LoTE profile URI value (Annex C.2) and the XSD fix (C.4) ·
**[SECTORAL]** depends on a register outside the trust framework

## E.1 Finnish VAT-ID — designated, type-level

### Establishing the authority

1. **[TODAY]** Finland notifies the Finnish Tax Administration into `EUPubEAAProvidersList`. Per
   Annex H, its `TETradeName` carries the official registration identifier and the `OJ:` reference
   to the law establishing it as responsible for the authentic source.
2. **[TODAY]** Its service entry carries `ServiceTypeIdentifier`
   `http://uri.etsi.org/19602/SvcType/PubEAA/Issuance`, `ServiceStatus` `notified`, and the signing
   certificate in `ServiceDigitalIdentity`.
3. **[TODAY]** The signing certificate carries the `QcPSB` qcStatement (TS 119 412-6
   PSB-8.3-01…04): `countryOfLegislation` = `FI`, `authSourceIdentification` = the VAT register,
   `legislationIdentification` = the establishing act.
4. **[TODAY]** Separately, it registers as a WRP with entitlement `PUB_EAA_Provider` and
   `providesAttestations` listing the VAT-ID type. The registrar verifies the entitlement against
   the Commission's Article 45f(3) list (CIR (EU) 2025/848 Annex III).
5. **[RULEBOOK]** The attestation rulebook declares `model: designated`, the expected
   `authSourceIdentification`, the `legalBasis` as an ELI URI, and `issuer[]` with the
   organisational identifier.
6. **[PROFILE]** The list entry's `TEInformationExtensions` carries `OtherAssociatedBodies` naming
   the VAT register, with `AssociatedBodyTypeIdentifier` set to the profile-defined
   `…/AssociatedBodyType/AuthenticSource`.

### Verifying a presented attestation

7. **[TODAY]** Verifier validates the SD-JWT VC signature and obtains the signing certificate.
8. **[TODAY]** Verifier resolves the credential's `vct` to its Catalogue of Schemes entry, reading
   `rulebookURI` and `trustedAuthorities`.
9. **[TODAY]** OpenID4VP enforces that the issuer chains to a trust anchor named in
   `trustedAuthorities`.
10. **[TODAY]** Verifier reads `QcPSB` from the certificate.
11. **[RULEBOOK]** Verifier compares `QcPSB.authSourceIdentification` and `countryOfLegislation`
    against the values the rulebook declares. **This is the authoritativeness check.**
12. **[PROFILE]** Optionally, verifier fetches the LoTE, locates the entity by certificate identity
    and confirms the associated body and its type identifier.

**Status: complete.** Steps 1–4 and 7–10 work today. The only additions are a rulebook record and,
optionally, one profile URI value. **The authority statement travels with the credential**, inside
the certificate — no external lookup is required at verification time.

## E.2 IBAN — instance-resolved

### Establishing the authority

1. **[TODAY]** The bank appears as a QTSP on a Member State's Article 22 national trusted list — a
   TS 119 612 trusted list, not a LoTE.
2. **[TODAY]** It registers as a WRP with entitlement `QEAA_Provider` and `providesAttestations`
   listing the IBAN attestation type. The registrar verifies qualified status against that Article
   22 list.
3. **No authentic-source binding is recorded anywhere.** There is no `QcPSB` (it is not a public
   sector body), no `OtherAssociatedBodies` (that extension exists only in TS 119 602), and no
   registered QEAA providers LoTE type at all. Steps 1 and 2 establish that the bank is a
   legitimate qualified provider — nothing more.
4. **[RULEBOOK]** The rulebook declares `model: instance-resolved`, requires the issuer's signing
   certificate to carry its LEI in the subject `organizationIdentifier` (`LEIXG-` form), and names
   the resolution method and register.

### Verifying a presented attestation

5. **[TODAY]** Verifier validates the signature, obtains the certificate and reads
   `organizationIdentifier` from the subject.
6. **[TODAY]** Verifier resolves the `vct` to the catalogue entry and the rulebook.
7. **[TODAY]** Verifier reads the disclosed IBAN value from the credential.
8. **[SECTORAL]** Verifier derives the institution identifier from the BBAN, whose layout is
   national and differs per country.
9. **[SECTORAL]** Verifier resolves that institution identifier to a legal entity and its LEI, via
   the register the rulebook names. **This is the fragile step** — per-country registers of varying
   quality, access terms and availability, with no single free authoritative EU-wide directory.
10. **[RULEBOOK]** Verifier compares the resolved LEI against the certificate's
    `organizationIdentifier`. **This is the authoritativeness check.**

**Status: partial.** Steps 5–7 work today and step 10 becomes a string comparison once step 4 is a
rulebook requirement. Steps 8 and 9 sit outside the trust framework entirely and carry no
availability guarantee.

## E.3 The asymmetry

| | Finnish VAT-ID | IBAN |
|---|---|---|
| Where the authority statement lives | in the issuer's certificate (`QcPSB`) | nowhere |
| Lookup needed at verification time | none | two external resolutions |
| Depends on registers outside eIDAS | no | yes, per country |
| Fails if a register is unavailable | no | yes |
| Blocking work | rulebook record | rulebook record **and** sectoral resolution |

The difference is not one of degree. For the public-sector case the trust framework carries the
authority claim with the credential, so verification is self-contained. For the instance-level case
it carries nothing, and every verifier must independently reach a banking register that no part of
the framework guarantees.

Recommending the same mechanism for both would obscure this. The type-level case is close to
finished; the instance-level case needs a sectoral resolution service that does not exist, and the
trust group should say so rather than imply the gap is of the same kind.

---

# Annex F — the full VAT-ID chain, verifier to trusted list field

Every hop, with the concrete field at each end. `[P]` marks a link that needs the profile URI value
(Annex C.2); `[R]` marks one that needs the rulebook record (Annex A). Everything unmarked exists
today.

```
  PRESENTED CREDENTIAL (SD-JWT VC)
    ├── vct ──────────────────────────────┐
    ├── claims: vat_id = "FI12345678"     │
    └── JOSE header x5c ───────┐          │
                               │          │
                               │          ▼
                               │   CATALOGUE OF SCHEMES (TS11 SchemaMeta)
                               │     ├── rulebookURI ──────────────► RULEBOOK  [R]
                               │     └── trustedAuthorities[]            │
                               │           {frameworkType:"etsi_tl",     │ model: designated
                               │            value:"<LoTE id>",           │ route: PUB_EAA_Provider
                               │            isLoTE:true} ──────┐         │ countryOfLegislation: FI
                               │                               │         │ legalBasis: <ELI URI>
                               ▼                               │         │ authSourceIdentification
                        SIGNING CERTIFICATE                    │         │ issuer[].organizationIdentifier
                          ├── subject.organizationIdentifier   │         │
                          ├── subject.countryName              │         │
                          └── QcPSB qcStatement                │         │
                                ├── countryOfLegislation ──────┼─────────┤ compare  [R]
                                ├── authSourceIdentification ──┼─────────┤ compare  [R]
                                └── legislationIdentification ─┼─────────┘ compare  [R]
                                     │                         │
                                     │                         ▼
                                     │              EUPubEAAProvidersList (TS 119 602 LoTE)
                                     │                LoTEType = .../LoTEType/EUPubEAAProvidersList
                                     │                SchemeTerritory = "EU"
                                     │                        │
                                     └── match on cert ──────►│
                                                              ▼
                                            TrustedEntityService/ServiceInformation
                                              ├── ServiceDigitalIdentity/DigitalId/X509Certificate  ◄── join
                                              ├── ServiceTypeIdentifier = .../SvcType/PubEAA/Issuance
                                              ├── ServiceStatus = .../PubEAAProvidersList/SvcStatus/notified
                                              └── StatusStartingTime
                                                              │
                                                       parent entity
                                                              ▼
                                            TrustedEntity/TrustedEntityInformation
                                              ├── TEName = "Verohallinto"
                                              ├── TETradeName
                                              │     ├── official registration identifier
                                              │     └── "OJ:FI:<law>"  (Annex H requirement)
                                              └── TEInformationExtensions  [P]
                                                    └── tie:OtherAssociatedBodies
                                                          └── tie:AssociatedBody
                                                                ├── AssociatedBodyName = "FI VAT register"
                                                                ├── AssociatedBodyTradeName = <reg. identifier>
                                                                └── AssociatedBodyTypeIdentifier
                                                                      = .../AssociatedBodyType/AuthenticSource
```

## F.1 The joins, in order

| # | From | To | Join key | Status |
|---|---|---|---|---|
| 1 | credential `vct` | Catalogue of Schemes `SchemaMeta` | attestation type identifier | today |
| 2 | `SchemaMeta.rulebookURI` | rulebook record | URL | today (record is **[R]**) |
| 3 | `SchemaMeta.trustedAuthorities[].value` | the LoTE to trust | list identifier, `isLoTE:true` | today |
| 4 | credential `x5c` | `ServiceDigitalIdentity/DigitalId/X509Certificate` | certificate equality | today |
| 5 | service entry | parent `TrustedEntityInformation` | XML containment | today |
| 6 | cert `QcPSB.authSourceIdentification` | rulebook `authSourceIdentification` | string comparison | **[R]** |
| 7 | cert `subject.organizationIdentifier` | `TETradeName` / rulebook `issuer[]` | organisational identifier | **[R]** |
| 8 | cert `QcPSB.legislationIdentification` | `TETradeName` `OJ:` value / rulebook `legalBasis` | law reference | **[R]**, see F.3 |
| 9 | entity entry | the authentic source itself | `AssociatedBodyTypeIdentifier` | **[P]** |

## F.2 What each check actually establishes

- Joins 3–5 establish **status**: the issuer is a notified Pub-EAA provider in good standing. This
  is what works today and is all that works today.
- Join 6 establishes **authoritativeness**: the issuer's declared authentic source is the one the
  rulebook says it should be for this attestation type. This is the check the whole note is about.
- Join 7 establishes **identity consistency** across certificate, list and rulebook.
- Join 9 makes the authentic source explicit in the list rather than only in the certificate, so it
  can be inspected without parsing a qcStatement.

## F.3 A defect this chain exposes — two incompatible encodings of the law

Join 8 does not work as specified, because the same fact is encoded two different ways:

| Source | Field | Format |
|---|---|---|
| TS 119 602 Annex H | `TETradeName` law reference | `"OJ:"` + `EU`\|country code + law identifier |
| TS11 | Attribute `legalBasis` | **SHOULD** be an ELI URI (`http://data.europa.eu/eli/...`) |
| TS 119 412-6 | `QcPSB.legislationIdentification` | unconstrained `UTF8String` |

Three references to the same national act, in three formats, none of which is required to be
convertible to the others. A verifier cannot compare them without a mapping that nobody publishes,
and `QcPSB.legislationIdentification` being unconstrained means even two Pub-EAA providers in the
same Member State may encode the same law differently.

**Recommendation:** whichever encoding is chosen, one should be mandated across all three, with ELI
the obvious candidate since it is already an EU-wide identifier scheme with a resolver. This is a
small, concrete, high-value ask and belongs in the escalation alongside the `AssociatedBodyType`
URI request.

---

# Annex G — provenance audit of the nine joins

Every element in Annex F, marked **[SPEC]** (exists, with citation), **[PROPOSED]** (this note's
invention), or **[ASSUMED]** (taken as given without a citation found — needs confirming).

## G.1 Join by join

| # | Element | Status | Basis |
|---|---|---|---|
| 1 | `vct` on the credential | **[SPEC]** | SD-JWT VC |
| 1 | `SchemaMeta` in Catalogue of Schemes | **[SPEC]** | TS11 |
| 1 | **`vct` resolves to a `SchemaMeta` entry** | **[ASSUMED]** | TS11's `SchemaMeta` fields are `id` (UUID), `version`, `rulebookURI`, `trustedAuthorities`, `attestationLoS`, `bindingType`, `supportedFormats`, `schemaURIs`. **No `vct` field was found.** The resolution may run through `schemaURIs[].uri`, but this note did not verify it. |
| 2 | `SchemaMeta.rulebookURI` | **[SPEC]** | TS11 |
| 2 | Attestation Rulebook as a document | **[SPEC]** | ARF |
| 2 | **The record structure inside it** (`model`, `route`, `authority`, `issuer[]`, `resolution`) | **[PROPOSED]** | Annex A — invented entirely by this note |
| 3 | `trustedAuthorities[]`, `frameworkType`, `value`, `isLoTE` | **[SPEC]** | TS11: "resolve to the applicable trust management scheme(s) … or trust anchor(s) … according to the options available in OpenID4VP" |
| 3 | Using it to select the LoTE | **[SPEC]** | that is its stated purpose |
| 4 | `x5c` in the JOSE header | **[SPEC]** | JOSE |
| 4 | `ServiceDigitalIdentity/DigitalId/X509Certificate` | **[SPEC]** | TS 119 602 clause 6.6.3.1; present in `1960201.xsd` |
| 4 | Matching a signing certificate to a list entry | **[SPEC]** | standard trusted list validation; see ETSI TS 119 615 |
| 5 | Service entry to parent `TrustedEntityInformation` | **[SPEC]** | XML containment in TS 119 602 |
| 6 | `QcPSB.authSourceIdentification` | **[SPEC]** | TS 119 412-6 PSB-8.3-03, Annex A ASN.1 |
| 6 | Rulebook's expected `authSourceIdentification` | **[PROPOSED]** | Annex A |
| 6 | **Comparing the two as a verification step** | **[PROPOSED]** | no specification describes or requires this check |
| 7 | `subject.organizationIdentifier` in the certificate | **[SPEC]** | EN 319 412-3 LEG-4.2.1-6 |
| 7 | `TETradeName` carrying the official registration identifier | **[SPEC]** | TS 119 602 Annex H, Pub-EAA profile |
| 7 | Rulebook `issuer[].organizationIdentifier` | **[PROPOSED]** | Annex A |
| 7 | The three-way comparison | **[PROPOSED]** | — |
| 8 | `QcPSB.legislationIdentification` | **[SPEC]** | TS 119 412-6 |
| 8 | `TETradeName` `OJ:` law reference format | **[SPEC]** | TS 119 602 Annex H, quoted verbatim |
| 8 | TS11 `legalBasis`, ELI URI | **[SPEC]** | TS11, "SHOULD be an ELI URI" |
| 8 | **The three-way encoding incompatibility** | **[SPEC-derived]** | an observation about published specifications, not an invention — the three formats are as cited |
| 8 | Comparing them as a verification step | **[PROPOSED]** | — |
| 9 | `OtherAssociatedBodies`, `AssociatedBody`, `AssociatedBodyName`, `AssociatedBodyTradeName` | **[SPEC]** | TS 119 602 clause 6.5.5.1 |
| 9 | `AssociatedBodyTypeIdentifier` | **[SPEC]** in the text, **absent from the XSD** | clause 6.5.5.1.6; see C.4 defect 1 |
| 9 | **The URI value `…/AssociatedBodyType/AuthenticSource`** | **[PROPOSED]** | no URI values are defined; clause 6.5.5.1.6 requires profiles to define them |
| 9 | Using the extension to name an authentic source | **[PROPOSED]**, consistent with clause 6.5.5.1.0 | — |

## G.2 Supporting values used in the mock

| Value | Status |
|---|---|
| `http://uri.etsi.org/19602/LoTEType/EUPubEAAProvidersList` | **[SPEC]** — Annex H registered type |
| `http://uri.etsi.org/19602/SvcType/PubEAA/Issuance` | **[SPEC]** — Annex H |
| `http://uri.etsi.org/19602/PubEAAProvidersList/SvcStatus/notified` | **[SPEC]** — Annex H |
| `…/PubEAAProvidersList/StatusDetn/EU`, `…/schemerules/EU`, `SchemeTerritory` = `EU`, `HistoricalInformationPeriod` = `65535` | **[SPEC]** — Annex H Pub-EAA profile |
| `NTRFI-0245437-2` | **fabricated sample.** The `NTRxx-` form follows EN 319 412-1 semantics; the number itself is not a real registration identifier and must be replaced |
| `OJ:FI:vero-laki-1558-1995` | **fabricated sample.** The `OJ:` construction is per Annex H; the law identifier is invented |
| `urn:eu.europa.ec.eudi:vat:fi:1` | **fabricated sample** — no such attestation type is registered |

## G.3 Summary

**Exists and works today:** joins 3, 4, 5 — the issuer's status as a notified Pub-EAA provider.
Every field in those joins is specified and deployed.

**Exists but is unused:** the `QcPSB` fields (join 6, 8) and `OtherAssociatedBodies` (join 9). These
are specified; nothing currently reads them for this purpose.

**Does not exist:** the rulebook record (joins 2, 6, 7), the `AssociatedBodyType` URI value
(join 9), and every comparison step that constitutes the actual authoritativeness check.

**Unverified:** join 1. The note assumes a credential's `vct` resolves to a Catalogue of Schemes
entry. That resolution was not confirmed against TS11 and should be, because the whole chain starts
there — if `vct` does not resolve to `SchemaMeta`, the verifier cannot reach the rulebook or
`trustedAuthorities` at all, and joins 2, 3, 6 and 7 have no entry point.

---

# Annex H — the IBAN chain, and where it stops

Same structure as Annex F, for comparison. (References to "TS 119 602 Annex H" elsewhere in this
note mean that specification's annex, not this one.)

## H.1 How the issuer is listed

A correction to Annex D.1: QEAA providers **are** properly listed, on Article 22 national trusted
lists, with a dedicated service type. TS 119 612 v2.4.1 defines four EAA service types:

| Service type URI | Definition |
|---|---|
| `…/Svctype/EAA/Q` | "The issuance of qualified electronic attestations of attributes by a qualified trust service provider" |
| `…/Svctype/EAA` | the same, not qualified |
| `…/Svctype/EAA/Pub-EAA` | "issued by or on behalf of a public sector body **responsible for an authentic source**" |
| `…/Svctype/EAAValidation` | validation of EAAs |

The asymmetry is visible in the definitions themselves. `EAA/Pub-EAA` carries the authentic source
in its *definition*. `EAA/Q` says nothing about what is attested — its only stated requirement is
that it uses PKI.

## H.2 The chain

```
  PRESENTED IBAN ATTESTATION
    ├── vct ────────────────► Catalogue of Schemes ──► rulebookURI ──► RULEBOOK  [R]
    │                              └── trustedAuthorities[] ──┐         model: instance-resolved
    ├── claim: iban = "FI21…"                                 │         resolution.method
    └── x5c ──────────┐                                       │         resolution.register
                      │                                       │         (no issuer[] — see H.3)
                      ▼                                       ▼
            SIGNING CERTIFICATE                   ARTICLE 22 NATIONAL TRUSTED LIST
              ├── subject.organizationName          (TS 119 612 TSL — not a LoTE)
              ├── subject.organizationIdentifier          │
              └── ✗ no QcPSB                              │
                      │                                   ▼
                      └── match on cert ────► TSPService/ServiceInformation
                                                ├── ServiceTypeIdentifier = …/Svctype/EAA/Q
                                                ├── ServiceStatus = granted
                                                └── ServiceInformationExtensions
                                                      ✗ nothing that can scope by attribute
                                                            │
                                                     parent entity
                                                            ▼
                                                  TSPInformation
                                                    ├── TSPName
                                                    └── TSPTradeName
                                                    ✗ no OtherAssociatedBodies (TS 119 602 only)

  ══════════ the chain ends here; authoritativeness is not reachable ══════════

  RESOLUTION MUST COME FROM OUTSIDE THE FRAMEWORK:
    iban value ──► institution identifier in the BBAN      [SECTORAL — national format]
               ──► legal entity + LEI via a register       [SECTORAL — no EU-wide directory]
               ──► compare to subject.organizationIdentifier  [R — needs LEIXG- requirement]
```

## H.3 The four gaps, in order of severity

**Gap 1 — no authentic-source statement in the certificate.** `QcPSB` is defined only in
TS 119 412-6 for PSBEAA providers. A QEAA signing certificate carries `organizationName` and
`organizationIdentifier` and nothing about what the issuer is authoritative for. There is also no
`QcType` OID for EAA — TS 119 412-6 defines only `id-etsi-qct-pid` and `id-etsi-qct-wal`.

**Gap 2 — no attribute scoping in the list entry.** `Svctype/EAA/Q` means "issues QEAAs", full
stop. The four TS 119 612 extensions cannot narrow it: `additionalServiceInformation` has a closed
URI set covering certificate and signature properties only, `Qualifications` matches certificate
criteria, `TakenOverBy` handles succession, `expiredCertsRevocationInfo` is about revocation data.

**Gap 3 — no "on behalf of" expression.** A bank may contract a QTSP to issue attestations for it.
The signing certificate then identifies the **QTSP**, not the bank. A verifier comparing a resolved
bank LEI against that certificate gets a mismatch and cannot tell a delegation from a fraud.
Pub-EAA solves this twice over — in the `EAA/Pub-EAA` service type definition ("by or **on behalf
of**") and in `OtherAssociatedBodies`. QEAA has neither, because `OtherAssociatedBodies` is a
TS 119 602 extension and Article 22 lists are TS 119 612.

**Gap 4 — no value-to-entity resolution.** Covered in Annex D.5. Nothing resolves
`FI21 1234 5600 0007 85` to a legal entity identifier comparable to a certificate subject.

## H.4 What a verifier can actually conclude today

| Question | Answerable? |
|---|---|
| Is the signature valid? | yes |
| Is the issuer a qualified trust service provider in good standing? | yes |
| Does it issue QEAAs? | yes |
| Does it issue **IBAN** attestations specifically? | no — `providesAttestations` is self-declared registry data, not in the list or certificate |
| Is it a credit institution at all? | no |
| Does it hold **this** account? | no |
| Is it acting for someone else? | no |

A verifier can establish that a legitimate qualified provider signed the attestation. It cannot
establish any connection between that provider and the account the attestation describes.

## H.5 Ordering of fixes

Gaps 1–3 are inside the framework and fixable by the requests already in the escalation draft, plus
one addition. Gap 4 is not, and no trust-layer work reaches it.

1. **Rulebook requirement** — issuer's LEI in `organizationIdentifier` (`LEIXG-` form). No
   specification change; makes the eventual comparison a string match.
2. **An "on behalf of" expression for QEAA** (gap 3). The cheapest route is porting
   `OtherAssociatedBodies` semantics into TS 119 612, or moving QEAA providers onto a LoTE. This
   should be added to the ETSI requests; it is currently missing from the escalation draft.
3. **An authentic-source statement for QEAA certificates** (gap 1) — already request 4 in the
   escalation draft.
4. **Gap 4 stays open.** State it as a banking-sector dependency rather than proposing a trust-layer
   mechanism for it.

Until 1–3 land, IBAN attestations are verifiable as *authentic* and not as *authoritative*, and any
conformance claim about them should say so explicitly.

---

# Annex I — where "which attestations does it issue" is recorded, and who can read it

The declaration exists. The problem is the reader.

## I.1 Where it lives

`providesAttestations` — "Set of sub-entitlements of the WRP, present only if any entitlement of
the WRP is of type QEAA_Provider, Non_Q_EAA_Provider, PUB_EAA_Provider or PID_Provider"
(TS 119 475 Annex I.13) — with `format`, `meta` and `claim` subfields (Table 8).

It is held in the national register and reachable two ways:

1. **From the register directly**, via `registryURI` — `[1..1]`, "The URL for the national registry
   API of the registered WRP" (Article 3(5)).
2. **From a WRPRC**, where a Member State mandates or allows them under CIR (EU) 2025/848
   Article 8. GEN-5.2.4-05 makes the Table 8 fields a **should** for attestation providers.

It is **not** in the trusted list and **not** in the signing certificate.

## I.2 Who it was designed for — the wallet, not the verifier

TS 119 475 is explicit about the consuming actor:

> "In cases where a WRPRC is not issued, the EUDIW retrieves the relevant information from the
> national register using the data structures specified in the present document."

> "If no WRPRC is available, the EUDIW retrieves the relevant registration data directly from the
> national register … In such cases, the register acts as the authoritative source of the relying
> party's identity, purpose, and entitlements, ensuring that the wallet can still enforce attribute
> access policies and provide informed user consent even without a WRPRC."

The reader is the **EUDIW**, at **interaction** time, enforcing attribute access policies and
supporting user consent. The design is sound for what it targets.

## I.3 Why a verifier cannot use it

A verifier holding a presented attestation has the issuer's **signing certificate**. Nothing
documented maps that certificate to a register entry:

- The certificate carries `organizationName` and `organizationIdentifier`; the register is keyed by
  the WRP's registered identifier (`sub`, e.g. `LEIXG-…`). No specification requires these to match
  or states how to join them.
- `registryURI` is a field **inside** the register entry. To read it you must already have the
  entry, so it cannot bootstrap a lookup from a certificate.
- There is no discovery mechanism from a certificate, or from a `vct`, to the national register of
  the Member State where the issuer is registered.

## I.4 What this means

| Question | Recorded? | Readable by the wallet at interaction time | Readable by a verifier holding an attestation |
|---|---|---|---|
| Which attestation types does the issuer declare? | yes, `providesAttestations` | yes | **no defined path** |
| Was that declaration verified? | see Finding 3 — entitlement only | — | — |
| Is the issuer authoritative for the type? | no | no | no |

So the honest answer to "what attestations does it issue" is: **the issuer says which ones, in the
register, and the wallet can read that at interaction time. A verifier examining an attestation
afterwards has no specified route to it — and even if it had, the declaration is of intent, not of
authority.**

## I.5 Added to the asks

This is a distinct gap from those in Annex H, and also missing from the escalation draft:

> **Define how a verifier resolves an attestation's signing certificate to the issuer's register
> entry** — a discovery path from certificate identity to national register, plus a requirement
> that the certificate's `organizationIdentifier` matches the registered `sub`. Without it,
> `providesAttestations` is invisible to the party that most needs it.

Note this would only expose a self-declaration. It is worth asking for because it is cheap and
closes a plumbing gap, not because it answers the authoritativeness question.
