
PerlCRTD.dll and PerlCRTD.lib fix problems with MSVCRT.DLL

MSVCRT.DLL has 2 known bugs.
  1) On Win95 when a socket handle is passed to the open_osfhandle
     the call to GetFileType() returns FILE_TYPE_UNKNOWN rather
     than FILE_TYPE_CHAR.
  2) When using read() on a file that was opened in text mode,
     and the read terminates between a CR and LF the CR is not
     translated to a LF.

-----------------------------------------------------------------

Changes fromn 1.0
 o Added _free_oshndl to exports

-----------------------------------------------------------------

Building Perl to use with PerlCRTD.dll and PerlCRTD.lib currently
requires VC 5.0 with Service pack 3
(Service pack 3 can be found at http://www.microsoft.com/vstudio/sp/)


Copy PerlCRTD.dll to %SystemRoot%\system32 directory.
Copy PerlCRTD.lib and PerlCRTD.pdb to a directory that is in the LIB environment variable.

If you are using nmake uncomment 'USE_PERLCRT	= define' in Makefile
If you are using dmake uncomment 'USE_PERLCRT	*= define' in makefile.mk

To build a version without DEBUGGING defined you will need the corresponding
non debug zip.


----------------------------------------------------------------

Diffs for those who are interested

diff -ruN src.orig/OSFINFO.C src/OSFINFO.C
--- src.orig/OSFINFO.C	Fri Jan 17 07:41:43 1997
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
diff -ruN src.orig/READ.C src/READ.C
--- src.orig/READ.C	Fri Jan 17 07:41:44 1997
+++ src/READ.C	Tue Jun 16 14:31:47 1998
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


