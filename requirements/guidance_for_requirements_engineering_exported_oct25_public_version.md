# **Guidance for Requirements Engineering**

**TL;DR** This document provides a standardized framework for writing requirements across software development projects. Requirements follow fundamental principles (atomicity, unambiguity, measurability, etc.) and are structured in three levels: (1) Stakeholder Requirements (SHR) describing business needs in business language, (2) Software Requirements (SWR) defining technical behavior without implementation, and (3) Software Specifications (SPEC) detailing concrete implementation. Use "shall" for mandatory and "should" for recommended requirements.

---

# **I) What is a Stakeholder Requirement vs. a Software Requirement vs. a Software Specification?**

## 1) Stakeholder Requirements (SHR) - Stakeholder View 

1. Express **WHAT** problem needs to be solved (SHRs are categorized into the following types: users, scientific, regulatory, performance, safety, business, and system requirements)

2. Owned by Product with input from stakeholders including business development, security, and regulatory teams

3. Use business/domain language only, i.e. don't use terms understood in software engineering or data science only

4. No technical solutions or implementations to be called out

5. Focus on measurable business or user value

6. As a rule of thumb there should not be a SHR with less than 3 connected SWRs

7. Example: "Customer data shall be protected from unauthorized access"

## 2) Software Requirements (SWR) - Solution View 

Describe **HOW** the software will behave / solve the problem. *The software should be viewed as a black box, i.e. the focus is on interaction and interfaces.*

1. Owned by Engineering with final review by Product Management

2. Use technical terms but stay implementation-agnostic

3. Define inputs and outputs

4. Define interfaces and behaviors

5. Include constraints and quality attributes, e.g. error rate < 0.1%; test coverage > 80%

6. Example: "System shall encrypt all stored customer data"

## 3) Software Item Specifications (SPEC) - Technical Contract and Design View 

1. Define **HOW** the system can be integrated / consumed by another system (contract) and how it is constructed to satisfy the expected behavior (design)

2. Owned by Engineering

3. Document interfaces, protocols, formats, and endpoint definitions (e.g. OpenAPI)

4. Provide implementation guidelines and architectural constraints

5. Define precise data structures and quality parameters

6. Support both integrators consuming the component and developers implementing it

7. Example: OpenAPI specification documenting REST endpoints with request/response schemas

**Where to document:** Specifications should be maintained in version control alongside the code, as markdown files, with clear references to the software requirements they fulfill.

---

# **II) Rules for Writing and Connecting Requirements (SHRs and SWRs)**

## 1. **Atomicity** - Each requirement addresses exactly one aspect or need

**Bad Example:** SHR-1: Patient data shall be secure and system must record all access for auditing

**Good Example:** Two separate requirements: 

* SHR-1: Patient data confidentiality shall be maintained in accordance with ISO 27001
* SHR-2: All access to patient data shall be logged according to ISO 27001

**Related SWRs that demonstrate atomicity:**

* For SHR-1:
  * SWR-1: System shall encrypt all stored patient data using approved cryptographic controls
  * SWR-2: System shall enforce role-based access control for all patient data
  * SWR-3: System shall automatically mask sensitive data in displays and reports
  * SWR-4: System shall enforce session timeout after 15 minutes of inactivity
  * SWR-5: System shall prevent concurrent user sessions
* For SHR-2:
  * SWR-6: System shall maintain access logs capturing user ID, timestamp, and type of access
  * SWR-7: System shall log all failed access attempts
  * SWR-8: System shall prevent modification of audit logs
  * SWR-9: System shall retain access logs for minimum of 2 years
  * SWR-10: System shall alert administrators of access patterns that could indicate fraudulent activity

## 2. **Unambiguous Language and Terminology Conformity** - Clear, precise language with single possible interpretation. Uses agreed glossary and domain vocabulary.

**Bad Example:** SHR-1: System should handle patient data securely (Ambiguous: What does "handle" and "securely" mean?)

**Good Example:** SHR-1: Patient data confidentiality shall be maintained according to ISO 27001

**Note on glossary:** Product-specific terms should be maintained in a central glossary accessible to all team members.

**Note on language:** 
- SHRs: Use the language of the specific user type(s) / persona. The language will depend on the domain, e.g. business, regulatory, medical.
- SWRs: Use the language of the specific engineering or science function. In other words, technical or scientific language.

**Note on standards references:** When referring to standards or regulations, refer to the standard and year without version numbers to avoid costly recertification on updates. For example, refer to ISO 27001:2022. Do not reference the specific section that the requirement meets.

## 3. **Verifiability, Measurability & Constraints** - Quantifiable and testable with concrete criteria. Try to put a numerical constraint wherever feasible. Clearly spell out fault tolerance, safety, and security requirements.

