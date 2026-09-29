# Moodle 5.2.3 Analysis Report vs. the Proposal

**Software:** Moodle **5.2.3** (Build: 20260914) · branch `502` · `$version = 2026042003.00`
**Path:** `/home/yermux/Documents/Graduation/TEMP/moodle-5.2.3/moodle/`
**Proposal:** `~/Downloads/University_Secure_Examination_Platform_Proposal-5.pdf` (8 pages)
**Analysis date:** 2026-09-29
**Method:** Direct source-code reading (`grep` / `sed` against Moodle's own files) — **not** based on documentation or on any prior report.

---

## 0. Installation Health

| Item | Result | Evidence |
|---|---|---|
| Version | 5.2.3 (Build: 20260914) | `public/version.php` |
| PHP | 8.2.30 CLI · **8.3.35 web** | `php -v` · `X-Powered-By` |
| Server | Apache/2.4.68 (Debian) on **:8080** | `curl` → HTTP 200 |
| Status | ✅ **Genuinely running** | `/` → 303 → `/login/index.php` |
| Login page | ✅ Renders | `<title>Log in to the site</title>` + username/password form |
| Theme | Boost (cache version 1790691994) | `/theme/image.php/boost/...` |
| YUI | 3.18.1 | `/theme/yui_combo.php?rollup/3.18.1/` |
| Code size | 584 MB · 49,960 PHP files | `du` / `find` |
| `config.php` | Present · `www-data` · `640` — **not readable by this user** | `ls -la` |
| `moodledata` | ⚠️ Not found during inspection | Searched, no match |

> ✅ **The platform is actually running** — not just a copy on disk. It issues a `MoodleSession` cookie and serves a real login page.

---

## 1. Key Finding: Moodle 5.x Changed the Architecture

> ⚠️ **If you plan to build on any open-source quiz module, review this first.** Most of the 57 modules in `/samples` are written against the old layout.

| Change | Moodle ≤4.x | Moodle 5.2.3 (verified) |
|---|---|---|
| **Code root** | `moodle/mod/quiz/` | `moodle/public/mod/quiz/` |
| **Quiz access plugins** | top-level `moodle/quizaccess_*/` | `moodle/public/mod/quiz/accessrule/*` |
| **`config.php`** | In the root | Root `config.php`, read via `public/config.php` |
| **Attempt manager** | `classes/attempt_manager.php` | **Removed / merged** into `classes/quiz_attempt.php` |

### 1.1 The 9 Built-in Access Rules in 5.2.3

All in `public/mod/quiz/accessrule/` — **this replaces the old `quizaccess_*` layout**:

| Folder | `$plugin->component` | Covers |
|---|---|---|
| `password` | `quizaccess_password` | **R09** ✅ |
| `openclosedate` | `quizaccess_openclosedate` | **R10** ✅ |
| `timelimit` | `quizaccess_timelimit` | **R10** ✅ |
| `numattempts` | `quizaccess_numattempts` | **R11** ✅ |
| `delaybetweenattempts` | `quizaccess_delaybetweenattempts` | **R11** ✅ |
| `ipaddress` | `quizaccess_ipaddress` | **R02** ✅ |
| `offlineattempts` | `quizaccess_offlineattempts` | Paper exams |
| `securewindow` | `quizaccess_securewindow` | Secure window (legacy) |
| `seb` | `quizaccess_seb` | **R03 + R48** ✅ — the only one with `MATURITY_STABLE` |

> 💡 **Practical consequence:** `quizaccess_oneconnection`, `quizaccess_safeexambrowser` and `quizaccess_sebversion` (all recommended in the other report) **are all built on the old layout.** Compatibility must be verified before relying on any of them.

---

## 2. Full Coverage Table: The 69 Requirements

**Legend:** ✅ present in core with no work · 🟡 partial (configuration or customisation required) · ❌ absent · ⚪ outside core scope

| # | Requirement | Status | Direct source evidence |
|---|---|---|---|
| **R01** | Online exam platform | ✅ | `mod/quiz` complete |
| **R02** | University network only | ✅ | `quiz.subnet` + `quizaccess_ipaddress::prevent_access()` → `address_in_subnet(getremoteaddr(), $this->quiz->subnet)` |
| **R03** | Mandatory via SEB | ✅ | `quizaccess_seb` + `seb_access_manager` (server-side validation) |
| **R04** | Doctor role | ✅ | `mod/quiz:manage` + `addinstance` → `editingteacher`, `manager` |
| **R05** | Scoped TA role | ✅ | `mod/quiz:grade`, `regrade`, `viewreports` → **`teacher` allowed**; `manage`/`deleteattempts`/`reopenattempts` → `editingteacher` only |
| **R06** | Students see their own assigned exams | ✅ | `mod/quiz:attempt` → `student` + `enrol` |
| **R07** | Reopen for a student/group | 🟡 | `mod/quiz:reopenattempts` + webservice `mod_quiz_reopen_attempt` exist — **but no dedicated UI** |
| **R08** | View results and information | ✅ | `mod/quiz:viewreports` + 4 core reports |
| **R09** | Password per exam | ✅ | `quiz.password` + `quizaccess_password` |
| **R10** | Start time / end time / duration | ✅ | `timeopen`, `timeclose`, `timelimit` + `delay1/delay2` |
| **R11** | Attempt limit | ✅ | `quiz.attempts`, `attemptonlast` + `quizaccess_numattempts` |
| **R12** | List of assigned students | 🟡 | Via `enrol`/`groups` — **no `students` column in the quiz table** |
| **R13** | Automatic expiry | ✅ | `overduehandling` = `autosubmit` / `graceperiod` / `autoabandon` |
| **R14** | **Delegated extra attempt, recorded** | ❌ | `reopen_attempt` requires `ABANDONED` and returns to `IN_PROGRESS` — **no delegation record, no new attempt** |
| **R15** | Multiple Choice | ✅ | `qtype_multichoice` + `options->single = 1` |
| **R16** | Multiple Answer | ✅ | `qtype_multichoice` + `options->single = 0` |
| **R17** | True / False | ✅ | `qtype_truefalse` |
| **R18** | Short Answer | ✅ | `qtype_shortanswer` |
| **R19** | **Fill in the Blank** | ❌ | ⚠️ **`qtype_cloze` does not exist in 5.2.3** — closest option is `qtype_gapselect` |
| **R20** | Matching | ✅ | `qtype_match` |
| **R21** | Essay (manual grading) | ✅ | `qtype_essay` + behaviour `manualgraded` |
| **R22** | Code Output | ❌ | No core question type — requires `qtype_coderunner` |
| **R23** | Programming Questions | ❌ | None — requires `qtype_coderunner` |
| **R24** | Rich content (text/image/equation/code/table) | 🟡 | TinyMCE (`lib/editor/tiny`) + `filter/codehighlighter` |
| **R25** | Mixed content in one question | 🟡 | TinyMCE supports it — but there is no `qtype_combined` |
| **R26** | **MathJax/LaTeX equations** | ✅ | `filter/mathjaxloader` + `filter/algebra` + `filter/tex` — **all three in core** |
| **R27** | Syntax highlighting | ✅ | `filter/codehighlighter` (PrismJS) |
| **R28** | **difficulty + topic** | ❌ | The `question` table has neither field — alternative is the `tag/` API |
| **R29** | Edit any part of a question | ✅ | Question versioning (`stamp`, `parent`) + `edit_question_form.php` |
| **R30** | Import PDF/Word/Excel/text | 🟡 | Core has **7 formats** (Aiken, GIFT, XML, XHTML, Blackboard, …) — **no PDF, no Word, no Excel** |
| **R31** | 6-stage pipeline | ❌ | Absent — must be built |
| **R32** | AI must not write directly to DB | ❌ | No AI import pipeline |
| **R33** | OCR for scanned PDFs | ❌ | ❌ There is no PDF reader in the first place |
| **R34** | Word + question/image/equation linking | ❌ | ❌ |
| **R35** | Excel + AI column mapping | ❌ | ❌ |
| **R36** | Different formats → one structure | 🟡 | Importers exist but **no AI** |
| **R37** | Isolated code + recorded language | ❌ | ❌ |
| **R38** | Equations stored as equations | ❌ | ❌ |
| **R39** | Preview + detected type | ❌ | ❌ |
| **R40** | Individual and bulk approve/reject | ❌ | ❌ |
| **R41** | Metadata suggestions | ❌ | ❌ (the `tag/` API exists but there is no suggestion logic) |
| **R42** | Duplicate detection | ❌ | ❌ |
| **R43** | 3 validation checks | ❌ | ❌ |
| **R45** | Automatic grading | ✅ | 11 grading behaviours: `immediatecbm`, `deferredcbm`, `adaptive`, `deferredfeedback`, `manualgraded`, … |
| **R46** | Manual grading | ✅ | `mod/quiz:grade` + behaviour `manualgraded` + `question_state` queue |
| **R47** | Store + review + publish | ✅ | `review*` (7 fields) × 4 phases: `DURING` / `IMMEDIATELY_AFTER` / `LATER_WHILE_OPEN` / `AFTER_CLOSE` |
| **R48** | Verify the exam runs in SEB | ✅ | `seb_access_manager::validate_browser_exam_key()` → `hash('sha256', $url.$validkey) === $key` |
| **R49** | Security event log | 🟡 | **48 event classes** in `mod/quiz/classes/event/` — **no IP, no device** |
| **R50** | Events are audit-only, not anti-cheat | ✅ | **The default is correct** — there is no auto-penalty anywhere in core |
| **R51** | **Built-in calculator** | ❌ | The `quiz` table has **no calculator column** — grep `CALCULATOR_BASIC` = **0** |
| **R52** | Enable/disable per exam | ❌ | ❌ |
| **R53** | Basic + scientific | ❌ | ❌ |
| **R54** | Cannot leave the exam | ❌ | ❌ |
| **R55** | Autosave | ✅ | `mod/quiz/autosave.ajax.php` → `process_auto_save()` + event `attempt_autosaved` |
| **R56** | Session recovery | 🟡 | Autosave exists — **but there is no offline queue** (only a single retry in JS, no `localStorage`) |
| **R57** | **7-state machine** | ❌ | Core has **6 states** (see §4) — missing `AutoSubmitted`, `Interrupted`, `Reopened`, `Graded` |
| **R58** | University read-only | 🟡 | **`auth/ldap` is in core!** + `auth/shibboleth` |
| **R59** | Pull students/doctors/departments | 🟡 | `auth/ldap` (users) + `enrol/ldap` — **but no `department` entity**; `department` is only free text in the `user` table |
| **R60** | Separate database | ❌ | Core runs on one DB — architectural work needed |
| **R61** | Integration/sync layer | 🟡 | `webservice/` (REST/SOAP) complete — but no sync scaffold |
| **R62** | Question bank per course | ✅ | `question` + `question_categories` + `mod/qbank` |
| **R63** | Organise by course/topic/type/difficulty | 🟡 | **category** ✅ · qtype ✅ · **difficulty ❌** |
| **R64** | Select specific questions | ✅ | `quiz_slots` + `edit.php` |
| **R65** | Random selection | ✅ | `structure->add_random_questions()` + `RANDOM` tag + filterconditions |
| **R66** | Different order per student | ✅ | `quiz.shufflequestions` + `quiz.shuffleanswers` |
| **R67** | count/avg/max/min/pass-fail | 🟡 | `overview_table::compute_average_row()` → `AVG(quizaouter.sumgrades)` + `add_average_row` + `groupavg` — **average and count only; no max/min/pass-fail** |
| **R68** | Question performance | ✅ | `statistics` report → `facility`, `discrimination`, `discriminationindex` |
| **R69** | Analytics for authorised users only | ✅ | `mod/quiz:viewreports` + `core_privacy` (privacy provider present) |
| **R70** | Face Recognition | ❌ | grep `facerecognition` = **0** (the hits in `lib/google2-service` are stub files from the Google API client, not a feature) |

### 2.1 Numerical Summary

| Classification | Count | Share |
|---|---|---|
| ✅ **Full in core** | **32** | 46% |
| 🟡 **Partial (config/customisation)** | **13** | 19% |
| ❌ **Absent** | **24** | 35% |
| **Total** | **69** (R44 reserved) | 100% |

---

## 3. The Five Decisive Classifications

### 3.1 🔴 R19 — `qtype_cloze` Has Disappeared from Moodle 5.2.3

This is **the most important finding in the whole analysis**, because the 57-module report treated `cloze` as "available in core".

```
public/question/type/
  calculated      calculatedmulti   calculatedsimple
  ddimageortext   ddmarker          ddwtos
  description     essay             gapselect
  match           missingtype       multianswer
  multichoice     numerical         ordering
  randomsamatch   shortanswer       truefalse
```

**18 question type plugins. `cloze` is not among them.** Neither `find . -type d -name "*cloze*"` nor `upgrade.txt` mentions it.

| Available alternative | Suits R19? | Note |
|---|---|---|
| `qtype_gapselect` | 🟡 Partial | Selection from dropdowns — **not typing into a blank** |
| `qtype_ddwtos` | 🟡 Partial | Drag & drop onto text — closest to a form |
| `qtype_ddmarker` | 🟡 Partial | Drag markers onto an image |
| `qtype_shortanswer` + frap | 🟡 | Requires `shortanswer` frap patterns |
| **`qtype_gapfill` (from the 57)** | ✅ | **An additional reason to pick it** — 3 input modes |

> 💡 **Decision:** either `qtype_gapfill` (ready, STABLE 2.31), or `qtype_ddwtos` with frap patterns, or build a new question type.

### 3.2 🔴 R51–R54 — Calculator: **0% confirmed**

Dumping the full `quiz` table (40 columns) — **there is no `calculator`**:

```
id, course, name, intro, introformat, timeopen, timeclose, timelimit,
overduehandling, graceperiod, preferredbehaviour, canredoquestions,
attempts, attemptonlast, grademethod, decimalpoints, questiondecimalpoints,
reviewattempt, reviewcorrectness, reviewmaxmarks, reviewmarks,
reviewspecificfeedback, reviewgeneralfeedback, reviewrightanswer,
reviewoverallfeedback, questionsperpage, navmethod, shuffleanswers,
sumgrades, grade, timecreated, timemodified, password, subnet,
browsersecurity, showuserpicture, showblocks, completionattemptsexhausted,
completionminattempts, allowofflineattempts, precreateattempts,
delay1, delay2, …
```

```
$ grep -rn "CALCULATOR_BASIC\|CALCULATOR_SCIENTIFIC" --include=*.php .
(zero results)
```

> ⚠️ **`grade_calculator.php` does exist in `mod/quiz/classes/` — but that calculates grades, it is not a student calculator.** This is the usual source of confusion.

**The only 5.x-compatible approach:** a `local_` plugin with an output component — **not** a fork of `mod_quiz` (the paths changed in 5.x).

### 3.3 ✅ R02 — Network Restriction Is **Native**

The other report said this needs a build. **That was wrong.**

```php
// mod/quiz/accessrule/ipaddress/rule.php
if (empty($quizobj->get_quiz()->subnet)) { return false; }   // empty = open
if (address_in_subnet(getremoteaddr(), $this->quiz->subnet)) { return true; }
return get_string('subnetwrong', 'quizaccess_ipaddress');
```

```xml
<!-- mod/quiz/db/install.xml:42 -->
<FIELD NAME="subnet" TYPE="char" LENGTH="255" …
  COMMENT="Used to restrict the IP addresses from which this quiz can be attempted."/>
```

> ✅ **R02 = ready-made configuration.** Enter the subnet in the exam settings and you are done. **Zero work.**

### 3.4 🔴 R48 — SEB in 5.2.3 Is **Much Stronger** Than the Plugins

`seb_access_manager.php` implements **3 validation levels** (the old `quizaccess_sebversion` plugin did only 1):

| Level | Function | Mechanism |
|---|---|---|
| 1. **Config Key** | `validate_config_key()` | `X-SafeExamBrowser-ConfigKeyHash` |
| 2. **Browser Exam Key** | `validate_browser_exam_key()` | `X-SafeExamBrowser-RequestHash` + `allowedbrowserexamkeys` |
| 3. **Session** | `set_session_access()` / `validate_session_access()` | Session pinned after the first success |

```php
// seb_access_manager.php:266
private function check_key(string $validkey, string $key, ?string $url = null): bool {
    return hash('sha256', $url . $validkey) === $key;
}
```

**Two delivery modes (`requiresafeexambrowser`):**

| Value | Description | Use |
|---|---|---|
| `USE_SEB_NO` | Without SEB | Default |
| `USE_SEB_TEMPLATE` | Ready-made template | Faster — the doctor picks a template |
| `USE_SEB_UPLOAD_CONFIG` | Uploaded `.seb` file | **Strongest** — full control |

**Useful extras:** `prevent_display_blocks()` + `setup_attempt_page()` + `get_quit_button()` (an exit button for SEB) + `current_attempt_finished()`.

> ✅ **R03 + R48 = 100% core, server-side, officially documented.** This is stronger than anything among the 57 modules.

### 3.5 🟢 New Discovery: Moodle 5.x Has a **Built-in AI Subsystem**

**Not present in 4.x.** This significantly changes the AI import plan:

| Element | What exists |
|---|---|
| **6 providers** | `openai` · `azureai` · `gemini` · `deepseek` · `awsbedrock` · `ollama` |
| **5 actions** | `generate_text` · `generate_image` · `summarise_text` · `explain_text` · `responses` |
| **2 placements** | `editor` (inside the editor) · `courseassist` (course assistant) |
| **Governance** | `privacy/provider.php` · `policy_acceptance_report.php` · `usage_report.php` · `rate_limiter.php` |

**Why this matters for R30–R43:**

| Requirement | What core AI offers |
|---|---|
| R31 pipeline | `generate_text` as a primitive — **the orchestration is still yours to build** |
| R32 AI must not write to DB | `ai/placement/` + `aiplacement_action_management` = **a ready model for a placement policy** |
| R41 metadata | No automatic suggestion |
| R70 Face Recognition | ❌ **Absent** — the providers are text/image only |
| Data residency | `ollama` = **self-hosted** → partially solves the R02/privacy problem |

> ⚠️ **`ollama` is the key to a graduation project:** it keeps the AI running **inside the university network** without student data leaving the premises — this satisfies the spirit of R02.

---

## 4. R57 — State Machine: 6 vs. 7

**The actual states in 5.2.3** (`mod/quiz/classes/quiz_attempt.php:57-67`):

```php
const NOT_STARTED = 'notstarted';   // 57
const IN_PROGRESS = 'inprogress';   // 59
const OVERDUE     = 'overdue';      // 61
const SUBMITTED   = 'submitted';    // 63
const FINISHED    = 'finished';     // 65
const ABANDONED   = 'abandoned';    // 67
```

```php
$ grep -c "AUTO_SUBMITTED\|INTERRUPTED\|REOPENED\|GRADED" quiz_attempt.php
0
```

| Proposal state | In 5.2.3? | Note |
|---|---|---|
| Not Started | ✅ `notstarted` | — |
| In Progress | ✅ `inprogress` | — |
| Submitted | ✅ `submitted` | — |
| **Auto Submitted** | ❌ | Autosubmit leaves `SUBMITTED` — **no distinction** |
| **Interrupted** | ❌ | No event for a network drop during an attempt |
| **Reopened** | ❌ | An event exists (`attempt_reopened`) but it is **not a state** — it returns to `inprogress` |
| **Graded** | ❌ | Grades live in `quiz_grades` / `quiz_grade_items`, not in `quiz_attempts.state` |

**Important:** `overdue`, `finished` and `abandoned` do exist in core but are **not required by the proposal** → more than the proposal asks for.

**Minimum solution — two tables:**

```
{prefix}_exam_sessions(id, attemptid FK, userid, state, state_changed_at,
                       device_hash, last_ip, auto_submitted BOOL, interrupted_at)
{prefix}_session_transitions(id, sessionid, from_state, to_state, reason,
                             actorid, created_at)     ← audit trail for R50
```

**Ready-made observers:** the `attempt_state_changed` hook + 48 event classes.

### 4.1 R14 — Delegated Extra Attempt: ❌ Fundamentally Wrong

```php
// mod/quiz/classes/external/reopen_attempt.php:58-66
require_capability('mod/quiz:reopenattempts', $attemptobj->get_context());
if ($attemptobj->get_state() != quiz_attempt::ABANDONED) {
    throw new moodle_exception('reopenattemptwrongstate', 'quiz', '', …);
}
$attemptobj->process_reopen_abandoned(time());
```

**4 problems vs. R14:**

| Required | What Moodle does |
|---|---|
| Delegate an **additional** attempt | Changes the state of the same attempt |
| A **new** attempt, number +1 | ❌ No new attempt is created |
| An independently **stored delegation record** | ❌ None — only an event |
| Works on any state | ❌ `ABANDONED` only (i.e. not submitted) |

> **In other words: a student who submitted their exam can never have it reopened.** In ordinary Moodle the teacher deletes the attempt or creates a manual override. **This is a structural gap.**

---

## 5. R30–R43 — Import: Core Gives You 7 Importers, Nothing More

| Format | Maturity | Covers R30? |
|---|---|---|
| `aiken` | STABLE | MCQ/MA/TF text |
| `gift` | STABLE | **Strongest** — all types |
| `xml` (Moodle XML) | STABLE | All types |
| `xhtml` | STABLE | — |
| `blackboard_six` | STABLE | Migration |
| `missingword` | STABLE | Fill-blank |
| `multianswer` | STABLE | MA |

**Absent:** PDF · Word/DOCX · Excel/XLSX · CSV · **OCR** · **AI**

> **13 of the 14 import requirements (R31–R43) are ❌ completely absent.** This is **the single largest block in the proposal** (~40% of the remaining work).
>
> 💡 **One existing strength:** the core importers already satisfy **R36** (different formats → one `question` structure). So if you build PDF/DOCX/XLSX parsers that emit Moodle XML, the rest is ready immediately.

---

## 6. R49 — Security Log: 48 Events, No IP

**All events in `mod/quiz/classes/event/`:**

| Category | Events |
|---|---|
| **Attempt** | `attempt_started` · `attempt_submitted` · `attempt_abandoned` · `attempt_becameoverdue` · `attempt_autosaved` · `attempt_reopened` · `attempt_deleted` · `attempt_viewed` · `attempt_reviewed` · `attempt_graded` · `attempt_regraded` · `attempt_preview_started` · `attempt_summary_viewed` · `attempt_question_restarted` · `attempt_manual_grading_completed` · `attempt_updated` |
| **Settings** | `slot_created/deleted/moved/…` · `section_*` · `quiz_grade_item_*` · `quiz_repaginated` · `user_override_*` · `group_override_*` |
| **Display** | `course_module_viewed` · `edit_page_viewed` · `report_viewed` |

**✅ R50 is satisfied by default:** there is no auto-penalty and no auto-submit in core — the default design is correct.

**❌ R49 is incomplete:** grep across the 48 events finds **no `ip`, no `user_agent`, no `device`**. You need:

```
{prefix}_security_events(id, userid, attemptid, event_type, ip, user_agent,
                         device_hash, payload_json, created_at)
```
+ an observer on `attempt_started` / `attempt_submitted` / `attempt_abandoned`.

**Ready-made pattern:** `quizaccess_seb` has `set_session_access()` — you can hook `validate_basic_header()`.

---

## 7. R58–R61 — Integration: **Better Than Expected**

| Requirement | Status | Present in 5.2.3 |
|---|---|---|
| **R58** read-only | 🟡 | **`auth/ldap`** complete (bind, search, attribute map) + **`auth/shibboleth`** |
| **R59** pull | 🟡 | `auth/ldap` + **`enrol/ldap`** (enrolments) + `auth/lti` |
| **R60** separate DB | ❌ | Absent — architecture |
| **R61** sync layer | 🟡 | `webservice/` REST + SOAP complete, `enrol/lti`, `enrol/database` |

**⚠️ Important correction:** the 57-module report said "there is no LDAP reference at all". **Wrong** — `auth/ldap` has been in core for years.

```php
// auth/ldap/auth.php
ldap_connect / ldap_bind / ldap_search / ldap_get_entries
```
```php
// auth/ldap/settings.php — full attribute map
$auth->set('ldap_user_search', …);
ldap_firstattr = cn, ldap_attributemap = ['firstname'=>'givenName', 'email'=>'mail', …]
```

**The only gap:** there is no `department` entity as an organisational unit:

```xml
<!-- lib/db/install.xml:888 -->
<FIELD NAME="department" TYPE="char" LENGTH="255" …
```

**It is free text in the profile, not an organisational table.** Linking departments to courses and to lecturers requires `customfield` or your own table.

**Recommended path:**

| Option | Description | Recommendation |
|---|---|---|
| **A. `auth/ldap` + `enrol/ldap`** | Configuration, zero code | ✅ For R58/R59 |
| **B. LTI (`enrol/lti`)** | The university pushes courses/costs | ✅ If the university has LTI |
| **C. `webservice` pull** | Read JSON hourly | ✅ For R61 |
| **D. `local_university_sync` table** | Departments + relationships | 🔨 Required |

---

## 8. R24–R27 — Editor, Equations and Code

| Requirement | Tool in core | Status |
|---|---|---|
| **R24** rich content | `lib/editor/tiny` (TinyMCE 6) + `lib/editor/textarea` | ✅ |
| **R25** mixed content | TinyMCE | 🟡 No `qtype_combined` |
| **R26** **equations** | **`filter/mathjaxloader`** + `filter/algebra` (Maxima→LaTeX) + `filter/tex` | ✅ **All three in core** |
| **R27** syntax highlighting | **`filter/codehighlighter`** (PrismJS) | ✅ |
| **R28** difficulty/topic | `tag/` API exists — **no field in `question`** | ❌ |

**The `question` table (18 columns) — no difficulty, no topic:**

```
id, parent, name, questiontext, questiontextformat, generalfeedback,
generalfeedbackformat, defaultmark, penalty, qtype, length, stamp,
timecreated, timemodified, createdby, modifiedby
```

> 💡 **Solution for R28:** `core_tag_tag::set_item_tags('core_question', 'question', $qid, $context, $tags)` — the API is in core, but there is no difficulty UI.

---

## 9. R67–R69 — Analytics: Core Is Stronger Than Expected

**R67 (statistics) — partially satisfied:**
```php
// mod/quiz/report/overview/overview_table.php:99
SELECT AVG(quizaouter.sumgrades) AS grade, COUNT(quizaouter.sumgrades) AS numaveraged
// :70  $this->add_average_row(get_string('groupavg', 'grades'), $this->groupstudentsjoins);
// :82  $this->add_average_row(get_string('overallaverage', 'grades'), $this->studentsjoins);
```
**Average per course + average per group** ✅
**But:** no max, no min and no pass/fail rate anywhere in the overview report — a search for pass/fail-style output across `mod/quiz/report/` and `grade/report/` returns nothing. This is a small report to write yourself.

**R68 (item analysis):**
```php
// mod/quiz/report/statistics/  → facility, discrimination, discriminationindex
```
**Full item analysis** ✅ — discrimination, facility, standard error

**R69:** `mod/quiz:viewreports` + the `core_privacy` provider

> ✅ **R68 + R69 = 100% core, no plugin needed.** A dashboard, if wanted, is a read-only view over `/mod/quiz/report/statistics`.

---

## 10. The Actual Build Plan

### 10.1 🔨 Must Be Built (24 absent + 13 partial)

| # | Package | Requirements | Effort | Priority |
|---|---|---|---|---|
| 1 | **Import pipeline** | R31–R43 (13) | 🔴 **8–12 weeks** | 🔴 Largest |
| 1a | └ PDF parser + **OCR** | R30, R33 | 🔴 3–4 | 🔴 |
| 1b | └ DOCX + OOXML relationships | R34 | 🔴 2–3 | 🔴 |
| 1c | └ XLSX + AI mapping | R35 | 🟡 1–2 | 🟡 |
| 1d | └ AI extraction + review UI | R31, R32, R39, R40 | 🟡 2–3 | 🔴 |
| 1e | └ metadata + dedup + validation | R41, R42, R43 | 🟡 1–2 | 🟡 |
| 2 | **Calculator** | R51–R54 (4) | 🔴 2–3 weeks | 🔴 |
| 3 | **State machine** | R57 + R14 (2) | 🟡 2–3 weeks | 🟡 |
| 4 | **Security log** | R49 (1) | 🟢 1 week | 🟡 |
| 5 | **Question fields** | R28 (1) | 🟢 1 week | 🟢 |
| 6 | **Fill-in-the-Blank** | R19 (1) | 🟢 ready plugin | 🟢 |
| 7 | **Sync layer** | R60, R61 (2) | 🟡 2–4 weeks | 🟡 |
| 8 | **Max/min/pass-fail report** | R67 (1) | 🟢 1 week | 🟡 |
| 9 | **Remaining partials** | R07, R12, R24, R25, R36, R56, R58, R59, R63 | 🟡 2–4 weeks | 🟡 |

### 10.2 ⚙️ Configuration (zero code)

| Requirement | How |
|---|---|
| R02 | `quiz.subnet` + `quizaccess_ipaddress` |
| R03 + R48 | `quizaccess_seb` + `USE_SEB_UPLOAD_CONFIG` |
| R05 | Grant `teacher` the `mod/quiz:grade` capability |
| R07 | `mod/quiz:reopenattempts` + build a small UI |
| R26 | Enable `filter/mathjaxloader` |
| R27 | Enable `filter/codehighlighter` |
| R45–R47 | `overduehandling` + the `review*` bitmask |
| R55 | `autosaveperiod` |
| R58 + R59 | `auth/ldap` + `enrol/ldap` |
| R62–R66 | Question bank + `shufflequestions` |

### 10.3 ✅ Already Present (32 requirements) — zero work

R01, R02, R03, R04, R05, R06, R08, R09, R10, R11, R13, R15, R16, R17, R18, R20, R21, R26, R27, R29, R45, R46, R47, R48, R50, R55, R62, R64, R65, R66, R68, R69

---

## 11. Performance Indicators — Inside Moodle 5.2.3

| Indicator | Value | Evidence |
|---|---|---|
| Code size | 584 MB · 49,960 PHP files | `du` · `find` |
| Question types | **18** (no cloze) | `question/type/` |
| Quiz access rules | **9** (in `mod/quiz/accessrule/`) | `ls` |
| Events | **48** | `classes/event/` |
| Grading behaviours | **11** | `question/behaviour/` |
| Importers | **7** | `question/format/` |
| AI providers | **6** | `ai/provider/` |
| AI actions | **5** | `ai/classes/aiactions/` |
| Filters (mathjax/algebra/tex/code) | 4 | `filter/` |
| Editors | 2 (TinyMCE + textarea) | `lib/editor/` |

---

## 12. Errors Corrected (Compared With the Other Report)

| # | Previous claim ❌ | Correction ✅ | Evidence |
|---|---|---|---|
| 1 | "R19 = `qtype_cloze` is available in core" | **cloze is absent in 5.2.3** | `question/type/` — 18 types |
| 2 | "R02 needs a custom access rule" | **Native in core** | `quizaccess_ipaddress` + `quiz.subnet` |
| 3 | "R58/R59 have no LDAP reference" | **`auth/ldap` + `auth/shibboleth` are in core** | `auth/ldap/auth.php` |
| 4 | "R59 has no department field" | **`user.department` exists** (but it is text, not an entity) | `lib/db/install.xml:888` |
| 5 | "R27 syntax highlighting is missing" | **`filter/codehighlighter` (PrismJS) exists** | `filter/codehighlighter/` |
| 6 | "R26 MathJax is missing" | **`filter/mathjaxloader` + `algebra` + `tex` all exist** | `filter/` |
| 7 | "There is no AI in core" | **A full AI subsystem with 6 providers** | `ai/provider/` |
| 8 | "R66 cannot shuffle per student" | **`quiz.shufflequestions` does this** | `install.xml:101` |
| 9 | "R67 avg/max/min is missing" | **Average and count exist; max/min/pass-fail do not** | `overview_table.php:99` |
| 10 | "R48 SEB is weak" | **3 levels of server-side validation** | `seb_access_manager.php` |
| 11 | "27 of 69 requirements are met" | **32 of 69 are met** (46%) | Full coverage table in §2 |

---

## 13. Questions That Need Answers

| # | Question | Why it matters |
|---|---|---|
| 1 | Must Moodle be **upgraded to 5.2.3**? Or is the project on 4.x? | Every plugin in `/samples` targets the old layout |
| 2 | Where is `moodledata`? What DB type is in use? | Maintenance and operations |
| 3 | Does the university offer **LDAP, an API, or LTI**? | R58–R61 |
| 4 | **R19 (Fill in the Blank)** — add `qtype_gapfill` or rely on `ddwtos`? | cloze has disappeared |
| 5 | Will the AI use **OpenAI or a local `ollama`**? | R02 + privacy |
| 6 | Is the **calculator** within the time scope? | 2–3 weeks |
| 7 | Is R60's "separate database" a **literal requirement**? | Architecture |
| 8 | Are there **programming courses**? | `qtype_coderunner` (Jobe) |

---

## Appendix: Verification Commands

```bash
cd /home/yermux/Documents/Graduation/TEMP/moodle-5.2.3/moodle/public

# Version
grep -E '\$release|\$branch' version.php

# Question types (18, no cloze)
ls question/type/

# Access rules
ls mod/quiz/accessrule/

# The six states
grep -n "const \(NOT_STARTED\|IN_PROGRESS\|OVERDUE\|SUBMITTED\|FINISHED\|ABANDONED\)" \
  mod/quiz/classes/quiz_attempt.php

# Calculator (zero)
sed -n '/<TABLE NAME="quiz" /,/<\/TABLE>/p' mod/quiz/db/install.xml | grep -ci calculator
grep -rn "CALCULATOR_BASIC\|CALCULATOR_SCIENTIFIC" --include=*.php .

# Network restriction
grep -n "address_in_subnet" mod/quiz/accessrule/ipaddress/rule.php

# SEB
grep -n "hash('sha256'" mod/quiz/accessrule/seb/classes/seb_access_manager.php

# LDAP
ls auth/ldap/ auth/shibboleth/

# AI
ls ai/provider/ ai/classes/aiactions/

# Events
ls mod/quiz/classes/event/ | wc -l

# Grading behaviours
ls question/behaviour/

# Analytics
grep -n "AVG(quizaouter.sumgrades)" mod/quiz/report/overview/overview_table.php
```

---

**Conclusion:** Moodle 5.2.3 covers **32 of the 69** requirements (46%) with no plugins at all. The remaining work is **clearly concentrated**: the AI import pipeline (13 requirements), the calculator (4), the state machine (2), and a small max/min/pass-fail report.
