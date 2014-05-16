---
layout: post
title: "Killing a fly with a hammer!<br /> NOT A GOOD IDEA!!"
description: "exiftool"
category: linux, exiftool
tags: [linux, exiftool]
comments: false
---
{% include JB/setup %}

Recently, I was browsing my old pictures and I wanted to find out the specific dates when I had been in vacation in Portugal about three years ago. A terrible idea!!

Over the years, I have changed several storage medias and I would usually copy/ move pictures from one storage media to another. And I would use the famous commands: 

   `cp -r ROOT_SOURCE ROOT_DESTINATION`
   `mv SOURCE_FOLDER DESTINATION_FOLDER`

Now that I needed my specific dates during the trip, I realized that using cp/ mv it was a terrible idea. So I set out a goal to save my pictures from now onward using time stamps for pictures just in case in the future.

I thought to myself, it might be a good idea if I could have folders in format such as:

      2012
	    2012_May_17
      2014
	    2014_January_23
	    2014_March_14


  When a folder has a specific date as name, it is a very useful piece of information, hence when I do a clumsy cp/ mv then folder names will be my guide in the future.


So I thought a script might be a good thing to automate such a job.

Well its easy, the script could perform the following tasks:
    Get SD Source
    Get Destination
    Check if both Source/ Destination are empty and are also valid directories
    then loop though the files to find the date of creation create the directories
    then loop through files again use to copy from source to destination
	`cp -r :SOURCE :DESTINATION`

  Then Who-la!! You are done.

  After spending like 1 hours writing this script, I realized date of creation is usually NOT there in Linux, I could use date of Modification or Last Access to create my directories. Then all of a sudden my script started to become longer and longer.

  Well, as I was progressing, I realized that, I hate my script because it had two loops and sometimes in one directory I could have 1000 or more pictures format of both: NEF and JPG.
  So I hated my script even before I could complete it.

  So I started searching around stackexchange, since it is the only place I visit when I get doubt or questions or need to learn about anything.

  Then I came across one questions about ["Rename JPG files according to date created"](http://stackoverflow.com/questions/4710753/rename-jpg-files-according-to-date-created)

One of the answers proposed to use EXIFTOOL to copy when playing with image files.

So I set out to check EXIFTOOL

it was the perfect tool..

So my script was reduced from being 50 lines to simply less than 2 lines and looked as follows:

    `exiftool -r -d ${DESTINATION_DIRECTORY}/%Y_%B_%d/%%f.%%e "-filename<filemodifydate" ${SD_CARD_PATH}`

One more point, is that I did not know each image has embedded more information details.

Through EXIFTOOL I could see on which date the image was taken.

`exiftool -d '%r %a, %B %e, %Y' -DateTimeOriginal -ext JPG . `

The above commands simply outputs Original Creation from meta-data for all files with extension of JPG in current directory. ( Beware . in after the JPG) 

Who-la!!
