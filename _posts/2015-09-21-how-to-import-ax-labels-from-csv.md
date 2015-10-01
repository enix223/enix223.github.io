---
layout: blog
title: "AX: How to import labels from csv file"
date: 2015-09-21 23:04:05 +0800
comments: true
categories: blog
---

## 1. Background
As our group need to launch Dynamic AX 2012 project in several countries, and we have developed several reports for our users, but our charllenges are we need to translate all the labels using in this reports to local language. The local translation are provided by our users, but all of them are keep in an excel. Then we have to copy and paste all the labels to the AOT label editor, you know, we have about 1000 labels, and you can imagine what a big work load to do this. And it is a simple manual task, copy and paste !!!! I don't want to do such stupid thing, so I will leave it to the machine. :)

### 1.1 Labels we already have
![]({{site.baseurl}}//images/ax/20150927-1-labels.PNG)

### 1.2 What Translation excel looks like
![]({{site.baseurl}}//images/ax/20150927-1-excel.PNG)

### 1.3 Target
Import all the labels into AOT for the specific language, for this post, the language is "zh-hans".

## 2. Start to work
Before coding, we need to understand how the label editor works. For the first step is to trace the work flow behind label editor.

### 2.1 Label editor
The best way to trace the program behind the GUI is using **personalize**. First open the label editor, and then right click the any position in the form, and then choose personalize. Then you can see the following screen.
![]({{site.baseurl}}//images/ax/20150927-1-label-editor.PNG)

So this form is called: SysLabelSearch. Then we can try to browse the code in this form. After debug and trace, finally, we got how it works. And here is the related class, form, table will be used by this form.

### 2.2 Related class and form
Form: SysLabelSearch (The label search/edit GUI)    
Class: SysLabelEdit (The class used to mainpulate labels)    
Table: TmpSysLabel (This table is used to keep the search result get from SysLabelEdit)    

### 2.3 Create a table to hold csv
A new table is copied from TmpSysLabel (this table is used to keep the search result in SysLabelEdit), and the name is **BIT_EY_SysLabel**
![]({{site.baseurl}}//images/ax/20150927-1-proj.PNG)

And I have written a helper to import csv data to table. 

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
        Common table;
        DictTable dictTable;
        int fieldId;

        str s11;
        int i, j, okCnt;
        Container c1,c2, colNames;
        CompanyInfo companyInfoLoc = CompanyInfo::find();
        Container filterCriteria;
        #File
        #avifiles
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
        dialogFileName.value("\\\\tsclient\\D\\AX\\Source\\dev\\import_label\\BIT_EY_SysLabel.csv");
        dialogTableName.value('BIT_EY_SysLabel'); // assign a defalt table name

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
        textIO = new TextIO(filename,#io_read, 54936);
        textIO.inFieldDelimiter(delimiter);
        dictTable = new DictTable(tableName2Id(tableName));

        header = true;
        okCnt = 0;

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
                    okCnt ++;
                }
                catch(Exception::Error)
                {
                    error(strfmt("Item : %1 not created", i));
                    continue;
                }

                //info(strfmt("Item : %1 has been created", i));
                sleep(10);

                i ++;
            }

            info(strFmt("Total records read: %1, total records imported: %2", i-1, okCnt));
        }
    }

Run this helper, and you can see this:
![]({{site.baseurl}}//images/ax/20150927-1-importer.PNG)

_Convention:_    

Thi job is used the csv column name to match the table field name, so please make sure all the csv column names should be exited in the target table field list.

_Parameters:_   
1. File name, the csv file path
2. Table to import, a dropdown list for you to choose the table you want to import the csv to.
3. Delimter, the delimiter which is used in your csv, default is ","
4. Clear the table, when this option is checked, then the target table will be cleared before importing the csv into it.

Fill the parameters, and the click confirm. Then the records will be imported into the table.

### 2.4 Write a job to import labels

Firstly, we will write a while loop to get each label imported from csv, and then I will use the labelID to lookup available labels with `sysLabelEdit.findLabel()`, after that, the result is saved in TmpSysLabel, so we can get the result back with `sysLabelEdit.findResults()`.

And if the label is the same as the label we imported to the csv, then, we will skip it, else, we will modify the label with `sysLabelEdit.labelModify`.

    static void BIT_EY_ImportLabels(Args _args)
    {
        Dialog dialog;

        BIT_EY_SysLabel myLabels;
        TmpSysLabel tmpSysLabel;
        SysLabelEdit sysLabelEdit;
        LabelId     labelId;
        #define.Language("zh-hans")

        // Init object
        sysLabelEdit                = new SysLabelEdit();
        dialog = new Dialog("Confirm to import labels?");

        dialog.run();
        if (dialog.run())
        {
            while select * from myLabels
            {
                sysLabelEdit.findLabel(#Language, myLabels.LabelId);
                tmpSysLabel = sysLabelEdit.findResults();
                while select * from tmpSysLabel
                    where tmpSysLabel.Language == #Language
                {
                    if(tmpSysLabel.Label == myLabels.Label)
                    {
                        info(strFmt("%1 not change.", tmpSysLabel.LabelId));
                    }
                    else
                    {
                        try
                        {
                            sysLabelEdit.labelModify(tmpSysLabel.Language, tmpSysLabel.LabelId,
                                myLabels.Label, myLabels.Description,  tmpSysLabel.SysLabelApplModule);

                            info(strFmt("%1 label change from: <%2> to: <%3>",tmpSysLabel.LabelId,
                                tmpSysLabel.Label, myLabels.Label));
                        }
                        catch
                        {
                            error(strFmt("%1 label change from: <%2> to: <%3> failed.",tmpSysLabel.LabelId,
                                tmpSysLabel.Label, myLabels.Label));

                            exceptionTextFallThrough();
                        }
                    }
                }
            }
        }
    }


After running with this job, you will find the info log similar as below:
![]({{site.baseurl}}//images/ax/20150927-1-infolog.png)







