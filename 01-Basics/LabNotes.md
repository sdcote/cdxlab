# 01 - Basics

This lab goes over basic operations of a data transfer job and how to install and call the Coyote DX tools.

## Simple Jobs

We will start with the classic first example.

### Hello World

You can call Coyote DX directly from the command line:
```
java -jar CoyoteDX-0.8.6.jar HelloWorld.json
```
This calls Coyote DX to read in its configuration from the file `HelloWorld.json` and perform the Read-Transform-Write operations it directs. Open `HelloWorld.json` and  review its parts:
* **Class** - The class of job it is. In this case it is a simple `Job`.
* **Job** - The Job configuration 
  *  **Name** - The optional name of the job. If omitted, the name of the configuration file would be used without its extension.
  * **Reader** The component that reads data.
    *  **class** - the class of reader to use. A `StaticReader` just reads the configured data.
    *  **fields** configures the `StaticReader`what fields to use.
  * **Writer** - The component that writes data
    * **class** - the class of writer to use. A `ConsoleWriter` writes the fields to the console.
    * **format** - what format to use.
    * **indent** - a flag indication if the format should be indented or not.

#### Job Directory

You will notice that a `HelloWorld` directory now exists in the lab directory. This is the Job directory. It's purpose is to provide separation for all the files a job may create in performing its function.

Job directories provide a file system context for all instances of a job. It can hold the results of processing for other jobs to read, or maintain state. In later exercises we will see state persistence in the form of persistent contexts, where contextual data can be stored between runs.
The Job Directory for the `HelloWorld` job is empty because it only wrote a message to the console.

## Reading and Writing
Now that the obligatory "Hello World" example is out of the way, here is a little more functional example:
```
java -jar CoyoteDX-0.8.6.jar Simple.json
```
If you look in the `Simple` job directory you will see the results of its processing, the `users.txt` file. It is a flat file representation of the data in the `users.csv` file. In this case we read in a CSV and generated a flat file with fixed column sizes, handy for older systems to read in.

Open the `Simple.json` file and you will see a configuration similar to the previous example. It contains a Reader and a Writer.  (We will get to transformations later.)

## The Template

Look in the `Template.json` file for an idea of what sections can exist in a Coyote data transfer job.

* **Context** - configures how data is populated in the transformation context of all job components.
* **PreProcess** - tasks that are to be perform before reading in records of processing.
* **Reader** - configures the component that reads in data for processing.
* **Filter** - configures the components that can filter out any unwanted data.
* **Validate** - configures components to validate data before they are processed. Invalid data is discarded if so configured.
* **Transform** - configures components to transform the data as they pass through the pipeline.
* **Mapper** - configures the component to map key-value pairs to other names.
* **Writer** - configures components to output the key-value pairs in a specific format.
* **PostProcess** configures tasks that are to be run once all the data has been written.
* **Listener** - configures components that will perform processing when events occur in the transform context. For example, what to do when invalid data is encountered.
* **Logging** - configures logging for the data transfer jobs.

Details of each section will be discussed in subsequent labs.

## The Installer

The Coyote DX JAR file is useful on its own, but its real power comes from other modules that can be optionally installed. To do this, Coyote DX can sense when it is being run as a part of a larger deployment and the data transfer jobs can reference classes in other modules.

There is a standard set of modules that come with Coyote, but not all of them may be needed. To keep the footprint small, it is possible to install only the required modules. This is helpful in resource constrained devices.

To help with the installation, Coyote is distributed as a cross-platform installer.

Download the installer by going to the [Coyote Release](https://github.com/sdcote/coyote/releases) page and choosing the release you want. If you want a pre-release version, select the "Tags" button for unreleased versions.

To run the installation, call `java -jar CoyoteInstaller-X.X.X.jar` and follow the prompts.

If you are running this on a windowless system such as a Unix compute instance, you can run the installer in "test mode":

```
java -jar CoyoteDXInstall.jar -console
```

### Execution Path

Although not strictly necessary, it is convenient to add the `bin` directory of the install to the systems execution path. That way is is possible to run `cdx` from anywhere in the system without having to specify a complete path  to the launch script.  You could also alias the command if your platform and command interpreter supports it.

Use the preferred method on your host to accomplish that. You could place it on the system path or you can place it in the user path, the choice is yours. Once this is done, you can call Coyote DX with the `cdx` command:

```
cdx HelloWorld.json
```

The above is the most common way to call Coyote DX.

### Uninstall

Just delete the directory. That's it; nothing else.

The installer only creates directories and copies files to them. No  updates are made to execution paths, registries or anything else. This  is a package of simple Java components and a few scripts to make calling them more convenient and not an application requiring complex setup.

If you updated your execution path to make running `cdx` simpler, then you may want to undo what you did to make that happen, but that is completely up to you.


### Directory Structure

The installer 

* **bin** - the directory containing the start scripts. This directory should be on your execution path.
* **cfg** - This directory contains all your globally accessible data transfer jobs.
* **lib** - all the libraries (JAR files) are contained in this directory. You can add your own JARs here.
* **doc** - this directory contains documentation and the license files for libraries.
* **demo** - if you choose to install demo configuration, this is where they will be found.
* **Uninstaller** this is the exact opposite of the installer, it will remove Coyote DX from your system.

There will be two other directories here, depending on the configuration of your jobs:

* **log** - this is where all logs will go unless they are configured with absolute paths.
* **wrk** - a common location for all the job directories.

The above will only appear if your jobs need them.

### Override Work Directory

When you run Coyote from a common location, as would be the case with the Installer, you might notice that your job directory is not created in the current directory, but in the `wrk` directory where you installed Coyote. This is by design, allowing all your work files to be in one location.

This can be overridden by using the `-owd` argument:

```cdx -owd HelloWorld.json```

This tells Coyote to **o**verride the **w**ork **d**irectory and use the current directory instead.

# Summary

At this point it should be clear how to install and use Coyote DX to call your data transfer job configurations.

If you are looking for logs and you can't find them in your local directory, check the `log` directory in Coyote DX home. The same is true for your job directories; check the `wrk` directory where you installed Coyote DX.