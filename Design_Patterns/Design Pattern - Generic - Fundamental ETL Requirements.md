# Design Pattern - Generic - Essential ETL Requirements

## Purpose
The purpose of this Design Pattern is to define a set of minimal requirements every single ETL process (mapping, module or package) should conform to. These essential design guidelines impact how all ETL processes behave under the architecture of the Data Integration Framework.

## Motivation
Regardless of (place in the) architecture or purpose every ETL process should be created to follow a distinct set of base rules. Essential concepts such as having ETL processes check if they have already run before inserting duplicates or corrupting data makes testing, maintenance and troubleshooting a more straightforward task. The ultimate motivation is to develop ETL which cannot cause errors due to unplanned or unwanted execution. Essentially, ETL must be able to be run and re-run at any point in time to support a fully flexible scheduling and implementation.

## Applicability
This Design Pattern applies to every ETL process.

## Structure
The requirements are as follows:
* ETL processes contain transformation logic for a single specific function or purpose (atomicity). This design decision follow ETL best practices to create many ETL processes that each address a specific function as opposed to few ETL processes that perform a range of activities. Every ETL process attempts to execute an atomic functionality. Examples of atomic Data Warehouse processes (which therefore are implemented as separate ETL processes) are key distribution, detecting changes and inserting records.
* Related to the above definition of designing ETL to suit atomic Data Warehouse processes, every ETL process can read from one or more sources but only write to a single target.
* ETL processes detect whether they should insert records or not, i.e. should not fail on constraints. This is the design decision that ETL handles referential integrity and constraints (and not the RDBMS).
* ETL can always be rerun without the need to manually change settings. This manifests itself in many ways depending on the purpose of the process (i.e. its place in the overall architecture). An ETL process that truncates a target table to load form another table is the most straightforward example since this will invariably run successfully every time. Another example is the distribution of surrogate keys by checking if keys are already present before they are inserted, or pre-emptively perform checksum comparisons to manage history. This requirement is also be valid for Presentation Layer tables which merge mutations into an aggregate. Not only does this requirement make testing and maintenance easier, it also ensures that no data is corrupted when an ETL process is run by accident.
* Source data for any ETL can always be related to, or be recovered. This is covered by correctly implementing an ETL control / metadata framework and concepts such as the Persistent Staging Area. This metadata model covers the audit trail and its ability to follow data through the Data Warehouse while the Persistent Staging Area enables a new initial load in case of disaster or to reload (parts of) the Data Vault.
* The direction of data is always ‘up’. The typical process of data is from a source, to Staging, Integration and ultimately Presentation. No regular ETL process should write data back to an earlier layer, or use access this information using a (key) lookup.  This implies that the required information is always available in the same Layer of the reference architecture.
* ETL processes must be able to process multiple intervals (changes) in one run. In other words, every time an ETL process runs it needs to process all data that it can (is available). This is an important requirement for ETL to be able to be run at any point in time and to support real-time processing. It means that ETL should not just be able to load a single snapshot or change for a single business keym but to correctly handle multiple changes in a single data set. For instance if the address of an employee changes multiple times during the day and ETL is run daily, all changes are still captures and correctly processed in a single run of the ETL process. This requirement also prevents ETL to be run many times for catch-up processing and makes it possible to easily change loading frequencies.
* ETL processes should automatically recover /rollback when failed. This means that if an error has been detected the ETL automatically repairs the information from the erroneous run and inserts the correct data along with any new information that has been sourced.

## Implementation guidelines
It is recommended to follow a ‘sortable’ folder structure to visibly order containers / folders where ETL processes are stored in a way that represents the flow of data. 

An example is as follows:
* 000_\<source systems\>, one for every source
* 100_Staging_Area
* 150_Persistent_Staging_Area
* 200_Integration_Layer
* 300_Presentation_Layer

ETL processes are recommended to be placed in the directory/folder where they pull data _to_. For instance the ETL logic for ‘Staging to History’ exists in the ‘150_History_Area’ folder and loads data from the ‘100_Staging_Area’.

## Consequences and considerations
In some situations specific properties of the ETL process may seem overkill or perhaps even redundant. This (perceived) additional effort will have its impact on developing duration. 

But in the context of maintaining a generic design (e.g. to support ETL generation and maintenance) this will still be necessary. Concessions may be made per architectural Layer (all ETL processes within a certain architecture step) but this is recommended to be motivated in the customised (i.e. project specific) Solution Architecture documentation.

## Related patterns
In the various Design and Implementation Patterns where detailed ETL design for a specific task is documented the requirements in this pattern will be adhered to.
