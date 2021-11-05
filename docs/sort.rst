.. _header-n0:

Sort
====

Sort with Morn is fast and simple, Generic is supported to some extent.

.. _header-n3:

API
---

The source code of `Morn <https://github.com/jingweizhanghuai/Morn>`__ sort is
`morn_sort.c <https://github.com/jingweizhanghuai/Morn/blob/master/src/math/morn_sort.c>`__, and APIs are defined in
`morn_math.h <https://github.com/jingweizhanghuai/Morn/blob/master/include/morn_math.h>`__.

.. tip:: 

   "Generic" is supported，``Type`` in following APIs can be ``int8_t``, ``uint8_t``,
   ``int16_t``, ``uint16_t``, ``int32_t``, ``uint32_t``, ``int64_t``,
   ``uint64_t``, ``float`` or ``double``.

.. _header-n7:

Ascending Sort
~~~~~~~~~~~~~~

.. code:: c

   void mAscSort(Type *data_in,int num);
   void mAscSort(Type *data_in,Type *data_out,int num);
   void mAscSort(Type *data_in,Type *data_out,int *index_out,int num);
   void mAscSort(Type *data_in,int *index_in,Type *data_out,int *index_out,int num);

``data_in`` is the input data before sort, and ``data_out`` is the sorted
outpu.

when ``data_out`` is equal as ``data_in`` (or ``data_out`` is not set,
or ``data_out`` ==NULL), the output data will overwrite the input.

``index_in`` is the index of ``data_in``, ``index_out`` is output index
for ``data_out``.

If neither is set(or both are set NULL), it means index is not care.

If ``index_in`` ==NULL and ``index_out``!=NULL, the default ascending
order(0, 1, 2, 3 etc.) is used.

If ``index_out`` == ``index_in`` or ``index_out`` ==NULL, the output
index will overwrite the input.

``num`` is the number of input.

Descending Sort
~~~~~~~~~~~~~~~

.. code:: c

   void mDescSort(Type *data_in,int num);
   void mDescSort(Type *data_in,Type *data_out,int num);
   void mDescSort(Type *data_in,Type *data_out,int *index_out,int num);
   void mDescSort(Type *data_in,int *index_in,Type *data_out,int *index_out,int num);

It is same as ``mAscSort``.

Minimum Subset
~~~~~~~~~~~~~~

.. code:: c

   Type mMinSubset(Type,Type *data_in,int num_in,int num_out);
   Type mMinSubset(Type,Type *data_in,int num_in,Type *data_out,int num_out);
   Type mMinSubset(Type,Type *data_in,int num_in,Type *data_out,int *index_out,int num_out);
   Type mMinSubset(Type,Type *data_in,int *index_in,int num_in, Type *data_out,int *index_out,int num_out);

This is used to select ``num_out`` smallest from all ``num_in`` data.

.. note:: 

   The data selected is not sorted in order.

``data_in``, ``data_out``, ``index_in``, ``index_out`` is same with
``mAscSort`` and ``mDescSort``

The return is threshold value, which is the largest one in all output.

Maximum Subset 
~~~~~~~~~~~~~~

.. code:: c

   Type mMaxSubset(Type,Type *data_in,int num_in,int num_out);
   Type mMaxSubset(Type,Type *data_in,int num_in,Type *data_out,int num_out);
   Type mMaxSubset(Type,Type *data_in,int num_in,Type *data_out,int *index_out,int num_out);
   Type mMaxSubset(Type,Type *data_in,int *index_in,int num_in, Type *data_out,int *index_out,int num_out);

It is same with ``mMinSubset``.

The return is threshold value, which is the smallest one in all output.

Sort List Element
~~~~~~~~~~~~~~~~~

All above APIs is for types of number, and Morn provides ``mListSort``
for ``MList``, which is a data container for all types. See
`MList <Morn:MList2>`__ for details.

Example
-------

Here is some simple examples. The complete code is `test_sort.c <https://github.com/jingweizhanghuai/Morn/blob/master/test/test_sort.c>`__.

Data Sort
~~~~~~~~~

.. code:: c

   #define N 16
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   mAscSort(data,N);
   printf( "\nout :");for(int i=0;i<N;i++) {printf("%d,",data[i]);}
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   mDescSort(data,N);
   printf( "\nout :");for(int i=0;i<N;i++) {printf("%d,",data[i]);}

Output is:

.. code:: 

   in :617,652,-370,597,-310,674,-335,-353,407,-630,-481,964,-454,-654,-146,-942,
   out :-942,-654,-630,-481,-454,-370,-353,-335,-310,-146,407,597,617,652,674,964,
   
   in :-878,-441,-239,-970,-803,-267,-718,749,-662,980,-955,922,-577,-991,-789,348,
   out :980,922,749,348,-239,-267,-441,-577,-662,-718,-789,-803,-878,-955,-970,-991,

