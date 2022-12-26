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
    - note: The macro variable must be in double quotation marks in order for the character value to be substituted. No substitution for single quotation marks.
  
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

  



# L4 Prepare Data

## Reading & Filtering

### basic data step:

```SAS
DATA=output-table;
        SET input-table;
RUN;
```

- if `output-table` already exist, this step will overwrite it!
- Any data source that can be read via a library can be used as the input table in the SET statement.



### data step process

- 2 phases: compilation, execution
  - compilation: SAS checks for syntax errors in the program and establishes the table metadata, including the column attributes. 
  - execution: SAS reads the data, processes it, and writes it to the output table one row at a time.

- execution process: DATA step execution is like an automatic loop - it uses streaming
  - The first time through the DATA step, the SET statement reads row number one from the input table
  - and then processes any other statements in sequence, manipulating the values within that row.
  - When SAS reaches the end of the DATA step, there is an implied OUPUT action, and the new row is written to the output table.
  - The DATA step then automatically loops back to the top and executes the statements in order again, this time reading, manipulating, and outputting the second row.
  - That implicit loop continues until all of the rows are read from the input table.



### example

```SAS
data myclass;
    set sashelp.class;
    where age >=15;
    *keep Name Age Height;
    *drop Sex Weight;
    format height 4.1 Weight 3.;
run;
```

- use `where` to filter data
- use `format` to set format for specified columns (the format will be applied in the output class!)
- use `keep` `drop` to set columns needed





## Computing New Columns

### use expression to create new columns

- syntax:

  ```SAS
  DATA output-table;
          SET input-table;
          new-column=expression;
  RUN;
  ```

  - note: character expression, like `country="AU"`, is **case sensitive**! 

- example:

  ```SAS
  data cars_new; 
      set sashelp.cars;
      where Origin ne "USA"; 
      Profit = MSRP-Invoice;
      Source = "Non-US Cars";
      format Profit dollar10.;
      keep Make Model MSRP Invoice Profit Source;
  run;
  ```

  - Without a KEEP statement, the output table would include all columns. But because there is a KEEP statement in this code, you must explicitly list the new columns, so that they are included in the new table.



### use functions to create new columns

- numeric functions:

|     Common Numeric Functions     |
| :------------------------------: |
|  **SUM** (*num1*, *num2*, ...)   |
|  **MEAN** (*num1*, *num2*, ...)  |
| **MEDIAN** (*num1*, *num2*, ...) |
| **RANGE** (*num1*, *num2*, ...)  |
|  **MIN** (*num1*, *num2*, ...)   |
|  **MAX** (*num1*, *num2*, ...)   |
|   **N** (*num1*, *num2*, ...)    |
| **NMISS** (*num1*, *num2*, ...)  |

- note these functions ignores missing values
  - `mean(10, ., 5, 9)` generate 8: (10 + 5 + 9)  / 3 = 8



- character functions:

  | Character Function                          | What it Does                                                 |
  | :------------------------------------------ | ------------------------------------------------------------ |
  | **UPCASE** (*char*) **LOWCASE**(*char*)     | Changes letters in a character string to uppercase or lowercase |
  | **PROPCASE** (*char*, <*delimiters*>)       | Changes the first letter of each word to uppercase and other letters to lowercase |
  | **CATS** (*char1*, *char2*, ...)            | Concatenates character strings and removes leading and trailing blanks from each argument |
  | **SUBSTR** (*char*, *position*, <*length*>) | Returns a substring from a character string                  |



- example

  ```SAS
  data storm_new;
      set pg1.storm_summary;
      drop Type Hem_EW Hem_NS MinPressure Lat Lon;
      * overwrite basin, name with proper case;
      Basin=upcase(Basin);
      Name=propcase(Name);
      * create hemisphere by combining two cols;
      Hemisphere=cats(Hem_NS, Hem_EW);
      * substr start from 2 (inclusive, 1-indexed), and has a fixed length of 1;
      Ocean=substr(Basin,2,1);
  run;   
  ```

  

