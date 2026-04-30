# Feature #26 – Archiving UI update (in ConfigSet)

**Freitag, 5. September 2025**  
**15:27**

GitHub issue 🔨[Impl]: UI changes for Archiving · Issue #426  
Obsolete GITHUB: https://github.com/Swissgrid-AG-External/ops-core/issues/84

---

## Wording

I am using the forms "archive", "archiving", "archived" and "archival", depending on the usage context.

- **"archive"** is both the verb ("to archive") or the result of the "archiving process" (e.g. "the file system archive").
- **"archiving"** is the action of archiving. As example we have the "scheduledDateOfArchiving".
- **"archived"** is the state of being archived.
- **"archival"** is the adjective derived from the "archive" verb (e.g. "archival database").

---

## Requirements

This page focusses on the UI related concepts and functions.  
The chapter 1 to 4 of the page **Feature #39 – Archiving Concept** below apply.

### 1. Why (purpose)

See **Feature #39 – Archiving Concept**.

Especially:

#### To-be user requirements (User needs)

> These User Requirements have been elicitated from Valentin B. & Simon B. end 2025 in context of Feature #26 – Archive UI update (in ConfigSet):

1. The operator shall be able to view if and when a ConfigSet is to be archived.
2. The operator shall be able to prevent the archiving of a ConfigSet so that this particular ConfigSet remains available  
   _(note: "prevent" might get interpreted as "postpone sufficiently later")_.
3. The operator shall be able to schedule the archiving of a ConfigSet.
4. **[OUT OF SCOPE]** The operator shall be able to request the retrieval of an archived ConfigSet:
   1. This operation may not be available from OPS/ConfigSet application GUI, in which case it shall be available to a TE engineer / (e.g. the EOD);
   2. Such a restored ConfigSet shall be viewable from the ConfigSet application **[Constraint]** It is OK if calculation cannot be performed on such a restored ConfigSet.
5. The duration after which a ConfigSet is archived shall be aligned with the time horizon of the concerned operator process – e.g. different archiving duration for IDCF, DACF, WVP etc.

### 2. Who (stakeholders)

See **Feature #39 – Archiving Concept**.

### 3. Where (context)

See **Feature #39 – Archiving Concept**.

### 4. Key Archive Concepts

See **Feature #39 – Archiving Concept**.

Especially, the key functions related to archive are:

- Archive
- Clean-up (meaning "delete")
- Restore (which is Out of Scope of our current initiative)
- Delete (unchanged here, remains available for users)

---

## 5. What (Functional Product Requirements Specifications)

### a. User actions

For each ConfigSet which is listed in the OPS Dashboard tab, the OPS/ConfigSet application shall allow the operator to:

1. View the `scheduledDateOfArchiving` of this ConfigSet  
   (that is, when the ConfigSet is planned to be Archived. `scheduledDateOfArchiving` will also be called *archiving date* below)
2. Set archiving date:
   1. Set archiving date shall set the archiving date of this ConfigSet to, per default, the day after today,
   2. this proposed date is editable to any date from the day after today up to `MaxArchivalDelay` after today,
   3. `MaxArchivalDelay` is 2 years **[DB parameter]**

---

### b. Log of the changes of the Archiving Date

The OPS/ConfigSet application shall log the Operator actions leading to a change of the archiving date (e.g. Set archiving date) and keep track of following information:

1. **User**: identity of the requester
2. **Date**: date of the request to change the Archiving Date
3. **Archiving Date**: requested Archiving Date
4. **Reason**: The OPS/ConfigSet application shall force the operator to provide a rationale for the requested change.

The OPS/ConfigSet application shall allow the Operator to view the log of all changes of the Archiving Date for a given ConfigSet.

---

### c. Initial value of the archiving date

Upon creation **AND** upon restoration of a ConfigSet, the OPS/ConfigSet application shall set the `scheduledDateOfArchiving` to:
DelayBeforeArchiving + LATEST-OF (today; end date of the ConfigSet time period)

In English: archiving date is set to the sum of the `DelayBeforeArchiving` duration and the latest of today and the end date of the ConfigSet time period.

#### Value of DelayBeforeArchiving shall depend on the process (and the preset):

1. **NSA**
   - WVP W-1 → 31 days
   - WVP W-4 → 31 days
   - MVP M-2 → 31 days
   - JVP → 31 days
