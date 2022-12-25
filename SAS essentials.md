# L1 essentials

## a basic SAS program:

```SAS
data myclass;							/* set up a new, temporary table, myclass */
    set sashelp.class;        			/* from existed data: class table in the sashelp dataset */
run;

proc print data=myclass;		/* print out the data */
run;
```



## steps

-  `data`, `proc`
- most steps end with `run`, a few `proc` steps end with `quit`



## statement

- a step was made of multiple statements

  - data, run, proc print, proc means, ...

  - set

  - assignment

- all stmt must end with semicolon `;`
- global statement:  title, options, libname ...
  - outside of steps; 
  - often involved with global/session setting; 
  - does not need `run`



## syntax

- spacing is not enforced;
  - try use auto formatting (format code button in the upper menu) in SAS studio for better readability!
- case-insensitive
- comment:
  - `/* comment style 1 */`
  - `*comment style 2;`
  - short cut key for comment: `ctrl + /` 



## misc

- comment is also considered a type of statement
- some misspelling, like `porc`, will be identified by SAS and SAS will execute successfully with notifying the user in the log.





# L2 accessing data

## Types of data

- `structured`  & `unstructured`
  - (sql v.s. non-sql)



## SAS data

- SAS table: `.sas7bdat` file
  - descriptor: table header with column names, and other meta data
  - data: rows of records
  - terms:
    - dataset = sas table
    - column = variable
    - row = observation



- defined column

  - a column in a SAS table must have 3 attribute to be defined:

    - name

      - 1-32 characters
      - recommended: start with underscore or letter, continue with underscore, letter or number
      - SAS can deal with special characters in column name or table name though
      - case doesn't matter

    - type

      - numeric
        - SAS dates: # days:  cell date - 01Jan1960
        - missing: indicated by `.`
      - character
        - missing: indicated by a blank column

    - length: numeric 8 bytes, character 1 - 32767 bytes
    
      

- listing attribute

  ```SAS
  /*template*/
  PROC CONTENTS DATA=data-set;
  RUN;
  
  /*use absolute path to access table file*/
  proc contents data="S:\workshop/EPG1V2/data/storm_summary.sas7bdat";
  run;
  ```

  

## library

- a temp variable to store data path; will be discarded after restarting SAS session; can be obtained again by using stored `.sas` program

- example:

  ```SAS
  LIBNAME libref engine "path";
  ```

  - LIBNAME: keyword

  - libref: custom variable name used to refer to the path

  - engine: use different engine to deal with different file

    - base: sas table; base is default, so can be skipped
    - excel, hadoop...

  - path: absolute path to a folder

  - example: `libname mylib base "s:/workshop/data";`

    

- automatically defined library
  - `work`
    - temporary: tables in work will be deleted at the end of the session
    - default: if not type in libref before a table name, SAS use work
  - `sashelp`
    - sample data and other files





- use library with excel

  ```SAS
  /* In this example, we use the OPTIONS statement to enforce SAS naming conventions for the columns. */
  options validvarname=v7;
  /*if use: `options option(s)`: no enforcement*/
  
  /*
  	Then, we create the 
  	xlclass library 
  	with the XLSX engine to read data from the class Excel workbook 
  	located in s:/workshop/data. 
  */
  
  libname xlclass xlsx "s:/workshop/data/class.xlsx";
  
  /*The PROC CONTENTS step is reading the class_birthdate worksheet in the class workbook.*/
  
  proc contents data=xlclass.class_birthdate;			/*sheet in xlsx  =  table in library*/
  run;
  
  /* At the end, we clear the xlclass libref, to remove the lock on the excel file to enable others access it */
  libname xlclass clear;
  ```

  



## import unstructured data

- point and click way: import wizard can be used to import unstructured data

- programming way: `proc import`

  ```SAS
  PROC IMPORT DATAFILE="path/filename" DBMS=filetype
                            OUT=output-table<REPLACE>;        /*replace: drop output table if exists at first*/
          <GUESSINGROWS=n|MAX;>                            
          /*set SAS will use how many rows to determine column type; default is 20*/
  RUN;
  ```

