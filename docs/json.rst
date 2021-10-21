.. _header-n0:

JSON
====

`Morn <https://github.com/jingweizhanghuai/Morn>`__ provides a library for parsing JSON files. It is
**simple** (with only two interfaces) and **fast** (much faster than
`rapidjson <https://github.com/Tencent/rapidjson>`__) .

This is a typical JSON string, we will take it as an example:

.. code:: json

   {
       "hello": "world",
       "t": true ,
       "f": false,
       "n": null,
       "i": 123,
       "pi": 3.1415926,
       "a1": [0,1,2,3],
       "a2": [[00,01,02,03],[10,11,12,13],[20,21,22,23]],
       "a3": [
           [[000,001,002],[010,011,012]],
           [[100,101,102],[110,111,112]],
           [[200,201,202],[210,211,212]]
       ],
       "a4": [{"value":0},{"value":1}],
       "date":
       {
           "year" :2021,
           "month":"June",
           "day":5
       },
       "city":[
           {"Beijing":["Dongcheng","Xicheng","Haidian","Chaoyang"]},
           "Shanghai",
           "Tianjin"
       ],
       "province":
       {
           "Hebei":["Shijiazhuang","Tangshan","Hengshui"],
           "Anhui":["Hefei","Huangshan"],
           "Gansu":"Lanzhou"
       }
   }

.. _header-n5:

API
---------

The source code of `Morn <https://github.com/jingweizhanghuai/Morn>`__ JSON is
`morn_json.c <https://github.com/jingweizhanghuai/Morn/blob/master/test/test_JSON_file.c>`__, and APIs are defined in
`morn_util.h <https://github.com/jingweizhanghuai/Morn/blob/master/include/morn_util.h>`__.

.. _header-n6:

JSON Node
~~~~~~~~~

JSON is organized with JSON nodes. In `Morn <https://github.com/jingweizhanghuai/Morn>`__ JSON node is the basic structure. 
These nodes have different types as key-value or single-value. We define these types as:

.. code:: c

   #define JSON_UNKNOWN     0
   #define JSON_KEY_UNKNOWN 1
   #define JSON_BOOL        2
   #define JSON_KEY_BOOL    3	//such as "t": true ,"f": false,
   #define JSON_INT         4	//such as 0,1,2,3
   #define JSON_KEY_INT     5	//such as "i": 123,
   #define JSON_DOUBLE      6
   #define JSON_KEY_DOUBLE  7	//such as "pi": 3.1415926,
   #define JSON_STRING      8	//such as "Dongcheng","Xicheng","Haidian","Chaoyang"
   #define JSON_KEY_STRING  9	//such as "hello": "world",
   #define JSON_LIST       10	//such as {"value":0}
   #define JSON_KEY_LIST   11	//such as "date":{"year":2021,"month":"June","day":5},
   #define JSON_ARRAY      12	//such as [10,11,12,13]
   #define JSON_KEY_ARRAY  13	//such as "a1": [0,1,2,3],

JSON Node is defined as:

.. code:: c

   struct JSONNode
   {
       union
       {
           bool     dataBool;   //valid when type is JSON_KEY_BOOL or JSON_BOOL
           int32_t  dataS32;    //valid when type is JSON_KEY_INT or JSON_INT
           double   dataD64;    //valid when type is JSON_KEY_DOUBLE or JSON_DOUBLE
           char    *string;     //valid when type is JSON_KEY_STRING or JSON_STRING
           uint16_t num;        //child node number, valid when type isJSON_KEY_ARRAY、JSON_ARRAY、JSON_KEY_LIST or JSON_LIST
       };
       char *key;
       int8_t type;
   };

.. _header-n11:

Load and Parse JSON
~~~~~~~~~~~~~~~~~~~

.. code:: c

   struct JSONNode *mJSONLoad(MFile *jsonfile);
   struct JSONNode *mJSONLoad(MString *jsondata);

The input can be a JSON file or a JSON string, the output is the parsed
root node.

It can be used as:

.. code:: c

   MFile *file = mFileCreate("./test_json.json");
   struct JSONNode *json=mJSONLoad(file);
   ...
   mFileRelease(file);

