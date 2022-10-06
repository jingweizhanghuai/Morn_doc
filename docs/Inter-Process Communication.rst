.. _header-n0:

Inter-Process Communication
===========================

`Morn <https://github.com/jingweizhanghuai/Morn>`__ provides a C
IPC(Inter-Process Communication) library, which is simple and
low-latency. The memory mapping is used by Morn IPC. Both Windows and
Linux are OK. 3 communicate modes are support.

-  Topic IPC Mode

-  Message-queue IPC Mode

-  Inter-Process Variate

.. _header-n4:

API
---

.. _header-n5:

Topic IPC Mode
~~~~~~~~~~~~~~

.. code:: c

   void *mProcTopicWrite(const char *topic_name,char *string);
   void *mProcTopicWrite(const char *topic_name,void *data,int size);
   
   void *mProcTopicRead(const char *topic_name);
   void *mProcTopicRead(const char *topic_name,void *data);
   void *mProcTopicRead(const char *topic_name,void *data,int *p_size);

Topic mode support one-writer one-reader, one-writer many-reader,
many-writer one-reader and many writer many-reader.

``mProcTopicWrite`` is used for publisher process, and
``mProcTopicRead`` is for subscriber.

For ``mProcTopicWrite``, the ``data`` or ``string`` would be copied to
the shared memory, and for ``mProcTopicRead`` the shared memory would be
copied to the user's ``data`` (if ``data`` is not NULL).

In project ``topic_name`` must be unique. Both sides of communication
must use the same ``topic_name``.

Note that: the topic IPC mode is non-blocking. For subscriber,
``mProcTopicRead`` would always read the latest topic data, and no
guarantee that all information will be read.

The return of both ``mProcTopicWrite`` and ``mProcTopicRead`` is the
data pointer of shared memory. For ``mProcTopicRead``, if no new topic
data has been write, it would return NULL.

.. _header-n10:

Message-queue IPC Mode
~~~~~~~~~~~~~~~~~~~~~~

.. code:: c

   void *mProcMessageWrite(const char *message_name,char *string);
   void *mProcMessageWrite(const char *message_name,void *data,int size);
   
   void *m_ProcMessageRead(const char *message_name);
   void *m_ProcMessageRead(const char *message_name,void *data,int *p_size);
   void *m_ProcMessageRead(const char *message_name,void *data,int *p_size);

Message-queue mode support one-writer one-reader, one-writer
many-reader, many-writer one-reader and many writer many-reader.

``mProcMessageWrite`` is used for publisher process, and
``m_ProcMessageRead`` is for subscriber.

For ``mProcMessageWrite``, the ``data`` or ``string`` would be copied to
the shared memory, and for ``m_ProcMessageRead`` the shared memory would
be copied to the user's ``data`` (if ``data`` is not NULL).

In project ``message_name`` must be unique. Both sides of communication
must use the same ``message_name``.

The return of both ``mProcMessageWrite`` and ``m_ProcMessageRead`` is
the data pointer of shared memory.

Note that: Different from the topic mode, the message-queue IPC mode is
blocking and guarantee every data would be read. That is if no new data
is write, ``mProcTopicRead`` would be blocked, and if too many data had
be write but not be read and the memory ring-buffer is full,
``mProcMessageWrite`` would be blocked.

.. _header-n135:

Inter-Process Variate
~~~~~~~~~~~~~~~~~~~~~

.. code:: c

   void *mProcMasterVariate(const char *variate_name,int size);
   void *mProcSlaveVariate(const char *variate_name,int size);

Different from topic mode and message-queue mode, to avoid conflicts,
the inter-process variate only support one-writer one-reader and
one-writer many-reader. That is in project, there can be only one
master-variate with a ``variate_name``, and may be many slave-variate in
different process with same ``variate_name``. multi-master would call an
error.

In project ``variate_name`` must be unique. Both sides of communication
must use the same ``variate_name``.

