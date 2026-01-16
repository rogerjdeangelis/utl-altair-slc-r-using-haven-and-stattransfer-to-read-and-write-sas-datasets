# utl-altair-slc-r-using-haven-and-stattransfer-to-read-and-write-sas-datasets
Altair slc r using haven and stattransfer to read and write sas datasets
    %let pgm=utl-altair-slc-r-using-haven-and-stattransfer-to-read-and-write-sas-datasets;

    %stop_submission;

    Altair slc r using haven and stattransfer to read and write sas datasets;

    Too long to post here, see github

    github
    https://github.com/rogerjdeangelis/utl-altair-slc-r-using-haven-and-stattransfer-to-read-and-write-sas-datasets


    CONTENTS

      1. input sas dataset
      2. stattransfer function (create sas dataset)
      3. process subset zipcode (sqlite select from zipcode where)
      4. output subsetted sas dataset
      5. log
      6. slc_rbegin macro
      7. slc_rend macro


    This post supports reading and writing sas datasets,
    without proc r in sas or in the altair slc.
    It also has the ability to do the reads and writes inside python.

    You do need to purchase, stattransfer, to create sas datasets.

    The solution uses macro sandwiches to execute r;

    /*   _                   _                        _       _                 _
    / | (_)_ __  _ __  _   _| |_   ___  __ _ ___   __| | __ _| |_ __ _ ___  ___| |_
    | | | | `_ \| `_ \| | | | __| / __|/ _` / __| / _` |/ _` | __/ _` / __|/ _ \ __|
    | | | | | | | |_) | |_| | |_  \__ \ (_| \__ \| (_| | (_| | || (_| \__ \  __/ |_
    |_| |_|_| |_| .__/ \__,_|\__| |___/\__,_|___/ \__,_|\__,_|\__\__,_|___/\___|\__|
                |_|
    */

    This solution requires

      c:/temp    for temporary files
      c:/wpsoto  for the stattransfer function

    /*--- autoexex has libname workx "d:/wpswrkx" ; ---*/
    proc datasets lib=workx kill;
    run;quit;

    data workx.zipcode;
      set sashelp.zipcode(obs=10 keep= zip x y statename);
    run;quit;

    /*---
    WORKX.ZIPCODE total obs=10

    Obs    ZIP        X          Y        STATENAME

      1    501    -73.0464    40.8131    New York
      2    544    -73.0493    40.8132    New York
      3    601    -66.7236    18.1660    Puerto Rico
      4    602    -67.1866    18.3830    Puerto Rico
      5    603    -67.1520    18.4332    Puerto Rico
      6    604    -67.1359    18.5053    Puerto Rico
      7    605    -67.1513    18.4361    Puerto Rico
      8    606    -66.9774    18.1856    Puerto Rico
      9    610    -67.1442    18.2859    Puerto Rico
     10    611    -66.7976    18.2877    Puerto Rico
    ---*/

    /*___        _        _   _                        __             __                  _   _
    |___ \   ___| |_ __ _| |_| |_ _ __ __ _ _ __  ___ / _| ___ _ __  / _|_   _ _ __   ___| |_(_) ___  _ __
      __) | / __| __/ _` | __| __| `__/ _` | `_ \/ __| |_ / _ \ `__|| |_| | | | `_ \ / __| __| |/ _ \| `_ \
     / __/  \__ \ || (_| | |_| |_| | | (_| | | | \__ \  _|  __/ |   |  _| |_| | | | | (__| |_| | (_) | | | |
    |_____| |___/\__\__,_|\__|\__|_|  \__,_|_| |_|___/_|  \___|_|   |_|  \__,_|_| |_|\___|\__|_|\___/|_| |_|

    save the r function in c:/wpsoto/fn_tosas9x.R. You on;y need to do this omce
    */

    %utlfkil(c:/wpsoto/fn_tosas9x.R); /*--- delete r function ---*/

    data _null_;
      file "c:/wpsoto/fn_tosas9x.R";
      input;
      put _infile_;
    cards4;
     fn_tosas9x<-function(
       inp    =NULL
      ,outlib =NULL
      ,outdsn =NULL
      )
     {
     rds <- tempfile(fileext = ".rds");
     saveRDS(inp, file = rds);
     stcmd <- tempfile(fileext = ".stcmd");
     writeLines(c(
     "set numeric-names      n              "
    ,"set log-level          e              "
    ,"set in-encoding        system         "
    ,"set out-encoding       system         "
    ,"set enc-errors         sub            "
    ,"set enc-sub-char       _              "
    ,"set enc-error-limit    100            "
    ,"set var-case-ci        preserve-always"
    ,"set preserve-str-widths n             "
    ,"set preserve-num-widths n             "
    ,"set recs-to-optimize   all            "
    ,"set factor-as-string   n              "
    ,"set sas-date-fmt       mmddyy         "
    ,"set sas-time-fmt       time           "
    ,"set sas-datetime-fmt   datetime       "
    ,"set write-file-label   none           "
    ,"set write-sas-fmts     n              "
    ,"set sas-outrep         windows_64     "
    ,"set write-old-ver      18             "
    ,paste("copy",rds,"sas9",
      paste0(outlib,outdsn,".sas7bdat"),"-T<outdsn")
    ,"quit"),stcmd);
     unlink(paste0(outlib,outdsn,".sas7bdat"));
     system(paste("c:/PROGRA~1/StatTransfer17-64/st.exe"
       ,stcmd));
      };
    ;;;;
    run;quit;

    /*____                                                 _              _        _                     _
    |___ /   _ __  _ __ ___   ___ ___  ___ ___   ___ _   _| |__  ___  ___| |_  ___(_)_ __   ___ ___   __| | ___
      |_ \  | `_ \| `__/ _ \ / __/ _ \/ __/ __| / __| | | | `_ \/ __|/ _ \ __||_  / | `_ \ / __/ _ \ / _` |/ _ \
     ___) | | |_) | | | (_) | (_|  __/\__ \__ \ \__ \ |_| | |_) \__ \  __/ |_  / /| | |_) | (_| (_) | (_| |  __/
    |____/  | .__/|_|  \___/ \___\___||___/___/ |___/\__,_|_.__/|___/\___|\__|/___|_| .__/ \___\___/ \__,_|\___|
            |_|                                                                     |_|
    */

    proc delete data=workx.want;
    run;quit;

    data workx.zipcode;
      set sashelp.zipcode(obs=10 keep= zip x y statename);
    run;quit;

    %slc_rbegin;
    cards4;
    library(haven)
    library(sqldf)
    source("c:/oto/fn_tosas9x.R")
    options(sqldf.dll = "d:/dll/sqlean.dll")
    zipcode<-read_sas("d:/wpswrkx/zipcode.sas7bdat")
    print(zipcode)
    want<-sqldf('
     select
         *
      from
         zipcode
      where
         statename = "New York"
      order
         by zip
    ')
    want
    fn_tosas9x(
          inp    = want
         ,outlib ="d:/wpswrkx/"
         ,outdsn ="want"
         )
    ;;;;
    %slc_rend;

    proc print data=workx.want;
    run;quit;


    /*  _                 _               _
    | || |     ___  _   _| |_ _ __  _   _| |_
    | || |_   / _ \| | | | __| `_ \| | | | __|
    |__   _| | (_) | |_| | |_| |_) | |_| | |_
       |_|    \___/ \__,_|\__| .__/ \__,_|\__|
                             |_|

    Altair SLC
    > library(haven)
    > library(sqldf)
    > source("c:/oto/fn_tosas9x.R")
    > options(sqldf.dll = "d:/dll/sqlean.dll")
    > zipcode<-read_sas("d:/wpswrkx/zipcode.sas7bdat")
    > print(zipcode)
    # A tibble: 10 Ã— 4
         ZIP     Y     X STATENAME
       <dbl> <dbl> <dbl> <chr>
     1   501  40.8 -73.0 New York
     2   544  40.8 -73.0 New York
     3   601  18.2 -66.7 Puerto Rico
     4   602  18.4 -67.2 Puerto Rico
     5   603  18.4 -67.2 Puerto Rico
     6   604  18.5 -67.1 Puerto Rico
     7   605  18.4 -67.2 Puerto Rico
     8   606  18.2 -67.0 Puerto Rico
     9   610  18.3 -67.1 Puerto Rico
    10   611  18.3 -66.8 Puerto Rico
    > want<-sqldf('
    +  select
    +      *
    +   from
    +      zipcode
    +   where
    +      statename = "New York"
    +   order
    +      by zip
    + ')
    > want
      ZIP        Y         X STATENAME
    1 501 40.81308 -73.04639  New York
    2 544 40.81322 -73.04929  New York
    > fn_tosas9x(
    +       inp    = want
    +      ,outlib ="d:/wpswrkx/"
    +      ,outdsn ="want"
    +      )
      [1] 0
    >

    Altair SLC

    Obs    rownames    ZIP       Y          X        STATENAME

     1         1       501    40.8131    -73.0464    New York
     2         2       544    40.8132    -73.0493    New York

    /*___    _
    | ___|  | | ___   __ _
    |___ \  | |/ _ \ / _` |
     ___) | | | (_) | (_| |
    |____/  |_|\___/ \__, |
                     |___/
    */

    1                                          Altair SLC        06:00 Friday, January 16, 2026

    NOTE: Copyright 2002-2025 World Programming, an Altair Company
    NOTE: Altair SLC 2026 (05.26.01.00.000758)
          Licensed to Roger DeAngelis
    NOTE: This session is executing on the X64_WIN11PRO platform and is running in 64 bit mode

    NOTE: AUTOEXEC processing beginning; file is C:\wpsoto\autoexec.sas
    NOTE: AUTOEXEC source line
    1       +  ï»¿ods _all_ close;
               ^
    ERROR: Expected a statement keyword : found "?"
    NOTE: Library workx assigned as follows:
          Engine:        SAS7BDAT
          Physical Name: d:\wpswrkx

    NOTE: Library slchelp assigned as follows:
          Engine:        WPD
          Physical Name: C:\Progra~1\Altair\SLC\2026\sashelp


    LOG:  6:00:34
    NOTE: 1 record was written to file PRINT

    NOTE: The data step took :
          real time : 0.020
          cpu time  : 0.015


    NOTE: AUTOEXEC processing completed

    1         proc delete data=workx.want;
    2         run;quit;
    NOTE: Deleting "WORKX.WANT" (memtype="DATA")
    NOTE: Procedure delete step took :
          real time : 0.000
          cpu time  : 0.000


    3
    4         data workx.zipcode;
    5           set sashelp.zipcode(obs=10 keep= zip x y statename);
    6         run;

    NOTE: 10 observations were read from "SASHELP.zipcode"
    NOTE: Data set "WORKX.zipcode" has 10 observation(s) and 4 variable(s)
    NOTE: The data step took :
          real time : 0.000
          cpu time  : 0.015


    6       !     quit;
    7
    8         %slc_rbegin;
    9         cards4;

    NOTE: The file 'c:\temp\r_pgm.r' is:
          Filename='c:\temp\r_pgm.r',
          Owner Name=SLC\suzie,
          File size (bytes)=0,
          Create Time=05:57:55 Jan 16 2026,
          Last Accessed=06:00:33 Jan 16 2026,
          Last Modified=06:00:33 Jan 16 2026,
          Lrecl=32767, Recfm=V


    2                                                                                                                         Altair SLC

    NOTE: 22 records were written to file 'c:\temp\r_pgm.r'
          The minimum record length was 80
          The maximum record length was 80
    NOTE: The data step took :
          real time : 0.015
          cpu time  : 0.000


    10        library(haven)
    11        library(sqldf)
    12        source("c:/oto/fn_tosas9x.R")
    13        options(sqldf.dll = "d:/dll/sqlean.dll")
    14        zipcode<-read_sas("d:/wpswrkx/zipcode.sas7bdat")
    15        print(zipcode)
    16        want<-sqldf('
    17         select
    18             *
    19          from
    20             zipcode
    21          where
    22             statename = "New York"
    23          order
    24             by zip
    25        ')
    26        want
    27        fn_tosas9x(
    28              inp    = want
    29             ,outlib ="d:/wpswrkx/"
    30             ,outdsn ="want"
    31             )
    32        ;;;;
    33        %slc_rend;

    NOTE: The infile rut is:
          Unnamed Pipe Access Device,
          Process=C:\Progra~1\R\R-4.5.2\bin\r.exe --vanilla --quiet --no-save < c:/temp/r_pgm.r 2> c:/temp/r_pgm.log,
          Lrecl=32756, Recfm=V

    > library(haven)
    > library(sqldf)
    > source("c:/oto/fn_tosas9x.R")
    > options(sqldf.dll = "d:/dll/sqlean.dll")
    > zipcode<-read_sas("d:/wpswrkx/zipcode.sas7bdat")
    > print(zipcode)
    # A tibble: 10 Ã— 4
         ZIP     Y     X STATENAME
       <dbl> <dbl> <dbl> <chr>
     1   501  40.8 -73.0 New York
     2   544  40.8 -73.0 New York
     3   601  18.2 -66.7 Puerto Rico
     4   602  18.4 -67.2 Puerto Rico
     5   603  18.4 -67.2 Puerto Rico
     6   604  18.5 -67.1 Puerto Rico
     7   605  18.4 -67.2 Puerto Rico
     8   606  18.2 -67.0 Puerto Rico
     9   610  18.3 -67.1 Puerto Rico
    10   611  18.3 -66.8 Puerto Rico
    > want<-sqldf('
    +  select
    +      *
    +   from
    +      zipcode
    +   where

    3                                                                                                                         Altair SLC

    +      statename = "New York"
    +   order
    +      by zip
    + ')
    > want
      ZIP        Y         X STATENAME
    1 501 40.81308 -73.04639  New York
    2 544 40.81322 -73.04929  New York
    > fn_tosas9x(
    +       inp    = want
    +      ,outlib ="d:/wpswrkx/"
    +      ,outdsn ="want"
    +      )
      [1] 0
    >
    NOTE: 40 records were written to file PRINT

    NOTE: 40 records were read from file rut
          The minimum record length was 2
          The maximum record length was 82
    NOTE: The data step took :
          real time : 3.170
          cpu time  : 0.015



    NOTE: The infile rut is:
          Unnamed Pipe Access Device,
          Process=C:\Progra~1\R\R-4.5.2\bin\r.exe --vanilla --quiet --no-save < c:/temp/r_pgm.r 2> c:/temp/r_pgm.log,
          Lrecl=32767, Recfm=V

    > library(haven)
    > library(sqldf)
    > source("c:/oto/fn_tosas9x.R")
    > options(sqldf.dll = "d:/dll/sqlean.dll")
    > zipcode<-read_sas("d:/wpswrkx/zipcode.sas7bdat")
    > print(zipcode)
    # A tibble: 10 Ã— 4
         ZIP     Y     X STATENAME
       <dbl> <dbl> <dbl> <chr>
     1   501  40.8 -73.0 New York
     2   544  40.8 -73.0 New York
     3   601  18.2 -66.7 Puerto Rico
     4   602  18.4 -67.2 Puerto Rico
     5   603  18.4 -67.2 Puerto Rico
     6   604  18.5 -67.1 Puerto Rico
     7   605  18.4 -67.2 Puerto Rico
     8   606  18.2 -67.0 Puerto Rico
     9   610  18.3 -67.1 Puerto Rico
    10   611  18.3 -66.8 Puerto Rico
    > want<-sqldf('
    +  select
    +      *
    +   from
    +      zipcode
    +   where
    +      statename = "New York"
    +   order
    +      by zip
    + ')
    > want
      ZIP        Y         X STATENAME
    1 501 40.81308 -73.04639  New York

    4                                                                                                                         Altair SLC

    2 544 40.81322 -73.04929  New York
    > fn_tosas9x(
    +       inp    = want
    +      ,outlib ="d:/wpswrkx/"
    +      ,outdsn ="want"
    +      )
      [1] 0
    >
    NOTE: 40 records were written to file PRINT

    NOTE: 40 records were read from file rut
          The minimum record length was 2
          The maximum record length was 82
    NOTE: The data step took :
          real time : 3.126
          cpu time  : 0.000



    NOTE: The infile 'c:\temp\r_pgm.log' is:
          Filename='c:\temp\r_pgm.log',
          Owner Name=SLC\suzie,
          File size (bytes)=873,
          Create Time=05:57:55 Jan 16 2026,
          Last Accessed=06:00:39 Jan 16 2026,
          Last Modified=06:00:39 Jan 16 2026,
          Lrecl=32767, Recfm=V

    Loading required package: gsubfn
    Loading required package: proto
    Loading required package: RSQLite
    Stat/Transfer - Command Processor (c) 1986-2024 Circle Systems, Inc.
    www.stattransfer.com
    Version 17.1.1820.1112 - 64 Bit Windows (10.0.26100)

    Serial:      HBRC-NEYA-UXDV-ELMT
    User:      Roger DeAngelis - CompuCraft
    License Type:      Single User Commercial Subscription
    Status:      License OK - Expires March 1, 2026
    Transferring from R Single object: C:\Users\suzie\AppData\Local\Temp\RtmpC4lZIo\file562058933d9d.rds
    Input file has 5 variables
    The input file contains string variables and although optimization is not required by the output format, S/T will enable it to make sure that the output strings will not get truncated.
    Optimizing (All records) ...
    Transferring to SAS Data File - Version Nine: d:/wpswrkx/want.sas7bdat

    2 cases were transferred(0.00 seconds)

    NOTE: 19 records were read from file 'c:\temp\r_pgm.log'
          The minimum record length was 0
          The maximum record length was 184
    NOTE: The data step took :
          real time : 0.000
          cpu time  : 0.000


    34
    35
    36        proc print data=workx.want;
    37        run;quit;
    NOTE: 2 observations were read from "WORKX.want"
    NOTE: Procedure print step took :
          real time : 0.000
          cpu time  : 0.000

    5                                                                                                                         Altair SLC



    38
    39
    ERROR: Error printed on page 1

    NOTE: Submitted statements took :
          real time : 6.517
          cpu time  : 0.093

    /*__         _              _                _
     / /_    ___| | ___   _ __ | |__   ___  __ _(_)_ __   _ __ ___   __ _  ___ _ __ ___
    | `_ \  / __| |/ __| | `__|| `_ \ / _ \/ _` | | `_ \ | `_ ` _ \ / _` |/ __| `__/ _ \
    | (_) | \__ \ | (__  | |   | |_) |  __/ (_| | | | | || | | | | | (_| | (__| | | (_) |
     \___/  |___/_|\___|_|_|   |_.__/ \___|\__, |_|_| |_||_| |_| |_|\__,_|\___|_|  \___/
                      |__|                 |___/

    %macro slc_rbegin;
      %utlfkil(c:/temp/r_pgm.r);
      %utlfkil(c:/temp/r_pgm.log);
      data _null_;
        file "c:/temp/r_pgm.r";
        input;put _infile_;
    %mend slc_rbegin;

    /*____       _                             _
    |___  |  ___| | ___   _ __   ___ _ __   __| | _ __ ___   __ _  ___ _ __ ___
       / /  / __| |/ __| | `__| / _ \ `_ \ / _` || `_ ` _ \ / _` |/ __| `__/ _ \
      / /   \__ \ | (__  | |   |  __/ | | | (_| || | | | | | (_| | (__| | | (_) |
     /_/    |___/_|\___|_|_|    \___|_| |_|\__,_||_| |_| |_|\__,_|\___|_|  \___/
                      |__|
    */

    %macro slc_rend(returnvar=N);
    * EXECUTE THE R PROGRAM;
    options noxwait noxsync;
    filename rut pipe "C:\Progra~1\R\R-4.5.2\bin\r.exe --vanilla --quiet --no-save < c:/temp/r_pgm.r 2> c:/temp/r_pgm.log";
    run;quit;
      data _null_;
        file print;
        infile rut recfm=v lrecl=32756;
        input;
        put _infile_;
        putlog _infile_;
      run;

      * use the clipboard to create macro variable;
      %if %upcase(%substr(&returnVar.,1,1)) ne N %then %do;
        filename clp clipbrd ;
        data _null_;
         length txt $200;
         infile clp;
         input;
         putlog "macro variable &returnVar = " _infile_;
         call symputx("&returnVar.",_infile_,"G");
        run;quit;
      %end;
    data _null_;
      file print;
      infile rut;
      input;
      put _infile_;
      putlog _infile_;
    run;quit;
    data _null_;
      infile "c:/temp/r_pgm.log";
      input;
      putlog _infile_;
    run;quit;

    %mend slc_rend;

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
