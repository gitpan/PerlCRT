NOTE: Please download the correct zip.

PerlCRT-1.02-bin-1-Alpha-VC.zip		non debug for the Alpha processor
PerlCRT-1.02-bin-1-debug-Alpha-VC.zip	debug for the Alpha processor

PerlCRT-1.02-bin-1-x86-VC.zip		non debug for the Intel processor
PerlCRT-1.02-bin-1-debug-x86-VC.zip	debug for the Intel processor

-----------------------------------------------------------------

PerlCRT.dll and PerlCRT.lib fix problems with MSVCRT.DLL

MSVCRT.DLL has the following known bugs.
  1) On Win95 when a socket handle is passed to the open_osfhandle
     the call to GetFileType() returns FILE_TYPE_UNKNOWN rather
     than FILE_TYPE_CHAR.
  2) When using read() on a file that was opened in text mode,
     and the read terminates between a CR and LF the CR is not
     translated to a LF.
  3) Several time-related functions (localtime(), findfirst(),
     findnext(), fstat(), stat() and time()) are affected by a
     miscalculation of when Daylight Saving Time goes into effect.
     When the begin-date for DST happens to fall on the first day
     of the month, DST is mistakenly reported as not starting until
     one week later.  For example, this causes localtime() to report
     time values that are off by an hour in North America for the
     first week of DST in April, 2001.

-----------------------------------------------------------------

Changes from 1.0
 o Added _free_oshndl to exports

Changes from 1.01
 o Fixed DST bug in time functions

-----------------------------------------------------------------

Building Perl to use with PerlCRT.dll and PerlCRT.lib currently
requires VC 5.0 with Service pack 3
(Service pack 3 can be found at http://www.microsoft.com/vstudio/sp/)


Copy PerlCRT.dll to %SystemRoot%\system32 directory.
Copy PerlCRT.lib to a directory that is in the LIB environment variable.

If you are using nmake uncomment 'USE_PERLCRT	= define' in Makefile
If you are using dmake uncomment 'USE_PERLCRT	*= define' in makefile.mk

To build a version without DEBUGGING defined you will need the corresponding
non debug zip.

To build a version with DEBUGGING defined you will need the corresponding
debug zip.

----------------------------------------------------------------

Diffs for those who are interested:

diff -ruN crt.orig/src/OSFINFO.C crt/src/OSFINFO.C
--- crt.orig/src/OSFINFO.C	Fri Jan 17 07:41:43 1997
+++ src/OSFINFO.C	Tue Jun 16 14:31:41 1998
@@ -335,17 +335,12 @@
         /* find out what type of file (file/device/pipe) */

         isdev = GetFileType((HANDLE)osfhandle);
-        if (isdev == FILE_TYPE_UNKNOWN) {
-            /* OS error */
-            _dosmaperr( GetLastError() );   /* map error */
-            return -1;
-        }

-        /* is isdev value to set flags */
-        if (isdev == FILE_TYPE_CHAR)
-            fileflags |= FDEV;
-        else if (isdev == FILE_TYPE_PIPE)
+		/* is isdev value to set flags */
+        if (isdev == FILE_TYPE_PIPE)
             fileflags |= FPIPE;
+        else
+            fileflags |= FDEV;


         /* attempt to allocate a C Runtime file handle */
diff -ruN crt.orig/src/READ.C crt/src/READ.C
--- crt.orig/src/READ.C	Fri Jan 17 07:41:44 1997
+++ crt/src/READ.C	Tue Jun 16 14:31:47 1998
@@ -221,12 +221,9 @@
                                have several possibilities:
                                1. disk file and char is not LF; just seek back
                                   and copy CR
-                               2. disk file and char is LF; seek back and
-                                  discard CR
-                               3. disk file, char is LF but this is a one-byte
-                                  read: store LF, don't seek back
-                               4. pipe/device and char is LF; store LF.
-                               5. pipe/device and char isn't LF, store CR and
+                               2. disk file and char is LF; store LF, don't seek back
+                               3. pipe/device and char is LF; store LF.
+                               4. pipe/device and char isn't LF, store CR and
                                   put char in pipe lookahead buffer. */
                             if (_osfile(fh) & (FDEV|FPIPE)) {
                                 /* non-seekable device */
@@ -239,7 +236,7 @@
                             }
                             else {
                                 /* disk file */
-                                if (q == buf && peekchr == LF) {
+                                if (peekchr == LF) {
                                     /* nothing read yet; must make some
                                        progress */
                                     *q++ = LF;
diff -ruN crt.orig/src/TZSET.C crt/src/TZSET.C
--- crt.orig/src/TZSET.C	Wed Nov 04 17:40:06 1998
+++ crt/src/TZSET.C	Tue Jan 19 18:13:49 1999
@@ -377,7 +377,7 @@
             /*
              * Figure the year-day of the transition date
              */
-            if ( monthdow < dayofweek )
+            if ( monthdow <= dayofweek )
                 yearday += (dayofweek - monthdow) + (week - 1) * 7;
             else
                 yearday += (dayofweek - monthdow) + week * 7;
End of Patch.

