### A First Command With Imaging Inputs

We are on XNAT to do neuroimaging, are we not?  The next step in getting to command mastery is to see if you can execute a command that takes neuroimaging inputs and outputs.  The XNAT team has already written a command to execute dcm2niix, and it was for that reason that we pulled the dcm2niix image.  

Here's the command (you can also find it [here](https://github.com/NrgXnat/docker-images/blob/master/dcm2niix/command.json)):

```
{
    "name": "dcm2niix",
    "description": "Runs dcm2niix",
    "info-url": "https://github.com/rordenlab/dcm2niix",
    "version": "1.4",
    "schema-version": "1.0",
    "type": "docker",
    "image": "xnat/dcm2niix",
    "command-line": "dcm2niix [BIDS] [OTHER_OPTIONS] -o /output /input",
    "mounts": [
        {
            "name": "dicom-in",
            "writable": "false",
            "path": "/input"
        },
        {
            "name": "nifti-out",
            "writable": "true",
            "path": "/output"
        }
    ],
    "inputs": [
        {
            "name": "bids",
            "description": "Create BIDS metadata file",
            "type": "boolean",
            "required": false,
            "default-value": false,
            "replacement-key": "[BIDS]",
            "command-line-flag": "-b",
            "true-value": "y",
            "false-value": "n"
        },
        {
            "name": "other-options",
            "description": "Other command-line flags to pass to dcm2niix",
            "type": "string",
            "required": false,
            "replacement-key": "[OTHER_OPTIONS]"
        }
    ],
    "outputs": [
        {
            "name": "nifti",
            "description": "The nifti files",
            "mount": "nifti-out",
            "required": "true"
        }
    ],
    "xnat": [
        {
            "name": "dcm2niix-scan",
            "description": "Run dcm2niix on a Scan",
            "contexts": ["xnat:imageScanData"],
            "external-inputs": [
                {
                    "name": "scan",
                    "description": "Input scan",
                    "type": "Scan",
                    "required": true,
                    "matcher": "'DICOM' in @.resources[*].label"
                }
            ],
            "derived-inputs": [
                {
                    "name": "scan-dicoms",
                    "description": "The dicom resource on the scan",
                    "type": "Resource",
                    "derived-from-wrapper-input": "scan",
                    "provides-files-for-command-mount": "dicom-in",
                    "matcher": "@.label == 'DICOM'"
                }
            ],
            "output-handlers": [
                {
                    "name": "nifti-resource",
                    "accepts-command-output": "nifti",
                    "as-a-child-of-wrapper-input": "scan",
                    "type": "Resource",
                    "label": "NIFTI"
                }
            ]
        }
    ]
}
``` 

Add this command to the dcm2niix image under Images & Commands.

### Mounts, Inputs, and Outputs

Here are some things to pay attention to:

1. In our `command-line` entry, there are now some values in brackets, "[BIDS]" and "[OPTIONS]".  Those are input replacement keys.  This command doesn't use the default for a replacement key, the name of the inputs surrounded by hashtags, but instead specifies a custom replacement key.  Under `inputs`, we now have an array of two items, a boolean specifying true or false for BIDS, and a string for any other options we want to pass to dcm2niix.

2. We now have mounts for both input and outpout.  A mount is a location in a file system where external storage can be accessed.  In this case, the file system is our container.  In order for our container to operate on input files from XNAT, they must be mounted within the container.  Similarly, the output files must have a defined location in the container directory tree so that XNAT can access them and move them into its own directory structure.

3. Under the `xnat` key we now have two different kinds of inputs, so there are now three kinds of inputs a command takes.

* The first kind is the input we've already seen: a value, either a string, a Boolean, or a number (?), that goes directly on the command line, replacing a replacement key.  You list these inputs under the `input` key on the top level of the JSON object.  

* The second is external input.  This is the path to an XNAT object.  This path is supplied in the request body when the program (i.e., you, using Swagger or any other client) makes a request. External inputs go in an array under the `external-inputs` key underneath the `xnat` key. 

