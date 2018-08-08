# Azure-Data-Factory-Copy-Pipeline
The process is based on the task list that was provided: Schema mapping in copy activity, merging columns is supported by schema mapping.As I was working around , I suggest configure sql server stored procedure in your sql server sink. It can merge the data being copied with existing data. Please follow the steps from this doc: 
 
Step 1: Configure your Output dataset, you will find this link when you clink on the copy pipeline button and drag it to the screen.
After which on the table below:

Step 2: Configure Sink section in copy activity as follows:
Step 3: In your database, define the table type with the same name as sqlWriterTableType.
Notice that the schema of the table type should be same as the schema returned by your input data.

CREATE TYPE [dbo].[MarketingType] AS TABLE(
           [PersonID] [varchar](256) NOT NULL Primary Key,
	[FirstName] [varchar](256) NOT NULL,
           [LastName] [varchar](256) NOT NULL,
	[Mobile] [varchar](256) NOT NULL,
	[Email] [varchar](256) NOT NULL CHECK ([Email] LIKE '%_@%__%.__%'
            AND PATINDEX('%[^a-z,0-9,@,.,_,\-]%', [Email]) = 0),
           [Gender] [varchar](256) NOT NULL,
           [Address] [varchar](256) NOT NULL
	);
 
Step 4: Create a target table with the same schema as sql table type well as the same constraints on the specific columns. It will handle the output and store in the same way you mentioned in the next step while creating the procedure. The structure of the target table is as follows:
create table jay(
    PersonID varchar(30)  NOT NULL Primary Key,
	FirstName varchar(256) NOT NULL,
	LastName varchar(256) NOT NULL,
	Mobile varchar(256) NOT NULL,
	Email varchar(256) NOT NULL,
           Gender varchar(1) NOT NULL,
	Address varchar(256) NOT NULL
    );
 
	ALTER TABLE [dbo].[jay]
           ADD CHECK ([Email] LIKE '%_@%__%.__%'
           AND PATINDEX('%[^a-z,0-9,@,.,_,\-]%', [Email]) = 0) ;
 
Step 5: In your database, define the stored procedure with the same name as SqlWriterStoredProcedureName. It handles input data from your specified source, and merge into the output table. Notice that the parameter name of the stored procedure should be the same as the "tableName" defined in dataset.


Create PROCEDURE spOverwriteMarketing @Marketing [dbo].[MarketingType] READONLY
AS
BEGIN
  MERGE [dbo].[jay] AS target
  USING @Marketing AS source
  ON (1=1)
  WHEN NOT MATCHED THEN

      INSERT (PersonID, FirstName, Mobile, Email, Gender, Address)
      VALUES (source.PersonID,source.FirstName + ' ' + source.LastName,source.Mobile,source.Email, UPPER(left(source.Gender,1)),source.Address)
END
 
Step 6: Now that you have created a source and target tables, as well as the stored  procedure, now click on the validate option to run your pipeline and click on the debug button. Now once the copy pipeline option is submitted, do the step 7 to get the final output. 
Step 7: To get the final view of the data, lets create a procedure and execute it.
Create procedure selectedcolumns
As
Begin
  select [PersonID],[FirstName] as 'Name',[Mobile],[Email],[Gender],[Address]
  from [dbo].[jay]
End

Step 8: Now just type selectedcolumns and click on execute button and see the final output.