The ``size`` is the variate bytes. A maximum of 128 bytes is supported.

The return is the pointer of inter-process variate. It can be used(write
or read) as a normal pointer.

For ``mProcMasterVariate``, the return pointer can be read and write,
and for ``mProcSlaveVariate`` the return pointer can only be read (can
not be write).

.. _header-n168:

example
-------

.. _header-n170:

Topic IPC Mode
~~~~~~~~~~~~~~

Here is a simple example to use Morn topic IPC mode.

.. code:: c

   #include "morn_ptc.h"
   
   void publisher()
   {
       char data[64];
       for(int i=0;i<100;i++)
       {
           mSleep(1000);
           mRandString(data,16,64);
           printf("string= %s\n",data);
           mProcTopicWrite("string",data);
       }
   }
   
   void subscriber()
   {
       while(1)
       {
           mSleep(100);
           char *p=mProcTopicRead("string");
           if(p!=NULL) 
           {
               if(p[0]==0) return;
               printf("string= %s\n",p);
           }
       }
   }
   
   int main(int argc,char *argv[])
   {
            if(strcmp(argv[1],"publisher" )==0)  publisher();
       else if(strcmp(argv[1],"subscriber")==0) subscriber();
       else if(strcmp(argv[1],"exit"      )==0) {char data=0;mProcTopicWrite("string",&data,1);}
       return 0;
   }

Here, the topic name is ``string``, you can use

.. code:: 

   test_process_topic.exe publisher

to start a publisher process, and use

.. code:: 

   test_process_topic.exe subscriber

to start a subscriber process.

the publisher transport a random string to the subscriber every seconds,
And the subscriber print the string on consoler when a new one is
received.

multi publisher and multi subscriber is support.

.. _header-n188:

Inter-Process Variate
~~~~~~~~~~~~~~~~~~~~~

Here is a simple example to use inter-process variate.

.. code:: c

   #include "morn_ptc.h"
   
   void slave1()
   {
       int *p = mProcSlaveVariate("test1",sizeof(int));
       while(1)
       {
           mSleep(100);
           if(*p<0) break;
           printf("data1=%d\n",*p);
       }
   }
   
   void slave2()
   {
       int *p = mProcSlaveVariate("test2",sizeof(int));
       while(1)
       {
           mSleep(100);
           if(*p<0) break;
           printf("data2=%d\n",*p);
       }
   }
   
   void master()
   {
       int *p1 = mProcMasterVariate("test1",sizeof(int));
       int *p2 = mProcMasterVariate("test2",sizeof(int));
       for(int i=0;i<100;i++)
       {
           *p1=i+i;
           *p2=i+i+1;
           printf("data1=%d,data2=%d\n",*p1,*p2);
           mSleep(1000);
       }
       *p1=-1;
       *p2=-1;
   }
   
   int main(int argc,char *argv[])
   {
       if(strcmp(argv[1],"slave1")==0) slave1();
       if(strcmp(argv[1],"slave2")==0) slave2();
       else                            master();
       return 0;
   }

Here, we used 2 inter-process variate, one named ``test1`` and another
named ``test2``. The master process write the variate every seconds, and
the slave process read and print it every 100 milliseconds .

You can use

.. code:: 

   test_process_variate.exe master

to start the master process, and use

.. code:: 

   test_process_variate.exe slave1
   test_process_variate.exe slave2

to start the slave process.

.. _header-n33:

Performance
-----------

Here we compared Morn IPC with famous IPC library
`iceoryx <https://github.com/eclipse-iceoryx/iceoryx>`__, which uses a
zero-copy, shared memory approach.

The measuring method comes from the iceoryx's
`iceperf <https://github.com/eclipse-iceoryx/iceoryx/tree/master/iceoryx_examples/iceperf>`__,
that is ping pong measurements. For one ping-pang transport, the leader
process send data to the follower, and wait to receive data from the
follower, the follower process forward the data from leader when
received it. We measured the time-use of 1000000 ping-pang transport.

