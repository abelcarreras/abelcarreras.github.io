Development Q-Chem
==================

How to add new REM variable
---------------------------
REM variables are, in principle, integer only variables. However some 
tricks can be done to use them as real.

New REM variables should be defined in file::

   config/rem.list


this file contains the list of all REM variables used in q-chem. Make sure your new
variable does not already exist. Each variable is associated to a number, the range
of numbers available for each user must be reserved in the upper section of the same
file. Make sure to not use more variables than the number of reserved numbers.

Each defined variable should be initializated in file::

   libgen/rem_default.C

The way to initalize a variable is by writting a line as::

   rem_write(0,NAME_OF_VARIABLE);

where 0 is the initial value(integer).

To use REM variables as real, the variable should be define as $10^n$ times the 
itended value, wheren is the maximum number of decimal places of accepted real.
This is defined  function **RemDoubleToValue** placed in file::
   
   qparser/read_rem.C

which should be edited to add the variable definition like::

   if (Loc == NAME_OF_VARIABLE){
     double dVal = TL[1].GetDouble();
     Val = INTEGER(dVal*10000);
     OK = 1;
   }

Then, within your code, in order to recover the intended real number the REM variable
should be divided by $10^n$::

   myrealvariable = rem_read(NAME_OF_VARIABLE) / 10000


