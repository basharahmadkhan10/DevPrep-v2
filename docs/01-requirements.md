# DevPrep --- Requirements

**Version:** 1.0\
**Status:** Baseline for design\
**Purpose:** Define what DevPrep must do before architecture, database,
API, and implementation decisions are finalized.

## 1. Product Overview

DevPrep is an AI-powered interview preparation platform for students and
job seekers targeting a specific company or role. It compares a
candidate's resume and background with a job description (JD),
identifies relevant strengths and skill gaps, and creates focused
preparation material for the week before an interview.

## 2. Problem and Target Users

**Problem:** Candidates often use generic preparation resources and
struggle to identify which skills matter most for a particular JD,
especially when interview time is limited.

**Target users:** - Students preparing for internships and campus
placements. - Job seekers targeting a specific company, role, or JD. -
Candidates with approximately one or two weeks to prepare.

**Product outcome:** Help candidates prioritize relevant
preparation---not promise an interview or job offer.

## 3. MVP Scope

### In scope

-   Email registration and email verification.
-   Login, JWT-based authentication, protected endpoints, logout, and
    forgot/reset password.
-   User profile viewing/editing, background information, and profile
    image upload using Multer and ImageKit.
-   PDF resume upload (maximum 5 MB) and parsing.
-   Job description input, with optional company and role.
-   Resume/JD match score with an explanation.
-   Evidence-based skill-gap analysis.
-   JD-specific technical and behavioral interview questions.
-   A personalized seven-day preparation plan.
-   Saving and viewing preparation history.
-   Exporting the report or preparation plan.
-   Request validation, centralized error handling, rate limiting, and
    AI-output validation.

### Out of scope for the initial MVP

-   LinkedIn/GitHub profile data import. Consider later, subject to
    permitted access methods and feasibility.
-   Guaranteeing interview success or presenting generated questions as
    verified company questions.
-   Automatic job applications.

## 4. Functional Requirements

Priorities: **Must** = required for MVP.

### FR-01 --- Registration and authentication \[Must\]

-   Users can register with an email and password and verify their email
    using a time-limited, single-use link or code.
-   Users can log in and access protected endpoints only when
    authorized.
-   Passwords are securely hashed; plaintext passwords are never stored.
-   JWT expiry, refresh/session handling, and logout behavior must
    follow a documented token policy.
-   The system provides forgot-password and reset-password flows using
    time-limited, single-use reset tokens.
-   Authentication responses should avoid unnecessarily revealing
    whether an email is registered.

### FR-02 --- Profile management \[Must\]

-   Authenticated users can view and update their own profile and
    background/about information.
-   Users can upload or replace a profile image through Multer and
    ImageKit.
-   Profile image uploads have file-type and size limits.
-   Users cannot read or modify another user's profile.

### FR-03 --- Resume upload and parsing \[Must\]

-   Users can upload PDF resumes only, with a maximum file size of 5 MB.
-   The server validates file type and size before processing.
-   Multer handles the upload and ImageKit is used for the
    upload/storage workflow.
-   The system extracts available skills, experience, education,
    projects, and certifications into structured data.
-   Parsing failures or insufficient text are reported clearly; missing
    details must not be invented.
-   The original resume and full raw parsed text must not be retained
    after processing. The analysis result and preparation records may be
    saved to the user's history.
-   Resume data must never be exposed to other users.

### FR-04 --- Job description input \[Must\]

-   Users can paste a target JD and optionally provide the company and
    role.
-   The system rejects empty or unusable JD input with a clear message.
-   The JD is associated with the relevant preparation session.
-   User-provided JD text is treated as data, not as instructions that
    override system behavior.

### FR-05 --- Resume/JD analysis and match score \[Must\]

-   The system analyzes the resume and user-provided background against
    the target JD.
-   It returns an indicative match score and an explanation of the
    evidence behind it.
-   It identifies demonstrated matches, partial matches, and skills not
    evidenced by the supplied information.
-   The score must not be represented as a probability of being hired.
-   The system must not state unsupported skills or experience as facts.

### FR-06 --- Skill-gap analysis \[Must\]

-   The system identifies skills emphasized by the JD that are missing,
    partially evidenced, or clearly demonstrated in the supplied
    profile.
-   It prioritizes gaps by relevance to the JD and explains the
    priority.
-   "Not evidenced" must not be treated as proof that the candidate does
    not have a skill.

### FR-07 --- Interview questions \[Must\]

-   The system generates technical and behavioral questions relevant to
    the JD and available candidate background.
-   Questions should be linked to relevant skills or role requirements
    when possible.
-   Generated questions must not be described as confirmed questions
    from the target company.
-   The system may suggest answer structures but must not invent the
    user's personal experiences.

### FR-08 --- Seven-day preparation plan \[Must\]

-   The system creates a focused seven-day plan based on the JD and
    identified skill gaps.
-   Each day contains actionable topics or tasks and an intended
    outcome.
-   The plan prioritizes important gaps and states or allows
    configuration of the assumed daily study time.

### FR-09 --- Preparation history and export \[Must\]

-   The system saves completed analysis results, questions, skill gaps,
    and the preparation plan.
-   Users can list and view only their own previous preparation
    sessions.
-   History endpoints must be paginated or bounded.
-   Users can export their report or preparation plan.
-   Data deletion and retention behavior must be documented before
    release.

