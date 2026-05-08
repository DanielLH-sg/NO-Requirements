# Replicated specification document
I'll attempt to replicate [Feature #26 - Archive UI update (in ConfigSet)](https://swissgrid.sharepoint.com/sites/sgteam000097-ConfigSet/_layouts/Doc.aspx?sourcedoc={89F86EB9-F76D-473D-8ED8-3981C10A6E52}&wd=target%28Features.one%7C35A97D03-4EBB-4F4B-A5C8-F21500E04120%2FFeature%20%2326%20-%20Archive%20UI%20update%20%28in%20ConfigSet%7C4C119274-737F-4BBF-8039-9764D87B6619%2F%29)


# Reference documentation:
* Focussing on the UI: Feature #26 - Archive UI update (in ConfigSet); 
* This page Feature #39 - Archiving Concept defines the big picture & key concepts
* ops-archivierung/docs/archivierung.md at main · Swissgrid-AG/ops-archivierung 
* Archivierungsworkflow.loop (possibly out of date)
* Archiving Concept  Archive Concept.docx  (possibly out of date)
	

# Requirements
## Why (purpose)
### Pain
		Despite large storage, the ConfigSet/OPS application deals with large data. We therefore need to find a way to remove obsolete data without excessively impacting the work of the Operators. The Archive functionnality aims at preventing following pains:
#### data container gets full and causes application failure
#### Relevant ConfigSet/OPS data is no more available when needed by the Operator
		
		The purpose of this feature is therefore to find the right trade-off between data availability and storage costs (one of the cost being here service failure cause shares full).
		
### To-be user requirements (User needs)
[These User Requirements have been elicitated from Valentin B. & Simon B. end 2025 in contex of Feature #26 - Archive UI update (in ConfigSet)]: 

			i. The operator shall be able to view if and when a ConfigSet is to be archived.
			ii. The operator shall be able to prevent the archival of a ConfigSet so that this particular ConfigSet remains available (note: "prevent" might get interpreted as "postpone sufficiently later").
			iii. The operator shall be able to schedule the archival of a ConfigSet.
			iv. [OUT OF SCOPE] The operator shall be able to request the retrieval of an archived ConfigSet:
				1) This operation may not be available from OPS/ConfigSet application Gui, in which case it shall be available to a TE engineer / (e.g. the EOD);
				2) Such a restored ConfigSet shall be viewable from the ConfigSet application [Constraint] It is OK if calculation cannot be performed on such a restored ConfigSet. 
			v. The duration after which a ConfigSet is archived shall be aligned with the time horizon of the concerned operator process  - e.g. different archival period for IDCF, DACF, WVP etc.
		
## Who (stakeholders)
		a. MA lead: Valentin B. (together with Simon B.)
		b. MA Owner for Presets WVP/MVP/JVP/Study -> Simon B.
		c. MA Owner for Presets IDCF DACF D2CF ->  Samuel L. resp. + Mael G.  FO team
		d. MA Owner for Presets OPFV** -> Emma Laub
		e. MA Owner for Presets EPW** incl EPV JVP -> Ronja ST
		f. PO for OPS Cluster - Michael Uggowitzer - > Coordination
		g. TE Dev Archi Db:  Michael Mahler
		h. TE follow up & stakeholder alignment:  Daniel Lucas-Hirtz 
		i. GUI -> Marion Bontron (Iconics)
		j. DB Frank Urmoneit 
	
## Where (context)
	
	
	
## What (Key Archive Concepts)
	
Key functions related to archive are:
		○ Archive
		○ Clean-up (meaning "delete")
		○ Restore (which is Out of Scope of our current initiative)
