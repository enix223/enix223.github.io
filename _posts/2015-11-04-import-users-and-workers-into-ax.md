---
layout: blog
categories: blog
tags: 
  - "null"
published: true
title: Import users and workers into AX
---


## PROGRAM ELEMENT

    +--------------+----------------------------+-------------------------------------------------+           
    |Element type  | Element                    |  Description                                    |        
    +--------------+----------------------------+-------------------------------------------------+          
    |Job	       | BIT_EY_ImportHCMWorker	    | Import worker and user dialog                   |     
    +--------------+----------------------------+-------------------------------------------------+         
    |Job           | BIT_EY_ImportCSVToTable | Load the csv data into AX table                 |     
    +--------------+----------------------------+-------------------------------------------------+         
    |Class         | BIT_EY_ImportHCMWorker     | Import worker and user main program             |     
    +--------------+----------------------------+-------------------------------------------------+         
    |Table         | BIT_EY_UserList	     | user and worker info to be imported             |                                                   
    +--------------+----------------------------+-------------------------------------------------+         
	Table 1  Program related elements

## IMPORT FLOW

### 1.	Deploy the latest program

a)	Export XPO from BI Sandpit     
![ax-2-1.PNG]({{site.baseurl}}/media/ax-2-1.PNG)         

b)	Import into Target server     
![ax-2-2.PNG]({{site.baseurl}}/media/ax-2-2.PNG)        
 

### 2.	Prepare the user list

a)	Prepare the user list according to the import template as below. Please note, NEVER CHANGE THE TEMPLATE HEADER NAME, as the header is used as the AX table field name.     
b)	Save the above data to CSV format, with comma as delimiter.     
c)	Run Job: BIT_EY_ImportCSVToTable to import the CSV into BIT_EY_UserList table         
![ax-2-3.PNG]({{site.baseurl}}/media/ax-2-3.PNG)            

d)	Please use the default setting, except the CSV path, and then click <OK>          
![ax-2-4.PNG]({{site.baseurl}}/media/ax-2-4.PNG)              

 
   Parameters    
   Table to be import: <Choose the table name which you want to import the csv to>    
   Delimiter: <The delimiter which you used in the CSV file>    
   Clear the table: <If checked, then the table will be cleared before importing the data into it>  

e)	If no error, you can find a similar infolog as below:       
![ax-2-5.PNG]({{site.baseurl}}/media/ax-2-5.PNG)     

f)	Double check your data with browsing the target table(BIT_EY_UserList)     
![ax-2-6.PNG]({{site.baseurl}}/media/ax-2-6.PNG)      
![ax-2-7.PNG]({{site.baseurl}}/media/ax-2-7.PNG)      

   
### 3.	Start to Import

a)	Switch to the target company. THIS IS IMPORTANT, PLEASE MAKE SURE YOU RUN THE PROGRAM UNDER THE CORRECT COMPANY.    
b)	Run Job BIT_EY_ImportHCMWorker             
![ax-2-8.PNG]({{site.baseurl}}/media/ax-2-8.PNG)              

c)	Choose the log level, and then click <OK>            
![ax-2-9.PNG]({{site.baseurl}}/media/ax-2-9.PNG)             

 
Log level:
1.	Debug: Verbose mode, and debug message will be display in the infolog
2.	Info: Just information, warning, error will be shown in the infolog
3.	Warning: Only warning, error message will be shown in the infolog
4.	Error: Only error message will be shown in the infolog

d)	Check the infolog and find out whether any warning and error message occurred.

<END OF IMPORT PROCESS>

---

## PROGRAM LOGIC

1. Read a record from BIT_EY_UserList, if no more record found, then go to END    
2. Lookup the worker with employee number, if record found then go to Step 3     
3. Lookup the worker with SameAs ad user, the first worker under this user will be used, if no record found, then raise an error message “SameAs user[xxxx] worker not found”, and go to Step 1, else if new worker found, then go to step 5, else go to step 4     
4. Create the worker, if failed, raise an error “Employee xxxx[xxx] create failed”, and go to step 1, else go to next step     
5. Copy order class if not existed. Copy from SameAs worker.     
6. Search the user with Domain and SDIAD field, if record found, then go to step 8, else go to step 7     
7. Import the user from AD Domain, if failed, then raise an error “User xxxx import failed”, and go to Step 1, else go to next step      
8. Copy the role from SameAs user if the role is not exited. If failed, then raise an error “User [user: %1, from: %2] role not copied”, and go to step 1, else go to next step     
9. Build user and worker relationship if the relationship is not exited. If failed, raise an error “User[%1] and Worker[%2] relation build faield”, and go to step 1, else go to next step    
10. Copy worker parameters, including Parts, WMS, Service, Power systems parameters. If failed, raise an error “Copy worker[from: %1, to:%2] parameters failed”, and go to step 1.    

