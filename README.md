# **Generating EJScreen RMP Facility Proximity**

EJScreen uses Apache Hadoop pig scripts to generate Risk Program Management (RMP) facility proximity. The Pig scripts were developed using Esri's [GIS Toolkit for Hadoop](https://esri.github.io/gis-tools-for-hadoop/) toolkit. It was run in an AWS EMR cluster environment. The source data came directly from EPA's RMP database with all active sites. The proximity process involves Pre-Hadoop processing, running Hadoop Pig scripts, and Post-Hadoop processing. The end results are Census block-group based proximity scores.

**Pre-Hadoop Processing:**

- Create a geodatabase table RMP_Work.gdb based on the download dataset (RMP_021623.csv).
- Drop records outside US and PR and create table RMP\_021623\_forHadoop.
- Export records to RMP\_021623\_forHadoop.csv with these columns: EPA\_ID, LATITUDE, LONGITUDE, CWEIGHT. Note that EPA\_ID is PGM\_SYS\_ID from the Envirofacts source file and CWEIGHT = 1 for all records.

**AWS Hadoop Processing:**

- Start an AWS EMR cluster.
- Rather than doing 52 separate state runs, combine the states into 22 state groupings.
- Run Step 1 for each of 22 state groups to generate weighted distances facility-block pairs. See **RMP\_US01\_02\_72\_proximity\_Step1.txt** for Pig script example.
- Run Step 2 for each of 22 state groups to generate block group summary for each subgroup. See **RMP\_US01\_02\_72\_proximity\_Step2.txt** for Pig script example.
- Use Athena to create BG-level results tables and download as csv files, for example, BG\_Scores\_01.csv to OutputfromHadoop folder.
- Repeat all steps for each state group.

**Post-Hadoop Processing:**

- Combine all BG score text files in OutputfromHadoop folder into one file (RMP\_BG\_Scores\_US.csv).
- Prep with text editor (Capitalize first header row and remove all other header rows).
- Import US csv file to Excel; make sure BLKGRP is text.
- Add the Excel file to the geodatabase (RMP\_Work.gdb) as a new table (RMP\_BG\_Scores\_US).
- Rename columns to STCNTRBG and BG\_SCORE, and name table RMP\_BG\_Scores\_Final.
- Provide datasets for testing. First create RMPProximity\_Testing.gdb
- Include US\_BG\_Scores\_Final and RMP\_021623\_forEJ tables.
- Add US\_RMPProx\_BG with BG shapes and BG\_SCORE, set NULL Scores to 0.

**EPA Disclaimer**

The United States Environmental Protection Agency (EPA) GitHub project code is provided on an "as is" basis and the user assumes responsibility for its use. EPA has relinquished control of the information and no longer has responsibility to protect the integrity, confidentiality, or availability of the information. Any reference to specific commercial products, processes, or services by service mark, trademark, manufacturer, or otherwise, does not constitute or imply their endorsement, recommendation or favoring by EPA. The EPA seal and logo shall not be used in any manner to imply endorsement of any commercial product or activity by EPA or the United States Government.
