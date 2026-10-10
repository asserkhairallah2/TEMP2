# System Requirements List

## University Secure Examination Platform

**Document Type:** Software Requirements Specification (SRS) — Software Requirements List
**Source:** University Secure Examination Platform — Graduation Project Documentation (September 2026)

---

## 1. Project Overview

The University Secure Examination Platform is a university online examination system designed to provide a **secure environment for conducting electronic exams**. The platform enables Doctors and Teaching Assistants to create and manage exams and questions for their assigned courses, while Students can access only the courses and exams assigned to them.

The system is intended to operate **within the university network** and requires exams to be accessed through **Safe Exam Browser (SEB)**. Its core focus areas are:

- Secure examination.
- Flexible question creation.
- Automatic and manual grading.
- Exam management (scheduling, timing, attempts, and reattempt control).
- Integration with the university's existing student and staff database.

---

## 2. User Roles & Permissions

The system defines **three main roles**: Doctor, Teaching Assistant, and Student. All access is limited to assigned courses.

### 2.1 Doctor

- Creates and manages questions and exams for assigned courses.
- Configures exam settings, including password, timing, and allowed attempts.
- Grades theoretical and essay questions.
- Grants or revokes Teaching Assistant permissions within a given course.
- May reopen an exam for a specific student or group of students when needed.
- Views results and exam information.
- Enables or disables the built-in calculator for each exam.

### 2.2 Teaching Assistant

- Operates within the permissions granted by the Doctor they are assigned to in a given course.
- Creates and manages questions and exams within assigned courses.
- Configures exam settings (password, timing, allowed attempts).
- Grades theoretical and essay questions, including grading exams **on the Doctor's behalf**.
- May reopen an exam for a specific student or group of students when needed.
- Views results and exam information.

### 2.3 Student

- Can access **only the courses and exams assigned to them**.
- Enters an exam using the required **exam password**.
- Completes the exam according to its configured rules.
- Submits their answers.
- Can view their results once they are **published**.

---

## 3. Functional Requirements

### 3.1 Exam Management

- **FR-EM-01:** Each exam shall be configured with a password, a start time, an end time, a duration, an attempt limit, a list of assigned students, and its set of questions.
- **FR-EM-02:** The exam shall automatically end when the configured duration expires.
- **FR-EM-03:** The exam shall automatically end when the configured exam end time is reached.
- **FR-EM-04:** By default, each student shall be given **one attempt only**.
- **FR-EM-05:** A Doctor or an authorized Teaching Assistant shall be able to allow a specific student or a group of students to take the exam again (reattempt authorization).
- **FR-EM-06:** When a reattempt is authorized, the system shall keep the original attempt and record the additional attempt authorization separately.

### 3.2 Question Editor & Question Types

- **FR-QE-01:** The system shall provide a rich Question Editor for Doctors and Teaching Assistants to create questions using different types of content.
- **FR-QE-02:** Supported question types shall include: Multiple Choice, Multiple Answer, True/False, Short Answer, Fill in the Blank, Matching, Essay, Code Output, and Programming Questions.
- **FR-QE-03:** The editor shall support rich content within a question, including text, images, mathematical equations, code blocks, tables, and lists.
- **FR-QE-04:** A single question shall be able to contain multiple types of content at once (e.g., text, image, mathematical equation, and code block combined).
- **FR-QE-05:** Mathematical equations shall be displayed using proper mathematical notation.
- **FR-QE-06:** Programming questions shall display code with syntax highlighting.
- **FR-QE-07:** Questions shall be creatable and editable individually rather than all at once.
- **FR-QE-08:** Creating a question shall include selecting or entering: Question Type, Question Content, Image, Equation, Code, Answers, Correct Answer, Marks, Difficulty, and Topic.
- **FR-QE-09:** A saved question shall be editable at any time, allowing any part of it (content, answers, or metadata) to be updated without recreating it.
- **FR-QE-10:** Saved questions shall be added to the Question Bank.

### 3.3 AI-Assisted Question Import Pipeline

