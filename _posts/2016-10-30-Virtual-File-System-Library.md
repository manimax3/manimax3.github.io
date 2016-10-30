---
layout: post
title:  "Virtual File System Library - Concept"
date:   2016-10-30 
categories: programming
author: "manimax3"
comments: true
---

So lets get started with a new project. The plan is a library which simulates a virtual file system.
The main target area is of course gamedev. My main goals for the library are:

1. Easy to use
2. Object Orientated
3. Makes use of C++11
4. Completly running in a seperate thread
5. Multiplatform

To be honest I dont have much experience in cross platform programming but I think thats something that might be doable
at least for Windows, Linux, Mac and maybe Android.

Also there is a code snippet showing how I imagine the Library to be used when its done at the end of this post.

The plan is that you have to create a System object which represents one local and physical file. Once you have created one of these
system you have to make a request to load a file which cause to system to do so, without a request the system doesnt load any files.

Once you made a request its added to a queue and after that one thread loads these files. There will never be running more than one thread 
per system (!). The advantage of this is that you can load your assets anytime in your game without having to worry about that the process
might be blocking your game if its a larger file.

Once the file is loaded there are two options depending on your LoadOptions (see the snippet) when the callback you specified is getting called.
First its getting called directly from the other thread. I wouldnt recommend doing this because you would have to deal with the typical Multithreading
proplems yourself.
Or you do in your main loop poll the finished requests which calls the callbacks synchronosly in your main thread.
Notice that in your callback you do get a void pointer pointing to some data you specifiedt yourself. This has the advantage that you dont have to 
deal with public refernces to your entity and stuff like that just to let the entity know that his sprite has finished loading.

When the file has been loading you can obtain the data by asking the system object to give it to you. And here you have again the options in what type
you would like to have your data for example as a stream or byte array.

For the future I'm planning some more cool features for example locking your files with a password, compression options, handling multiple systems in 
one handler and more...



{% highlight c++ %}

/*
	Virtual File Lib Example Usage
	**WARINING**
	This is just a prototype the final implementation might be used different
*/

#include "VFSLib.h"

vfl::System system("/path/to/physical/file/");

void callback(std::string file_name, bool success, void* yourcustomdata)
{
	//Notice your game that file has been loaded
	char* data;
	if(!system.get_file_data(file_name, data))
		//Error occurred
}

int main()
{

	vfl::FileLoadRequestOptions options;
	options.callback = callback;//Type of this field is std::function<void(std::string, bool, void*)>
	options.customdata = //eg. a gameobject. This is the void pointer you get in your callback
	options.asyncallback = true;//Should the callback called directly from the different thread or via the polling?
	//More Options will later be available

	system.request("filename/inside/the/virtual/system", options);
  //File gets loaded in a seperate thread

	//If asyncallback = false was specified you have to do sth like this
	while(gameLoopRunning)
	{
		system.poll(); //Calls the callbacks of the finished files
	}

	return 0;
}

{% endhighlight %}