- import excel:

  ```SAS
  PROC IMPORT DATAFILE="path/file-name.xlsx" DBMS=XLSX
                            OUT=output-table <REPLACE>;
          SHEET=sheet-name;
  RUN;
  ```

  - difference with using library:
    - when using library with xlsx engine, SAS will lock the file and read it, so the file is always up-to-date
    - when using proc import, SAS will create a copy of excel sheet, in the format of SAS table. So, the table in the future may not be consistent with the file. To update the table, re-import is needed.



- import file with custom delimiter

  ```SAS
  proc import datafile="S:/workshop/EPG1V2/data/np_traffic.dat"
  		dbms=dlm					/*set dbms to dlm*/
  		out=traffic2 replace;
  		guessingrows=max;
  		delimiter="|"; 				  /*set the delimiter explicitly*/
  run;
  
  proc contents data=traffic2;
  run;
  ```

  



# L3 Explore and Validating Data

## Procedures

- procedures that help to explore data  (`content` show the descriptor part)

  - `print`: show all data
  - `means`: simple summary stat: N, mean, STD, min, max
  - `univariate`: more detailed summary stat related to distribution, 5 extreme low/high value
  - `freq`: create frequency tables for each column in input table

- official documentation for procedures: https://documentation.sas.com/doc/zh-CN/pgmsascdc/9.4_3.5/allprodsproc/procedures.htm

- examples

  ```SAS
  															/*obs: list first 10 rows*/
  proc print data=pg1.storm_summary(obs=10);
      var Season Name Basin MaxWindMPH MinPressure StartDate EndDate;  /*select variables to display*/
  run;
  
  
  /*calculate summary statistics*/
  proc means data=pg1.storm_summary;
      var MaxWindMPH MinPressure;        /*must be numeric variables!*/
  run;
  
  
  /*examine extreme values*/
  proc univariate data=pg1.storm_summary;
      var MaxWindMPH MinPressure;
  run;
  
  
  /*list unique values and frequencies*/
  proc freq data=pg1.storm_summary;
      tables Basin Type Season;					/*for freq, use table instead of using var!*/
  run; 
  
  ```

  



## Filter rows

- use `where` step to filter rows

```SAS
PROC procedure-name . . . ;
         WHERE expression;
RUN;
```

- expression:

  - structure: `column` `operator` `value`
  - operators:
    - **=** or **EQ**
    - **^=** or **~=** or **NE**
    - **>** or **GT**
    - **<** or **LT**
    - **>=** or **GE**
    - **<=** or **LE**
  - values:
    - string: double/single quotation;  **case-sensitive**
    - number
    - date: use SAS date constant: 
      - `"ddmmmyyy"d`: eg. `"1jan15"d`,  `"01JAN2015"d`
      - 1 or 2 digit day, 3-letter case-insensitive month, and 2 or 4 digits year

- multiple expression can be combined by `and`, `or`

- `in`: SQL-like filtering:

  ```SAS
  PROC ...;
  	WHERE col-name IN (value-1<…,value-n>);					/* just like sql! */
  	WHERE col-name NOT IN (value-1<…,value-n>);
  RUN;
  ```

  

- some column has missing values. In comparision, **missing values are treated as smallest possible value.**

- misc
  - find missing value
    - `where Type=. or Type=" ";`
    - **WHERE** *col-name* **IS MISSING;**
      **WHERE** *col-name* **IS NOT MISSING;**
    - for data coming from DBMS distinguished missing and null:
      - `where Item is null;`
  - between and: inclusive
    - **WHERE** *col-name* **BETWEEN** *value-1* **AND** *value-2*;
  - pattern matching: like
    - **WHERE** *col-name* **LIKE "**value**";**
    - wildcard: `%`: any character of any length (>= 1); `_`: any character, length of 1

 	

### Macro Variables

- macro variable: a way to use custom defined variable
  - temporary: only lives in current session