- **FR-AI-01:** The system shall allow Doctors and Teaching Assistants to import questions from existing files, including PDF, Word, Excel, and plain text documents.
- **FR-AI-02:** The system shall use an AI-assisted pipeline to interpret varying instructor formats and convert them into a single, consistent question structure.
- **FR-AI-03:** The import process shall follow a defined pipeline rather than relying on AI alone to make final decisions.

**Import Pipeline Stages:**

- **FR-AI-04 (File Upload):** The Doctor or Teaching Assistant uploads a PDF, Word, Excel, or text file.
- **FR-AI-05 (File Parsing):** The system extracts the raw content of the file, including text, tables, and images.
- **FR-AI-06 (AI Question Understanding):** The extracted content is analyzed to identify question boundaries, question type, options, correct answers, code blocks, and equations.
- **FR-AI-07 (Structuring):** The identified content is converted into the platform's standard question format.
- **FR-AI-08 (Human Review):** The Doctor or Teaching Assistant reviews the structured questions before they are saved.
- **FR-AI-09 (Question Bank):** Approved questions are added to the Question Bank.

**Human Oversight & Data Integrity:**

- **FR-AI-10:** The AI shall **not** save questions directly to the database.
- **FR-AI-11:** Extracted questions shall always be presented for review first.
- **FR-AI-12:** Questions shall be editable, approvable, or rejectable individually or in groups.

**Handling Different File Formats:**

- **FR-AI-13:** Text-based PDFs shall be processed through direct text extraction.
- **FR-AI-14:** Scanned PDFs shall first be processed using OCR before the extracted text is passed to the AI structuring step.
- **FR-AI-15:** Word files shall be processed by extracting paragraphs, tables, images, and headings, while preserving the relationship between a question and its related content (such as an image, equation, or code block).
- **FR-AI-16:** Excel files shall be importable using a defined column template (e.g., Question, Type, Option A to D, Correct Answer, and Marks).
- **FR-AI-17:** If different column names are used, the AI shall assist in mapping the columns to the system's expected fields.

**Flexible Formatting Between Instructors:**

- **FR-AI-18:** The AI processing step shall interpret differing instructor formats (e.g., options, correct answers, question numbering) and convert them into the same internal question structure, without requiring a single fixed template.

**Code and Mathematical Content:**

- **FR-AI-19:** When an imported question contains a code block, the system shall distinguish the question text from the code and store the code separately, along with its programming language where identifiable.
- **FR-AI-20:** Mathematical expressions shall be preserved as formulas rather than converted into plain text, so they can be displayed using proper mathematical notation.

**Import Review and Validation:**

- **FR-AI-21:** After processing a file, the system shall present a preview of detected questions before import, including a summary such as the number of questions found and their detected types.
- **FR-AI-22:** Each imported question shall be editable, approvable, or rejectable individually, or approvable as a group.
- **FR-AI-23 (Metadata Suggestions):** The system shall suggest topic, difficulty, and marks for each imported question, which the reviewer can accept or edit.
- **FR-AI-24 (Duplicate Detection):** The system shall identify questions that are similar to existing questions already in the Question Bank.
- **FR-AI-25 (Basic Validation Checks):** The system shall flag possible issues such as an unclear correct answer, incomplete code, or an equation that could not be fully parsed.
- **FR-AI-26:** Validation checks are intended to support the reviewer and shall **not** replace the Doctor's or Teaching Assistant's final judgment.

### 3.4 Grading System

- **FR-GR-01:** The platform shall support both automatic and manual grading.
- **FR-GR-02:** Automatically graded question types shall include: Multiple Choice, Multiple Answer, True/False, Matching, Fill in the Blank, selected short-answer questions, and code-based questions where applicable.
- **FR-GR-03:** Theoretical and essay questions shall be manually gradable by the Doctor or an authorized Teaching Assistant.
- **FR-GR-04:** The system shall store grades.
- **FR-GR-05:** The system shall allow authorized staff to review and publish results.

### 3.5 Question Bank Management

