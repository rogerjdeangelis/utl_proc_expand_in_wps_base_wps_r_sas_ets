# utl_proc_expand_in_wps_base_wps_r_sas_ets
Compute average rents for a rolling window of three floors from penthouse to street level.  Keywords: sas sql join merge big data analytics macros oracle teradata mysql sas communities stackoverflow statistics artificial inteligence AI Python R Java Javascript WPS Matlab SPSS Scala Perl C C# Excel MS Access JSON graphics maps NLP natural language processing machine learning igraph DOSUBL DOW loop stackoverflow SAS community.
    SAS/ETS Proc expand in Base WPS, WPS Proc R and SAS/ETS

    github
    https://github.com/rogerjdeangelis/utl_proc_expand_in_wps_base_wps_r_sas_ets

    inspired by
    https://stackoverflow.com/questions/50402807/moving-variance-with-aggregation

    Compute average rents for a rolling window of three floors from penthouse to street level.

    INPUT
    =====
                                  |    RULES
    SD1.HAVE total obs=16         |
                                  |
    Obs    BLDG   FLOOR    RENT   |   Rolling Average
                                  |
      1      1      8      800    |    .    not enough info for average
      2      1      7      700    |    .    not enough info for average
      3      1      6      600    |   700   (800+700+600)/3 = 700
      4      1      5      500    |   600   (700+600+500)/3 = 600
      5      1      4      400    |
      6      1      3      300    |
      7      1      2      200    |
      8      1      1      100    |
                                  |
      9      2      9      900    |
     10      2      8      800    |
     11      2      7      700    |
     12      2      6      600    |
     13      2      5      500    |
     14      2      4      400    |
     15      2      3      300    |
     16      2      2      200    |


    PROCESS
    =======

     Base WPS and SAS (proc expand is in base WPS)

      proc expand data=sd1.have out=wantwps(drop=time) method=none;
        by bldg;
        convert rent = rent_movave /transformin=(movave 3 trimleft 2);
      run;quit;

     WPS Proc R  (working code)

      myfun = function(x) rollmean(x, k = 3, fill = NA, align = "right");
      have %>% group_by(BLDG) %>% mutate_each(funs(myfun), RENT) -> wantwps;


    OUTPUT
    =====

    SAME RESULTS WPS and SAS

    WORK.WANTWPS total obs=16

                                       RENT_
    Obs    BLDG    FLOOR    RENT      MOVAVE

      1      1       8       800          .
      2      1       7       700          .
      3      1       6       600        700
      4      1       5       500        600
      5      1       4       400        500
      6      1       3       300        400
      7      1       2       200        300
      8      1       1       100        200

      9      2       9       900          .
     10      2       8       800          .
     11      2       7       700        800
     12      2       6       600        700
     13      2       5       500        600
     14      2       4       400        500
     15      2       3       300        400
     16      2       2       200        300

    WPS PROC R

    The WPS System
                       RENT_
      BLDG    FLOOR    MOVAVE

        1       8         .
        1       7         .
        1       6       700
        1       5       600
        1       4       500
        1       3       400
        1       2       300
        1       1       200
        2       9         .
        2       8         .
        2       7       800
        2       6       700
        2       5       600
        2       4       500
        2       3       400
        2       2       300

    *                _               _       _
     _ __ ___   __ _| | _____     __| | __ _| |_ __ _
    | '_ ` _ \ / _` | |/ / _ \   / _` |/ _` | __/ _` |
    | | | | | | (_| |   <  __/  | (_| | (_| | || (_| |
    |_| |_| |_|\__,_|_|\_\___|   \__,_|\__,_|\__\__,_|

    ;

    options validvarname=upcase;
    libname sd1 "d:/sd1";
    data sd1.have;
    input bldg floor rent;
    cards4;
     1 8 800
     1 7 700
     1 6 600
     1 5 500
     1 4 400
     1 3 300
     1 2 200
     1 1 100
     2 9 900
     2 8 800
     2 7 700
     2 6 600
     2 5 500
     2 4 400
     2 3 300
     2 2 200
    ;;;;
    run;quit;


    SAS/ETS
    =======

    proc expand data=sd1.have out=wantwps(drop=time) method=none;
      by bldg;
       convert rent = rent_movave /transformin=(movave 3 trimleft 2);
    run;quit;

    title "Transformed Series";
    proc print data=out;
    run;

    BASE WPS

    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    proc expand data=sd1.have out=wantwps(drop=time) method=none;
    by bldg;
    convert rent = rent_movave /transformin=(movave 3 trimleft 2);
    run;quit;
    proc print data=wantwps;
    run;quit;
    ');

    WPS PROC R
    ==========

    options ls=171;
    %utl_submit_wps64('
    libname sd1 "d:/sd1";
    options set=R_HOME "C:/Program Files/R/R-3.3.2";
    libname wrk  sas7bdat "%sysfunc(pathname(work))";
    proc r;
    submit;
    source("C:/Program Files/R/R-3.3.1/etc/Rprofile.site", echo=T);
    library(haven);
    have<-read_sas("d:/sd1/have.sas7bdat");
    library(dplyr);
    library(zoo);
    myfun = function(x) rollmean(x, k = 3, fill = NA, align = "right");
    have %>% group_by(BLDG) %>% mutate_each(funs(myfun), RENT) -> wantwps;
    endsubmit;
    import r=wantwps   data=wrk.want;
    run;quit;
    proc print data=wrk.want;
    run;quit;
    ');

