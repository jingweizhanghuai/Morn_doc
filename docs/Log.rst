
Log
===

`Morn <https://github.com/jingweizhanghuai/Morn>`__ provides a C logging
library. It is **simple** (with only one interface) and
**high-performance** with almost no learning-costs.

-  Multiple outputs, including dynamic files, console, or user-defined
   outputs.

-  Support log level to selective log output. and customize log level
   also support.

-  Preset log format, and custom format is support.

-  Support automatically log-split with file size.

-  Thread safety.

-  Synchronous logging mode.


API
---

``mLog`` is the only logging API, witch is used to generate and output
the logging information.


Logging
~~~~~~~

.. code:: c

   void mLog(int Level,const char *format,...);

It is so simple, an example is:

.. code:: c

   int main()
   {
       mLog(MORN_INFO, "this is a Morn log, num=%d\n",1);
       return 0;
   }

the result is print the information below on console.

.. code:: 

   this is a Morn log, num=1


Logging Format
~~~~~~~~~~~~~~

**Preset Logging Format**

Morn use ``mLogFormat`` (which is syntactic sugar) to use preset
format.

.. code:: c

   int main()
   {
       mLog(MORN_INFO, mLogFormat1("this is a Morn log, format %d"),1);
       mLog(MORN_INFO, mLogFormat2("this is a Morn log, format %d"),2);
       mLog(MORN_INFO, mLogFormat3("this is a Morn log, format %d"),3);
       mLog(MORN_INFO, mLogFormat4("this is a Morn log, format %d"),4);
       mLog(MORN_INFO, mLogFormat5("this is a Morn log, format %d"),5);
       return 0;
   }

The output is:

.. code:: 

   [test_log.c,line 16,function main]Info: this is a Morn log, format 1
   [2020.09.13 18:03:43]Info: this is a Morn log, format 2
   [thread001]Info: this is a Morn log, format 3
   [2020.09.13 18:03:43 thread001]Info: this is a Morn log, format 4
   [2020.09.13 18:03:43 thread001 test_log.c,line 20,function main]Info: this is a Morn log, format 5

All built-in Morn log output (such as timing information and exception
information) is in format 1.

**User-defined Format**

According to Morn, any preset format may not meet all the needs of
users, Morn encourages users to define their own log format.

For example, when coder name is needed in log, you can do it as:

.. code:: c

   mLog(MORN_INFO, "[%s,line %d,function %s,author:%s]%s: this is a Morn log" ,__FILE__,__LINE__,__FUNCTION__,"JingWeiZhangHuai",mLogLevel(),1);

or

.. code:: c

   mLog(MORN_INFO, mLogFormat1("author:%s. this is a Morn log"),"JingWeiZhangHuai",1);

But both of these are cumbersome, Morn encourages users define a new
format:

.. code:: c

   //defined in .h files
   #define MY_FORMAT(Message) "[%s,line %d,function %s,author:%s]%s:"  Message  "\n", __FILE__,__LINE__,__FUNCTION__,"JingWeiZhangHuai",mLogLevel()
   
   //using in .c files
   mLog(MORN_INFO, MY_FORMAT("this is a Morn log"));

The following APIs may be used to define new logging format.

-  Logging Level

.. code:: c

   const char *mLogLevel();

It will return the logging level string which is input with ``mLog``. it
may be: "Debug", “Info", "Warning" or "Error".

-   Current Time

.. code:: c

   const char *mTimeString(int64_t time_value,const char *format);

See `Morn Data and Time <./Morn：时间和日期>`__ for
details. ``mTimeString(DFLT,NULL)`` may useful.

-  Thread ID

.. code:: c

   int mThreadID();

It returns the thread ID. note: these thread ID is not the system ID,
but an integer that starts with 1. that means for the first thread, it
returns 1, and for the second it returns 2.

.. _header-n59:

Logging Property
~~~~~~~~~~~~~~~~

``mPropertyWrite`` can be used to set logging property. The name of
logging module is "Log".

