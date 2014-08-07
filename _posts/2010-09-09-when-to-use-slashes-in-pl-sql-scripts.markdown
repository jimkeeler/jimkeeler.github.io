---
layout: post
title: "When to Use Slashes in PL/SQL Scripts"
date:  2010-09-09 11:34:00
---

I had a PL/SQL script defect today because I left out a slash.  It seemed Toad may have been a little more forgiving than SQL*Plus when I was doing my testing.  This isn't the first time I've been caught by this subtlety.  Asking my colleagues for an explanation yielded various opinions rather than fact, so I decided to try my luck in the mountains of Oracle documentation.  Surprisingly, it didn’t take much effort to find this:

>SQL*Plus treats PL/SQL subprograms in the same manner as SQL commands, except that a semicolon (;) or a blank line does not terminate and execute a block. Terminate PL/SQL subprograms by entering a period (.) by itself on a new line. You can also terminate and execute a PL/SQL subprogram by entering a slash (/) by itself on a new line.

I had been treating my PL/SQL blocks like SQL statements and terminating them with a semicolon.  Doesn't  DECLARE-BEGIN-END; appear to compose a complete and terminated statement?  Regardless; this will be at the forefront of my mind when writing my next script.
