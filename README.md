# utl-proof-of-concept-using-dosubl-to-create-a-fcmp-like-function-for-a-rolling-sum-of-size-three
Proof of concept using dosubl to create a fcmp like function for a rolling sum of size three 
    %let pgm=utl-proof-of-concept-using-dosubl-to-create-a-fcmp-like-function-for-a-rolling-sum-of-size-three;

    Proof of concept using dosubl to create a fcmp like function for a rolling sum of size three

    Compute a rolling sum of size 3 using a dosubl function.

    Dosubl executes at datastep execution time. Wrapping dosubl in a macrois potentially more pwerfull than FCMP.
    In this case below the macro doe not generate static code before the datasteps starts.

    Why dosubl is potentially more powerfull than FCMP

       1. DOSUBL supports file and datastep IO
       2. DOSUBL supports more datastep statments and functions than FCMP
          (FCMP appears to be a subset of the SAS datastep statements)


    SAS neeed to enhance dosubl with common shared storage, and perhaps make it an executable.
    see
    https://github.com/rogerjdeangelis/utl_sharing_a_block_of_memory_with_dosubl

    github
    https://tinyurl.com/ykpjvubr
    https://github.com/rogerjdeangelis/utl-proof-of-concept-using-dosubl-to-create-a-fcmp-like-function-for-a-rolling-sum-of-size-three

    /*               _     _
     _ __  _ __ ___ | |__ | | ___ _ __ ___
    | `_ \| `__/ _ \| `_ \| |/ _ \ `_ ` _ \
    | |_) | | | (_) | |_) | |  __/ | | | | |
    | .__/|_|  \___/|_.__/|_|\___|_| |_| |_|
    |_|
    */

    /**************************************************************************************************************************/
    /*                                     |                                                                  |               */
    /*                                     |                                                                  |               */
    /*                                     |                                                                  | ROLLING SUM 3 */
    /*            INPUT                    |                   PROCESS                                        |    OUTPUT     */
    /*                                     |                                                                  |               */
    /*  array vec[9]  (1,2,3,4,5,6,7,8,9); |  data want;                                                      | Obs SUMWINDOW */
    /*                                     |                                                                  |               */
    /*                                     |      %let window=3;                                              |  1       6    */
    /*                                     |                                                                  |  2       9    */
    /*                                     |      array vec[9]  (1,2,3,4,5,6,7,8,9);                          |  3      12    */
    /*                                     |                                                                  |  4      15    */
    /*                                     |      retain sumwindow 0;               /*----  FCMP ARGS  ----*/ |  5      18    */
    /*                                     |                                                                  |  6      21    */
    /*                                     |      call symputx("varadr",put(addrlong(vec1),hex16.),"G");      |  7      24    */
    /*                                     |      call symputx("varret",put(addrlong(sumwindow),hex16.),"G"); |               */
    /*                                     |                                                                  |               */
    /*                                     |      do beg=1 to (dim(vec) - &window + 1);                       |               */
    /*                                     |                                                                  |               */
    /*                                     |         call symputx('beg',put(beg,2.));                         |               */
    /*                                     |         call_sumwindow = %sumwindow; /*---- CALL FCMP     ----*/ |               */
    /*                                     |                                                                  |               */
    /*                                     |         if beg <= (dim(vec) - &window +1) then output;           |               */
    /*                                     |      end;                                                        |               */
    /*                                     |                                                                  |               */
    /*                                     |      keep sumwindow;                                             |               */
    /*                                     |                                                                  |               */
    /*                                     |   run;quit;                                                      |               */
    /*                                     |                                                                  |               */
    /*                                     |                                                                  |               */
    /*                                     |   MACRO                                                          |               */
    /*                                     |                                                                  |               */
    /*                                     |   %macro sumwindow ;  /*---- SIMULATE FCMP                ----*/ |               */
    /*                                     |    dosubl('                                                      |               */
    /*                                     |      data _null_;                                                |               */
    /*                                     |        retain cum 0;                                             |               */
    /*                                     |        chr = peekclong("&varadr,"x,72);                          |               */
    /*                                     |        do idx=symget("beg") to (symget("beg") + %eval(&window-1))|;              */
    /*                                     |          cum=cum+input(substr(chr,1+ 8*(idx-1),8),rb8.);         |               */
    /*                                     |        end;                                                      |               */
    /*                                     |        call pokelong(cum,"&varret."x, 8);                        |               */
    /*                                     |      run;quit;                                                   |               */
    /*                                     |   ');                                                            |               */
    /*                                     |   %mend sumwindow;                                               |               */
    /*                                     |                                                                  |               */
    /**************************************************************************************************************************/

    /*                   _      ___
    (_)_ __  _ __  _   _| |_   ( _ )    _ __  _ __ ___   ___ ___  ___ ___
    | | `_ \| `_ \| | | | __|  / _ \/\ | `_ \| `__/ _ \ / __/ _ \/ __/ __|
    | | | | | |_) | |_| | |_  | (_>  < | |_) | | | (_) | (_|  __/\__ \__ \
    |_|_| |_| .__/ \__,_|\__|  \___/\/ | .__/|_|  \___/ \___\___||___/___/
            |_|                        |_|
    */

    proc datasets lib=work mt=data mt=view; delete want; run;quit;

    data want;

        %let window=3;

        %symdel varadr varret beg / nowarn;

        array vec[9]  (1,2,3,4,5,6,7,8,9);

        retain sumwindow 0;               /*----  FCMP ARGS  ----*/

        call symputx("varadr",put(addrlong(vec1),hex16.),"G");
        call symputx("varret",put(addrlong(sumwindow),hex16.),"G");

        do beg=1 to (dim(vec) - &window + 1);

           call symputx('beg',put(beg,2.));
           call_sumwindow = %sumwindow;   /*---- CALL FCMP   ----*/

           if beg <= (dim(vec) - &window +1) then output;
        end;

        keep sumwindow;

     run;quit;

    %macro sumwindow ;  /*---- SIMULATE FCMP                  ----*/
      dosubl('
        data _null_;
          retain cum 0;
          chr = peekclong("&varadr,"x,72);
          do idx=symget("beg") to (symget("beg") + %eval(&window-1)) ;
            cum=cum+input(substr(chr,1+ 8*(idx-1),8),rb8.);
          end;
          call pokelong(cum,"&varret."x, 8);
        run;quit;
     ');
    %mend sumwindow;


    /**************************************************************************************************************************/
    /*                                                                                                                        */
    /*   Obs    SUMWINDOW                                                                                                     */
    /*                                                                                                                        */
    /*    1          6                                                                                                        */
    /*    2          9                                                                                                        */
    /*    3         12                                                                                                        */
    /*    4         15                                                                                                        */
    /*    5         18                                                                                                        */
    /*    6         21                                                                                                        */
    /*    7         24                                                                                                        */
    /*                                                                                                                        */
    /**************************************************************************************************************************/

    REPO
    ------------------------------------------------------------------------------------------------------------------------------------------
    https://github.com/rogerjdeangelis/utl_sharing_a_block_of_memory_with_dosubl
    https://github.com/rogerjdeangelis/utl_dosubl_subroutine_interfaces
    https://github.com/rogerjdeangelis/-utl-delete-dosubl-created-sas-macro-libraries
    https://github.com/rogerjdeangelis/Dynamic_variable_in_a_DOSUBL_execute_macro_in_SAS
    https://github.com/rogerjdeangelis/utl-DOSUBL-running-sql-inside-a-datastep-to-check-if-variables-exist-in-another-table
    https://github.com/rogerjdeangelis/utl-No-need-to-convert-your-datastep-code-to-macro-code-use-DOSUBL
    https://github.com/rogerjdeangelis/utl-a-better-call-execute-using-dosubl
    https://github.com/rogerjdeangelis/utl-academic-pipes-dosubl-open-defer-and-dropping-dowm-to-multiple-languages-in-one-datastep
    https://github.com/rogerjdeangelis/utl-adding-female-students-to-an-all-male-math-class-using-sql-insert_and_dosubl
    https://github.com/rogerjdeangelis/utl-adding-summary-statistics-to-your-datastep-input-table-macro-dosubl
    https://github.com/rogerjdeangelis/utl-append-and-split-tables-into-two-tables-one-with-common-variables-and-one-without-dosubl-hash
    https://github.com/rogerjdeangelis/utl-applying-meta-data-and-dosubl-to-create-mutiple-subset-tables
    https://github.com/rogerjdeangelis/utl-cleaner-macro-code-using-dosubl
    https://github.com/rogerjdeangelis/utl-dosubl-a-more-powerfull-macro-sysfunc-command
    https://github.com/rogerjdeangelis/utl-dosubl-and-do-over-as-alternatives-to-explicit-macros
    https://github.com/rogerjdeangelis/utl-dosubl-more-precise-eight-byte-float-computations-at-macro-excecution-time
    https://github.com/rogerjdeangelis/utl-dosubl-persistent-hash-across-datasteps-and-procedures
    https://github.com/rogerjdeangelis/utl-dosubl-remove-text-within-parentheses-of-macro-variable-using-regex
    https://github.com/rogerjdeangelis/utl-dosubl-using-meta-data-with-column-names-and-labels-to-create-mutiple-proc-reports
    https://github.com/rogerjdeangelis/utl-drop-down-using-dosubl-from-sas-datastep-to-wps-r-perl-powershell-python-msr-vb
    https://github.com/rogerjdeangelis/utl-embed-sql-code-inside-proc-report-using-dosubl
    https://github.com/rogerjdeangelis/utl-embedding-dosubl-in-a-macro-and-returning-an-updated-environment-variable-contents
    https://github.com/rogerjdeangelis/utl-error-checking-sql-and-executing-a-datastep-inside-sql-dosubl
    https://github.com/rogerjdeangelis/utl-extracting-sas-meta-data-using-sas-macro-fcmp-and-dosubl
    https://github.com/rogerjdeangelis/utl-get-dataset-attributes-at-macro-time-within-a-datastep-using-attrn-dosubl-macros
    https://github.com/rogerjdeangelis/utl-in-memory-hash-output-shared-with-dosubl-hash-subprocess
    https://github.com/rogerjdeangelis/utl-let-dosubl-and-the-sas-interpreter-work-for-you
    https://github.com/rogerjdeangelis/utl-load-configuation-variable-assignments-into-an-sas-array-macro-and-dosubl
    https://github.com/rogerjdeangelis/utl-loop-through-one-table-and-find-data-in-next-table--hash-dosubl-arts-transpose
    https://github.com/rogerjdeangelis/utl-macro-klingon-solution-or-simple-dosubl-you-decide
    https://github.com/rogerjdeangelis/utl-macro-with-dosubl-to-compute-last-day-of-month
    https://github.com/rogerjdeangelis/utl-maitainable-macro-function-code-using-dosubl
    https://github.com/rogerjdeangelis/utl-passing-a-datastep-array-to-dosubl-squaring-the-elements-passing-array-back-to-parent
    https://github.com/rogerjdeangelis/utl-potentially-useful-dosubl-interface
    https://github.com/rogerjdeangelis/utl-re-ordering-variables-into-alphabetic-order-in-the-pdv-macros-dosubl
    https://github.com/rogerjdeangelis/utl-rename-variables-with-the-same-prefix-dosubl-varlist
    https://github.com/rogerjdeangelis/utl-sas-array-macro-fcmp-or-dosubl-take-your-choice
    https://github.com/rogerjdeangelis/utl-select-high-payment-periods-and-generating-code-with-do_over-and-dosubl
    https://github.com/rogerjdeangelis/utl-some-interesting-applications-of-dosubl
    https://github.com/rogerjdeangelis/utl-transpose-multiple-rows-into-one-row-do_over-dosubl-and-varlist-macros
    https://github.com/rogerjdeangelis/utl-twelve-interfaces-for-dosubl
    https://github.com/rogerjdeangelis/utl-use-dosubl-to-save-your-format-code-inside-proc-report
    https://github.com/rogerjdeangelis/utl-using-dosubl-and-a-dynamic-arrays-to-add-new-variables
    https://github.com/rogerjdeangelis/utl-using-dosubl-to-avoid-klingon-obsucated-macro-coding
    https://github.com/rogerjdeangelis/utl-using-dosubl-to-avoid-macros-and-add-an-error-checking-log
    https://github.com/rogerjdeangelis/utl-using-dosubl-to-exceute-r-for-each-row-in-a-dataset
    https://github.com/rogerjdeangelis/utl-using-dosubl-with-data-driven-business-rules-to-split-a-table
    https://github.com/rogerjdeangelis/utl-using-dynamic-tables-to-interface-with-dosubl
    https://github.com/rogerjdeangelis/utl_avoiding_macros_and_call_execute_by_using_dosubl_with_log
    https://github.com/rogerjdeangelis/utl_dosubl_do_regressions_when_data_is_between_dates
    https://github.com/rogerjdeangelis/utl_dosubl_macros_to_select_max_value_of_a_column_at_datastep_execution_time
    https://github.com/rogerjdeangelis/utl_dynamic_subroutines_dosubl_with_error_checking
    https://github.com/rogerjdeangelis/utl_overcoming_serious_deficiencies_in_call_execute_with_dosubl
    https://github.com/rogerjdeangelis/utl_pass_character_and_numeric_arrays_to_dosubl
    https://github.com/rogerjdeangelis/utl_passing-in-memory-sas-objects-to-and-from-dosubl
    https://github.com/rogerjdeangelis/utl_read_all_datasets_in_a_library_and_conditionally_split_them_with_error_checking_dosubl
    https://github.com/rogerjdeangelis/utl_using_dosubl_instead_of_a_macro_to_avoid_numeric_truncation
    https://github.com/rogerjdeangelis/utl_using_dosubl_to_avoid_klingon_macro_quoting_functions
    https://github.com/rogerjdeangelis/utl_why_proc_import_export_needs_to_be_deprecated_and_dosubl_acknowledged

    /*              _
      ___ _ __   __| |
     / _ \ `_ \ / _` |
    |  __/ | | | (_| |
     \___|_| |_|\__,_|

    */
