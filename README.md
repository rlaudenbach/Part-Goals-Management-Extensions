# Part Goals Management extensions v1
- The purpose of this add-on package is to extent the Part Goals functionality of Product Engineering.
- Target values are now decoupled from the other goal values (separate rollups). 
- Thus Target values can be maintained on parts where applicable  (likely on higher and top levels of the BOM).
- Other goal values now have a precedence of: actual_value --> estimated_value --> calculated_value
- Goal rows can no longer be added or deleted manually. Actions for Administrators are to be used instead.
- An action to recalculate the rollup within a BOM structure is available 
	(replaces the "Rollup all parts in DB" action)
- Rollup calculations are also triggered automatically from events on "Part Goal" and “Part BOM”.
- A method to rollup all Part Goals in DB is available for Administrators.
- Two new reports are added for target costing and target weight. 
	- The new reports apply color coding to indicate when target values are met or missed.
- Generic rollup utilities are used.
	- These allow a "Rollup Definition" by a structure relationship type . Here "Part BOM"
	- A tested configuration of the "Rollup Definition" used for "Part BOM" is part of the package

Future enhancements:
	- Currently rollups are done regardless whether parts are released or not. (as done in OOTB Product Engineering)
	- For released parts values should not be changed, but a “not current flag” should be set. 
	  Or a separeate "released_goal" value should be maintained. That is not changed on released parts.

## Pre-requisites
Aras Innovator Release: 12SP5. 12SP8, 12SP9
Product Engineering Release: 12R2
Rollup Calculation Utitlities v1

## Implementation Details
This package installs on top of standard PE. It extends these elements. 

- Part (ItemType) - addl. Properties (cost_target,cost_unit, weight_target, weight_unit) 
- Part (Form) - added new fields for properties from above 
- Part Goal (RelationshipItemType) - addl. Properties (goal_unit) 
- New Logic on Part (onAfterUpdate, onAfterAdd) - this logic overrides "PE_RollupPart" and actually does nothing. Rollup are no triggerd in different places
- New Logic on Part Goal rows (onAfterUpdate) - triggers the rollup action, if manual values for cost or target are saved
- New Logic on Part Goal rows (onBeforeAdd, onBeforeDelete) - throws an error to prevent manual add and delete actions
- New Logic on Part BOM rows (onAfterUpdate, onAfterDelete) - to set a default unit for "Cost" or "Weight"

- New Action "Add PartGoals to Structure (down)" - Addd part goal rows to all parts in BOM down, if goals do not exist.
- New Action "Delete PartGoals from Structure (down)" - permissions for Administrators only. Deletes all part goal rows of a selected type in BOM down. the runs goal rollups on all parts botton up.
- New Action "Calculate All Part Goals in BOM" - overrides standard action "PE Rollup all Parts in DB". New action resolves BOM down and run rollups on all parts bottom up.

- New Query Definition "Part_CostAndWeightGoalsReport" to resovlve BOM Structures down and return goal related properties to be rendered by XSLTs of the new reports.
- New Report "BOM Cost Goals Report" - "cloned" and extended columns and formatting  for cost_unit and cost_target and added color coding (green, red) when compared against "target_cost" column. 
- New Report "BOM Weight Goals Report" - cloned cost goals report using weight columns instead of cost columns.

For Administrators to use. New Method: "Adm PartGoals Update Calculation"
	It can be run (with action: Run Server Method) to do Part Goals rollups on all parts in DB.  
	It creates a list of all parts without children. And then runs the goals rollup for each of these parts bottom up.

NOTE:
The rollup and update methods use apply SQL intentionally.
For rollups, permissions and item locks can be ignored.
The original rollup logic in PE also uses the same approach.

## Installation

The installation can be done in 2 ways:

1. Automated: Using the Aras Update tool (recommended) - Update version 1.9 or higher
2. Manually: Using the packages import tool 

NOTE: in both cases you must do the manual configuration steps as administrator described in section "Post Import Steps" !!!

1. Backup your database and store the BAK file in a safe place.

### Automated: Using Aras Update

Log on as "admin"

1. start the Aras Update Tool
2. choose "local"
3. Click on "Add package reference"
4. On file dialog find installation folder "Installation" (where you extracted the solution installation package locally) and click "OK"
5. Click on "install"
6. Review the install selection. Click on "Next"
7. Select "Detailed logging" and click on "Next"
8. Click on "Browse" and select your Aras installation folder (choose folder "Innovator")
9. Click on "Import into Innovator DB" and enter your connect parameters for "Innvator URL, Database, Innovator user (admin), User Password"
10. Click "Install" - wait until success message appears

NOTE: you must clear the browser cache of all clients after deploying the code tree patches.


### Manual Code Tree and Import Steps

#### Deploy Code Tree Changes
CodeTree patches are located in folder "...Installation\CodeTree\Innovator" of this solution

1. Select folder "Innovator" and copy it
2. Navigate to the root folder of your Aras installation.  (i.e. "C:\Program Files (x86)\Aras_Innovator")
3. Paste the copied folder to the Aras installation root.
4. Confirm "Overwrite" warning.

NOTE: you must clear the browser cache of all clients after deploying the code tree patches.

#### Import Packages
Import must be done in 4 steps in the indicated sequence.

Import packages are located in folder "...Installation\imports" of this solution

Use Package Utilities and log on as "admin"

1. select path to mf file "1_imports.mf"  --> click import button	- imports "Rollup Calculations Utitlities"
2. select path to mf file "2_imports.mf"  --> click import button	- imports "Part Goals Management Extenstions"
3. select path to mf file "3_imports.mf"  --> click import button	- imports modifications to PE (PLM)
4. select path to mf file "4_imports.mf"  --> click import button   - imports a configuration of "Rollup Definition" for "Part BOM / Part Goal"


## Usage
If using the Makerbot demo database, and after installing Part Goal Management extensions, do these steps:
- open top part of Makerbot
- run action "Add PartGoals to Structure (down)"
- run action "Calculate All Part Goals in BOM"
- do a refresh on part
- run report "BOM Cost Goals Report"

## Credits


## License
This project is published under the MIT license. See the [LICENSE file](./LICENSE.md) for license rights and limitations.)
