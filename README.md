<p align="center">
  <img src="img/showcase-header.svg" alt="Contract Creator — Equipment maintenance contracts" width="100%">
</p>

<p align="center">
  <strong>Prepare equipment maintenance contracts, issue PDFs, and follow every revision under one reference.</strong>
</p>

<p align="center">
  <img src="img/badges/python.svg" alt="Python" height="28">
  <img src="img/badges/django.svg" alt="Django" height="28">
  <img src="img/badges/tailwind.svg" alt="Tailwind CSS" height="28">
</p>

<p align="center">
  <a href="#the-problem">Problem</a> ·
  <a href="#the-solution">Solution</a> ·
  <a href="#app-walkthrough">Walkthrough</a> ·
  <a href="#application-architecture">Architecture</a> ·
  <a href="#data-architecture">Data model</a> ·
  <a href="#engineering-decisions">Engineering</a> ·
  <a href="#testing-and-limitations">Testing</a>
</p>

## The problem

To prepare an equipment maintenance contract, staff had to look up equipment in the ERP, find the applicable rates, and enter coverage and exclusions into a Word template. Manual lookups and transcription made contracts harder to prepare consistently and sometimes led to contracts being signed below the applicable price.

After issuance, historical contracts were available as scanned files. Staff lacked a searchable register that showed which contracts had been issued and who prepared them.

## The solution

Contract Creator guides staff through equipment selection, coverage, exclusions, and pricing, then generates a PDF from a Word template. A searchable register keeps the issued PDFs, their authors, and their follow-up history together.

<p align="center">
  <a href="img/dashboard.png"><img src="img/dashboard.png" alt="Contract Creator dashboard showing recent contracts, document availability, and notifications" width="720"></a>
</p>

*Recent contracts, available documents, and notifications in one staff workspace.*

## App walkthrough

Follow preparation, revision, and document follow-up. The screenshots illustrate individual steps rather than one continuous version sequence.