or as:

.. code:: c

   MString *string = mStringCreate("{\"hello\":\"world\",\"t\":true,\"i\":123}");
   struct JSONNode *json=mJSONLoad(string);
   ...
   mStringRelease(string);

For file parsing, you can use ``mJSONLoad`` directly, or can read file and parse it as JSON string.

.. _header-n20:

Read JSON Node
~~~~~~~~~~~~~~

.. code:: c

   struct JSONNode *mJSONRead(struct JSONNode *node);
   struct JSONNode *mJSONRead(struct JSONNode *node,int n);
   struct JSONNode *mJSONRead(struct JSONNode *node,const char *key);
   struct JSONNode *mJSONRead(struct JSONNode *node,struct JSONNode *dst);
   struct JSONNode *mJSONRead(struct JSONNode *node,int n,struct JSONNode *dst);
   struct JSONNode *mJSONRead(struct JSONNode *node,const char *key,struct JSONNode *dst);

The input node must with type of list(``JSON_LIST`` or 
``JSON_KEY_LIST``) or array(``JSON_ARRAY`` or ``JSON_KEY_ARRAY``), It
returns NULL when read failure.

For example:

.. code:: c

   struct JSONNode *child;
   child = mJSONRead(mother);          //mother is list or array, read the first node of mother
   child = mJSONRead(mother,5);        //mother is list or array, read the fifth node of mother
   child = mJSONRead(mother,"[5]");    //mother is array, read the fifth node of mother
   child = mJSONRead(mother,"child5"); //mother is list, read the node with key is "child5"
   child = mJSONRead(mother,"a.b[3].c.d[6]");   //read further child node

Or it can be writen as:

.. code:: c

   struct JSONNode child;
   mJSONRead(mother,&child);           //mother is list or array, read the first node of mother
   mJSONRead(mother,5,&child);         //mother is list or array, read the fifth node of mother
   mJSONRead(mother,"[5]",&child);     //mother is array, read the fifth node of mother
   mJSONRead(mother,"child5",&child);  //mother is list, read the node with key is "child5"
   mJSONRead(mother,"a.b[3].c.d[6]",&child);    //read further child node

.. _header-n28:

Example
-------

Complete example file is
`test_JSON_file.c <https://github.com/jingweizhanghuai/Morn/blob/master/test/test_JSON_file.c>`__

Getting Start
~~~~~~~~~~~~~

Taking the beginning JSON file as an example, it can be read as:

.. code:: c

   char *jsontype[15]={"UNKNOWN","KEY_UNKNOWN","BOOL","KEY_BOOL","INT","KEY_INT","DOUBLE","KEY_DOUBLE","STRING","KEY_STRING","LIST","KEY_LIST","ARRAY","KEY_ARRAY","UNKNOWN"};

   int main()
   {
       MFile *file = mFileCreate("./test_json.json");

       struct JSONNode *json=mJSONLoad(file);
       printf("json->type=%s\n",jsontype[json->type]);
       printf("json->num=%d\n",json->num);

       struct JSONNode *node;
       node=mJSONRead(json,"hello");
       printf("node->type=%s\n",jsontype[node->type]);
       printf("node->key=%s\n",node->key);
       printf("node->string=%s\n",node->string);
       
       mFileRelease(file);
   }

In this example, two nodes have been read: root-node and hello-node. Output is:

.. code:: 

   json->type=LIST
   json->num=13
   node->type=KEY_STRING
   node->key=hello
   node->string=world

Code can also be written as following forms:

.. code:: c

   node=mJSONRead(json,"t");
   if(node!=NULL)
   {
       if(node->type==JSON_KEY_BOOL)
           printf("t=%d\n",node->dataBool);
   }

   struct JSONNode f_node;
   node=mJSONRead(json,"f",&f_node);
   printf("f=%d\n",f_node.dataBool);

   int i=*(int *)mJSONRead(json,"i");
   printf("i=%d\n",i);

   double *pi=(double *)mJSONRead(json,"pi");
   printf("pi=%lf\n",*pi);

Output is:

.. code:: 

   t=1
   f=0
   i=123
   pi=3.141592