The test code is:

.. code:: c

   #include "morn_ptc.h"
   #include "iceoryx_binding_c/runtime.h"
   #include "iceoryx_binding_c/publisher.h"
   #include "iceoryx_binding_c/subscriber.h"
   
   #define T 1000000
   
   ////////////////////////////////////iceoryx//////////////////////////////////////
   iox_pub_storage_t m_publisherStorage;
   iox_sub_storage_t m_subscriberStorage;
   iox_pub_t m_publisher;
   iox_sub_t m_subscriber;
   
   void construct(const char * publisherName, const char * subscriberName)
   {
       iox_pub_options_t publisherOptions;
       iox_pub_options_init(&publisherOptions);
       publisherOptions.historyCapacity = 0U;
       publisherOptions.nodeName = "SlapStick";
       m_publisher = iox_pub_init(&m_publisherStorage, "Comedians", publisherName, "Duo", &publisherOptions);
   
       iox_sub_options_t subscriberOptions;
       iox_sub_options_init(&subscriberOptions);
       subscriberOptions.queueCapacity = 10U;
       subscriberOptions.historyRequest = 0U;
       subscriberOptions.nodeName = "Slapstick";
       m_subscriber = iox_sub_init(&m_subscriberStorage, "Comedians", subscriberName, "Duo", &subscriberOptions);
   }
   
   void init()
   {
       iox_pub_offer(m_publisher);
       iox_sub_subscribe(m_subscriber);
   
       while (iox_sub_get_subscription_state(m_subscriber) != SubscribeState_SUBSCRIBED) mSleep(1);
       while (!iox_pub_has_subscribers(m_publisher)) mSleep(1);
   }
   
   void destruct()
   {
       iox_pub_deinit(m_publisher);
       iox_sub_deinit(m_subscriber);
   }
   
   void sendPerfTopic(int count, int size)
   {
       void* userPayload = NULL;
       if (iox_pub_loan_chunk(m_publisher, &userPayload, size) == AllocationResult_SUCCESS)
       {
           int *data = (int *)userPayload;
           data[0]=count;data[1]=size;
           iox_pub_publish_chunk(m_publisher, userPayload);
       }
   }
   
   void receivePerfTopic(int *count,int *size)
   {
       int hasReceivedSample=0;
       while(!hasReceivedSample)
       {
           const void* userPayload = NULL;
           if (iox_sub_take_chunk(m_subscriber, &userPayload) == ChunkReceiveResult_SUCCESS)
           {
               int *data = (int *)userPayload;
               *count = data[0];*size=data[1];
               hasReceivedSample = 1;
               iox_sub_release_chunk(m_subscriber, userPayload);
           }
       }
   }
   
   void pingPongLeader(int size)
   {
       mTimerBegin("iceoryx");
       int count=0;
       for (int i = 0; i < T; ++i)
       {
           sendPerfTopic(count,size);
           receivePerfTopic(&count,&size);
           count++;
       }
       mTimerEnd("iceoryx");
       printf("count=%d,size=%d\n\n",count,size);
   }
   
   void pingPongFollower()
   {
       int count,size;
       while(1)
       {
           receivePerfTopic(&count,&size);
           if(count<0) break;
           count++;
           sendPerfTopic(count,size);
       }
   }
   
   void iceoryx_leader()
   {
       iox_runtime_init("iox-leader-app");
       construct("to_follower","to_leader");
       init();
       for(int size=64;size<=256*1024;size*=4)
           pingPongLeader(size);
       sendPerfTopic(-1,2*sizeof(int));
       destruct();
   }
   
   void iceoryx_follower()
   {
       iox_runtime_init("iox-follower-app");
       construct("to_leader","to_follower");
       init();
       pingPongFollower();
       destruct();
   }
   ///////////////////////////////////////////////////////////////////////////////
   
   //////////////////////////////////Morn/////////////////////////////////////////
   void morn_leader()
   {
       int size=256*1024;
       int *data=mMalloc(size);
       int *p=NULL;
       mPropertyWrite("to_follower","topic_size",&size,sizeof(int));
       for(size=64;size<=256*1024;size*=4)
       {
           data[0]=0;
           mTimerBegin("morn");
           for(int i=0;i<T;i++)
           {
               mProcTopicWrite("to_follower",data,size);
               do{p=mProcTopicRead("to_leader");}while(p==NULL);
               data[0]=p[0]+1;
           }
           mTimerEnd("morn");
           printf("count=%d,size=%d\n\n",data[0],size);
       }
       data[0]=-1;
       mProcTopicWrite("to_follower",data,sizeof(int));
       mFree(data);
   }
   
   void morn_follower()
   {
       int *p=NULL;int size=256*1024;
       mPropertyWrite("to_leader","topic_size",&size,sizeof(int));
       while(1)
       {
           do{p=mProcTopicRead("to_follower",NULL,&size);}while(p==NULL);
           if(p[0]<0) break;
           p[0]++;
           mProcTopicWrite("to_leader",p,size);
       }
   }
   ///////////////////////////////////////////////////////////////////////////////
   
   ////////////////////////////////////memcpy/////////////////////////////////////
   void test_memcpy()
   {
       int size = 1024*1024;
       int *data1=mMalloc(size);
       int *data2=mMalloc(size);
       for(size=64;size<=256*1024;size*=4)
       {
           data1[0]=0;
           mTimerBegin("memcpy");
           for(int i=0;i<T;i++)
           {
               memcpy(data2,data1,size);
               data2[0]++;
               memcpy(data1,data2,size);
               data1[0]++;
           }
           mTimerEnd("memcpy");
           printf("count=%d,size=%d\n\n",data1[0],size);
       }
       mFree(data1);
       mFree(data2);
   }
   //////////////////////////////////////////////////////////////////////////////////
   
   int main(int argc,char *argv[])
   {
            if(strcmp(argv[1],"iceoryx_leader"  )==0) iceoryx_leader();
       else if(strcmp(argv[1],"iceoryx_follower")==0) iceoryx_follower();
       else if(strcmp(argv[1],   "morn_leader"  )==0) morn_leader();
       else if(strcmp(argv[1],   "morn_follower")==0) morn_follower();
       else if(strcmp(argv[1],     "memcpy"     )==0) test_memcpy();
       return 0;
   }