- **FR-QB-01:** Each Doctor and Teaching Assistant shall be able to maintain questions related to their assigned courses.
- **FR-QB-02:** Questions shall be organizable by Course, Topic, Question Type, and Difficulty.
- **FR-QB-03:** Questions shall be selectable when creating an exam.
- **FR-QB-04:** The system shall support randomized question selection.
- **FR-QB-05:** The system shall support different question orders for students.

### 3.6 In-Exam Features

**Built-in Calculator:**

- **FR-IE-01:** The platform shall provide a built-in calculator inside the exam interface.
- **FR-IE-02:** The Doctor shall be able to enable or disable the calculator for each exam.
- **FR-IE-03:** The calculator shall support basic arithmetic and scientific calculations.
- **FR-IE-04:** The calculator shall be available inside the examination environment without requiring the Student to leave the exam.

**Auto Save and Exam Recovery:**

- **FR-IE-05:** Student answers shall be automatically saved during the examination.
- **FR-IE-06:** If a temporary network or device problem occurs, the system shall preserve the student's current exam session and allow recovery according to the exam rules.
- **FR-IE-07:** The system shall record the exam session state.

**Exam Session States:**

- **FR-IE-08:** The system shall support the following exam session states: Not Started, In Progress, Submitted, Auto Submitted, Interrupted, Reopened, and Graded.

### 3.7 Exam Analytics

- **FR-AN-01:** The platform shall provide basic exam and result analytics for authorized staff.
- **FR-AN-02:** Analytics shall include: number of students, average score, highest score, lowest score, passed/failed results, question performance, and number of correct and incorrect answers.
- **FR-AN-03:** Analytics shall help Doctors and Teaching Assistants review exam performance.

---

## 4. Security & Non-Functional Requirements

### 4.1 Safe Exam Browser (SEB) Integration & Environment Constraints

- **NFR-SE-01:** The examination platform shall operate only within the university network.
- **NFR-SE-02:** Students must access exams through Safe Exam Browser (SEB).
- **NFR-SE-03:** The system shall verify that the exam is being accessed through the required environment.

### 4.2 Network Restriction Rules

- **NFR-NR-01:** Exam access shall be restricted to the university network only.

### 4.3 Audit and Security Event Logging

- **NFR-AU-01:** The platform shall record examination security events, including:
  - Login time.
  - Exam start time.
  - Exam submission.
  - Network disconnect/reconnect.
  - IP address changes.
  - Device/session changes.
  - Other relevant security events.
- **NFR-AU-02:** These events shall be stored as audit information.
- **NFR-AU-03:** These events shall **not** be automatically treated as cheating.

---

## 5. System Integration Requirements

- **INT-01:** The platform shall be integrated with the university's existing system.
- **INT-02:** The university database shall be treated as a **read-only source** for existing university information.
- **INT-03:** The platform shall be able to retrieve information including: Students, Doctors, Teaching Assistants, Courses, Departments, Course Enrollments, and User Roles.
- **INT-04:** The examination platform shall **not** directly modify the university's original database.
- **INT-05:** The platform shall maintain its own database for examination-related data, including: Questions, Question Banks, Exams, Attempts, Answers, Grades, Exam Sessions, Reattempt Authorizations, and Audit Logs.
- **INT-06:** A separate integration/synchronization layer may be used between the university system and the examination platform.

---

## 6. Future / Optional Features

- **OPT-01 (Face Recognition):** Face Recognition may be added as an optional future feature for student identity verification during login or examination sessions.
- **OPT-02:** Face Recognition is **not required** for the initial system, as its implementation depends on university infrastructure, available cameras, privacy requirements, and deployment conditions.

---

## 7. Expected System Output (Consolidated)

The final system is expected to provide:

- A secure university examination platform.
- Three role-based interfaces for Doctor, Teaching Assistant, and Student.
- Secure exams through Safe Exam Browser.
- University network restriction.
- Rich question creation with text, images, mathematical equations, and code.
- AI-assisted question import from PDF, Word, Excel, and text files.
- Automatic and manual grading.
- Question Bank management.
- Exam scheduling and attempt control.
- Reattempt authorization.
- Built-in calculator.
- Automatic answer saving and exam recovery.
- Exam analytics.
- Audit and security logging.
- Read-only integration with the university system.