<img width="609" height="301" alt="image" src="https://github.com/user-attachments/assets/94f80b37-8af2-44c0-ba03-1f2ea5195226" />
	

	
## What (Functional Product Requirements Specifications)
	
		a. User actions
		
		For each ConfigSet which is listed in the OPS Dashboard tab, the OPS/ConfigSet application shall allow the operator to:
			i. View the scheduledDateOfArchival of this ConfigSet (that is, when the ConfigSet is planned to be Archived)
			ii. Mark the ConfigSet for archive 
				1) Mark for archive shall set the scheduledDateOfArchival of this ConfigSet to, per default, the day after today, 
				2) this proposed date is editable to any date from the day after today up to MaxArchiveDelay after today. 
				3) MaxArchiveDelay is [2 years - to be confirmed - is a DB parameter]
				4) As the operator is proposed to edit the scheduledDateOfArchival as above, the current scheduledDateOfArchival shall be visible.

		b. Log of archive actions
		
		The OPS/ConfigSet application shall log the Mark for archive Operator actions and keep track of following information:
			i. Who-log: identity of the requester
			ii. When-log-date-of-request: date of the request
			iii. When-log-requested-archive-date: requested archive date
			iv. Why-log: The OPS/ConfigSet application shall force the operator to provide a rationale for the archive request.
			
		The OPS/ConfigSet application shall allow the Operator to view the logs of a given ConfigSet (that is, the list of all historical logs for this ConfigSet). 
			
		c. Initial value of scheduledDateOfArchival 

		Upon creation AND upon restoration of a ConfigSet, the OPS/ConfigSet application shall set the scheduledDateOfArchival to: 
			[DelayBeforeArchive (defined below) + LATEST-OF (today; end date of the ConfigSet time period)]
			In English: scheduledDateOfArchival is set to the sum of the DelayBeforeArchive duration and the latest of today and the end date of the ConfigSet time period.
			
			Value of DelayBeforeArchive depends on the preset:
				1) WVP W-1 –> 31 days
				2) WVP W-4 –> 60 days
				3) MVP NSA –> 90 days
				4) IDCF –> to be defined by Daniel LH with the MA process owner
				5) OPFV(*) –> to be defined by Daniel LH with the MA process owner
				6) DACF  –> to be defined by Daniel LH with the MA process owner
				7) D2CF  –> to be defined by Daniel LH with the MA process owner
				8) EPW(*) –> to be defined by Daniel LH with the MA process owner
				9) JVP  –> to be defined by Daniel LH with the MA process owner
				10) Study –> to be defined by Daniel LH with the MA process owner
				
			[For Information] 
				1) end date of the ConfigSet time period depends on the preset, it can be in a few days for a daily preset (e.g. DACF), or in more than one year for a yearly preset.  
				2) As example, if you create a WVP ConfigSet in the past e.g. 2024, its scheduledDateOfArchival will be set to "today + 31 days". And you'll be able to postpone this archival date as explained below.

			[For Information] The flags ToBeArchived and DoNotArchive are removed and replaced by the scheduledDateOfArchival mechanism defined above. 
		
		d. Can not list archived ConfigSets
		[Constraint] OPS/ConfigSet application does not allow the operator to list the ConfigSets which have been archived or deleted. 
[Rationale] the number of archived ConfigSet may grow large, and ensuring a consistent display and search GUI for Archived COnfigSets would be costly. 
		
		e. Archive process

			i. Every night starting at TIME-ARCHIVE the OPS/ConfigSet application shall archive the ConfigSets which scheduledDateOfArchival has been reached (TIME-ARCHIVE is sometime in the middle of the night). 
			ii. "Archive" a ConfigSet above means 
				1) move the ConfigSet archive data from the LIVE environment to the Archive environment (see context diagram above). 
				2) "moving" here means that the data is no longer available on PROD after archive
			iii. search and view non-GUI tool: 
				1) The OPS/ConfigSet non-GUI administration toolset shall allow the TE Engineer (e.g. the EOD) to search for and view a ConfigSet which have been archived on the Archive DB by the OPS/ConfigSet application. Search criteria to be at least ConfigSet ID and name.  
				2) This tool may be made available for MA operators. 
					
		f. Restore process
		
			i. Restore is Out of Scope of the first step
			ii. Second Step - plan: 
				1) [Constraint] OPS/ConfigSet GUI application does not allow the operator to restore a ConfigSet which has been archived. 
[Rationale] same as for "listing" above - effort. 
				2) The OPS/ConfigSet non-GUI administration toolset shall allow the TE Engineer (e.g. the EOD) to restore a ConfigSet - that is, to restore the "ConfigSet archive data" defined above.
				3) "Restore" means the opposite operation as "Archive" above:
					a) move the ConfigSet archive data back from the Archive environment to the PROD environment (see context diagram above). 
					b) Which (may) includes both DB data & File referred data 
				4) There may be restrictions in the restored ConfigSet
					a) Either some actions are not available (e.g. retricted from viewing only, PTI module calculation are not available)
					b) Or some data is not available (TBD)
					c) Or both
					
		g. Clean-up process
		Clean-Up process (data deletion) has no operator UI & is out of scope of this document. 
See  Feature #39 - Archiving Concept .
						
	
## How
			
		a. How (GUI)
		As defined above, following actions are to be available on the UI for the Operators:
			i. View the scheduledDateOfArchival of a ConfigSet 
			This action shall be available as a column in the table of ConfigSet of the OPS Dashboard tab.
[Constraint] not too large. "ArchiveDate" ? "DateOfArchival" ?

			ii. Mark the ConfigSet for archive 
