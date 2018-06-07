# Getting Started with XNAT's Container Service: a Tutorial

## [Part 1. Installing the Container Service and Executing a Command: Hello, World](./tutorial_part1.md)
  [What This Tutorial Covers](./tutorial_part1.md/#)  
  [What You Need Before You Begin](./tutorial_part1.md#what-you-need-before-you-begin)  
  [Installing the Container Service Plugin](./tutorial_part1.md#installing-the-container-service-plugin)  
  [Installing Images for the Container Service](./tutorial_part1.md#installing-images-for-the-container-service)  
  [Setting Up a First Command](./tutorial_part1.md#setting-up-a-first-command)  
  [Interacting With the REST API](./tutorial_part1.md#interaction-with-the-rest-api)  
  [Running Our Hello World Command](./tutorial_part1.md#running-our-hello-world-command)  
  [Investigating the Command History](./tutorial_part1.md#investigating-the-command-history)  
  [Changing the Input](./tutorial_part1.md#changing-the-input)    
  [Error Logging](./tutorial_part1.md#error-logging)  

## [Part 2. Manipulating Data from XNAT: dcm2niix](./tutorial_part2.md)
  [A First Command With Imaging Inputs](./tutorial_part2.md#a-first-command-with-imaging-inputs)  
  [Mounts, Inputs, and Outputs](./tutorial_part2.md#mounts-inputs-and-outputs)  
  [A Simile Involving a Wizard You Can Skip If It's Confusing](./tutorial_part2.md#a-simile-involving-a-wizard-you-can-skip-if-its-confusing)  
  [A Closer Look at Mounts](./tutorial_part2.md#a-closer-look-at-mounts)  
  [XNAT's Data Organization](./tutorial_part2.md#xnats-data-organization)  
  [A Closer Look at Wrapper Inputs](./tutorial_part2.md#a-closer-look-at-wrapper-inputs)  
  [Output Handling](./tutorial_part2.md#output-handling)  
  [One Common Error](./tutorial_part2.md#one-common-error)

## Part 3. Accepting Arbitrary Inputs and Outputs: FSL Operations

## Glossary

Command: a JSON file that gives XNAT the information it needs to run processes in a Docker Container.

Command Input: the third kind of input, along with the two wrapper inputs.  Command inputs are strings supplied directly to the command line.  They are matched to a position in the command line by the input  replacement key.

Command Line: in this context, the string provided to the container by the command that actually runs at the container's shell prompt.  ("Shell prompt" is another term for command line.  Another way of writing this definition would be to say the command line is what runs at the container's command line.)

Derived Input: one of the two kinds of wrapper inputs.  It is an XNAT object that is not passed to the API in a post request, but whose API 

Docker Container: a specific instance of a machine, derived from a Docker Image.  Docker containers can run processes.

Docker Hub: a repository with pre-built Docker Images.

Docker Image: an image is a snapshot of a machine that has the capacity to run processes.  It form the basis for a Docker container. 

Experiment: in the XNAT context, experiment is what we might elsewhere call a session -- an object representing the discrete period of time the participant was in the lab and completed one or more scans. It is the level of organization between Project and Scan

External Input: one of the two kinds of wrapper inputs. It is a path to an XNAT object, passed to the REST API in a POST request.  

Input Replacement Key: a string in the command-line value of the command that will be matched with, and replaced by, a string from a command input provided when the container is launched.

JSON: a common data format for objects.  The object is represented within curly braces, keys are always strings, and values can be strings, numbers, booleans, arrays, objects, or null.  The command is written in JSON.

Matcher: a [JSONPath filter](https://wiki.xnat.org/display/CS/Command#Command-jsonpath-filters) expression that tells XNAT about the correct characteristics of the derived input.

Mount: a location in a file system where external storage can be accessed.

Object: in programming generally, a data type that can have properties and methods associated with it.  In JSON, a series of key value pairs enclosed in curly braces.  Scan, Resource, and File are examples of first kind of object in XNAT.  The command is a JSON object.

Resource: in this context, a resource is an XNAT object that represents a category of files in a scan.  It is the level of organization between Scan and File.  A resource contains files, and can be provided to a mount in a container and act like a directory. 

REST API: a set of conventions wherein a program can send requests to a application via a URI to either get information from the application's back end, or to provide data to the application.

Scan: the level of XNAT data organization between Experiment and Resource.  

Standard Error (StdErr): the abstract place where a computer sends information about the errors encountered in running a program.  Standard Error can be printed to a screen, saved to a file, etc.  In the case of the container service, it is saved to a log file.

Standard Output (StdOut): the abstract place where a computer sends information about the output generated while running a program.  Standard Output can be printed to a screen, saved to a file, etc.  In the case of the container service, it is saved to a log file.

Swagger: a web interface to XNAT's API.

Wrapper: in this context, the part of the command that gives XNAT the information it needs to resolve an API route into mounted directories and files a container can use, and when the container has completed its process, to take output and store it within XNAT's own directory tree.