- date functions

  | Date Function            | What it Does                                                 |
  | :----------------------- | ------------------------------------------------------------ |
  | **MONTH** (*SAS-date*)   | Returns a number from 1 through 12 that represents the month |
  | **YEAR** (*SAS-date*)    | Returns the four-digit year                                  |
  | **DAY** (*SAS-date*)     | Returns a number from 1 through 31 that represents the day of the month |
  | **WEEKDAY** (*SAS-date*) | Returns a number from 1 through 7 that represents the day of the week (Sunday=1) |
  | **QTR** (*SAS-date*)     | Returns a number from 1 through 4 that represents the quarter |



| Date Function                             | What it Does                                                 |
| ----------------------------------------- | ------------------------------------------------------------ |
| **TODAY** ()                              | Returns the current date as a numeric SAS date value (no argument is required because the function reads the system clock) |
| **MDY** (*month*, *day*, *year*)          | Returns a SAS date value from numeric month, day, and year values |
| **YRDIF** (*startdate*, *enddate*, 'AGE') | Calculates a precise age between two dates (has decimal places) |



- example

  ```SAS
  data storm_new;
      set pg1.storm_damage;
      drop Summary;
      * calculate the age in years between `date` column and today;
      YearsPassed=yrdif(Date,today(),"age");
      * generate a date that has the same mon, day as `date`, but the year of current year;
      Anniversary=mdy(month(Date),day(Date),year(today()));
      format YearsPassed 4.1 Date Anniversary mmddyy10.; 
  run; 
  ```





### **Creating Character Columns with the LENGTH statement**

- when assigning a value to a new column for the first time, the length of the value will be used by SAS as the length of the column. If we assigned another value that is longer, this value will be truncated.

  ```SAS
  data cars2;
      set sashelp.cars;
      * the first time assigning value, so length of cartype is set to be 5;
      if MSRP<60000 then CarType="Basic";
      * in result, cartype will be "Luxur", truncated to length of 5;
      else CarType="Luxury";
      keep Make Model MSRP CarType;
  run;
  ```

  

- to avoid such problem, we can declare the length of the new column at first: 

  - **LENGTH** *char-column* $ _length_;

  - must be placed before the first assignment, or in the execution phase SAS will still determine the length by the first assignment

  - ```SAS
    data cars2;
        set sashelp.cars;
        LENGTH CarType $ 6;
        if MSRP<60000 then CarType="Basic";
        * now in result, cartype will be "Luxury";
        else CarType="Luxury";
        keep Make Model MSRP CarType;
    run;
    ```

    





## Conditional Processing

### if then

```SAS
data storm_new;
    set pg1.storm_summary;
    keep Season Name Basin MinPressure PressureGroup;
    if MinPressure=. then PressureGroup=.;
    if MinPressure<=920 then PressureGroup=1;
    if MinPressure>920 then PressureGroup=0;
run; 
```



### if then else

```SAS
data cars2;
    set sashelp.cars;
    if MSRP<20000 then Cost_Group=1;
    else if MSRP<40000 then Cost_Group=2;
    else if MSRP<60000 then Cost_Group=3;
    else Cost_Group=4;
    keep Make Model Type MSRP Cost_Group;
run;
```



- note: only 1 executable statement allowed after `then`
- use `then do`, `else do` to execute multiple statement



### if then do end

- use `if then do end` to output data to 2 different tables

```SAS
data under40 over40;
    set sashelp.cars;
    keep Make Model msrp cost_group;
    if MSRP<20000 then do;
       Cost_Group=1;
       * use output to manually control which table that this row will go to;
       output under40;
    end;
    else if MSRP<40000 then do;
       Cost_Group=2;
       output under40;
    end;
    else do;
       Cost_Group=3;
       output over40;
    end;
run;
```









# L5 Analyzing and Reporting on Data

## Enhancing Reports with Titles, Footnotes, and Labels

### titles & footnote

- syntax:
  - **TITLE**<n> "*title-text*";
  - **FOOTNOTE**<n> "*footnote-text*"**;**
  - `n` means the line number in title/footnote

- note
  - title, footnote are global stmt, so they live through the SAS session
  - sometimes it might happens that you current report has titles / footnote set previously (in another program)
  - to avoid such situation, use null title stmt to eliminate previous titles:
    - **TITLE**;
    - **FOOTNOTE**;
    - it's a good habit to call it at the end of each .SAS program

- turn off auto generated procedure title: `ods noproctitle`



### Using Macro Variables and Functions in Titles and Footnotes

