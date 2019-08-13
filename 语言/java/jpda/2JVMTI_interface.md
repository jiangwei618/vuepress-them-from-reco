# JVMTI:Java 虚拟机工具接口
&emsp;&emsp;JVMTI（Java Virtual Machine Tool Interface）即指 Java 虚拟机工具接口，它是一套由虚拟机直接提供的 native 接口，它处于整个 JPDA 体系的最底层，所有调试功能本质上都需要通过 JVMTI 来提供。通过这些接口，开发人员不仅调试在该虚拟机上运行的 Java 程序，还能查看它们运行的状态，设置回调函数，控制某些环境变量，从而优化程序性能。

&emsp;&emsp;JVMTI 并不一定在所有的 Java 虚拟机上都有实现，不同的虚拟机的实现也不尽相同。不过在一些主流的虚拟机中，比如 Sun 和 IBM，以及一些开源的如 Apache Harmony DRLVM 中，都提供了标准 JVMTI 实现。

## 1. JVMTI与Agent的关系
&emsp;&emsp;JVM TI is implemented by HotSpot and allows a native code ‘agent’ to inspect and modify the state of the JVM.（参看http://openjdk.java.net/groups/hotspot/docs/Serviceability.html#battach 中的说明）
**Agent 即 JVMTI 的客户端，它和执行 Java 程序的虚拟机运行在同一个进程上，通过调用 JVMTI 提供的接口和虚拟机交互，负责获取并返回当前虚拟机的状态或者转发控制命令**。把 Agent 编译成一个动态链接库之后，我们就可以在 Java 程序启动的时候来加载它（启动加载模式），也可以在 Java 5 之后使用运行时加载（活动加载模式）。

&emsp;&emsp;总结：个人感觉，JVMTI一种操作JVM的规范与接口。开发者通过Agent的方式使用该套接口。同时Agent的逻辑运行机制依赖与JVMTI接口的实现。

## 2. gent的工作过程
&emsp;&emsp;我们使用 JVMTI 的过程，主要是设置 JVMTI 环境，监听虚拟机所产生的事件，以及在某些事件上加上我们所希望的回调函数。

&emsp;&emsp;Agent 的主要功能是通过一系列的在虚拟机上设置的回调（callback）函数完成的，一旦某些事件发生，Agent 所设置的回调函数就会被调用，来完成特定的需求。

## 3. JVM启动时加载
&emsp;&emsp;Agent 是在 Java 虚拟机启动之时加载的，这个加载处于虚拟机初始化的早期，在这个时间点上：

1. 所有的 Java 类都未被初始化；
1. 所有的对象实例都未被创建；
1. 因而，没有任何 Java 代码被执行；但在这个时候，我们已经可以：
操作 JVMTI 的 Capability 参数；
使用系统参数；