**Bad Example:** SWR-1: System shall handle many concurrent users quickly (Not measurable: What's "many" and "quickly"?)

**Good Example:** SWR-1: System shall support 1000 concurrent authenticated users with response times under 200ms at P95

**What is meant by "testable"?** 
- For SHRs: the user, resp. the business validates the requirement
- For SWRs: the engineer validates through automated tests

**What does it mean to constrain?** Instead of saying "Multiple users can log in," define how many.

## 4. **Non-redundancy** - No duplicate or overlapping requirements

**Bad Example:** 
- SWR-1: System shall encrypt patient data
- SWR-2: All patient information shall be stored in encrypted form
- SWR-3: Patient records must be encrypted in the database

**Good Example:** SWR-1: System shall encrypt all patient data at rest using an advanced encryption method

## 5. **Consistency** - No contradictions with other requirements

**Bad Example:** 
- SWR-1: System shall encrypt all patient data using AES-256
- SWR-2: System shall store sensitive patient data using 3DES encryption

**Good Example:** 
- SWR-1: System shall encrypt all patient data using an advanced encryption method
- SWR-2: System shall encrypt all audit logs using an advanced encryption method

## 6. **Completeness** - Each requirement will cover all necessary aspects with no undefined use cases or business goals. The requirement should make everything explicit. Don't assume everybody knows xyz - but make xyz a requirement.

**Bad Example:** SWR-1: System shall authenticate users (Incomplete: missing password policies, lockout, reset flows)

**Good Example:** 
- SWR-1: System shall authenticate users via username and password
- SWR-2: System shall enforce password minimum length of 12 characters
- SWR-3: System shall lock accounts for 1 hour after 3 failed login attempts
- SWR-4: System shall provide secure password reset via email verification

*Note: This is an incomplete example for demonstration purposes.*

## 7. **Necessity** - Each requirement is essential and needed. If it's not needed but just an idea, move it to the next version's requirement backlog.

**Bad Example:** SWR-1: System shall support dark mode for night operation (Nice to have feature without business/regulatory need)

**Good Example:** SWR-1: System shall support high contrast display mode for accessibility compliance (Maps to regulatory requirement for accessibility)

## 8. **Binding** - Clear distinction between mandatory and optional requirements (shall to be used instead of will or must for mandatory requirements; should for optional)

**Bad Example:** 
- SWR-1: System requires authentication
- SWR-2: Data encryption needed

**Good Example:** 
- SWR-1: System shall authenticate all users before access
- SWR-2: System shall encrypt all patient data at rest
- SWR-3: (optional) The interface should support customizable keyboard shortcuts for commonly used analysis functions
- SWR-4: (optional) Report exports should be customizable with user institutional logos

## 9. **Implementation Independence** - Requirements specify expected input, output and behavior, without specifying how the behavior is implemented. I.e. the implementation must be able to change without having to change requirements. The SWRs should not be overly prescriptive, stay away from mentioning specific solutions/methods of implementation.

**Bad Example:** SWR-1: System shall use FastAPI with Pydantic models and SQLAlchemy ORM to validate and store user data in PostgreSQL. (Specifies Python frameworks and database technology)

**Good Example:** 
- SWR-1: System shall validate all user input against defined schemas
- SWR-2: System shall persist user data ensuring referential integrity
- SWR-3: System shall reject malformed data with descriptive error messages
- SWR-4: System shall prevent SQL injection attacks
- SWR-5: System shall log all validation failures

**Where to document implementation details:** In the software specifications. The software specification serves three key purposes: it documents the technical contracts (APIs, data formats, protocols) needed by integrators to consume the component, provides implementation guidelines to help developers build the right solution, and defines clear validation criteria to verify all software requirements are met.

## 10. **Conciseness** - Each requirement shall be brief and direct without redundant elaboration.

**Bad Example:** SWR-1: In order to maintain the highest standards of data protection and privacy in our cutting-edge healthcare system, and considering the critical importance of maintaining patient trust while adhering to industry best practices, the system shall thoughtfully and carefully implement state-of-the-art cryptographic protection mechanisms for safeguarding sensitive information.

**Good Example:** SWR-1: System shall encrypt all patient data using an advanced encryption method.

**Note on rationale:** Elaborations and reasoning are relevant for alignment. Consider adding a rationale/justification field to your requirements management system where this context can be documented separately.

## 11. **Clearly Define Users** - When a requirement refers to a user, the exact user must be defined. Each system will have different types of users which will be defined at the start of the project (e.g., system admin, internal engineer/user, external customer/user) and these specific user roles should be referred to throughout the requirements.

**Bad Example:** SHR: The Platform API must provide detailed information about an application version at a user's request.

**Good Example:** SHR: The Platform API must provide detailed information about an application version at a data scientist's request.

**Note on user definition:** Users should be defined in product design documentation, clarifying the different user types (usually primary users - who are the end users, secondary users, and indirect users such as support engineers). The software requirement should clearly indicate which user type it applies to.

## 12. **Traceability** - Requirements must be traceable in two dimensions: vertically through requirement levels (SHR to SWR to SPEC, showing implementation relationships) and horizontally where SHRs link to validation tests (proving we built the right thing) and SWRs link to verification tests. Additionally, risk controls trace to requirements that implement them, and requirements (both SHR and SWR) trace to risks they mitigate.

**Bad Example:** 
- SHR-1: Patient data shall be protected (no SWRs linked)
- SHR-2: System shall protect data (no validation test)
- SWR-1: System shall validate input (no verification test)

**Good Example:** 
- SHR-1: Patient data confidentiality shall be maintained → **VAL-23**
  - SWR-1: System shall encrypt all patient data at rest → **VER-12**, RISK-2
    - SPEC-1: Data encryption interface

**Note on traceability management:** Traceability should be managed through your requirements management system with regular monitoring to ensure all links remain valid.

## 13. **Complete and Explicit Dependencies** - When a software requirement builds upon or specializes functionality defined in another software requirement, this dependency shall be made explicit through dual parentage, linking to both its stakeholder requirement and the software requirement(s) it depends on. Note that a SWR may have dependencies to other SWRs (relationships between requirements are both vertical (to SHRs and specs) and horizontal (to other SWRs)).

**Bad Example:** 
- SHR-1: System shall comply with security audit requirements
  - SWR-1.1: System shall maintain comprehensive system audit logs
- SHR-2: Organization administrators shall be able to monitor actions of their organization's users
  - SWR-2.1: System shall display organization activity logs

*Bad, as SWR-2.1 depending on SWR-1.1 is not made explicit*

**Good Example:** 
- SHR-1: System shall comply with security audit requirements
  - SWR-1.1: System shall maintain comprehensive system audit logs
- SHR-2: Organization administrators shall be able to monitor actions of their organization's users
  - SWR-2.1: System shall display organization activity logs
    - Additional Parent: SWR-1.1

**Note on implementation:** Use your requirements management system's relationship features to link requirements to multiple parents as needed.

---

# **III) Shall, Should**

**SHALL**

1. Indicates a mandatory requirement

2. Used for essential functionality or regulatory obligations

3. Example: "The system shall log all user actions with timestamp and user ID."

**SHOULD**

1. Indicates recommended but not mandatory requirements

2. Used for desired features that allow some flexibility

3. Example: "The system should display a warning when memory usage exceeds 80%."

---

# **IV) FAQs**

**Q1: What do we do about SHRs which are for strictly business purposes and don't link to software requirements?**

These are not required for product development, and a justification can be provided for why there is no implementation/traceability.

**Q2: What is the scope of software requirements?**

Software requirements should specify the following (Sourced from: General Principles of Software Validation; Final Guidance for Industry and FDA Staff):

1. All software system inputs
2. All software system outputs
3. All functions that the software system will perform
4. All performance requirements that the software will meet (e.g., data throughput, reliability, and timing)
5. The definition of all external and user interfaces, as well as any internal software-to-system interfaces
6. How users will interact with the system
7. What constitutes an error and how errors should be handled
8. Required response times
9. The intended operating environment for the software, if this is a design constraint (e.g., hardware platform, operating system)
10. All ranges, limits, defaults, and specific values that the software will accept
11. All safety related requirements, specifications, features, or functions that will be implemented in software
12. Software requirements should not capture design decisions (i.e., the implementation method of a requirement)

**Q3: How do I reference a SWR from a specification file?**

Reference requirements using metadata in your specification files (e.g., in the front-matter of markdown files) according to your requirements management system's conventions. This ensures traceability between specifications and the requirements they fulfill.

**Q4: What is a "system"?**

A system is a piece of software that fulfills specific requirements and can be independently tested, released, deployed and monitored. A system is owned by a team and includes everything needed to run: code, infrastructure, documentation, and interfaces to other systems. Systems can be part of larger "systems of systems," i.e. systems can depend on each other. Systems can contain "sub-systems" (also called "components"), i.e. functional pieces that may not be independently deployable.

**Q5: Is it important to differentiate 'as-built' vs 'as-designed'?**

No. There should not be discrepancies between as-built and as-designed. If there is a discrepancy, this should be documented. The software requirements should reflect the final product. If the product cannot meet a software requirement, this should be documented, and the software requirement should be updated to specify whatever the 'as-built' version will be - this may occur in the next revision of the product.

---

# **Appendix A: References**

1. [General Principles of Software Validation; Final Guidance for Industry and FDA Staff](https://www.fda.gov/media/73141/download)

2. [Why Using the Correct Requirements Terms Matters](https://argondigital.com/blog/product-management/using-the-correct-terms-shall-will-should/)

3. Requirements Engineering Fundamentals, Rooky Nook, Inc., 2011, 1st edition, ISBN 98-1-933952-81-9.

---

*This document provides guidance on requirements engineering best practices for software development projects.*