```SAS
%let age=13;

title1 "Class Report";
title2 "Age=&age";
footnote1 "School Use Only";

proc print data=pg1.class_birthdate;
    where age=&age;
run;

title;
footnote;
```



### Applying Temporary Labels to Columns

- syntax: **LABEL** *col-name=*"*label-text*"**;**

- example:

  ```SAS
  proc means data=sashelp.cars;
      where type="Sedan";
      var MSRP MPG_Highway;
      label MSRP="Manufacturer Suggested Retail Price"
            MPG_Highway="Highway Miles per Gallon";
  run; 
  ```

- using label outside the datastep will result in temporary labels; however, if put in data step, these labels will become permanent

  - you can observe the label attribute by proc content
  - still need to explicitly use label option for proc print to display the label

- misc:

  - PROC MEANS, PROC FREQ, and most other procedures automatically **display the labels in the results as a column**, but PROC PRINT is an exception. Because the main purpose of PROC PRINT is to examine the data, column names are displayed by default.

  - To display labels instead, you must add the LABEL option in the PROC PRINT statement.

    ```SAS
    * add `label` for proc print
    proc print data=sashelp.cars label;
        where type="Sedan";
        var Make Model MSRP MPG_Highway MPG_City;
        label MSRP="Manufacturer Suggested Retail Price"
              MPG_Highway="Highway Miles per Gallon";
    run; 
    ```

  - to remove the `obs` column in result, add `noobs` in proc print

    

### Segmenting Reports

- syntax: **BY** _variable(s)_;

- note: To do this, you must sort the data first using the same BY-column.

- example:

  ```SAS
  * sort at first;
  proc sort data=sashelp.cars 
            out=cars_sort;
      by Origin;
  run;
   
  * for each value of origin, generate a report table on Type;
  proc freq data=cars_sort;
      by Origin;
      tables Type;
  run;
  ```

  



## Creating Frequency Reports

### Creating Frequency Reports and Graphs

- a Basic Frequency Report is based on individual columns. By default, each column listed in the TABLES statement generates a separate frequency table that includes the number and percentage of rows for each value in the data, as well as a cumulative frequency and percent.

- There are options that you can add to the PROC FREQ statement and options you can add to the TABLES statement that you can use to customize the output, and include additional statistics.

- syntax:

  ```SAS
  proc freq data=input-table <proc-options>;
      tables col-name(s) / options;
      format StartDate monname.;
  run;
  ```

  

- examples:

  - ```SAS
    * order=freq : in report, order values by freq, instead of the value itself;
    * nlevels: summarzing the number of distinct values for each variable;
    proc freq data=pg1.storm_final order=freq nlevels;
        tables BasinName Season;
    run; 
    ```

    - `order=formatted | data | freq`
    - `nlevels`

  - 

    ```SAS
    proc freq data=pg1.storm_final order=freq nlevels;
        * nocum: suppress the cumulative columns;
        tables BasinName Season / nocum;
    run; 
    ```

    - `nocum` `nopercent` `out=output-table`

  - 

    ```SAS
    proc freq data=pg1.storm_final order=freq nlevels;
        * 2) so now the startdate table will only have 12 values, and we can count storm happend in each month;
        tables BasinName StartDate / nocum;   /* options have to be added after the left dash*/
        * 1) use monname. to format the startdate, let it becomes names of months
        format StartDate monname.;
    run;
    ```

    

  - ```SAS
    * call graph library. it remains for the session and has no reason to shut it down;
    ods graphics on;
    proc freq data=pg1.storm_final order=freq nlevels;
        tables BasinName StartDate /
        	   * call freq plot, set orient and scale
               nocum plots=freqplot(orient=horizontal scale=percent);
        format StartDate monname.;
    run;
    ```

    

  - ```SAS
    ods graphics on;
    * remove procedure title;
    ods noproctitle;
    title "Frequency Report for Basin and Storm Month";
    proc freq data=pg1.storm_final order=freq nlevels;
        tables BasinName StartDate / 
               nocum plots=freqplot(orient=horizontal scale=percent);
        format StartDate monname.;
        label BasinName="Basin"
              StartDate="Storm Month";
    run;
    title;
    * re start procedure title;
    ods proctitle; 
    ```



## Creating Two-Way Frequency Reports