Sort with Index
~~~~~~~~~~~~~~~

.. code:: c

   #define N 16
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   mAscSort(data,NULL,index,N);
   printf(" \nout :");for(int i=0;i<N;i++) {printf("%d(%d),",data[i],index[i]);}
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   mDescSort(data,NULL,index,N);
   printf( "\nout :");for(int i=0;i<N;i++) {printf("%d(%d),",data[i],index[i]);}

Output is:

.. code:: 

   in :928,730,543,999,-955,343,718,351,-369,444,-37,172,-130,-154,-905,582,
   out :-955(4),-905(14),-369(8),-154(13),-130(12),-37(10),172(11),343(5),351(7),444(9),543(2),582(15),718(6),730(1),928(0),999(3),
   
   in :63,416,461,-475,835,873,804,320,522,-898,630,-316,766,641,-591,104,
   out :873(5),835(4),804(6),766(12),641(13),630(10),522(8),461(2),416(1),320(7),104(15),63(0),-316(11),-475(3),-591(14),-898(9),

Minimum/Maximum Subset
~~~~~~~~~~~~~~~~~~~~~~

.. code:: c

   #define N 16
   #define M 4
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   threshold=mMinSubset(data,N,M);
   printf( "\nout :");for(int i=0;i<M;i++) {printf("%d,",data[i]);}
   printf("\nthreshold=%d\n",threshold);
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   threshold=mMaxSubset(data,N,M);
   printf( "\nout :");for(int i=0;i<M;i++) {printf("%d,",data[i]);}
   printf("\nthreshold=%d\n",threshold);

Output is:

.. code:: 

   in :89,163,-627,-103,-8,848,-419,11,79,761,183,589,542,-318,-364,0,
   out :-419,-364,-627,-318,
   threshold=-318

   in :521,890,155,-136,580,-535,-196,-704,766,-269,-388,116,335,651,-238,-922,
   out :580,890,651,766,
   threshold=580

Minimum/Maximum Subset with Index
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: c

   #define N 16
   #define M 4
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   threshold=mMinSubset(data,N,NULL,index,M);
   printf( "\nout :");for(int i=0;i<M;i++) {printf("%d(%d),",data[i],index[i]);}
   printf("\nthreshold=%d\n",threshold);
   
   printf("\n\nin :");for(int i=0;i<N;i++) {data[i] = mRand(-1000,1000);printf("%d,",data[i]);}
   threshold=mMaxSubset(data,N,NULL,index,M);
   printf( "\nout :");for(int i=0;i<M;i++) {printf("%d(%d),",data[i],index[i]);}
   printf("\nthreshold=%d\n",threshold);

Output is:

.. code:: 

   in :-126,189,-359,-455,-786,118,695,868,907,673,-63,379,-275,849,-868,296,
   out :-786(4),-868(14),-455(3),-359(2),
   threshold=-359
   
   in :-640,-920,411,-341,573,630,-462,-780,73,860,-109,-36,-782,638,880,-499,
   out :880(14),638(13),860(9),630(5),
   threshold=-630

Performance
-----------

Complete testing code is: `test_sort2.cpp <https://github.com/jingweizhanghuai/Morn/blob/master/test/test_sort2.cpp>`__.
Compile command for these testings is:

.. code:: shell

   g++ -Ofast -DNDEBUG test_sort2.cpp -o test_sort2.exe -lgsl -lgslcblas -lmorn

Data Sort
~~~~~~~~~

Here, we compared Morn with other 3 libraries: ``qsort`` in C standard
library, ``gsl_sort`` in `GSL(The GNU Scientific Library) <https://www.gnu.org/software/gsl/>`__ and
``std::sort`` in C++ STL.

Testing Code is:

