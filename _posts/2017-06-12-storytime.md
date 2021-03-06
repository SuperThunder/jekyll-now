---
layout: post
title: "Sysadmin Storytime"
date: 2017-06-12
---

# Making a Python CLI reader for a story collection
As I was reading through this collection of [sysadmin horror stories](http://www-uxsup.csx.cam.ac.uk/misc/horror.txt) and often lost my place it struck me it must be easy to make a simple command line viewer to show one story at a time. 

The stories have a fairly simple format, very usefully deliminated by 77 dashes between each story.
This makes it fairly easy to split to stories up by doing a

    text.split('-'*77)

Here is an example story:

    ----------------------------------------------------------------------------

    From: barrie@calvin.demon.co.uk (Barrie Spence)
    Organization: DataCAD Ltd, Hamilton, Scotland

    In article <1992Oct13.014245.24930@ccu1.aukuni.ac.nz> russells@ccu1.aukuni.ac.nz (Russell Street) writes:
    >rca@Ingres.COM (Bob Arnold) writes:
    >>      9) It's a lot less painful to learn from someone else's experience
    >>         than your own (that's what this thread is about, I guess :-) )
    >
    >With out trying to wander off the thread tooooo much ... In my
    >experience the best experiences to learn off are your own :)
    >I wonder how many stories we have got so far about "I will never
    >type rm -r /" as root. (And no I have not done that _yet_, but
    >the day will come :()
    >

    My mistake on SunOS (with OpenWindows) was to try and clean up all the
    '.*' directories in /tmp. Obviously "rm -rf /tmp/*" missed these, so I
    was very careful and made sure I was in /tmp and then executed
    "rm -rf ./.*".

    I will never do this again. If I am in any doubt as to how a wildcard
    will expand I will echo it first.

    Barrie

    -----------------------------------------------------------------------------

An appreciable amount of space in the stories can be taken up by email information (to/from/re/org).
Quotes can also take up a lot of space. Sometimes the quotes are new and interesting, but sometimes they've been quoted several times already. Additionally, There's all that article information above the quote taking up space.  To effectively use terminal space we can add options to the program to remove email header information.

How do we cut down on metadata-ish information? At a simple level we could strip out any line that looks like it's email header info (using string searches or regex). We can also look for quotes, and maybe keep track of them to only show them the first one or two times.

# A really simple reader
This snippet will show the stories one at a time. This makes reading them a bit easier, but we can't move forward or back and have to start at the beginning. It also does not incorporate any stripping of extraneous information.

	with open("horror.txt", 'r') as horrorfile:
		filecont = horrorfile.read()
		stories = filecont.split('-'*77)

	import os
	for story in stories:
		print(story)
		os.system('clear')
		input()
		
# But how will we add features?
Having never done 'live' input with Python, I needed to do a stack overflow on how to incorporate moving forward/back in response to keys. [SA, of course, delivers](https://stackoverflow.com/questions/9013078/how-to-read-users-input-when-in-loop-and-without-blocking-work-in-this-loop).

An input-only thread that puts keypresses into an input queue seems like a reasonable design. The main thread can watch the queue and execute the keypresses, which are at this point only to move forward or back; maybe to toggle showing metadata. 
Unfortunately, things get complicated here. To accept input without needing the user to type the character in and hit enter, you start getting into platform specific libraries. Words like 'msvcrt' and 'curses' start coming up. We'll sidestep the issue (which also allows us to sidestep threading) for now but look at solutions later.


To implement this cleanly I redesigned the script into a more reliable form:
    
        def main():
            import time # for sleeping - waiting without busy loops
            import os # for clearing the console
            #import threading # for later

            inputqueue = []
            CurrentStory = 0
            stories = []
            
            # Handle the initial case of setting up the program
            def init():
                nonlocal stories
                with open("horror.txt", 'r') as horrorfile:
                    content = horrorfile.read()
                    stories = content.split('-'*77)
                    print("Stories: %d\n"%len(stories))
                
                
                
            # Handles the actual printing of the text; including whether or not to clear
            def display(text, clear=True):
                if( clear == True ):
                    os.system('clear')

                print(text)
            
            
            # Main control loop - handles showing text
            def control():
                nonlocal CurrentStory # get around namespace issue - we can fix this later with OOP
                nonlocal stories
                
                    
                while True:
                    display( stories[CurrentStory] )
                    WaitForInput()
                    command = inputqueue.pop(0)
                    # f: move forward a story
                    if( command == 'f' ):
                        # Don't want to show a story that doesn't exist
                        if( CurrentStory < len(stories) ):
                            CurrentStory += 1
                        
                    # b: move backward
                    elif( command == 'b' ):
                        if( CurrentStory > 0 ):
                            CurrentStory -= 1

                    # Allow user to directly type in story to jump to
                    elif( command.isnumeric() ):
                        if( int(command) > 0 and int(command) < len(stories) ):
                            CurrentStory = int(command)
                            
                    # else: input error
                    else:
                        # Do nothing on invalid input for now
                        pass            
            
            
            # Input function
            def WaitForInput():
                print("\nCurrent: %d\tCommand: "%CurrentStory,)
                inputqueue.append( input() )

            # Init first to display the first story
            init()
            # After that go into control which has a loop
            control()

            return 0

    main()

These are quite a few more changes than I expected to make at first! We have special functions for initializing, display, control, and input. The values entered by the user are fully error checked to avoid out of bounds index access. Clearing the console only works in shell terminals (not IDLE or cmd). 
Moving around is a bit cumbersome as you need to hit enter, but jumping to arbitrary stories was easily implemented with this input method.
I was surprised by how pleasant the program was to use even the first time I ran it! The script (Python3 only due to use of nonlocal keyword) and source files [can be found here](https://github.com/SuperThunder/SuperThunder.github.io/tree/master/content/HorrorStories).

# What next?
I think I'll be leaving this program alone for now (after I've read the rest of the sysadmin stories, of course), but it's a good candidate for future enhancement. I've always wanted to use (n)curses, and two nice features would be non-blocking input and coloured text.
It might also be possible to replicate this script with a shell one liner or a very minimal bash script.

I plan to put more projects on this blog, writing my thoughts and plans down as I program (and in a way that explains to other people) seems to help to a surprising degree with productivity.
