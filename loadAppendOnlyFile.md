# Redis中AOF文件载入过程

##1、创建为客户端，打开.aof文件
struct redisClient *fakeClient;
fakeClient = createFakeClient()

（while循环，直到将所有行数据读完）
PS：文件中内容的协议格式：*3\r\n$3\r\nSET\r\n$3\r\nKEY\r\n$5\r\nVALUE\r\n
##2、确认协议格式，以*开始。
if (buf[0] != '*') goto fmterr;

##3、获得参数个数argc = 3
atoi()函数是将字符串转化成整数，跳过前面的空格字符，直到遇上数字或正负号才开始做转换，而再遇到非数字或字符串时（'\0'）才结束转化，并将结果返回（返回转换后的整型数）。
argc = atoi(buf+1);


（for循环，循环次数是argc=3，只有第4条执行for循环）
##4、获得参数的值argv[][]
fgets()遇见换行或结尾就结束，按行读取；fread()遇见结束或是读取到指定字节数结束。获得3和对应3个字节的SET值。
//例如读取‘3\r\n’的内容

if (fgets(buf,sizeof(buf),fp) == NULL) goto readerr;
//判断参数是以’$’开始
if (buf[0] != '$') goto fmterr;

// 读取参数值的长度
len = strtol(buf+1,NULL,10);   //wyj:   buf+1的字符串转换成10进制的长整形,遇到非数字或'\0'结束转换
// 读取参数值
argsds = sdsnewlen(NULL,len);
if (len && fread(argsds,len,1,fp) == 0) goto fmterr;
// 为参数创建对象
argv[j] = createObject(REDIS_STRING,argsds);

//wyj:  \r\n
if (fread(buf,2,1,fp) == 0) goto fmterr; 

##5、根据argv[0]即SET去查找相应的指令redisCommand。
struct redisCommand *cmd;
cmd = lookupCommand(argv[0]->ptr);

##6、调用伪客户端去执行命令set key value
fakeClient->argc = argc;
fakeClient->argv = argv;
cmd->proc(fakeClient);

##7、最后关闭aof文件，释放伪客户端
// 关闭 AOF 文件
fclose(fp);
// 释放伪客户端
freeFakeClient(fakeClient);