## 4. JVM运行态加载
基于jvm attach机制。
关于dynamic attach: Dynamic Attach. This is a Sun private mechanism that allows an external process to start a thread in HotSpot that can then be used to launch an agent to run in that HotSpot, and to send information about the state of HotSpot back to the external process.(参见 http://openjdk.java.net/groups/hotspot/docs/Serviceability.html#battach )

&emsp;&emsp;++attach是Sun的私有实现（并不是所有的jvm都提供该功能），该机制允许 外部进程 在JVM（该JVM指运行被监控、需要被操控的Java程序的JVM）中启动一个线程,该线程随后会启动加载agent，并且将本JVM的状态发送给 外部进程。++

## 5. JVMTIAgent 示例
&emsp;&emsp;该Agent通过监听 JVMTI_EVENT_METHOD_ENTRY 事件，注册对应的回调函数来响应这个事件，来输出所有被调用函数名。

&emsp;&emsp;具体实现都在 MethodTraceAgent 这个类里提供。按照顺序，他会处理环境初始化、参数解析、注册功能、注册事件响应，每个功能都被抽象在一个具体的函数里。

```
MethodTraceAgent.h

#include "jvmti.h"

class AgentException 
{
 public:
	AgentException(jvmtiError err) {
		m_error = err;
	}

	char* what() const throw() { 
		return "AgentException"; 
	}

	jvmtiError ErrCode() const throw() {
		return m_error;
	}

 private:
	jvmtiError m_error;
};


class MethodTraceAgent 
{
 public:

	MethodTraceAgent() throw(AgentException){}

	~MethodTraceAgent() throw(AgentException);

	void Init(JavaVM *vm) const throw(AgentException);
        
	void ParseOptions(const char* str) const throw(AgentException);

	void AddCapability() const throw(AgentException);
        
	void RegisterEvent() const throw(AgentException);
    
	static void JNICALL HandleMethodEntry(jvmtiEnv* jvmti, JNIEnv* jni, jthread thread, jmethodID method);

 private:
	static void CheckException(jvmtiError error) throw(AgentException)
	{
		// 可以根据错误类型扩展对应的异常，这里只做简单处理
		if (error != JVMTI_ERROR_NONE) {
			throw AgentException(error);
		}
	}
    
	static jvmtiEnv * m_jvmti;
	static char* m_filter;
};
```
```
MethodTraceAgent.cpp

#include <iostream>

#include "MethodTraceAgent.h"
#include "jvmti.h"

using namespace std;

jvmtiEnv* MethodTraceAgent::m_jvmti = 0;
char* MethodTraceAgent::m_filter = 0;

MethodTraceAgent::~MethodTraceAgent() throw(AgentException)
{
    // 必须释放内存，防止内存泄露
    m_jvmti->Deallocate(reinterpret_cast<unsigned char*>(m_filter));
}

void MethodTraceAgent::Init(JavaVM *vm) const throw(AgentException){
    jvmtiEnv *jvmti = 0;
	jint ret = (vm)->GetEnv(reinterpret_cast<void**>(&jvmti), JVMTI_VERSION_1_0);
	if (ret != JNI_OK || jvmti == 0) {
		throw AgentException(JVMTI_ERROR_INTERNAL);
	}
	m_jvmti = jvmti;
}

void MethodTraceAgent::ParseOptions(const char* str) const throw(AgentException)
{
    if (str == 0)
        return;
	const size_t len = strlen(str);
	if (len == 0) 
		return;

  	// 必须做好内存复制工作
	jvmtiError error;
    error = m_jvmti->Allocate(len + 1,reinterpret_cast<unsigned char**>(&m_filter));
	CheckException(error);
    strcpy(m_filter, str);

    // 可以在这里进行参数解析的工作
	// ...
}

void MethodTraceAgent::AddCapability() const throw(AgentException)
{
    // 创建一个新的环境
    jvmtiCapabilities caps;
    memset(&caps, 0, sizeof(caps));
    caps.can_generate_method_entry_events = 1;
    
    // 设置当前环境
    jvmtiError error = m_jvmti->AddCapabilities(&caps);
	CheckException(error);
}
  
void MethodTraceAgent::RegisterEvent() const throw(AgentException)
{
    // 创建一个新的回调函数
    jvmtiEventCallbacks callbacks;
    memset(&callbacks, 0, sizeof(callbacks));
    callbacks.MethodEntry = &MethodTraceAgent::HandleMethodEntry;
    
    // 设置回调函数
    jvmtiError error;
    error = m_jvmti->SetEventCallbacks(&callbacks, static_cast<jint>(sizeof(callbacks)));
	CheckException(error);

	// 开启事件监听
	error = m_jvmti->SetEventNotificationMode(JVMTI_ENABLE, JVMTI_EVENT_METHOD_ENTRY, 0);
	CheckException(error);
}

void JNICALL MethodTraceAgent::HandleMethodEntry(jvmtiEnv* jvmti, JNIEnv* jni, jthread thread, jmethodID method)
{
	try {
        jvmtiError error;
        jclass clazz;
        char* name;
		char* signature;
        
		// 获得方法对应的类
        error = m_jvmti->GetMethodDeclaringClass(method, &clazz);
        CheckException(error);
        // 获得类的签名
        error = m_jvmti->GetClassSignature(clazz, &signature, 0);
        CheckException(error);
        // 获得方法名字
        error = m_jvmti->GetMethodName(method, &name, NULL, NULL);
        CheckException(error);
        
        // 根据参数过滤不必要的方法
		if(m_filter != 0){
			if (strcmp(m_filter, name) != 0)
				return;
		}			
		cout << signature<< " -> " << name << "(..)"<< endl;

        // 必须释放内存，避免内存泄露
        error = m_jvmti->Deallocate(reinterpret_cast<unsigned char*>(name));
		CheckException(error);
        error = m_jvmti->Deallocate(reinterpret_cast<unsigned char*>(signature));
		CheckException(error);

	} catch (AgentException& e) {
		cout << "Error when enter HandleMethodEntry: " << e.what() << " [" << e.ErrCode() << "]";
    }
}
```

&emsp;&emsp;Agent_OnLoad 函数会在 Agent 被加载的时候创建这个类，并依次调用上述各个方法，从而实现这个 Agent 的功能。Agent_OnUnload函数，在agent卸载时调用。

&emsp;&emsp;(通过在vm参数里 加上-agentlib，则会VM启动时，执行VM JVMTIAgent的Agent_OnLoad函数 ）

&emsp;&emsp;（Agent_OnAttach函数：如果agent不是在启动时加载的，而是我们先attach到目标进程上，然后给对应的目标进程发送load命令来加载，则在加载过程中会调用Agent_OnAttach函数。）
```
Main.cpp

#include <iostream>

#include "MethodTraceAgent.h"
#include "jvmti.h"

using namespace std;

JNIEXPORT jint JNICALL Agent_OnLoad(JavaVM *vm, char *options, void *reserved)
{
    cout << "Agent_OnLoad(" << vm << ")" << endl;
    try{
        
        MethodTraceAgent* agent = new MethodTraceAgent();
		agent->Init(vm);
        agent->ParseOptions(options);
        agent->AddCapability();
        agent->RegisterEvent();
        
    } catch (AgentException& e) {
        cout << "Error when enter HandleMethodEntry: " << e.what() << " [" << e.ErrCode() << "]";
		return JNI_ERR;
	}
    
	return JNI_OK;
}

JNIEXPORT void JNICALL Agent_OnUnload(JavaVM *vm)
{
    cout << "Agent_OnUnload(" << vm << ")" << endl;
}
```

&emsp;&emsp;Agent 编译和运行
```
g++ -I${JAVA_HOME}/include/ -I${JAVA_HOME}/include/linux 
MethodTraceAgent.cpp Main.cpp -fPIC -shared -o libagent.so
```

用于测试Java程序，一个简单类：
```
public class MethodTraceTest{

	public static void main(String[] args){
		MethodTraceTest test = new MethodTraceTest();
		test.first();
		test.second();
	}
	
	public void first(){
		System.out.println("=> Call first()");
	}
	
	public void second(){
		System.out.println("=> Call second()");
	
	}
}
```
运行Java程序，并指定Agent

java -agentlib:Agent=first MethodTraceTest

&emsp;&emsp;当程序运行到到 MethodTraceTest 的 first 方法是，Agent 会输出这个事件。“ first ”是 Agent 运行的参数，如果不指定话，所有的进入方法的触发的事件都会被输出，如果读者把这个参数去掉再运行的话，会发现在运行 main 函数前，已经有非常基本的类库函数被调用了。

## 6. Agent 时序图
在这里插入图片描述
![](https://raw.githubusercontent.com/jiangwei618/note/master/assets/image/2JVMTI_interface.md-2019-08-06-15-06-44.png)

## 7. 补充说明：Java Agent
我们通过-javaagent来指定我们编写的agent的jar路径（./myagent.jar），以及要传给agent的参数（mode=test），在启动的时候这个agent就可以做一些我们希望的事了。

javaagent的主要功能如下：

1. 可以在加载class文件之前做拦截，对字节码做修改
可以在运行期对已加载类的字节码做变更，但是这种情况下会有很多的限制，后面会详细说
2. 获取所有已经加载过的类
3. 获取所有已经初始化过的类（执行过clinit方法，是上面的一个子集）
4. 获取某个对象的大小
5. 将某个jar加入到bootstrap classpath里作为高优先级被bootstrapClassloader加载
6. 将某个jar加入到classpath里供AppClassloard去加载
7. 设置某些native方法的前缀，主要在查找native方法的时候做规则匹配

&emsp;&emsp;JavaAgent，必须要讲的是一个叫做instrument的JVMTIAgent（Linux下对应的动态库是libinstrument.so），因为javaagent功能就是它来实现的，另外instrument agent还有个别名叫JPLISAgent(Java Programming Language Instrumentation Services Agent)，这个名字也完全体现了其最本质的功能：就是专门为Java语言编写的插桩服务提供支持的。

instrument agent的核心数据结构如下：
```
struct _JPLISAgent {
    JavaVM *                mJVM;                   /* handle to the JVM */
    JPLISEnvironment        mNormalEnvironment;     /* for every thing but retransform stuff */
    JPLISEnvironment        mRetransformEnvironment;/* for retransform stuff only */
    jobject                 mInstrumentationImpl;   /* handle to the Instrumentation instance */
    jmethodID               mPremainCaller;         /* method on the InstrumentationImpl that does the premain stuff (cached to save lots of lookups) */
    jmethodID               mAgentmainCaller;       /* method on the InstrumentationImpl for agents loaded via attach mechanism */
    jmethodID               mTransform;             /* method on the InstrumentationImpl that does the class file transform */
    jboolean                mRedefineAvailable;     /* cached answer to "does this agent support redefine" */
    jboolean                mRedefineAdded;         /* indicates if can_redefine_classes capability has been added */
    jboolean                mNativeMethodPrefixAvailable; /* cached answer to "does this agent support prefixing" */
    jboolean                mNativeMethodPrefixAdded;     /* indicates if can_set_native_method_prefix capability has been added */
    char const *            mAgentClassName;        /* agent class name */
    char const *            mOptionsString;         /* -javaagent options string */
};

struct _JPLISEnvironment {
    jvmtiEnv *              mJVMTIEnv;              /* the JVM TI environment */
    JPLISAgent *            mAgent;                 /* corresponding agent */
    jboolean                mIsRetransformer;       /* indicates if special environment */
};
```
&emsp;&emsp;这里解释一下几个重要项：

1. mNormalEnvironment：主要提供正常的类transform及redefine功能。

1. mRetransformEnvironment：主要提供类retransform功能。

1. mInstrumentationImpl：这个对象非常重要，也是我们Java agent和JVM进行交互的入口，或许写过javaagent的人在写premain以及agentmain方法的时候注意到了有个Instrumentation参数，该参数其实就是这里的对象。

1. mPremainCaller：指向sun.instrument.InstrumentationImpl.loadClassAndCallPremain方法，如果agent是在启动时加载的，则该方法会被调用。

1. mAgentmainCaller：指向sun.instrument.InstrumentationImpl.loadClassAndCallAgentmain方法，该方法在通过attach的方式动态加载agent的时候调用。

1. mTransform：指向sun.instrument.InstrumentationImpl.transform方法。

1. mAgentClassName：在我们javaagent的MANIFEST.MF里指定的Agent-Class。

1. mOptionsString：传给agent的一些参数。

1. mRedefineAvailable：是否开启了redefine功能，在javaagent的MANIFEST.MF里设置Can-Redefine-Classes:true。

1. mNativeMethodPrefixAvailable：是否支持native方法前缀设置，同样在javaagent的MANIFEST.MF里设置Can-Set-Native-Method-Prefix:true。

1. mIsRetransformer：如果在javaagent的MANIFEST.MF文件里定义了Can-Retransform-Classes:true，将会设置mRetransformEnvironment的mIsRetransformer为true。

## 8. 在启动时加载instrument agent
&emsp;&emsp;正如前面“概述”里提到的方式，就是启动时加载instrument agent，具体过程都在InvocationAdapter.c的Agent_OnLoad方法里，这里简单描述下过程：

1. 创建并初始化JPLISAgent
1. 监听VMInit事件，在vm初始化完成之后做下面的事情：
1. 创建InstrumentationImpl对象
1. 监听ClassFileLoadHook事件
1. 调用InstrumentationImpl的loadClassAndCallPremain方法，在这个方法里会调用javaagent里MANIFEST.MF里指定的Premain-Class类的premain方法
解析javaagent里MANIFEST.MF里的参数，并根据这些参数来设置JPLISAgent里的一些内容

## 9. 在运行时加载instrument agent
&emsp;&emsp;上面会通过JVM的attach机制来请求目标JVM加载对应的agent，过程大致如下：

1. 创建并初始化JPLISAgent
1. 解析javaagent里MANIFEST.MF里的参数
1. 创建InstrumentationImpl对象
1. 监听ClassFileLoadHook事件
1. 调用InstrumentationImpl的loadClassAndCallAgentmain方法，在这个方法里会调用javaagent里MANIFEST.MF里指定的Agent-Class类的agentmain方法
&emsp;&emsp;在运行时加载instrument agent大致按照如下方式进行：

1. VirtualMachine vm = VirtualMachine.attach(pid);  （根据PID获取 运行被监控的JVM实例）
1. vm.loadAgent(agentPath, agentArgs); （通知该JVM去加载agent）