* The third is derived input.  Derived input are XNAT objects or strings that you get from other XNAT objects by referencing their properties or their children.  We'll go into more detail about this in a bit.  Since really all the container knows about are what it executes from the command line and what files it has mounted, your derived input needs at some point to be mounted or translated into the first kind of input to be used.  This kind of input is stored in an array under the `derived-inputs` key underneath the `xnat` key.  The way the developers think about it, the information under the `xnat` key, that is, the wrapper, wraps around the container and allows it to communicate with XNAT. 

4. We also now have outputs.  After the container executes whatever process is necessary, it must copy output from the container directory tree to the XNAT directory tree.  The wrapper provides the information about how to do that. 

### A Simile Involving a Wizard You Can Skip If It's Confusing

 It's like the container is an old wizard in a cave, and the command is an emmissary who comes bearing delicious fruit that has been enchanged to look like useless lumps of coal. (The fruits are our imagine files.)  But the lumps of coal are covered in a piece of paper, the wrapper, on which is a spell that allows the wizard to break the enchantment and unlock their true fruit nature.  Similarly, the container wizard is so grateful to the command emissary for bringing her fruit that she offers the emissary the pits, wrapped in the same paper.  When the emissary recites the second spell on the paper the pits turn into gold coins.

 ### More Details About Mounts

 To make sure we understand what's happening with our mounts, lets look at again at the command and this page from the [dcm2niix readme](https://github.com/rordenlab/dcm2niix).  


 > The minimal command line call would be dcm2niix /path/to/dicom/folder. However, you may want to   invoke additional options, for example the call dcm2niix -z y -f %p_%t_%s -o /path/ouput /path/to/dicom/folder will save data as gzip compressed, with the filename based on the protocol name (%p) acquisition time (%t) and DICOM series number (%s), with all files saved to the folder "output". For more help see help: dcm2niix -h.

```
  "command-line": "dcm2niix [BIDS] [OTHER_OPTIONS] -o /output /input",
    "mounts": [
        {
            "name": "dicom-in",
            "writable": "false",
            "path": "/input"
        },
        {
            "name": "nifti-out",
            "writable": "true",
            "path": "/output"
        }
    ],
```

So dcm2niix takes two directories in the command line.  Our command creates two directories in the container file system, one at /output and one at /intput.  It's next job is going to make sure that /input has the files that dcm2niix expects, which it's going to do with the external inputs and derived inputs in the wrapper.  

# More Details About Inputs

Let's look at the external inputs and derived inputs sections of the command again.

```
"external-inputs": [
                {
                    "name": "scan",
                    "description": "Input scan",
                    "type": "Scan",
                    "required": true,
                    "matcher": "'DICOM' in @.resources[*].label"
                }
            ],
            "derived-inputs": [
                {
                    "name": "scan-dicoms",
                    "description": "The dicom resource on the scan",
                    "type": "Resource",
                    "derived-from-wrapper-input": "scan",
                    "provides-files-for-command-mount": "dicom-in",
                    "matcher": "@.label == 'DICOM'"
                }
            ],
```

So the external input, the thing the program (in other words, us at the Swagger UI) is going to be the path to an XNAT scan.  

In order to understand how external and derived inputs relate to each other, you need to understand the abstract heirarchy XNAT uses to organize data. It goes

Project -> Experiment -> Scan -> Resource -> File

Project is parent to Experiment (and an Experiment is a Project's child), Experiment is parent to Scan, and so on.

A Project we've already seen.  This naming scheme is pretty confusing; I, at least, would be tempted to think of Projects and Experiments as synonymous.  But instead, in XNAT, Projects are entire studies, and an Experiment is more like a session -- a period of time the subject was in the scanner.  The Scan is a single neuroimaging run, a T1 or anything else.  A Resource is an abstraction that you can think of as a subdirectory that holds data of a certain format.  So when the description for "scan-dicoms" says "The dicom resource on the scan", that's what it means 