This action shall be available as a right click menu action on a ConfigSet of the table view of the OPS Dashboard tab - replacing the action "Archive" which should be removed. See illustration below (is-situation). Action text is "Mark for archive". 


			
			iii. View the archive actions logs
			
[BrainStorm with Dimitri & Misha 22.04.26]
				□ An "I" or "Edit" icon next to the "scheduledDateOfArchival " column, whithin each ConfigSet, where the user can access the 
Logs.

				
				□ Alt. Right click "MarkForArchive" leads to a single popup similar as below containing as well the "MarkForArchive" datae edition, the mandatory log text input, and the history of logs.

			

		b. How (Dynamic behaviour)
		Intentionally empty
		
		c. How (Static data model)
Intentionally empty
		
		d. How (Key Quality & Performance)
Intentionally empty
		
		e. How (Solution design)
Intentionally empty
		
		
	 
# Verification test cases:

Here TE's verification test strategy for this feature.
Note that we mean here strictly the "Verification" test, i.e. step 3 of the test strategy defined in paragraph 4 below.
The "verification" test is to be performed on INT, by TE, after integration, and before the Validation test phase from MA on INT (see the "test steps" below).
Purpose of Verification test phase is to check that the behaviour and characteristics of the realized solution are conform with the specifications.

	1. Test dataset:  
		a. configset ID x y z
		b. A custom configset with XYZ 
	
	2. Verification test - What (Functional Product Requirements Specifications)
	
	For a relevant part of the test dataset defined above, for each requirement statements in the chapter 4. "What (Functional Product Requirements Specifications)" above, test whether the behaviour and characteristics of the application is conform. 
	
	3. Verification test - How 
	
	For a relevant part of the test dataset defined above, for each requirement statements in the chapter 5. "How" above, test whether the behaviour and characteristics of the application is conform.
	
	4. Overall Test strategy
	Here the test phases which may be performed during the realisation of this feature. 
	In this document we focus strictly on the phase 3 "Verification tests":
		a. Unit tests (during development, typically on the various TEST sub-systems, from TE or the realisation team e.g. ICONICS)
		b. Integration tests (during integration on INT, typically on INT, from TE, purpose is to check the integration of the feature in the overall system)
		c. Verification tests (before release for validation on INT, on INT, from TE, purpose is to check that the application is conform to its specifications)
		d. Validation tests (possibly on INT, from MA, purpose is to check that the needs & intended characteristics are fulfilled)
		e. Regression tests (possibly on INT, from MA, purpose is to check that the inclusion of the feature has not caused regression on other functions and characteristics of the overall application)
		f. Smoke tests (sampling test, on PROD, after roll out, from MA, purpose is to quickly check that the INT behaviour is indeed reflected on PROD as expected - with a small subset of tests)
	
						

# Historical discussions: 
(one row one statement)
Name	Datetime	Statement / Question / Answer
Daniel LH	2.09.2025	Today's UI of the ConfigSet Archive process has some flaws (see "Pain list" below).  Purpose of this feature is to get a consistent user interaction. 
Initial alignment meeting with Valentin Bürgler & Simon Baumgartner, Michael Mahler & Daniel Lucas-Hirtz.
Daniel LH	~5.09.2025	Alignment on a concept in the chat MS Teams Chat "Archivierung Detail Specs" with the four above.
Michael Uggowitzer	17.09.2025	Leankit card created  TE Quartalsziele (L2) 
		
Gerold F.	22.10.2025	As-is archive behaviour has caused an issue during Regression tests - documented in Issue R25.3#12 - Configsets deleted  
The team has decided to deactivate the archival process on INT in order to avoid further issues during Regression tests.
Purpose is to have the issue fixed by the current feature #26 
Daniel LH	27.10.2025	Detailed design & specs documented below. Some open question. I need to align first with TE (& Frank) then we'll feedback with MA.
Misha & Daniel	6.11.25	[Team meeting 6.11.25] We've decided to deliver a first MVP with "Scheduled date of archival set to: [DelayBeforeArchive + LATEST-OF (today; end date of the ConfigSet time period)]". Realized by Micha with https://agilesg.leankit.com/card/31512306800997
Daniel LH	15.04.25	Refactoring this spec following GitHub issue 🔨[Impl]: UI changes for Archiving · Issue #426
Any further discussion to be tracked in GitHub (either #426 or  Concept Archiving · Issue #427)

**********************************************************************************************************************
		Grey text highlight: as-is behaviour (as existing end of 2025).
		


************************************************************************************************************************
Below working documents, please ignore, partly Wrong
<img width="915" height="4398" alt="image" src="https://github.com/user-attachments/assets/956cd4d3-7620-45dc-925a-b80c369e65c1" />