.. code:: c

   #include <algorithm>
   #include <gsl/gsl_sort_double.h>
   #include "morn_math.h"
   
   int compare(const void *v1, const void *v2) {return ((*((double *)v1))>(*((double *)v2)))?1:-1;}
   void test1()
   {
       double *data1= (double *)mMalloc(10000000* sizeof(double));
       double *data2= (double *)mMalloc(10000000* sizeof(double));
       double *data3= (double *)mMalloc(10000000* sizeof(double));
       double *data4= (double *)mMalloc(10000000* sizeof(double));
    
       for(int n=1000;n<=10000000;n*=10)
       {
           printf("\n%d data sort for %d times:\n",n,10000000/n);
           for(int i=0;i<10000000;i++)
           {
               data1[i]=((double)mRand(-10000000,10000000))/((double)mRand(1,10000));
               data2[i]=data1[i];data3[i]=data1[i];data4[i]=data1[i];
           }
           
           mTimerBegin("qsort");
           for(int i=0;i<10000000;i+=n) qsort(data1+i,n,sizeof(double),compare);
           mTimerEnd("qsort");
           
           mTimerBegin("gsl")
           for(int i=0;i<10000000;i+=n) gsl_sort(data2+i,1,n);
           mTimerEnd("gsl");
           
           mTimerBegin("stl");
           for(int i=0;i<10000000;i+=n) std::sort(data3+i,data3+i+n);
           mTimerEnd("stl");
           
           mTimerBegin("Morn");
           for(int i=0;i<10000000;i+=n) mAscSort(data4+i,n);
           mTimerEnd("Morn");
       }
       
       mFree(data1); mFree(data2); mFree(data3); mFree(data4);
   }

In above program, we firstly generate some random double precision floats, and then
measure time-consume of: 1. sorting 1000 data for 10000times, 2.
sorting 10000 data for 1000times, 3.sorting 100000 data for 100 times,
4.sorting 1000000 data for 10 times and 5.sorting all 10000000 data for
1 time. Output is:

|image1|

It can be seen that: 1. ``std::sort`` **and** ``mAscSort`` **is the
fastest**, 2.for small amount of data, ``gsl_sort`` is faster then
``qsort``, but for the large amount ``qsort`` is faster.

Sort with Index
~~~~~~~~~~~~~~~

Sorting performance is always a concern, Here we compared ``mAscSort`` in Morn and ``gsl_sort_index`` in `GSL <https://www.gnu.org/software/gsl/>`__.
Testing code is:

.. code:: c

   void test2()
   {
       double *data1 = (double *)mMalloc(10000000* sizeof(double));
       double *data2 = (double *)mMalloc(10000000* sizeof(double));
       size_t *index1= (size_t *)mMalloc(10000000* sizeof(size_t));
       int    *index2= (int    *)mMalloc(10000000* sizeof(int   ));

       for(int n=1000;n<=10000000;n*=10)
       {
           printf("\n%d data sort with index for %d times:\n",n,10000000/n);
           for(int i=0;i<10000000;i++)
           {
               data1[i]=((double)mRand(-10000000,10000000))/((double)mRand(1,10000));
               data2[i]=data1[i];
           }
           mTimerBegin("gsl");
           for(int i=0;i<10000000;i+=n) gsl_sort_index(index1,data1+i,1,n);
           mTimerEnd("gsl");
           
           mTimerBegin("Morn");
           for(int i=0;i<10000000;i+=n) mAscSort(data2+i,NULL,index2,n);
           mTimerEnd("Morn");
       }
       
       mFree(data1); mFree(data2);mFree(index1);mFree(index2);
   }

In above program, we firstly generate some random double precision floats, and then
measure time-consume of: 1. sorting 1000 data for 10000 times, 2.
sorting 10000 data for 1000 times, 3.sorting 100000 data for 100 times,
4.sorting 1000000 data for 10 times and 5.sorting all 10000000 data for
1 time. Output is:

|image2|

Obviously: **Morn sort is faster than GSL**. And as the amount
increases, the speed gap widens.

.. note::

   ``gsl_sort_index`` and ``mAscSort`` are different with:
   ``gsl_sort_index`` Outputs only sorted index, without sorted data, But
   ``mAscSort`` Outputs sorted data and sorted index.

.. _header-n72:

Minimum/Maximum Subset
~~~~~~~~~~~~~~~~~~~~~~

Firstly, we compared ``mMinSubset`` in Morn and ``std::nth_element`` in
C++ STL. Test code is:

.. code:: c

   void test3_1()
   {
       double *data1= (double *)mMalloc(10000000*sizeof(double));
       double *data2= (double *)mMalloc(10000000*sizeof(double));
       for(int n=100000;n<=10000000;n*=10)
           for(int m=n/10;m<n;m+=n/5)
           {
               printf("\nselect %d from %d data for %d times\n",m,n,10000000/n);
               for(int i=0;i<10000000;i++)
               {
                   data1[i]=((double)mRand(-1000000,1000000))/((double)mRand(1,1000));
                   data2[i]=data1[i];
               }
               mTimerBegin("stl");
               for(int i=0;i<10000000;i+=n) std::nth_element(data1+i,data1+i+m-1,data1+i+n);
               mTimerEnd("stl");
               
               mTimerBegin("Morn");
               for(int i=0;i<10000000;i+=n) mMinSubset(data2+i,n,m);
               mTimerEnd("Morn");
           }
       mFree(data1);mFree(data2);
   }

