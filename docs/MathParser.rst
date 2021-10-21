.. _header-n0:

Math Parser
===========

`Morn <https://github.com/jingweizhanghuai/Morn>` Provides functions for math expression parsing, which is **small,
simple and extensible**. And you can use it building your own calculator.

.. _header-n5:

API
---

The source code of `Morn <https://github.com/jingweizhanghuai/Morn>`__ math parser is
`morn_calculate.c <https://github.com/jingweizhanghuai/Morn/blob/master/src/math/morn_calculate.c>`__, and APIs are defined in
`morn_math.h <https://github.com/jingweizhanghuai/Morn/blob/master/include/morn_math.h>`__.

.. _header-n6:

Calculate from Expression
~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: c

   double mCalculate(char *str);

The ``str`` is string of math expression. It returns the calculate result.

Math expression here can be arithmetic operation whit ``+``, ``-``,
``*`` ,\ ``/`` and other predefined functions.

For details, see the example below.

.. _header-n80:

Register for Custom Functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. code:: c

   void mCalculateFunction(void *func);
   void mCalculateFunction(char *name,void *func);

Except for the predefined functions, custom functions is also supported.
You must register your function by ``mCalculateFunction`` before use it
by ``mCalculate``.
You can give a new name for the function, if nessary.

The new custom function for register must like this:

.. code:: c

   double myfunc(double a,double b,...);

Thus, 1.Parameter type of the function must be ``double``, 2.The function must has at
least 1 parameter, and at most 8. 3.Return value is a must, and it must be ``double``.

.. _header-n108:

Example
-------

Complete example file is
`test_calculate.c <https://github.com/jingweizhanghuai/Morn/blob/master/test/test_calculate.c>`__

.. _header-n110:

Getting Start
~~~~~~~~~~~~~

.. code:: c

   int main()
   {
       printf("1+2*3/4            = %f\n",mCaculate("1+2*3/4"));
       printf("pi                 = %f\n",mCaculate("pi"));
       printf("sin(pi/2)          = %f\n",mCaculate("sin(pi/2)"));
       printf("ln(e)              = %f\n",mCaculate("ln(e)"));
       printf("sqrt(3^2+pow(4,2)) = %f\n",mCaculate("sqrt(3^2+pow(4,2))"));
       return 0;
   }

Output is:

.. code:: 

   1+2*3/4            = 2.500000
   pi                 = 3.141593
   sin(pi/2)          = 1.000000
   ln(e)              = 1.000000
   sqrt(3^2+pow(4,2)) = 5.000000

.. _header-n158:

Binding with Custom Functions
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this example, we defined our own function ``r2a`` and ``a2r``, which
can converse values between angle and radian.

.. code:: c

   double r2a(double r)
   {
       return r*180.0/MORN_PI;
   }

   double a2r(double a)
   {
       return a*MORN_PI/180.0;
   }

We must register the two functions by ``mCaculateFunction`` firstly.

.. code:: c

   int main()
   {
       mCaculateFunction(a2r);
       printf("sin(a2r(30)) = %f\n",mCaculate("sin(a2r(30))"));
       
       mCaculateFunction(r2a);
       printf("r2a(atan(1)) = %f\n",mCaculate("r2a(atan(1))"));
       
       return 0;
   }

Output is:

.. code:: 

   sin(a2r(30)) = 0.500000
   r2a(atan(1)) = 45.000000

Another example: we defined a function to find the mid-value from 3 data.

.. code:: c

   double get_mid_value(double a,double b,double c)
   {
       if((a>b)==(c>=a)) return a;
       if((a>b)==(b>=c)) return b;
       return c;
   }
   
   int main()
   {
       mCaculateFunction("mid",get_mid_value);
       char *str = "mid(5,1,2)";
       printf("%s = %f\n",str,mCaculate(str));

       return 0;
   }

Output is:

.. code:: 

   mid(5,1,2) = 2.000000

.. _header-n169:

Regulation
----------

.. _header-n171:

predefined functions
~~~~~~~~~~~~~~~~~~~~

In API ``mCalculate``, the below functions are supporter:

-  abs(x): calculate absolute value, ``abs(-2)``\ is 2.
-  min(x,y): select the minimum value of x and y, ``min(1,2)`` is 1.
-  max(x,y): select the maximum value of x and y ``max(1,2)`` is 2.
-  ceil(x): round up to an integer, ``ceil(1.6)`` is 2.
-  floor(x): round down to an integer, ``floor(1.6)`` is 1.
-  round(x): round to an integer nearest, ``round(1.6)`` is 2.
-  sqrt(x): square root, ``sqrt(9)`` is 3.
-  sqr(x): square, ``sqr(9)`` is 81.
-  exp(x): exponent with ``e``, same as ``e^x``.
-  pow(x): same as ``x^y``.
-  ln(x): natural logarithm, same as ``log(e,x)``, ``ln(e)`` is 1.
-  log10(x): logarithm with 10, same as ``log(10,x)``, ``log(100)`` is 2.
-  log(y,x): logarithm with y, ``log(3,9)`` is 2.
-  sin(x): sine, ``sin(pi/6)`` is 0.5.
-  cos(x): cosine, ``cos(0)`` is 1.
-  tan(x): tangent, ``tan(0)`` is 0.
-  cot(x): cotangent, ``cot(pi/4)`` is 1.
-  asin(x): anti-sine, ``asin(0.5)`` is 0.523598775598298.
-  acos(x): anti-cosine, ``acos(0)`` is 0.
-  atan(x): anti-tangent, ``atan(0)`` is0.
-  acot(x): anti-cotangent, ``acot(1)`` is 0.7853981633974483.

.. note:: 

   The input of sin(x), cos(x), tan(x) and cos(x) is radian (not angle).

   The return of asin(x), acos(x), atan(x), acot(x) is radian (not angle).

All these above function names are case-insensitive.

For These above functions, parentheses is a must. So ``ln5`` is invalid,
and ``ln(5)`` is OK.

.. _header-n245:

precedence
~~~~~~~~~~

**The parentheses have the highest precedence,** followed by the
exponential operations (^), then multiplication, division, and mod (*,
/, %), the lowest is addition and subtraction (+, -).

So ``-3^2``, for example, would result of -9 (instead of 9).

For continuous power expression, it will be calculated from right to
left, such as ``2^3^2``, is actually same as ``2^(3^2)``, resulting 512.

For other operations of same priority except for power, it will be
calculated from left to right.

.. tip:: 

   precedence is complex, parentheses is simple.

.. _header-n266:

Others
~~~~~~

-  ``%`` in expression means taking the remainder (instead of
   percent-sign). So ``5%+2`` means 5 mod +2, the result is 1(instead of
   2.05).

-   Spaces key play no role in expression, so you can write '10000' or
   '10 000', but not '10,000'.

-  ``pi`` (3.1415926) and ``e`` (2.718281828) are 2 constants, and they are
   case-insensitive.

-  The multiplication sign ``*`` cannot be omitted, ``2pi`` is invalid,
   and ``2*pi`` is OK.

.. _header-n60:

Tool
----

`Morn <https://github.com/jingweizhanghuai/Morn>` provides a command-line calculator, It is simple and easy:

.. code:: 

   >1+2*3/4
   result is 2.500000
   >
   >pi
   result is 3.141593
   >
   >sin(pi/2)
   result is 1.000000
   >
   >ln(e)
   result is 1.000000
   >
   >sqrt(3^2+pow(4,2))
   result is 5.000000
   >
   >5/0
   result is 1.#INF00
   >
   >exit

typing ``exit`` when exit from this calculator.