END

---

## X++ Code

### BIT_EY_ImportHCMWorker

{% highlight c %}
static void BIT_EY_ImportHCMWorker(Args _args)
{
  BIT_EY_ImportHCMWorker service;
  Dialog dialog;
  dialog = new Dialog('Start to import Workers?');
  dialog.run();
  if (dialog.run())
  {
    service = new BIT_EY_ImportHCMWorker();
    service.setLogLevel(BIT_EY_LogLevel::Debug);
    service.run();
  }
}
{% endhighlight %}


### BIT_EY_ImportCSVToTable


{% highlight c %}
    static void BIT_EY_ImportCSVToTable(Args _args)
    {
        Dialog dialog;
        DialogField dialogFileName;
        DialogField dialogDelimiter;
        DialogField dialogHeader;
        DialogField dialogClearTable;
        DialogField dialogTableName;
    
        Filename filename;
        Delimiter delimiter;
        NoYesId header;
        NoYesId clearTable;
        TableName tableName;
    
        FileIOPermission permission;
        TextIO textIO;
        BIT_EY_UserList tbl; // Table to be imported
        Common table;
        DictTable dictTable;
        int fieldId;
    
        str s11;
        int i, j;
        Container c1,c2, colNames;
        CompanyInfo companyInfoLoc = CompanyInfo::find();
        Container filterCriteria;
    File
    avifiles
        ;
    
        dialog = new Dialog("Importing Text File");
        dialogFileName = dialog.addField(extendedTypeStr(Filenameopen), "File Name");
        dialogTableName = dialog.addField(extendedTypeStr(TableName), "Table to imported");
        dialogDelimiter = dialog.addField(extendedtypestr(Delimiter), "Delimiter");
        dialogClearTable = dialog.addField(extendedTypeStr(NoYesId), "Clear the table?");
    
        filterCriteria = ['*.csv'];
        filterCriteria = dialog.filenameLookupFilter(filterCriteria);
    
        // Default value
        dialogDelimiter.value(',');
        dialogClearTable.value(NoYes::Yes);
        dialogTableName.value('BIT_EY_UserList'); // assign a defalt table name
    
        dialog.run();
        if (dialog.run())
        {
            filename = dialogFileName.value();
            delimiter = dialogDelimiter.value();
            clearTable = dialogClearTable.value();
            tableName = dialogTableName.value();
        }
        else
        {
            return;
        }
    
        if(!filename || !delimiter || !tableName)
        {
            info("Filename/delimiter/table name must be filled");
            return;
        }
    
        permission = new fileIOpermission(filename,#io_read);
        permission.assert();
        textIO = new TextIO(filename,#io_read);
        textIO.inFieldDelimiter(delimiter);
        dictTable = new DictTable(tableName2Id(tableName));
    
        header = true;
    
        if(textIO)
        {
            // Clear the table if needed
            if(clearTable)
            {
                table = BIT_EY_Helper::getTable(tableName2Id(tableName), true);
    
                ttsBegin;
                while select table
                {
                    table.delete();
                }
                ttsCommit;
                info(strFmt('Table %1<id:%2> cleared', tableName, tableName2Id(tableName)));
            }
    
            i = 1;
            while(textIO.status() == IO_Status::Ok)
            {
                c1 = textIO.read();
    
                if(header)
                {
                    colNames = c1;
                    header = false;
                    continue;
                }
    
                // Skip empty rows
                if(conLen(c1) == 0)
                {
                    continue;
                }
    
                // Create a new record
                table = dictTable.makeRecord();
    
                // Loop over the colunm names and assign values to the table
                for(j = 1; j <= conLen(colNames); j ++)
                {
                    fieldId = fieldName2id(tableName2Id(tableName), conPeek(colNames, j));
                    table.(fieldId) = conPeek(c1, j);
                }
    
                // Skip when row insert got error
                try
                {
                    table.insert();
                }
                catch(Exception::Error)
                {
                    error(strfmt("Item : %1 not created", i));
                    continue;
                }
    
                info(strfmt("Item : %1 has been created", i));
                sleep(10);
    
                i ++;
            }
        }
    }
{% endhighlight %}