# utl-extract-icd9-codes-from-strings-clinical
Extract icd9 codes from strings clinical.
    Extract icd9 codes from strings clinical

    All 14,567 ICD9 codes with short and long descriptions in a more commonly used format.
    https://tinyurl.com/ycrglpfd
    https://github.com/rogerjdeangelis/utl-extract-icd9-codes-from-strings-clinical/blob/master/icd9.xlsx

    github
    https://github.com/rogerjdeangelis/utl-extract-icd9-codes-from-strings-clinical

    SAS-L
    https://listserv.uga.edu/cgi-bin/wa?A2=SAS-L;ea73bd41.1901a

    You might want to take a look at R packages ICD, ICD9, Comobidity

    Many of your codes are not ICD9 codes.
    I added some additional valid ICD9 codes.

      Solution

        1. Download all ICD9 codes with code, long and short descriptions(adjusted for commonly used format)
           https://tinyurl.com/ycrglpfd
        2. Load codes into an array
        3. Cycle through 14567 codes looking for a matches.
        4. I joined back to the full list so you can ad valid descriptions.


    INPUT
    =====

    WORK.HAVE total obs=26

    Obs    DESCRIPTION

      1    417.1
      2    4722
      3    J73.457
      4    diabetes,hypertension,enlarged prostate
      5    melanoma in 803.5
      6    dizziness 418.2
      7    PTSD anxiety 609 diagnoses in 2007
      8    STC 755.3
      9    207.23
     10    herniated disc numbness in leg 587.1
     11    C25.12
     12    Prostate cancer 43.
     13    32.
     14    blurry vision, lower back pain migraines 588.
     15    Depression opioid addiction
     16    2008 - Colon cancer ,421 - balance problems
     17    COPD 707 Peripheral Arterial Disease E849.0
     18    Heart disease in 2006 stent put in
     19    Bladder cancer C32.87
     20    Bedford MA Hospital System VISN V12.3
     21    Chronic heart disease 587.
     22    diagnosis in 2001.
     23    high blood pressure dystonia
     24    tumor in arm, classified as malignant D34.23
     25    Shortness of breath 25/15 mg of respiratory meds
     26    VISN 12 Brewster Medical Center 472.


    EXAMPLE OUTPUT  (surpessed long description)
    --------------------------------------------

     CMS_CODE    CMS_SHORT_DESCRIPTION       RAW_TEXT

      417.1      Pulmon artery aneurysm      417.1
      NA                                     4722
      NA                                     J73.457
      NA                                     diabetes,hypertension,enlarged prostate
      803.5      Open skull fracture NEC     melanoma in 803.5
      NA                                     dizziness 418.2
      NA                                     PTSD anxiety 609 diagnoses in 2007
      755.3      Reduction deform leg NOS    STC 755.3
      NA                                     207.23
      NA                                     herniated disc numbness in leg 587.1
      NA                                     C25.12
      NA                                     Prostate cancer 43.
      32.        Faucial diphtheria          32.
      588.       Renal osteodystrophy        blurry vision, lower back pain migraines 588.
      NA                                     Depression opioid addiction
      NA                                     2008 - Colon cancer ,421 - balance problems
      E849.0     Accident in home            COPD 707 Peripheral Arterial Disease E849.0
      NA                                     Heart disease in 2006 stent put in
      NA                                     Bladder cancer C32.87
      V12.3      Hx-blood diseases           Bedford MA Hospital System VISN V12.3
      587.       Renal sclerosis NOS         Chronic heart disease 587.
      NA                                     diagnosis in 2001.
      NA                                     high blood pressure dystonia
      NA                                     tumor in arm, classified as malignant D34.23
      NA                                     Shortness of breath 25/15 mg of respiratory meds
      472.       Chronic rhinitis            VISN 12 Brewster Medical Center 472.


    PROCESS
    =======

    data want( rename=(
        diagnosis_code    = cms_code
        short_description = cms_short_description
        description       = raw_text
        ));

        * IMPORT EXCEL ICD9 CODES AND DESCRIPTIONS;

        if _n_=0 then do; %let rc=%sysfunc(dosubl('
         libname xel "d:/xls/icd9.xlsx";
          proc sql;
             create
                table icd9s(index=(diagnosis_code/unique)) as
             select
                *
             from
                xel.icd9
          ;quit;
          libname xel clear;
          %put &=sqlobs;
          '));
       end;

       * LOAD ARRAY FOR LOOKUP;

       length diagnosis_code $16;
       array icd9s[&obsSql.] $16 _temporary_ (&sqlobs.*"0");
       do idx=1 to &obsSql.;
          set icd9s;
          icd9s[idx]=diagnosis_code;
       end;

       * LOOKUP AND ADD DESCRIPTIONS;

       do until(dne);
         set have end=dne;
         do idx=1 to &obsSql;
            if indexw(description,strip(icd9s[idx]),' ') > 0  then do;
                  diagnosis_code=trim(icd9s[idx]);
                  set icd9s key=diagnosis_code/unique;
                  output;
                  call missing(long_description,short_description);
                  leave;
            end;
         end;
         if idx=&obsSql + 1 then do;
             diagnosis_code="NA";
             call missing(long_description,short_description);
             output;
         end;
       end;
       keep diagnosis_code description long_description short_description;
       stop;
    run;quit;


    proc print data=want(drop=long_description) width=minimum;
    run;quit;


    OUTPUT
    ======

    see above;