- syntax:

  ```SAS
  PROC FREQ DATA=input-table < options >;
          * note: here two col were connected with *;
          TABLES col-name*col-name < / options >;
  RUN;
  ```

  - output is a 2x2 table, with freq, percent, row pct and col pct in each cell:

    <img src="SAS essentials.assets/image-20221225183624852-16720113911231.png" alt="image-20221225183624852" style="zoom:50%;" />



- options

  - ```SAS
    proc freq data=pg1.storm_final;
    	* remove row/col_pct and percent
        tables BasinName*StartDate / norow nocol nopercent;
        * same trick to let startdate be month name;
        format StartDate monname.;
        label BasinName="Basin"
              StartDate="Storm Month";
    run;
    ```

  - `tables BasinName*StartDate / crosslist;`

    - same statistics, different arrangement 
    - very much alike to the following one but omit duplicate value in the first category

  - `tables BasinName*StartDate / list;`

    - same statistics, different arrangement (tidy data)

  - ```SAS
    * noprint: suppress auto print;
    proc freq data=pg1.storm_final noprint;
        * output the freq report table;
        tables StartDate*BasinName / out=stormcounts;
        format StartDate monname.;
        label BasinName="Basin"
              StartDate="Storm Month";
    run;
    ```

    

## Creating Summary Statistics Reports

- The MEANS procedure is great for calculating basic summary statistics and looking for numeric values that might be outside of an expected range. But you can also use PROC MEANS to generate complex reports that include various statistics and groupings within the data.

- syntax:

  ```SAS
  PROC MEANS DATA=input-table <stat-list>;
          VAR col-name(s);
          CLASS col-name(s);
          WAYS n;
  RUN;
  ```

  

- examples:

  - ```SAS
    * here we specified stat-list;
    * add the MAXDEC=0 option to round statistics to the nearest integer;
    proc means data=pg1.storm_final mean median min max maxdec=0;
        var MaxWindMPH;
    run;  
    ```

  - segmentation:

    ```SAS
    proc means data=pg1.storm_final mean median min max maxdec=0;
        var MaxWindMPH;
        * class col-name was used to calculate statistics for groups
        class BasinName;
    run;   
    ```

  - two-way segmentation:

    ```SAS
    proc means data=pg1.storm_final mean median min max maxdec=0;
        var MaxWindMPH;
        * the two col will be combined for segmentation
        class BasinName StormType;
    run;   
    ```

  - control ways of segmentation

    ```SAS
    proc means data=pg1.storm_final mean median min max maxdec=0;
        var MaxWindMPH;
        class BasinName StormType;
        * when you add `ways`, col-names in class stmt become candidates;
        * ways 0: do not do segmentation;
        * ways 1: do segmentation for each single candidates;
        * ways 2: do segmentation for the 2 candidates combined;
        ways 0 1 2;
    run;
    ```

    

## Creating an Output Summary Table

- You can use the OUTPUT statement in PROC MEANS to create a summary data set. The OUT= option names the output table. In the OUTPUT statement you can generate output statistics and name a column to store them in.
- syntax:
  - **OUTPUT OUT=**output-table <statistic(col-name)=col-name> </ option(s)>**;**

- example:

  ```SAS
  proc means data=sashelp.heart noprint;
      var Cholesterol Weight;
      class Chol_Status Smoking_Status;
      output out=heart_stats mean=AvgWeight; 
  run; 
  ```

  







# L6 Export Result

## Exporting data

### proc export

- SAS programming environments includes point-and-click tools that export data to various delimited text formats
- syntax
  - **PROC EXPORT DATA=***input-table* **OUTFILE=**"*output-file*"
                  <**DBMS** =*identifier>* **<REPLACE>**;
    **RUN;**

- example

  ```SAS
  * OUTFILE= option specifies the full path and file name of the exported data file;
  * the path that you specify in the OUTFILE= option must be relative to the location of SAS;
  * The DBMS= option tells SAS how to format the output; options: CSV, TAB, DLM, or XLSX;
  * REPLACE option tells SAS to overwrite the output file if it already exists;
  
  proc export data=sashelp.cars
       outfile="s:/workshop/output/cars.txt"
       dbms=tab replace;
  run; 
  ```

  

  ## libname output

- Another easy way to export data is to use a LIBNAME engine. In this scenario, you are creating the table rather than exporting an existing table.