Note here that: ``nul`` will be understood as a null string:

.. code:: c

   node = mJSONRead(json,"n");
   printf("type=%s,nul=%p\n",jsontype[node->type],node->string);

Output is:

.. code:: 

   type=KEY_STRING,nul=0000000000000000

Read from List
~~~~~~~~~~~~~~

For further child node, it can be read layer by layer, for example:

.. code:: c

   node=mJSONRead(json,"date");
   struct JSONNode *year=mJSONRead(node,"year");
   printf("date.year=%d,type=%s\n",year->dataS32,mJSONNodeType(year));
   struct JSONNode *month=mJSONRead(node,"month");
   printf("date.month=%s,type=%s\n",month->dataS32,mJSONNodeType(month));
   struct JSONNode *day=mJSONRead(node,"day");
   printf("date.day=%d,type=%s\n",day->dataS32,mJSONNodeType(day));

Or it can be read cross layers:

.. code:: c

   struct JSONNode *year=mJSONRead(json,"date.year");
   printf("date.year=%d,type=%s\n",year->dataS32,mJSONNodeType(year));
   struct JSONNode *month=mJSONRead(json,"date.month");
   printf("date.month=%s,type=%s\n",month->dataS32,mJSONNodeType(month));
   struct JSONNode *day=mJSONRead(json,"date.day");
   printf("date.day=%d,type=%s\n",day->dataS32,mJSONNodeType(day));

Outputs of these above two programs are the same:

.. code:: 

   date.year=2021,type=KEY_INT
   date.month=June,type=KEY_STRING
   date.day=5,type=KEY_INT

.. tip:: 
   If you want to traverse all the child nodes, Read layer-by-layer is faster than reading cross layers.

Read from Array
~~~~~~~~~~~~~~~

Several flexible forms for reading from arrays are provided as:

.. code:: c

   struct JSONNode *p;
   node=mJSONRead(json,"a1");
   p = mJSONRead(node);
   printf("a1[0]=%d\n",p->dataS32);
   p = mJSONRead(node,1);
   printf("a1[1]=%d\n",p->dataS32);
   p = mJSONRead(node,"[2]");
   printf("a1[2]=%d\n",p->dataS32);
   p = mJSONRead(json,"a1[3]");
   printf("a1[3]=%d\n",p->dataS32);

Output is:

.. code:: 

   a1[0]=0
   a1[1]=1
   a1[2]=2
   a1[3]=3

Multidimensional Array can also be read as further child with cross layers read:

.. code:: c

   node = mJSONRead(json,"a2[1][2]");

And also can be read layer by layer:

.. code:: c

   struct JSONNode *a2=mJSONRead(json,"a2");
   for(int j=0;j<a2->num;j++)
   {
       struct JSONNode *p1=mJSONRead(a2,j);
       for(int i=0;i<p1->num;i++)
       {
           struct JSONNode *p2=mJSONRead(p1,i);
           printf("%02d,",p2->dataS32);
       }
       printf("\n");
   }

Output is:

.. code:: 

   00,01,02,03,
   10,11,12,13,
   20,21,22,23,

Mixed Read
~~~~~~~~~~

Node can also be read from mixed list and array, as:

.. code:: c

   node = mJSONRead(json,"province.Hebei[0]");
   printf("%s\n",node->string);
   node = mJSONRead(json,"province.Anhui[0]");
   printf("%s\n",node->string);
   node = mJSONRead(json,"province.Gansu"   );
   printf("%s\n",node->string);

Output is:

.. code:: 

   Shijiazhuang
   Hefei
   Lanzhou

.. _header-n69:

Performance
-----------
Here, `Morn <https://github.com/jingweizhanghuai/Morn>`__ is compared with:
`cjson <https://github.com/DaveGamble/cJSON>`__, `jsoncpp <https://github.com/open-source-parsers/jsoncpp>`__, 
`nlohmann <https://github.com/nlohmann/json>`__, `rapidjson <https://github.com/Tencent/rapidjson>`__, and `yyjson <https://github.com/ibireme/yyjson>`__