Here, we measured the time-use for transport with 64 bytes, 256 bytes,
1k bytes, 4k bytes, 16k bytes, 64k bytes and 256k bytes. The result is:

For iceoryx:

|image1|

For Morn IPC:

|image2|

And as a comparison, we measured the time-use with memory copy use
``memcpy`` in a process, the result is:

|image3|

As shown above: Morn is much faster (lower latency) than iceoryx when
transport with not a lot of data(<100k). And thanks to the zero-copy
feature, iceoryx would faster than Morn when transport with lots of
data. The time-use is almost the same with different data size using
iceoryx, and is increased using Morn IPC.

Compared with inner-process memory copy, it shows the major time sink is
the data copy. That is because Morn did not use zero-copy approach like
iceoryx. According to Morn, data-copy usually is necessary, the
difference is simply that who copy the data, the users or the library.
With data-copy by library, the transport can be safer, and the interface
can be simpler.

.. |image1| image:: https://s1.ax1x.com/2022/09/16/vzaNQS.png
   :target: https://imgse.com/i/vzaNQS
.. |image2| image:: https://s1.ax1x.com/2022/09/16/vzaJRf.png
   :target: https://imgse.com/i/vzaJRf
.. |image3| image:: https://s1.ax1x.com/2022/09/16/vzaYz8.png
   :target: https://imgse.com/i/vzaYz8