- ```SAS
  libname myxl xlsx "&outpath/cars.xlsx";
  
  data myxl.asiacars;
      set sashelp.cars;
      where origin='Asia';
  run;
  
  libname myxl clear;
  ```

  - This code extracts data about cars manufactured in Asia from **sashelp.cars**, and writes the result directly into the **asiacars** worksheet in the **cars** workbook.

- example: 

  ```SAS
  * xlout is bounded to the xlsx file by the libname stmt;
  libname xlout xlsx "&outpath/southpacific.xlsx";
  
  * the data step created the work sheet South_Pacific in the xlsx file;
  data xlout.South_Pacific;
      set pg1.storm_final;
      where Basin="SP";
  run;
  
  * the proc means step also created the work sheet Season_Stat in the xlsx file;
  proc means data=pg1.storm_final noprint maxdec=1;
      where Basin="SP";
      var MaxWindKM;
      class Season;
      ways 1;
      output out=xlout.Season_Stats n=Count mean=AvgMaxWindKM max=StrongestWindKM;
  run; 
  ```

  



## Exporting Report

### Using the SAS Output Delivery System

- The SAS Output Delivery System (or ODS) is programmable and flexible, making it simple to automate the entire process of exporting reports to other formats.

  - In ODS terminology, an output format is called a destination.
  - Common destinations of this type include XLSX for Excel, RTF for Microsoft Word, PPTX for PowerPoint, and PDF. 

- syntax:

  - ```SAS
    ODS <destination> <destination-specifications>;
    
    /* SAS code that produces output */
    
    ODS <destination> CLOSE;
    ```

    

### Exporting Results to CSV

- syntax:

  ```SAS
  ODS CSVALL FILE="filename.csv";
  
  /* SAS code that produces output */
  
  ODS CSVALL CLOSE;
  ```

- example:

  ```SAS
  ods csvall file="&outpath/cars.csv";
  proc print data=sashelp.cars noobs;
      var Make Model Type MSRP MPG_City MPG_Highway;
      format MSRP dollar8.;
  run;
  ods csvall close;
  ```

  - difference from `proc export`: The ODS statements are exporting the results of a procedure, so by using ODS CSVALL with PROC PRINT, you can specify the order and format of the columns in your output CSV file.



### Exporting Results to Excel

- syntax:

  ```SAS
  ODS EXCEL FILE="filename.xlsx" STYLE=style
          OPTIONS(SHEET_NAME='label');
  
  /* SAS code that produces output */
  
  ODS EXCEL CLOSE;
  ```

- example

  ```SAS
  * open a new file
  ods excel file="&outpath/wind.xlsx";
  
  * main content;
  title "Wind Statistics by Basin";
  ods noproctitle;
  proc means data=pg1.storm_final min mean median max maxdec=0;
      class BasinName;
      var MaxWindMPH;
  run;
  
  title "Distribution of Maximum Wind";
  proc sgplot data=pg1.storm_final;
      histogram MaxWindMPH;
      density MaxWindMPH;
  run; 
  title;  
  ods proctitle;
  
  * close the file;
  ods excel close;
  ```

  - this code will output the result of the middle SAS code to the file with default style

  

- adjust style

  - use `proc template` to see style options:

    ```SAS
    proc template;
        list styles;
    run;
    ```

  - set style:

    ```SAS
    * style=sasdocprinter: set style (a plain style that similar to most xlsx files);
    * options(sheet_name='Wind Stats'): set sheet name;
    ods excel file="&outpath/wind.xlsx" style=sasdocprinter
              options(sheet_name='Wind Stats');
    title "Wind Statistics by Basin";
    ods noproctitle;
    proc means data=pg1.storm_final min mean median max maxdec=0;
        class BasinName;
        var MaxWindMPH;
    run;
    
    * options(sheet_name='Wind Stats'): set sheet name;
    ods excel options(sheet_name='Wind Distribution');
    title "Distribution of Maximum Wind";
    proc sgplot data=pg1.storm_final;
        histogram MaxWindMPH;
        density MaxWindMPH;
    run;
    title;
    ods proctitle;
    ods excel close; 
    ```

    



### Exporting Results to PowerPoint and Microsoft Word