Complete test file is
`test_JSON_file2.cpp <https://github.com/jingweizhanghuai/Morn/blob/master/test/test_JSON_file2.cpp>`__

Following command is used to compile this program:

.. code:: shell

   g++ -O2 test_JSON_file2.cpp -o test_JSON_file2.exe -lcjson -ljsoncpp -lyyjson -lmorn

.. _header-n72:

Test 1
~~~~~~

Parsing
`citm_catalog.json <https://github.com/miloyip/nativejson-benchmark/blob/master/data/citm_catalog.json>`__,
and reading "areaId" in the file, then measure time-consume of parsing and reading. This
is a fragment of the program using `Morn <https://github.com/jingweizhanghuai/Morn>`__:

.. code:: c

   int Morn_test1()
   {
       MObject *jsondata=mObjectCreate();
       mFile(jsondata,"./citm_catalog.json");
       
       mTimerBegin("Morn Json");
       struct JSONNode *json = mJSONLoad(jsondata);
       int n=0;
       struct JSONNode *performances_array = mJSONRead(json,"performances");
       for(int i=0;i<performances_array->num;i++)
       {
           struct JSONNode *performances = mJSONRead(performances_array,i);
           struct JSONNode *seatCategories_array = mJSONRead(performances,"seatCategories");
           for(int j=0;j<seatCategories_array->num;j++)
           {
               struct JSONNode *seatCategories = mJSONRead(seatCategories_array,j);
               struct JSONNode *areas_array = mJSONRead(seatCategories,"areas");
               for(int k=0;k<areas_array->num;k++)
               {
                   struct JSONNode *areas = mJSONRead(areas_array,k);
                   struct JSONNode *areaId=mJSONRead(areas,"areaId");
                   int id=areaId->dataS32;
                   n++;
                   // printf("id=%d\n",id);
               }
           }
       }
       mTimerEnd("Morn Json");

       mObjectRelease(jsondata);
       return n;
   }

   int test1()
   {
       int n=Morn_test1();
       printf("get %d areaId\n\n",n);
   }

Result is:

|image1|

.. _header-n78:

Test 2
~~~~~~

parsing
`canada.json <https://github.com/miloyip/nativejson-benchmark/blob/master/data/canada.json>`__
and reading all of coordinates, then measure time-consume of parsing and
reading. This is a fragment of the program using `Morn <https://github.com/jingweizhanghuai/Morn>`__:

.. code:: c

   int Morn_test2()
   {
       MObject *jsondata=mObjectCreate();
       mFile(jsondata,"./canada.json");
       
       mTimerBegin("Morn json");
       struct JSONNode *json=mJSONLoad(jsondata);
       int n=0;
       struct JSONNode *coordinates0=mJSONRead(json,"features[0].geometry.coordinates");
       for (int j=0;j<coordinates0->num;j++)
       {
           struct JSONNode *coordinates1 = mJSONRead(coordinates0,j);
           for (int i=0;i<coordinates1->num;i++)
           {
               struct JSONNode *coordinates2 = mJSONRead(coordinates1,i);
               double x=mJSONRead(coordinates2,0)->dataD64;
               double y=mJSONRead(coordinates2,1)->dataD64;
               n++;
               // printf("x=%f,y=%f\n",x,y);
           }
       }
       mTimerEnd("Morn json");
       
       mObjectRelease(jsondata);
       return n;
   }

   void test2()
   {
       int n=Morn_test2();
       printf("get %d coordinates\n\n",n);
   }

Result is:

|image2|

It can be seen: 1. `rapidjson <https://github.com/Tencent/rapidjson>`__ `yyjson <https://github.com/ibireme/yyjson>`__ and `Morn <https://github.com/jingweizhanghuai/Morn>`__ is much faster than other
json library (`cjson <https://github.com/DaveGamble/cJSON>`__ is OK in Test 1,but is slowest in test 2), 2. 
`yyjson <https://github.com/ibireme/yyjson>`__ and `Morn <https://github.com/jingweizhanghuai/Morn>`__ is faster than `rapidjson <https://github.com/Tencent/rapidjson>`__.

.. _header-n85:

Test 3
~~~~~~