**Logging level**

.. code:: c

   mPropertyWrite("Log","log_level",(int *)p_log_level,sizeof(int));

The property name is "log_level", and it is an ``int``.

Logs are output only when the level input by ``mLog`` is greater than or
equal to this level_level. Otherwise, logs are ignored.

The preset level by Morn is
``MORN_DEBUG`` , ``MORN_INFO`` , ``MORN_WARNING`` and
``MORN_ERROR``, they are defined as:

.. code:: c

   #define MORN_DEBUG    0
   #define MORN_INFO    16
   #define MORN_WARNING 32
   #define MORN_ERROR   48

For example, To set the property "log_level" with ``MORN_WARNING``, 
the code can be:

.. code:: c

   int log_level = MORN_WARNING;
   mPropertyWrite("Log","log_level",&log_level,sizeof(int));

And if no property "log_level" is set, the default level is
``MORN_INFO`` (for release version) or ``MORN_DEBUG`` (for debug
version).

User-defined logging level is also support. For example you would define
a level with ``NOTICE``, which is Higher than ``MORN_INFO`` and lower
than ``MORN_WARNING`` , the code can be:

.. code:: c

   #define NOTICE (MORN_INFO+1)
   mLog(NOTICE, "this is a Morn log, num=%d\n",1);

A sample program is as follows:

.. code:: c

   int main()
   {
       int log_level = MORN_INFO;
       mPropertyWrite("Log","log_level",&log_level,sizeof(int));
       
       mLog(MORN_DEBUG  ,"this is a debug log\n");
       mLog(MORN_INFO   ,"this is a info log\n");
       mLog(MORN_WARNING,"this is a warning log\n");
       mLog(MORN_ERROR  ,"this is a error log\n\n");

       #define REMARK MORN_INFO+1
       log_level = REMARK;
       mPropertyWrite("Log","log_level",&log_level,sizeof(int));

       mLog(MORN_DEBUG  , "this is a debug log\n");
       mLog(MORN_INFO   , "this is a info log\n");
       mLog(REMARK      , "this is a remark log\n");
       mLog(MORN_WARNING, "this is a warning log\n");
       mLog(MORN_ERROR  , "this is a error log\n\n");
       
       return 0;
   }

The output will be:

.. code:: 

   this is a info log
   this is a warning log
   this is a error log

   this is a remark log
   this is a warning log
   this is a error log

**Logging File**

.. code:: c

   mPropertyWrite("Log","log_file",(const char *)filename); //output log to file
   mPropertyWrite("Log","log_file","exit"); 				 //ending output to file

The property name is ``"log_file"``, and it is a string.

Only when property "log_file" is write,it will output the log
information to the file.

And if you want ending log file output, write "exit" to this property.

**Logging File Size**

.. code:: c

   mPropertyWrite("Log","log_filesize",(int *)p_filesize,sizeof(int));

The property name is "log_filesize", and it is a ``int``, means the
bytes of output file.

this property works only when property "log_file" is write.

Only when this property be write, the log file will split into multiple
files. That is when file size greater than "log_filesize", a new log
file will be create, and the older one will be saved.

**Logging Console**

.. code:: c

   mPropertyWrite("Log","log_console",&p_log_console,sizeof(int));

The property name is ``"log_filesize"``, and it is a ``int``, Non-0
means enable console printing, and 0 means disable.

Console print is the default way of logging output.

The following program provides an example of a log's mixed file and
console output:

.. code:: c

   int main()
   {
       mLog(MORN_INFO, "this is log No.1\n");
       
       mPropertyWrite("Log","log_file","./test_log.log");
       mLog(MORN_INFO, "this is log No.2\n");
       
       mPropertyWrite("Log","log_file","exit");
       mLog(MORN_INFO, "this is log No.3\n");
       
       mPropertyWrite("Log","log_file","./test_log2.log");
       mLog(MORN_INFO, "this is log No.4\n");
       int log_console = 1;
       
       mPropertyWrite("Log","log_console",&log_console,sizeof(int));
       mLog(MORN_INFO, "this is log No.5\n");
       
       log_console = 0;
       mPropertyWrite("Log","log_console",&log_console,sizeof(int));
       mLog(MORN_INFO, "this is log No.6\n");
       
       mPropertyWrite("Log","log_file","exit");
       mLog(MORN_INFO, "this is log No.7\n");
       
       return 0;
   }