- syntax:
  - define: **%LET** *macro-variable*=*value*;
  - use: **&** *macro-var*

- example:

  ```SAS
  %let CarType=Wagon;
  
  proc print data=sashelp.cars;
      where Type="&CarType";
      var Type Make Model MSRP;
  run;
  
  proc means data=sashelp.cars;
      where Type="&CarType";
      var MSRP MPG_Highway;
  run;
  
  proc freq data=sashelp.cars;
      where Type="&CarType";
      tables Origin Make;
  run;
  ```

  



## Formatting

### Format values in results

- syntax:

  ```SAS
  PROC PRINT DATA=input-table;
            FORMAT col-name(s) format;
  RUN;
  ```

  - `FORMAT`: keyword

  - `col-name(s)`: columns that needed formatting, can be multiple columns separated by space

  - `format`: SAS format

    - syntax: `<$>format-name<w>.<d>`
    - `$`: indicating a character format
    - `format-name`: SAS format name
    - `w`: total width, including decimal places and special characters
    - `.`: a required delimiter (suggesting this is a SAS format)
    - `d`: for numeric format, specifying number of decimal places

  - example: (does not change raw data!)

    ```SAS
    proc print data=pg1.class_birthdate;
        format Height Weight 3. Birthdate date9.;
    run;
    /*format height, weight with 3. and format birthdate with date9.*/
    ```

    

### common format for numeric values

| Format Name | Example Value | Format Applied | Formatted Value |
| :---------: | :-----------: | :------------: | :-------------: |
|     w.d     |   12345.67    |       5.       |      12346      |
|     w.d     |   12345.67    |      8.1       |     12345.7     |
|  COMMAw.d   |   12345.67    |    COMMA8.1    |    12,345.7     |
|  DOLLARw.d  |   12345.67    |   DOLLAR10.2   |   $12,345.67    |
|  DOLLARw.d  |   12345.67    |   DOLLAR10.    |     $12,346     |
|   YENw.d    |   12345.67    |     YEN7.      |     ¥12,346     |
|  EUROXw.d   |   12345.67    |   EUROX10.2    |   €12.345,67    |

- `w`: maximum total width, `d`: decimal place, `COMMA`: add comma



### common format for date values

| Format Name | Example Value | Format Applied           |
| ----------- | ------------- | ------------------------ |
| 21199       | DATE7.        | 15JAN18                  |
| 21199       | DATE9.        | 15JAN2018                |
| 21199       | MMDDYY10.     | 01/15/2018               |
| 21199       | DDMMYY8.      | 15/01/18                 |
| 21199       | MONYY7.       | JAN2018                  |
| 21199       | MONNAME.      | January                  |
| 21199       | WEEKDATE.     | Monday, January 15, 2018 |



### setting width

- by default, numeric values will have commas
- when width is not enough for displaying numeric values, 
  - commas will be eliminated at first
  - then eliminate dollar or other money sign
  - then SAS use scientific notation to represent the number
- so, if you observed a scientific notation in the output, you have set a too small width.



## Sorting & Removing Duplicate

### Sorting Data

```SAS
PROC SORT DATA= input-table<OUT=output table>;
         BY <DESCENDING> col-name(s);
RUN;
```

-  if you don't include the OUT= option, PROC SORT changes the order of the rows in the input table.
- PROC SORT doesn't generate printed output, so you have to open or print the sorted table if you want to look at it.



### Removing Duplicate

```SAS
PROC SORT DATA=input-table <OUT=output-table>
                        NODUPKEY  <DUPOUT=output-table>;
        BY _ALL_;
RUN;
```

- PROC SORT is also used to remove duplicate

- `BY _ALL_`: use all columns

- `DUPOUT`: by specifying this, we can observe removed columns and validate

- mechanism: sort columns at first, then only keep the first occurence for duplicated rows

  

- remove dup by specific columns:

  ```SAS
  PROC SORT DATA=input-table <OUT=output-table>
                          NODUPKEY <DUPOUT=output-table>;
          BY col-name(s);
  RUN;
  ```

  