Comparing the performance of `rapidjson <https://github.com/Tencent/rapidjson>`__, `yyjson <https://github.com/ibireme/yyjson>`__ and `Morn <https://github.com/jingweizhanghuai/Morn>`__ by parsering many
different json files. `rapidjson <https://github.com/Tencent/rapidjson>`__ and `yyjson <https://github.com/ibireme/yyjson>`__ are both known for their high performance.

The testing file are: `canada.json <https://github.com/miloyip/nativejson-benchmark/blob/master/data/canada.json>`__, 
`citm_catalog.json <https://github.com/miloyip/nativejson-benchmark/blob/master/data/citm_catalog.json>`__, 
`twitter.json <https://github.com/chadaustin/sajson/blob/master/testdata/twitter.json>`__, 
`github_events.json <https://github.com/chadaustin/sajson/blob/master/testdata/github_events.json>`__, 
`apache_builds.json <https://github.com/chadaustin/sajson/blob/master/testdata/apache_builds.json>`__, 
`mesh.json <https://github.com/chadaustin/sajson/blob/master/testdata/mesh.json>`__, 
`mesh.pretty.json <https://github.com/chadaustin/sajson/blob/master/testdata/mesh.pretty.json>`__, 
and 
`update-center.json <https://github.com/chadaustin/sajson/blob/master/testdata/update-center.json>`__

In this program we parse each of these files for 100 times and measure
the time-consume.

Testing program is:

.. code:: c

   #define TEST_TIME 100

   void rapidjson_test3(const char *filename)
   {
       MString *jsondata=mObjectCreate();
       mFile(jsondata,filename);
   
       mTimerBegin("rapidjson");
       for(int i=0;i<TEST_TIME;i++)
       {
           rapidjson::Document doc;
           doc.Parse(jsondata->string);
       }
       mTimerEnd("rapidjson");
       mObjectRelease(jsondata);
   }
   
   void yyjson_test3(const char *filename)
   {
       MString *jsondata=mObjectCreate();
       mFile(jsondata,filename);
   
       mTimerBegin("yyjson");
       for(int i=0;i<TEST_TIME;i++)
           yyjson_doc_get_root(yyjson_read(jsondata->string,jsondata->size-1,0));
       mTimerEnd("yyjson");
       mObjectRelease(jsondata);
   }
   
   void Morn_test3(const char *filename)
   {
       MString *jsondata=mObjectCreate();
       mFile(jsondata,filename);
   
       mTimerBegin("Morn json");
       for(int i=0;i<TEST_TIME;i++)
           mJSONLoad(jsondata);
       mTimerEnd("Morn json");
       mObjectRelease(jsondata);
   }
   
   void test3()
   {
       const char *filename;
   
       filename = "./canada.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   
       filename = "./citm_catalog.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
       
   
       filename = "./testdata/twitter.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   
       filename = "./testdata/github_events.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   
       filename = "./testdata/apache_builds.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   
       filename = "./testdata/mesh.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   
       filename = "./testdata/mesh.pretty.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   
       filename = "./testdata/update-center.json";
       printf("\nfor %s:\n",filename);
       rapidjson_test3(filename);
       yyjson_test3(filename);
       Morn_test3(filename);
   }

Result is:

|image3|

It can be seen that: 1.`Morn <https://github.com/jingweizhanghuai/Morn>`__ and `yyjson <https://github.com/ibireme/yyjson>`__ are much faster then `rapidjson <https://github.com/Tencent/rapidjson>`__ with
2 to 5 times, 2.In most cases `Morn <https://github.com/jingweizhanghuai/Morn>`__ is faster then `yyjson <https://github.com/ibireme/yyjson>`__.

.. |image1| image:: https://z3.ax1x.com/2021/10/13/5KQ2Is.png
   :target: https://imgtu.com/i/5KQ2Is
.. |image2| image:: https://z3.ax1x.com/2021/10/13/5KQWin.png
   :target: https://imgtu.com/i/5KQWin
.. |image3| image:: https://z3.ax1x.com/2021/10/13/5KK5Yq.png
   :target: https://imgtu.com/i/5KK5Yq