[Issue a contract](#issue-a-contract) → [Revise an issued contract](#revise-an-issued-contract) → [Record receipt and signature](#record-receipt-and-signature)

### Issue a contract

Choose a client, select equipment from that client’s list, and set the maintenance coverage and exclusions. The wizard calculates a default price for each equipment line. After staff review the totals, the app generates the contract PDF and records it as V1.

<p align="center">
  <a href="img/contract-client-selection.png"><img src="img/contract-client-selection.png" alt="Client selection showing the seven preparation stages and a searchable client list" width="720"></a>
</p>

<details>
<summary><strong>Select equipment and define its coverage</strong></summary>

Search the selected client’s equipment by identifier, designation, or serial number, then choose the items to include in the contract.

<p align="center">
  <a href="img/contract-equipment-coverage0.png"><img src="img/contract-equipment-coverage0.png" alt="Client-scoped equipment selection with search and multiple-selection checkboxes" width="720"></a>
</p>

Set exclusions for each equipment item. The form distinguishes optional selections from mandatory exclusions.

<p align="center">
  <a href="img/contract-equipment-coverage1.png"><img src="img/contract-equipment-coverage1.png" alt="Per-equipment exclusions showing selected options and a mandatory exclusion" width="720"></a>
</p>

*Equipment scope and exclusions are defined before the pricing review.*

</details>

The pricing review brings together equipment-line rates, annual totals, tax, and the amount for each billing period.

<p align="center">
  <a href="img/contract-tarification.png"><img src="img/contract-tarification.png" alt="Equipment pricing controls and summary of annual charges, tax, and billing-period amount" width="580"></a>
</p>

*Review the equipment pricing before issuing the contract.*

<details>
<summary><strong>Review a pricing exception</strong></summary>

An authorized user can adjust a rate or price on an equipment line. The app recalculates the totals and records the exception.

</details>

### Revise an issued contract

Open the contract in history and start a revision. The wizard restores the previous inputs for editing. When staff issue the correction, the register keeps both PDFs under the same contract reference and records the revision author, reason, and changes.

<p align="center">
  <a href="img/contract-history0.png"><img src="img/contract-history0.png" alt="Contract detail showing current V3, earlier V2, authorship, and expandable changes and document follow-up" width="720"></a>
</p>

*Earlier issues remain accessible while the current version is clearly identified.*

<details>
<summary><strong>Find a contract in the register</strong></summary>

Search by reference, client, or identifier, then filter by author, contract type, status, and dates.

<p align="center">
  <a href="img/contract-history.png"><img src="img/contract-history.png" alt="Contract register with search, author and status filters, dates, and current versions" width="720"></a>
</p>

</details>

### Record receipt and signature

Staff record receipt and signature against the relevant version. The register stores timestamps for those milestones and provides a place to save a link to the signed document.

The version page keeps the revision’s change summary and follow-up together. This example shows changes to duration, payment terms, and periodicity, with receipt marked for the technico-commercial staff and client. The client-signature checkbox remains unchecked.

<p align="center">
  <a href="img/contract-revision-changes-follow-up.png"><img src="img/contract-revision-changes-follow-up.png" alt="Expanded V2 change summary and receipt follow-up, with both receipt checkboxes marked and client signature still pending" width="720"></a>
</p>

*Document availability and signature status are separate: generating a PDF does not establish that it has been received or signed.*

## Application architecture

Python and Django handle sign-in, the contract wizard, pricing, and history. Django templates, HTMX, and Tailwind CSS provide the staff interface. Client and equipment reference data from the ERP inform equipment selection and pricing. Pricing also uses maintained rules for the selected coverage and equipment configuration.

<p align="center">
    <a href="img/application-architecture.svg"><img src="img/application-architecture.svg" alt="Contract Creator architecture: staff interface, reference data and pricing rules feed Django preparation, revisions, permissions and follow-up; SQLite stores history, and Word documents are converted and archived through Microsoft Graph and OneDrive." width="800"></a>
</p>

*The Django block groups the application’s core responsibilities. Arrows show the reference-data, persistence, and document flows.*

<details>
<summary><strong>Document generation and access</strong></summary>

The document generator populates Word templates with contract details, equipment tables, exclusions, and pricing. The application uses Microsoft Graph for PDF conversion and OneDrive storage. Staff use authenticated accounts, and permissions control actions such as pricing adjustments and account administration.

<p align="center">
  <a href="img/contract-onedrive.png"><img src="img/contract-onedrive.png" alt="OneDrive settings showing the central connection for document conversion and PDF archiving" width="720"></a>
</p>

| Component | Responsibility |
| --- | --- |
| Python and Django | Authentication, wizard validation, pricing, revisions, and history. |
| Templates, HTMX, Tailwind CSS | Staff pages and interactive workflow. |
| Reference-data adapter | Client and equipment data and maintained pricing inputs. |
| Word-template engine | Assemble contract details, equipment annexes, coverage, and exclusions. |
| Microsoft Graph and OneDrive | Convert documents to PDF and store issued files. |
| SQLite | Application state, versions, authorship, and follow-up in the documented build. |

The application defines standard, elevated, and manager roles. Managers have broader history and account-administration access. DOCX export permissions depend on the application build.

</details>

## Data architecture

The contract record retains one reference across revisions. Each issued version has its own number, author, timestamp, saved inputs, and PDF reference. Receipt and signature are recorded for the specific version.

<p align="center">
  <a href="img/data-architecture.svg"><img src="img/data-architecture.svg" alt="Contract Creator data architecture showing ERP exports, normalized reference tables, contract records, revisions, user attribution, and OneDrive storage." width="800"></a>
</p>

*Reference-data and application schema.*

<details>
<summary><strong>Identity, integrity, and history</strong></summary>

The data model enforces unique contract references and unique version numbers within a contract. The wizard checks that selected equipment belongs to the chosen client and that exclusions apply to the selections. Saved inputs and change summaries preserve the context for each revision.

</details>

## Engineering decisions

**Calculate defaults and record exceptions.** Pricing rules account for the client profile, contract type, equipment modality, and applicable exclusions. Recording overrides makes exceptions traceable. Default prices depend on the quality of the source data and maintained rules.

**Preserve each issued version.** A correction creates a new PDF and saves a new snapshot under the existing contract reference. The wizard checks restored selections against current reference data before issuing the revision.

<details>
<summary><strong>Fit document generation into existing tools</strong></summary>

Word templates give staff a familiar format for maintaining contract layouts. Generation relies on the template markers and layout being maintained. OneDrive fits the company’s existing Office environment and keeps the generated documents in its document storage.

</details>

## Testing and limitations

This showcase includes screenshots of contract preparation, pricing review, revisions, and document follow-up. Automated test results are not included in this repository.

- **Pricing:** default prices depend on the ERP reference data and maintained rules; staff review totals before issuance.
- **Documents:** generation depends on maintained Word-template markers and the Microsoft Graph / OneDrive integration.
- **Follow-up:** receipt and signature are recorded by staff for each version. PDF availability alone does not establish receipt or signature.

---

*This repository showcases the project. The codebase is not shared because it contains sensitive business data and related business information. Identifying details in screenshots have been redacted; client data, pricing tables, and legal templates remain private.*
