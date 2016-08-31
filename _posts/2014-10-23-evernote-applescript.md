---
layout : post
title: Evernote AppleScript
categories: blog
permalink: /blog/Evernote-AppleScript/
comments : true
excerpt: How I wrote an AppleScript to add an Evernote entry
seo__desc : AppleScript to create and append top journal entries in Evernote
seo__key : AppleScript, Evernote, script, productivity, workflow
---
I use Evernote on a daily basis for many things and find it to be a valuable tool to organize my thoughts, notes, important emails, etc. After all, that is what Evernote is designed for, right? On my mobile I have the app 'vJournal' which is simple and straight-forward for making journal entries to Evernote. For those not familiar with the app...It lets you quickly type your entry, add a picture and location if you like, and then upload to your Evernote account. Once the entry reaches Evernote it is entered into a notebook with the current year as its title. This notebook is a sub-notebook of the notebook titled 'My Journal'. The note is titled with the month, day and year in the following format: October 23, 2014. If a note with this name already exists, the entry submitted is appended to the note. Each entry is also logged with the time of day it was created on top of a gradient background, providing a nice separator for multiple entries.

<img src="/assets/images/ENgrad1.png" alt="first Evernote gradient example">

The fore mentioned process for logging journal entries to Evernote is great, when I have my phone in hand, but what about the majority of the day when my hands are trying to be productive banging away on my Mac? I thought to myself:

>"Wouldn't it be great to have a shortcut, launching a dialog box, accepting
my thoughts and sending them off to my journal in Evernote?"

This is when the idea sparked to explore writing an AppleScript to accomplish this task. I started by heading straight to [the source](https://dev.evernote.com/doc/articles/applescript.php "Evernote AppleScript doc") for documentation. Scanning through the examples outlined, I was quickly able to gather my thoughts and start coding out the script.

The first step is to display a dialog box that allows for the journal entry.
{% highlight applescript %}
display dialog "Journal Entry" default answer ""
set journal_entry to text returned of result  
{% endhighlight %}
Once the entry is complete it is stored in a variable titled *journal_entry*. Now the call to tell Evernote to launch is evoked. I did not want the process of logging a journal entry to interrupt my workflow by switching to Evernote so I also tell <b>System Events</b> to set the Evernote process visibility to false.
{% highlight applescript %}
tell application id "com.evernote.Evernote" to launch
delay 2.0

tell application "System Events" to set visible of process "Evernote" to false
{% endhighlight %}

Now that Evernote is running a command is sent to check is a notebook titled with the current year exists and if it does not, create one.
{% highlight applescript %}
tell application id "com.evernote.Evernote"
    if (not (notebook named "2014" exists)) then
        make notebook with properties {name:"2014"}
    end if
end tell
{% endhighlight %}

The next step was to set some variables for time and date that will be used within the journal entry title and the entry separator.
{% highlight applescript %}
set notebook1 to "2014"

set theDate to current date

set titleDate to short date string of (current date)

set _time to time string of theDate
{% endhighlight %}

The next variable set is one I titled *journalAppend*. This is the content that gets added as each journal entry within the notebook inside Evernote. The first part is the HTML styling that renders each entry as its own item when there are multiple entries for one date. The background gradient produces a nice green gradient or the time of each entry. This helps visually separate each entry. Looking towards the end of the code you see *_time* which was set in the previous steps and places the time the entry was created, on top of the background gradient. You will also find *journal_entry* which is the actual entry that was captured by the dialog box in the beginning.
{% highlight applescript %}
set journalAppend to
    "<br>
        <div>
            <div style=
                'background-image: linear-gradient(right , rgb(255,255,255) 0%, rgb(59, 180, 40) 85%);                                
                background-image: -o-linear-gradient(right , rgb(255,255,255) 0%, rgb(59, 180, 40) 85%);                             
                background-image: -moz-linear-gradient(right , rgb(255,255,255) 0%, rgb(59, 180, 40) 85%);                               
                background-image: -webkit-linear-gradient(right , rgb(255,255,255) 0%, rgb(59, 180, 40) 85%);                                
                background-image: -ms-linear-gradient(right , rgb(255,255,255) 0%, rgb(59, 180, 40) 85%);                                
                background-image: -webkit-gradient(linear,right top,left top,color-stop(0, rgb(255,255,255)),color-stop(0.85, rgb(59, 180, 40)));                                
                height:26px;                                 
                margin-left:5px;                                 
                margin-right:0px;
                padding-left: 5px;
                border-top-left-radius: 4px 4px;
                border-bottom-left-radius: 4px 4px;
                color:rgb(220,220,220);                                
                text-align:left;'>" & _time & "
            </div>
            <br>
            <div>
                <p>" & journal_entry & "</p>
            </div>
        </div>"
{% endhighlight %}

Now that *journalAppend* has been set with the content for the entry, we proceed with telling Evernote to either find or create a note in our notebook with the date. Remember that the date, set by *titleDate*, is the title of our entry each day into the journal. I'll first post the whole command and then explain the steps:
{% highlight applescript %}
tell application id "com.evernote.Evernote"
    set foundNotes to find notes "notebook:\"" & "2014" & "\"" & " intitle:\"" & titleDate & "\""
    set found to ((length of foundNotes) is not 0)
    if not found then
        create note title titleDate with html journalAppend notebook notebook1
    else
        repeat with curnote in foundNotes
            tell curnote to append html journalAppend
        end repeat
    end if
{% endhighlight %}
I am setting a variable titled *foundNotes* with an Evernote action to **find notes** and then telling it to look in the notebook "2014" with a title of current date (*titleDate*)
{% highlight applescript %}
set foundNotes to find notes "notebook:\"" & "2014" & "\"" & " intitle:\"" & titleDate & "\""
{% endhighlight %}
The next line sets a variable, *found*, to count how many notes were found from the previous command
{% highlight applescript %}
set found to ((length of foundNotes) is not 0)
{% endhighlight %}
The next step says if *found* returns no results, meaning there is not already a note with the current date as the title, then create one - using the current date (*titleDate*), appending it with the HTML entry (*journalAppend*), under the notebook "2014" (*notebook1*)
{% highlight applescript %}
if not found then
        create note title titleDate with html journalAppend notebook notebook1
{% endhighlight %}
Finally, if *foundNotes* did return a result we want to append the journal entry to the note that has already been created for today
{% highlight applescript %}
else
        repeat with curnote in foundNotes
            tell curnote to append html journalAppend
        end repeat
    end if
{% endhighlight %}

That's it! I have successfully developed a solution to a help in my daily productivity by allowing me to quickly enter thoughts into Evernote without interrupting my current flow.

<img src="/assets/images/ENpost.png" alt="entry to Evernote via Applescript">