### FR-10 --- API and failure handling \[Must\]

-   Endpoints validate request bodies, parameters, query values, and
    uploaded files.
-   The backend uses consistent error responses and appropriate HTTP
    status codes.
-   Rate limits apply to authentication-sensitive and resource-intensive
    endpoints; limits must be configurable.
-   AI-provider timeouts, quota errors, malformed output, and temporary
    failures must be handled explicitly.
-   Unexpected errors must not expose stack traces, secrets, tokens, or
    sensitive user data to clients.

### FR-11 --- AI output safeguards \[Must\]

-   AI responses are validated against a defined output schema before
    being saved or shown.
-   Analysis should be grounded in the supplied resume/background and
    JD; insufficient evidence must be identified explicitly.
-   Malformed, incomplete, or unsupported output must be rejected or
    retried safely.
-   Where practical, important conclusions should be traceable to input
    evidence.
-   The system must reduce the risk of hallucination but must not claim
    that hallucinations can be eliminated completely.
-   A failed AI request must not be presented as a successful analysis.

## 5. Non-Functional Requirements

### Security

-   Enforce authentication, authorization, and resource ownership checks
    on every user-specific endpoint.
-   Keep secrets and API keys out of source control.
-   Use secure transport, appropriate CORS/security headers, input
    validation, upload restrictions, and rate limiting.
-   Never log passwords, tokens, or unnecessary resume/profile content.

### Privacy

-   Collect only data needed for the product.
-   Document how resume files, profile images, JDs, and generated
    results are processed and stored.
-   Delete the original resume and raw parsed text after processing;
    retain only the intended analysis/history records.
-   Define deletion behavior for profile images and saved preparation
    records.

### Reliability and performance

-   Set explicit timeouts and failure states for parsing, email, file
    storage, and AI calls.
-   Do not mark a session complete until required output is validated
    and saved.
-   Paginate or bound history queries.
-   Establish measurable latency and usage targets before production; do
    not assume scale without estimating expected traffic.

### Maintainability and testability

-   Keep routes, controllers, business logic, database access, and
    external integrations separated.
-   Use consistent validation and error conventions.
-   Test authentication, authorization/ownership, the core analysis
    flow, upload validation, and AI-provider failures.
-   Document environment variables, API contracts, setup steps, and
    significant design decisions.

### Usability

-   Provide clear loading, success, empty, and error states.
-   Explain match scores and skill-gap categories in understandable
    language.
-   Support common desktop and mobile screen sizes.

## 6. Primary User Journey

1.  Register and verify email.
2.  Log in and add profile/background information.
3.  Upload a PDF resume (maximum 5 MB).
4.  Enter the target JD and optionally company/role.
5.  Receive a match score, explanation, skill gaps, technical questions,
    behavioral questions, and a seven-day plan.
6.  Save and export the results.
7.  Return later to view previous preparation sessions.

## 7. MVP Acceptance Criteria

The main workflow is accepted when: - A verified user can submit a valid
PDF resume under 5 MB and a usable JD. - Invalid files and unusable
inputs are rejected with clear messages. - Successful analysis returns
all required outputs in the expected schema. - Results do not present
unsupported personal skills or experiences as facts. - The original
resume and raw parsed text are not retained after processing. - AI or
parsing failures do not produce fabricated success results. - Results
are saved and retrievable by their owner only. - Tests cover successful
analysis, invalid upload/input, unauthorized access, and AI-provider
failure. - A user can view and export their own saved results.

## 8. Risks and Constraints

  -----------------------------------------------------------------------
  Risk                                Mitigation
  ----------------------------------- -----------------------------------
  AI invents skills or experience     Ground results in input evidence,
                                      represent uncertainty, validate
                                      output, and test edge cases

  Resume parsing fails                Validate PDF input, handle
                                      extraction errors, and report
                                      insufficient text

  AI latency or cost grows            Apply limits and timeouts, measure
                                      usage, and evaluate background jobs
                                      if needed

  Private data is exposed             Enforce ownership checks, minimize
                                      logging, and define deletion
                                      behavior

  Duplicate or partial results occur  Define session states and
                                      retry/idempotency behavior during
                                      design
  -----------------------------------------------------------------------

## 9. Decisions Required Before Implementation

Resolve these during design: 1. Email provider and
verification/reset-token delivery. 2. JWT access/refresh-token policy,
storage, rotation, and revocation. 3. Resume processing sequence and how
the uploaded file is deleted from ImageKit after parsing. 4. AI
provider/model, output schema, timeout, retry policy, and cost limits.
5. Match-score methodology and how its consistency will be evaluated. 6.
Expected initial usage and measurable performance targets. 7. Whether AI
analysis is synchronous for the MVP or uses background jobs. 8.
Retention and deletion rules for profile images and saved preparation
history. 9. Export format (PDF, etc.) and daily study-time assumption.

## 10. Next Design Steps

1.  Review and approve these requirements.
2.  Write user stories and acceptance criteria for the MVP.
3.  Draw the primary user flow.
4.  Create a system-context and container architecture diagram.
5.  Design MongoDB data models, indexes, and ownership rules.
6.  Define REST API contracts and error conventions.
7.  Break the work into implementation milestones and tests.

**Note:** This document defines intended behavior. Database schemas,
exact endpoint paths, architecture choices, and quantitative production
targets are not finalized here.