In above program, we generate some double precision floats, and then test:
1.selecting 10000, 30000, 50000, 70000, 90000 data from 100000 for 100
times, 2.selecting 100000, 300000, 500000, 700000, 900000 data from 1000000
for 10 times, 3.selecting 1000000, 3000000, 5000000, 7000000, 9000000 data
from 10000000 for 1 time. The testing code is:

|image3|

It shows that: ``mMinSubset`` **and** ``std::nth_element`` **perform at
roughly the same level**.

.. note::

   ``mMinSubset`` and ``std::nth_element`` have some difference. For top-N
   program, these 2 functions all output unsorted subset, but
   ``std::nth_element`` outputs the threshold in array position n,
   ``mMinSubset`` outputs the threshold as return.

And then, we compared ``mMinSubset`` in Morn and ``gsl_sort_smallest``
in `GSL <https://www.gnu.org/software/gsl/>`__. Testing code is:

.. code:: c

   void test3_2()
   {
       int n=1000000;int m;
       double *in  = (double *)mMalloc(n * sizeof(double));
       double *out1= (double *)mMalloc(n * sizeof(double));
       double *out2= (double *)mMalloc(n * sizeof(double));
       for (int i=0;i<n;i++) in[i] = ((double)mRand(-10000,10000))/10000.0;
       
       for(m=100000;m<n;m+=200000)
       {
           printf("\nselect %d from %d data\n",m,n);
           mTimerBegin("gsl" ); gsl_sort_smallest(out1,m,in,1,n); mTimerEnd("gsl" );
           mTimerBegin("Morn"); mMinSubset(in,n,out2,m);          mTimerEnd("Morn");
       }

       mFree(in); mFree(out1); mFree(out2);
   }

Here, we select 100000, 300000, 500000, 700000, 900000 data from 1000000.
Output is:

|image4|

It shows that: gap of time-consume between Morn and `GSL <https://www.gnu.org/software/gsl/>`__ is huge.

.. note::

   ``gsl_sort_smallest`` and ``mMinSubset`` are different: the output of
   ``gsl_sort_smallest`` is sorted, which is similarity as
   ``std::partial_sort``, and the output of ``mMinSubset`` is unsorted.

.. _header-n88:

Minimum/Maximum Subset with Index
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Here we compared ``mMaxSubset`` in Morn and ``gsl_sort_largest_index``
in `GSL <https://www.gnu.org/software/gsl/>`__. Testing code is:

.. code:: c

   void test4()
   {
       int n=1000000;int m;
       double *in  = (double *)mMalloc(n * sizeof(double));
       size_t *out1= (size_t *)mMalloc(n * sizeof(size_t));
       int    *out2= (int    *)mMalloc(n * sizeof(int   ));
       for (int i=0;i<n;i++) in[i] = ((double)mRand(-10000,10000))/10000.0;
       
       for(m=100000;m<n;m+=200000)
       {
           printf("\nselect %d from %d data with index\n",m,n);
           mTimerBegin("gsl" ); gsl_sort_largest_index(out1,m,in,1,n); mTimerEnd("gsl" );
           mTimerBegin("Morn"); mMaxSubset(in,n,NULL,out2,m);          mTimerEnd("Morn");
       }

       mFree(in); mFree(out1); mFree(out2);
   }

Here, we select 100000, 30000, 500000, 700000, 900000 largest data from
1000000. Testing code is:

|image5|

Obviously: Morn is much faster then `GSL <https://www.gnu.org/software/gsl/>`__.

.. note::

   ``gsl_sort_largest_index`` and ``mMaxSubset`` are also different:
   ``gsl_sort_largest_index`` output only index, and it is sorted,
   ``mMaxSubset`` outputs the index and data, but it is unsorted.

.. |image1| image:: https://z3.ax1x.com/2021/04/11/c0WVPA.png
   :target: https://imgtu.com/i/c0WVPA
.. |image2| image:: https://z3.ax1x.com/2021/04/11/c0fVwF.png
   :target: https://imgtu.com/i/c0fVwF
.. |image3| image:: https://z3.ax1x.com/2021/04/11/c0htBT.png
   :target: https://imgtu.com/i/c0htBT
.. |image4| image:: https://z3.ax1x.com/2021/04/12/c07YuR.png
   :target: https://imgtu.com/i/c07YuR
.. |image5| image:: https://z3.ax1x.com/2021/04/12/c07Gv9.png
   :target: https://imgtu.com/i/c07Gv9
