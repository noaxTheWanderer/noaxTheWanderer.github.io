---
title: "Dealing with USPTO Patents Data for Application and Grants"
layout: post
tagline: "Linux xml tools xslt"
tags : linux
comments: true
category: Linux, xml_pp, xml_grep, xsltproc, dtd
---
{% include JB/setup %}

My journey in the IT world takes me to different places, some tough, some easy 
and some complecated.

Lately, I have been working to import USPTO data to MySQL data. It was a quite 
challenging task which I thought it was easy in the beginning but turned out to 
be extremely complex.

Since the requirement was pretty easy, that is all, data should be in MySQL it 
made life easy. But where do I start.

As usual, I love short-cuts, so I googled around to see if other people well 
people had done it, and it was not what exactly what I needed. In short, it did 
NOT solve my problems at all.

So I set out and started looking around at the uspto files.

The files are in zip files and it is simply one big file with thousands and 
thousands of patents bundled XML files together.

The first thing I did was that I wrote a Bash script which would download all 
files of the same year and place them in same folder. Then I would execute 
another Bash Script to test the Zipped file is valid. If the Zip file is valid, 
then unzip to the directory with zip name. After that, I would change the 
current working directory to the unzipped folder.

Then I would pass the huge file name as an argument to a Python 
Script that would split this huge file into smaller files as shown below:

The solution was pretty easy, a simple python script did this as follows:
{% highlight python %}
  import sys;
  import os;
  MASTER_XML_FILE=sys.argv[1];
  L = file(MASTER_XML_FILE, "r").read().split("<?xml ")
  i = 0
  for l in L:
      i = i + 1
      f = file("file%d.xml" % i , "w")

      print >>f, "<?xml " + l

{% endhighlight %}

The Python script was simple and good enough for my job. I knew that if I could 
manage to have individual XML files splitted from large file, I was close to 
the solution.

After this, the rest was simply easy, using xsltproc I transformed the xml 
files to the required new xml format which I would then later import to the 
MySQL database.

{% highlight bash %}

ls *.xml | xargs -n1 -I {} sh -c 'xsltproc '"${XSLT_TRANSFORMER}"'  {} > _{}' 
2&> "${BAR}_xsltTransformationErrors.txt"
{% endhighlight %}

However, the above explanation is not all of what happened. I can share my 
issues on this as follows:

#### Malformed Zip files
Some of the zip files, downloaded where malformed and can not remember exact 
count.

#### DTD issues
The DTD's provided by the USPTO where not correct and had so many problems. 
Some of the problems I encountered where:

* Hard coded file paths
* Unnecessary duplicates of some of the DTD's lead to lots of confusion
* Scientific files will likely mess up your new transformed files and result 
into empty files. The script below did the magic work of renaming the files.

{% highlight bash %}
#!/bin/bash

START_TIME=$(date +%s)

 for X in `ls file*.xml `
   do
    IS_US_SEQUENCE_LISTING_AVAILABLE=$(head -n 3 "$X" | grep 
"us-sequence-listing-2004-03-09.dtd" )
    if [[ -n $IS_US_SEQUENCE_LISTING_AVAILABLE ]]; then

        FILE=${X#*/}

        NEW_FORMED_FILE="${FILE%.*}._xml_"

        mv ${FILE} ${NEW_FORMED_FILE}
        [[ -f ${NEW_FORMED_FILE}  ]] && echo "Created new file 
${NEW_FORMED_FILE}"
        [[ ! -f ${FILE} ]] && echo "Old file ${FILE} is NOT available any more"
    fi
   done;
END_TIME=$(date +%s)

CONSUMED_TIME=$(( $END_TIME - $START_TIME ))
echo "It took $CONSUMED_TIME seconds to rename the whole directory"
{% endhighlight %}

Last after renamig the empty files, I wrote a simple script to create sql load 
statements

{% highlight bash %}

    DUMP_FILE="${DIRECTORY_NAME}_sql_loadtatements.txt"
    cat /dev/null > ${DUMP_FILE}

    for XML_FILE in $(find ${BASE_WORKING_DIRECTORY}/ -type f -iname 
"_file*.xml")
    do
        FILE_PATH="$XML_FILE"
        echo "LOAD XML LOCAL INFILE '"${FILE_PATH}"' INTO TABLE applicants ROWS 
IDENTIFIED BY '<applicants>';" >> "${DUMP_FILE}"
    done
{% endhighlight %}

After this point, my work was done and I simply run mysql import from command 
line and it did the magic!!

It is fun to work in this project and it has helped me sharpen my skills in xml 
and xslt.
I also got to learn working with different XML tools especially the XML::Twig 
library. The library had excellent resources such as xml_pp, xml_grep, 
xml_split.
In a single line:
{% highlight bash %}
xml_pp file.xml
{% endhighlight %}

This made it easy when trying to browse large files with character entity 
errors.
As for xml_grep was handy when trying to grab some element tags such as below:

{% highlight bash %}
xml_grep XML_ELEMENT_NAME file*.xml 
#This will grab XML_ELEMENT_NAME from all files in directory
{% endhighlight %}
xml_grep provided a lot of power and flexibility when playing around with xml 
files.

I found a lot of solutions online on how to import the data using 
other tools but I find this solution easy.

### WHY!? 
Majority of the times one does not need all of the data from the files hence 
performing transformations using XSLT is cheap unlike writing a whole program 
in Java or Python. With XSLT I could perform validations on the fly and get 
clean data.

Many times, in the research I am participating we need some extra fields, since 
the whole job is automated already, then what I do is change the XSLT 
transformer, run the whole script to generate new XML files to be imported with 
the new fields.

One final key point, in this project I had to choose machine time vs developer 
time. The code might appear sloppy but I believe you can not import all patents 
data in one day. Hence, I chose machine time.