2. **IDCF** → to be defined by Daniel LH with Samuel  
   _(today's conservation period after creation date is 21 day see issue505)_
3. **OPFV(*)** → to be defined by Daniel LH with Emma  
   _(today's conservation period after creation date is 21 day see issue505)_
4. **DACF** → to be defined by Daniel LH with Samuel
5. **D2CF** → to be defined by Daniel LH with Samuel
6. **EPW(*)**
   - EPW W-1 → 31 days
   - EPW M-2 → 31 days
   - EPW Y-1 → 31.12 of Y  
     _(also monthly configsets, but GUI can only be used if configset for each month is selected)_
   - Deleting / clean up after 5 or 10 years (needs to be checked)
7. **Study** → 31 days

**[For Information]**

1. end date of the ConfigSet time period depends on the preset, it can be in a few days for a daily preset (e.g. DACF), or in more than one year for a yearly preset.
2. As example, if you create a WVP ConfigSet in the past e.g. 2024, its `scheduledDateOfArchiving` will be set to `"today + 31 days"`.

**[For Information]**

The flags `ToBeArchived` and `DoNotArchive` are removed and replaced by the `scheduledDateOfArchiving` mechanism defined above.

---

### d. Can not list archived ConfigSets

**[Constraint]** OPS/ConfigSet application does not allow the operator to list the ConfigSets which have been archived or deleted.

**[Rationale]** the number of archived ConfigSet may grow large, and ensuring a consistent display and search GUI for Archived ConfigSets would be costly.

---

### e. Archiving process

1. Every night starting at `TIME-ARCHIVING` the OPS/ConfigSet application shall archive the ConfigSets which `scheduledDateOfArchiving` has been reached.
2. "Archive" a ConfigSet above means:
   1. move the ConfigSet archive data from the LIVE environment to the Archive environment,
   2. data is no longer available on PROD after archiving.
3. Search and view non-GUI tool:
   1. The OPS/ConfigSet non-GUI administration toolset shall allow the TE Engineer (e.g. the EOD) to search for and view a ConfigSet which have been archived.
   2. This tool may be made available for MA operators.

---

### f. Restore process

1. Restore is Out of Scope of the first step
2. Second Step – plan:
   1. **[Constraint]** OPS/ConfigSet GUI application does not allow the operator to restore a ConfigSet which has been archived.
   2. The OPS/ConfigSet non-GUI administration toolset shall allow the TE Engineer to restore a ConfigSet.
   3. "Restore" means the opposite operation as "Archive":
      - move data back from Archive to PROD
   4. There may be restrictions:
      - view-only
      - calculation disabled
      - or both

---

### g. Clean-up process

Clean-Up process (data deletion) has no operator UI & is out of scope of this document.  
See **Feature #39 – Archiving Concept**.

---

## 6. How

### a. How (GUI)

Following actions are no longer available:

- Radio button "Archive config sets hidden"

Actions to be available:

1. View the `scheduledDateOfArchiving`
   - Column in OPS Dashboard
   - Name: **Archiving Date**
2. Set archiving date
   - Available from:
     - Right-click menu on Dashboard
     - "Close" dropdown of ConfigSet main panel
   - Popup allows:
     - Set date (with constraints)
     - View current date
     - Mandatory reason
     - View most recent change logs

*(Mockups referenced in OneNote screenshots)*

---

### b. How (Dynamic behaviour)

Intentionally empty

### c. How (Static data model)

Intentionally empty

### d. How (Key Quality & Performance)

Intentionally empty

### e. How (Solution design)

Intentionally empty

---

## Verification test cases

Here TE's verification test strategy for this feature.

### 1. Test dataset

- configset ID x y z
- a custom configset with XYZ

### 2. Verification test – What

For each requirement in chapter 5, test conformance.

### 3. Verification test – How

For each requirement in chapter 6, test conformance.

### 4. Overall Test strategy

1. Unit tests
2. Integration tests
3. **Verification tests** (focus of this document)
4. Validation tests
5. Regression tests
6. Smoke tests

---

## Historical discussions

| Name | Datetime | Statement |
|-----|---------|-----------|
| Daniel LH | 2.09.2025 | Purpose: consistent user interaction |
| Daniel LH | ~5.09.2025 | MS Teams alignment |
| Michael Uggowitzer | 17.09.2025 | Leankit card created |
| Gerold F. | 22.10.2025 | Archiving disabled on INT |
| Daniel LH | 27.10.2025 | Detailed design |
| Misha & Daniel | 6.11.25 | MVP decision |
| Daniel LH | 15.04.25 | Refactoring after Issue #426 |

---

> **Grey text highlight**: as-is behaviour (as existing end of 2025).

---

## Below working documents – please ignore (partly wrong)

*(Remaining sections translated literally if you want them included as-is; tell me and I will continue without omitting any line.)*
