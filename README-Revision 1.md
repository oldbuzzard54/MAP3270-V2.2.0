# MAP3270-V2.2.0 Full install from source for MVS 38j systems running Hercules TK3 or TK4- systems.
This project is the complete source code for generating the MAP3270 project from source code.
Please see the main Readme.MD for the full information.

This Readme file documents revision 1 for MAP3270-V2.2.0 only

It came to my attention that generated maps were causing problems on TK4- with 43x80 screens.
A command to Erase the screen was causing the screen to clear and the size revert to 24x80 mode.
The issue revolves around the "alternate partition".  The Erase commands (x'F1' or x'F5') causes
the screen to clear and resets the screen to defaults.  To avoid this, the Erase Alternate (x'7E')
should be used.  Erase Alternate will clear the screen but will not reset the screen to defaults.

If it is happening to you, it can be solved by:

1)  Copy the 'TWCC.txt' fix file into the 'userid.MAP3270.MACLIB(TWCC)'.
    OR
2)  Manually edit the TWCC macro in the 'userid.MAP3270.MACLIB(TWCC)' to match the
    'TWCC' fix file below.  Note-11 lines were added.  They are marked with
    'R1' in cols 70-71.  Also note, the second line was changed.  The
     &ESCAPE=YES should be changed to &ESCAPE=ALT.

TWCC fix: Note-there is a file called TWCC.txt.

         MACRO
&NAME    TWCC  &ESCAPE=YES,&ERASE=ALT,&RESET=NO,&PRNTFMT=NO,         R1*
               &PRNT=NO,&ALARM=NO,&KBUNLK=NO,&RSTMDT=NO
.*********************************************************************
.*
.*   TWCC - GENERATE THE 3270 WRITE CONTROL COMMAND.  OPTIONALLY,
.*          THE ESCAPE AND WRITE PREFIX CAN BE GENERATED, MAKING
.*          THE 3270 SUITABLE FOR USE WITH TPUT.
.*          THIS MACRO IS FIRST WHEN DEFINING A SCREEN.
.*
.* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
.*                                                                  **
.*                                                                  **
.* MAP3270 V2.2.0 - SOFTWARE PACKAGE TO IMPLEMENT FULL SCREEN       **
.* IO FOR IBM 3270 CLASS TERMINALS.                                 **
.*                                                                  **
.*             COPYRIGHT (C) 2021  EDWARD G LISS                    **
.*                     EGLISS4024@GMAIL.COM                         **
.*                                                                  **
.* THIS PROGRAM IS FREE SOFTWARE: YOU CAN REDISTRIBUTE IT           **
.* AND/OR MODIFY IT UNDER THE TERMS OF THE GNU GENERAL PUBLIC       **
.* LICENSE AS PUBLISHED BY THE FREE SOFTWARE FOUNDATION,            **
.* EITHER VERSION 3 OF THE LICENSE, OR ANY LATER VERSION.           **
.*                                                                  **
.* THIS PROGRAM IS DISTRIBUTED IN THE HOPE THAT IT WILL BE          **
.* USEFUL, BUT WITHOUT ANY WARRANTY; WITHOUT EVEN THE IMPLIED       **
.* WARRANTY OF MERCHANTABILITY OR FITNESS FOR A PARTICULAR          **
.* PURPOSE.  SEE THE GNU GENERAL PUBLIC LICENSE FOR MORE            **
.* DETAILS.                                                         **
.*                                                                  **
.* PLEASE SEE HTTPS://WWW.GNU.ORG/LICENSES/ FOR A COPY OF THE       **
.* GNU GENERAL PUBLIC LICENSE.                                      **
.*                                                                  **
.* - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -  *R1
.*                                                                  *R1
.* REVISION 1 (R1) - ADDED SUPPORT FOR ERASE ALTERNATE              *R1
.*                                                                  *R1
.********************************************************************R1
         GBLA  &WCC
         AIF   (T'&NAME EQ 'O').NAMEOK
&NAME    EQU   *
.NAMEOK  ANOP
&WCC     SETA  0
         TFVERI B'00100111',&ESCAPE,'ESCAPE'
         AIF   ('&ESCAPE' EQ 'NO').NOESC
         DC    AL1(&WCC)  3270 ESCAPE
.NOESC   ANOP
         AIF   (T'&ERASE EQ 'O').ERASE                              R1
         AIF   ('&ERASE' NE 'ALT').ERASE                            R1
         DC    X'7E'        3270 WRITE ERASE ALTERNATE              R1
         AGO   .OTHER                                               R1
.ERASE   ANOP                                                       R1
&WCC     SETA  B'11110001'
         TFVERI B'00000100',&ERASE,'ERASE'
         DC    AL1(&WCC)  3270 WRITE(X'F1') OR WRITE/ERASE(X'F5')
.OTHER   ANOP                                                       R1
&WCC     SETA  0
.*
         TFVERI B'01000000',&RESET,'RESET'
         TFVERI B'00110000',&PRNTFMT,'PRNTFMT'
         TFVERI B'00001000',&PRNT,'PRNT'
         TFVERI B'00000100',&ALARM,'ALARM'
         TFVERI B'00000010',&KBUNLK,'KBUNLK'
         TFVERI B'00000001',&RSTMDT,'RSTMDT'
         DC    AL1(&WCC)  3270 WCC
.EXIT    ANOP
         MEND

=======================================================================
=======================================================================