- ppt:
  - **ODS POWERPOINT FILE=**"*filename.pptx*" **STYLE**=*style***;**
    /* SAS code that produces output */
    **ODS POWERPOINT CLOSE;**

- word:
  - **ODS RTF FILE=**"*filename.rtf*" **STARTPAGE=NO****;**
    /* SAS code that produces output */
    **ODS RTF CLOSE;**



### Exporting Results to PDF

- **ODS PDF FILE=**"*filename.pdf*" **STYLE=**style
          **STARTPAGE=NO PDFTOC=**n***;**
  **ODS PROCLABEL "***label**";**

  /* SAS code that produces output */

  **ODS PDF CLOSE;**

  

- example:

  ```SAS
  * startpage=no: eliminate page breaks between procedures;
  * style=journal: just try a different style
  * pdftoc=1: bookmarks are expanded only one level when the PDF is opened
  ods pdf file="&outpath/wind.pdf" startpage=no style=journal pdftoc=1; 
  
  * main content;
  ods noproctitle;
  
  ods proclabel "Wind Statistics";
  title "Wind Statistics by Basin";
  proc means data=pg1.storm_final min mean median max maxdec=0;
      class BasinName;
      var MaxWindMPH;
  run;
  
  * ods proclabel: customize the bookmark labels;
  ods proclabel "Wind Distribution"; 
  title "Distribution of Maximum Wind";
  proc sgplot data=pg1.storm_final;
      histogram MaxWindMPH;
      density MaxWindMPH;
  run;
  title;
  
  ods proctitle;
  
  * close file;
  ods pdf close; 
  ```

  





# L7 Using SQL in SAS

## basic usage

### SQL and SAS

- SAS can integrate SQL
- SQL is typically used in the Prepare data and Analyze and report on data phases of the SAS programming process. It's alternative to DATA step and certain PROC steps.
- Although DATA and PROC steps and SQL definitely have some overlap in terms of functionality, they process data differently behind the scenes, and they each have their own unique advantages. That's why it's valuable to know how to use both native SAS syntax and PROC SQL in your SAS programs.



### PROC SQL

```SAS
PROC SQL;

* SQL stmt;
SELECT clause
          FROM clause
                    <WHERE clause>
                              <ORDER BY clause>;

QUIT;
```

- note: each SQL statement executes immediately and independently.

- example:

  ```SAS
  proc sql;
  * we can still use format in sql part, although this is not standard SQL
  select Name, Age, Height, Birthdate format=date9.
      from pg1.class_birthdate;
  quit; 
  ```





### filter & sorting

```SAS
proc sql;
select Name, Age, Height, Birthdate format=date9.
    from pg1.class_birthdate
    * use SQL to filter and sort;
    where age > 14
    order by Height desc;
quit; 
```



```SAS
title "International Storms since 2000";
title2 "Category 5 (Wind>156)";
proc sql;
* SAS function can still be used in SQL part;
select Season, propcase(Name) as Name, 
       StartDate format=mmddyy10., MaxWindMPH 
    from pg1.storm_final
    * advantage of SQL is easy filtering and sorting;
    where MaxWindMPH > 156 and Season > 2000
    order by MaxWindMPH desc, Name;
quit;
title;   
```



### Creating and Deleting Tables in SQL

- create table

  - **CREATE TABLE** *table-name* **AS**

  - ```SAS
    proc sql;
    create table work.myclass as
        select Name, Age, Height
            from pg1.class_birthdate
            where age > 14
            order by Height desc;
    quit;
    ```

- drop table

  - **DROP TABLE** *table-name*;

  - ```SAS
    proc sql;
        drop table work.myclass;
    quit;
    ```





## Joining Tables Using SQL in SAS

### Creating Inner Joins in SQL

- **FROM** *table1* **INNER JOIN** table2
  **ON** *table1.column* = *table2.column*

- ```SAS
  proc sql;
  select Grade, Age, Teacher 
      from pg1.class_update inner join pg1.class_teachers
      on class_update.Name = class_teachers.Name;
  quit;  
  ```



### Comparing SQL and DATA step

| DATA Step                                                    | PROC SQL                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| provides more control of reading, writing, and manipulating data | ANSI-standard language used by most databases        |
| can create multiple tables in one step                       | code can be more streamlined                         |
| includes looping and array processing                        | can manipulate, summarize, and sort data in one step |
