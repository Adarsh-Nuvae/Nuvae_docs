# Pre-Auth Agent Explained Like a Story

## Meet the Team
- **Pre-Auth Agent (apps/pre-auth-agent/agents/pre_auth_agent.py)** is the main hero. Think of it as a super-smart helper who fills out medical permission slips (pre-authorizations) on insurance websites.
- **BaseHealthcareAgent (apps/pre-auth-agent/agents/base_agent.py)** is the heros backpack. It carries the tools for talking to web pages and to a bigger brain called Agno.
- **PayerPortalService (apps/pre-auth-agent/services/payer_portals.py)** is the map. It knows where each insurance website lives, which buttons to click, and what rules they follow.
- **CredentialManager (apps/pre-auth-agent/services/credential_manager.py)** is the keyring that keeps usernames and passwords locked up safely until the hero needs them.
- **DocumentProcessor (apps/pre-auth-agent/services/document_processor.py)** is the librarian. It checks that every file is the right type and size, and can read text from PDFs or pictures.
- **MedicalRecordsRetriever, ClinicalDataExtractor, DocumentGenerator, FHIRClient (apps/pre-auth-agent/tools/)** are the research squad. They gather patient facts, dig through medical history, and write the letters the hero must submit.
- **pre_auth_tools.py** is a bundle of quick info buttons ("Give me encounter info!", "Show me prior auth history!") that the hero can press while working.
- **Models (apps/pre-auth-agent/models/pre_auth.py)** are the heros notebooks with neat forms to hold every detail about patients, doctors, insurance, and submission results.

## What Problem Are We Solving?
Hospitals must ask insurance companies for a "yes" before doing many treatments. This request is called a **pre-authorization**. Filling out these forms by hand is slow. Our hero automates that chore while staying careful, safe, and organized.

## How the Adventure Unfolds
1. **Starting Up**
   - The hero wakes up with settings like how long to work and which AI brain to use.
   - It grabs helper objects: portal map, medical records squad, document writers, and FHIR data fetcher.

2. **Receiving a Mission**
   - The hero gets a `PreAuthTask`, which includes:
     - Patient, doctor, and insurance info (`PreAuthRequest`).
     - Login details for the insurance website.
     - Rules like "try again if it fails" or "save screenshots." 

3. **Understanding the Portal**
   - Using `PayerPortalService`, the hero finds the right website link, login boxes, and which forms need filling.
   - `CredentialManager` hands over the username and password securely.

4. **Gathering Evidence**
   - **MedicalRecordsRetriever** pulls lab results, imaging, doctor notes, and past treatments.
   - **ClinicalDataExtractor** picks the most important facts and checks how they prove the treatment is needed.
   - **DocumentGenerator** writes helpers like letters of medical necessity, clinical summaries, and treatment plans using Azure OpenAI.
   - **DocumentProcessor** double-checks that every file fits size rules and can be read.

5. **Talking to the Portal**
   - The hero uses `interact_with_browser()` to give clear instructions to a browser automation buddy (browser-use library).
   - It logs in, explores menus, and finds the "new authorization" form.
   - If a form field needs info, the hero presses a tool button like `get_patient_info` or `get_procedure_requirements` to fetch the answer.

6. **Filling and Submitting**
   - The hero types in patient, provider, and clinical info using selectors from the portal map.
   - It uploads supporting documents and double-checks values (for example, using `validate_clinical_codes`).
   - When everything looks right, it clicks submit and waits for the confirmation page.
   - It grabs the reference number, takes a screenshot, and stores the result as a `SubmissionResult` inside a `PreAuthResponse`.

7. **Handling Trouble**
   - If the website complains (bad login, missing info, code errors), the hero reads the error, fetches more data if possible, fixes the form, and tries again.
   - If the whole mission fails and `auto_retry_on_failure` is turned on, the hero rests for a few seconds and starts over.

## Important Safety Rules
- **HIPAA & Security**: Credentials stay encrypted. Sensitive data isn
