# Using Java to play a sound

Recently, I want to using java play a mp3 sound. I have try a lot:

* javafx
* javazoom basicplay 
* other from google search

What confused me is that each of them works well in my Eclipse environment. Unfortunately, when I export the java code as a runnable jar, none work well any more. Most common error is missing files or can not find the file - the mp3 file. Well, it is really embarrassing. I am sure there must be a lot of ways, but I can't find one.

Hopefully, I got one. Maybe it is not a good one. But it works well for me.

The code is easy:

```java
import sun.audio.AudioPlayer;
import sun.audio.AudioStream;

	InputStream in;
	try {
      in = new FileInputStream(new File("./resource/alarm.mp3"));
      AudioStream audioStream = new AudioStream(in);
      AudioPlayer.player.start(audioStream);
	} catch (IOException e) {
      //
	}
```

What I do is

```
1. codindg as above
2. put all resource in a folder named resource and put it in the same foot folder or src
	- father folder
		- src
		- resource
3. using Eclipse export it as a runnable jar -> you can search it online
4. create a new folder like application, and put the jar and resource folder in the application folder like
	- application
		- resource
		- application.jar
5. now, you can run it as an application. You can run in a command or terminal, or just double click or write a shell or bat

```

It works well for me, just hope it can help. If you have any questions, send me an email:

**guzuxing1991@gmail.com** 