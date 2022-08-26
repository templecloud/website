# An Introduction to Functional Programming

Preamble: to start building our program, we do need to to a bit of programming and C++ first. We are going to review some programming concepts and write some code that will have little to do with computer graphics but will set a general framework with wich we will work until the end.

Studying an existing high-quality API or open-source project is a really good way of learning (actually based on personal experience, we think it is the best method). Not only we can learn about computer graphics techniques but also improve our knowledge about programming in general. One aspect of Embree that is particularly interesting is its API which is described by the programmers as a **fully functional programming API**. By fully they probably mean that the API strictly follows the rule of functional programming but we won't get into defining whether this is true or not. What we care about here is: what is **functional progamming**?

Generally speaking when you create an API for other people to use, you don't want them to start messing up with whatever data you feel is strictly internal to the system.  Assuming your code does something useful, functional programming allows you to protect external users from accessing data held by your code directly while still providing them with ways to obviously make use of your code. Code written that way is easier to debug (we will explain why later). But is harder for beginners to understand.

Functional programming primarily relies on the concept of **pure function**. There are some definition out there for what a pure function is but they can be quite abstract. So rather than going through a long explanation let's write some code.

Did we mention this wasn't going to be easy? Ok so now we did. We will do out best to provide the clearest possible explanation but believe us, this is something that is not easy to explain. Hang in there, while nerdy, it's cool stuff, and most importantly this will really lay the foundations of everything else we will be doing next. The code we will have at the end of this chapter will fit on a page. The original code is spread across dozens of files, so we've done everything we could already to make the process much simpler (yes you can be grateful).

```
class RenderDevice
{
public:
	typedef struct __RTHandle {}* RTHandle;
	typedef struct __RTSomething : public __RTHandle {}* RTSomething;
	
   	Device::RTSomething rtNewSomething(int a) 
   	{
        return (RenderDevice::RTSomething) new Something(a);
	}
    void                rtCommit(Device::RTHandle handle) {}
    void                rtSomeTask(Device::RTSomething something_i) {}
};

RenderDevice* g_device = nullptr;

int main(int argc, char** argv)
{
	g_device = new Device;
	Device::RTSomething g_something = g_device->rtNewSomething(1);
	g_device->rtCommit(g_something);
	g_device->rtSomeTask(g_something);
	delete g_device;
}
```