In this program:

The 1st log, no property was write, it will print on console.

The 2nd log, property "log_file" is write, it will output to file
"./test_log.log", and console disabled.

The 3rd log, "exit" is write to property "log_file", the file output is
terminate, and it will print on console.

the 4th log, a new file name is write to property "log_file", it will
output to file "./test_log2.log", and console disabled.

the 5th log, property "log_console" is enabled, it will output both to
file and to console.

the 6th log, since "log_console" is disabled, it will only output to
file.

the 7th log, "exit" is write to property "log_file", console output
will be the only way, whether "log_console" is enabled or not .

So on console:

.. code:: 

   this is log No.1
   this is log No.3
   this is log No.5
   this is log No.7

in file "./test_log.log":

.. code:: 

   this is log No.2

in file "./test_log2.log":

.. code:: c

   this is log No.4
   this is log No.5
   this is log No.6

**Logging User-defined output**

In addition to output log to console or file, user-defined way of
logging output is support. a user function can be write to property
"log_function"

.. code:: c

   mPropertyWrite("Log","log_function",&func,sizeof(void *));  //设置日志输出函数

and if function parameter is necessary, property "log_func_para"
would also be write.

.. code:: c

   mPropertyWrite("Log","log_func_para",&para,sizeof(void *));	//设置日志函数的参数

According to Morn, any preset logging output way may not meet all the
needs of users, a better way to do this is to allow users to customize
how logs are output.

The following example shows a user-defined way: transmit to another
cloud computer with UDP protocol, and the IP and port of the
cloud-computer are used as function parameters.

.. code:: c

   void send_log(char *log,int size,char *addr)
   {
       mUDPWrite(addr,log,size);
   }
   
   int main()
   {
       void *func=send_log;
       mPropertyWrite("Log","log_function",&func,sizeof(void *));
       
       char *para = "192.168.1.111:1234";
       mPropertyWrite("Log","log_func_para",&para,sizeof(char *));
       
       mLog(MORN_INFO, "this is a Morn log\n");
       return 0;
   }

.. _header-n116:

Performance
----

Here we test the performance of Morn logging, compared with other 3
famous logging library:
`glog <https://github.com/google/glog#getting-started>`__,
`spdlog <https://github.com/gabime/spdlog>`__ and
`log4cpp < https://github.com/orocos-toolchain/log4cpp>`__

The test code is:

.. code:: cpp

   #include "glog/logging.h"
   
   #include "spdlog/spdlog.h"
   
   #include "log4cpp/Category.hh"
   #include "log4cpp/FileAppender.hh"
   #include "log4cpp/Priority.hh"
   #include "log4cpp/PatternLayout.hh"
   
   #include "morn_ptc.h"
   
   struct LogData
   {
       int *datai;
       double *datad;
       char *datas;
       int N;
   };
   
   void test_glog(struct LogData *p)
   {
       google::InitGoogleLogging("test_log");
       google::SetLogDestination(google::GLOG_INFO, "./test_log_glog.log");
       for(int n=0;n<p->N;n++)
       {
           int i=n%100;
           LOG(INFO)<<": Hello glog, "<<"datai="<<p->datai[i]<<", datad="<<p->datad[i]<<", datas="<< p->datas+i*32;
       }
       google::ShutdownGoogleLogging();
   }
   
   void test_spdlog(struct LogData *p)
   {
       auto console2 = spdlog::basic_logger_mt("test_log","./test_log_spdlog.log");
       spdlog::set_pattern("[%Y.%m.%d %H:%M:%S thread%t]%l: %v");
       for(int n=0;n<p->N;n++)
       {
           int i=n%100;
           console2->info("[{} line {},function {}] Hello spdlog, datai={}, datad={}, datas={}",__FILE__,__LINE__,__FUNCTION__,p->datai[i],p->datad[i],p->datas+i*32);
       }
   }
   
   void test_log4cpp(struct LogData *p)
   {
       log4cpp::FileAppender * appender = new log4cpp::FileAppender("appender","./test_log_log4cpp.log");
       log4cpp::PatternLayout* pLayout = new log4cpp::PatternLayout();
       pLayout->setConversionPattern("[%d thread%t %c]%p: %m%n");
       appender->setLayout(pLayout);
       log4cpp::Category& root =log4cpp::Category::getRoot();
       log4cpp::Category& infoCategory =root.getInstance("test_log");
       infoCategory.addAppender(appender);
       infoCategory.setPriority(log4cpp::Priority::INFO);
       for(int n=0;n<p->N;n++)
       {
           int i=n%100;
           infoCategory.info("[%s,line %d,function %s] Hello log4cpp, datai=%d, datad=%f, datas=%s",__FILE__,__LINE__,__FUNCTION__,p->datai[i],p->datad[i],p->datas+i*32);
       }
       log4cpp::Category::shutdown();
   }
   
   void test_morn(struct LogData *p)
   {
       mPropertyWrite("Log","log_file","./test_log_morn.log");
       for(int n=0;n<p->N;n++)
       {
           int i=n%100;
           mLog(MORN_INFO,mLogFormat5("Hello Morn, datai=%d, datad=%f, datas=%s"),p->datai[i],p->datad[i],p->datas+i*32);
       }
   }
   
   int main(int argc, char** argv)
   {
       char datas[100][32];
       int datai[100];
       double datad[100];
       for(int i=0;i<100;i++)
       {
           datai[i]=mRand(DFLT,DFLT);
           datad[i]=(double)mRand(-1000000,1000000)/1000000.0;
           mRandString(&(datas[i][0]),15,31);
       }
       struct LogData data={.datai=datai,.datad=datad,.datas=(char *)datas,.N=1000000};
       
       mTimerBegin("glog");
       test_glog(&data);
       mTimerEnd("glog");
       
       mTimerBegin("spdlog");
       test_spdlog(&data);
       mTimerEnd("spdlog");
       
       mTimerBegin("log4cpp");
       test_log4cpp(&data);
       mTimerEnd("log4cpp");
       
       mTimerBegin("Morn");
       test_morn(&data);
       mTimerEnd("Morn");
       
       return 0;
   }

Here we test the time cost for 1000000 logs output to file. The output
of the four logs is as follows:

-  glog

.. code:: c

   I20200913 21:14:36.839823   148 test_log2.cpp:48] : Hello glog, datai=17729, datad=-0.761655, datas=cuppeubmapohxinsmwoumohsrmfdi

-  spdlog

.. code:: c

   [2020.09.13 21:14:44 thread148]info: [test_log2.cpp line 59,function main] Hello spdlog, datai=17729, datad=-0.761655, datas=cuppeubmapohxinsmwoumohsrmfdi

-  log4cpp

.. code:: 

   [2020-09-13 21:14:45,783 thread148 .\test_log2.exe]INFO: [test_log2.cpp,line 75,function main] Hello log4cpp, datai=17729, datad=-0.761655, datas=cuppeubmapohxinsmwoumohsrmfdi

-  Morn

.. code:: 

   [2020.09.13 21:15:15 thread001 test_log2.cpp,line 85,function main]Info: Hello Morn, datai=17729, datad=-0.761655, datas=cuppeubmapohxinsmwoumohsrmfdi

The result is:

|image1|

As shown above, Morn is the fastest, followed by spdlog, and log4cpp is
the slowest.

.. |image1| image:: https://s1.ax1x.com/2022/09/24/xAniX4.png
   :target: https://imgse.com/i/xAniX4
