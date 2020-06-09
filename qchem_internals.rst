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

To use REM variables as real, the variable should be define as :math:`10^n` times the
intended value, where *n* is the maximum number of decimal places of accepted real.
This is defined in **RemDoubleToValue** function placed in file::
   
   qparser/read_rem.C

which should be edited to add the variable definition like::

   if (Loc == NAME_OF_VARIABLE){
     double dVal = TL[1].GetDouble();
     Val = INTEGER(dVal*10000);
     OK = 1;
   }

Then, within your code, in order to recover the intended real number the REM variable
should be divided by :math:`10^n` ::

   myrealvariable = rem_read(NAME_OF_VARIABLE) / 10000



Manage scratch files
--------------------

**FileMan** function manages the scratch data stored in files. They are useful to store large
arrays in one section of the code to be used in another part.

Before using a file it has to be defined in *include/fileman.h*. This file contains a list of
all files defined along Q-Chem where each one is associated to a number. This number has to be
unique and is more or less taken in a first come first served basis. In order to organize this file
a little, developers usually reserve some range of numbers for themselves by writing a short
comment.

Opening file
^^^^^^^^^^^^
To read/store data the file first should be opened ::

    # Open to write data
    FileMan(FM_OPEN_W, FILE_NAME, 0, 0, 0, 0, 0);

    # Open to read data
    FileMan(FM_OPEN_R, FILE_NAME, 0, 0, 0, 0, 0);

where **FM_OPEN_R**/**FM_OPEN_W** labels indicate the type. These labels are already defined in Q-Chem
and are available everywhere. **FILE_NAME** is the name of the file defined in *fileman.h* file.

.. note::
    It is interesting to be noted that all these labels are just variables that contain integer numbers
    defined in the internals of Q-Chem. Its probably done this way for interoperability C/Fortran.

Writing data
^^^^^^^^^^^^

To store data in the file ::

    double data[N];
    FileMan(FM_WRITE,FILE_NAME,FM_DP,N,0,FM_BEG, &data);

where **FM_WRITE** labels indicates that the data will be written, **FM_DP** label indicates the data type
(double), **FM_BEG** indicates that the data will be stored from the beginning of the file and **data** is
the array of data to be stored.

.. note::
    Note that *data* argument should be passed to *FileMan* as a memory address.


Reading data
^^^^^^^^^^^^

To determine the length of the array ::

    int N;
    FileMan(FM_GET_LEN,FILE_NAME,FM_INT,0,0,0,&N);

where **FM_GET_LEN** indicates that the length will be retrieved, **FM_INT** indicates the data type
(integer in this case) and **N** is the variable that will contain the array length.

To read the data form file ::

    double data[N];
    FileMan(FM_READ,FILE_NAME,FM_DP,N,0,FM_BEG,&data);

where **FM_READ** indicated reading mode, and **N** is the length of the array.


Closing file
^^^^^^^^^^^^

Once the data is read/written the file should be close by ::

    FileMan(FM_CLOSE, FILE_NAME, 0, 0, 0, 0, 0);


Final notes
^^^^^^^^^^^

When working with armadillo objects (matrices & vectors) the raw data can be accessed
by the methods *.begin()* & *.memptr*. This exposes the memory address which allows
to use FileMan directly on these arrays. The difference between this methods is that *.memptr*
is read only, so it is safer than *.begin()* if the data have only to be read from the array ::

    arma::Col<double> data_array(N);

    FileMan(FM_WRITE,FILE_NAME,FM_DP,N,0,FM_BEG,data_array.begin());
    FileMan(FM_READ,FILE_NAME,FM_DP,N,0,FM_BEG,data_array.memptr);

