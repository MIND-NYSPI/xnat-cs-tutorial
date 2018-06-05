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

### XNAT's Data Organization

In order to understand how external and derived inputs relate to each other, you need to understand the abstract heirarchy XNAT uses to organize data. It goes

Project -> Experiment -> Scan -> Resource -> File

Project is parent to Experiment (and an Experiment is a Project's child), Experiment is parent to Scan, and so on.

A Project we've already seen.  This naming scheme is pretty confusing; I, at least, would be tempted to think of Projects and Experiments as synonymous.  But instead, in XNAT, Projects are entire studies, and an Experiment is more like a session -- a period of time the subject was in the scanner.  The Scan is a single neuroimaging run, a T1 or anything else.  A Resource is an abstraction that you can think of as a subdirectory that holds data of a certain format.  So when the description in the dcm2niix command for "scan-dicoms" says "The dicom resource on the scan", it's referring to a collection of dicoms that can act like a directory when attached to a mount.  A File, to XNAT, is an abstraction too.  It's a File object that itself has properties, like a name. 

We've already seen a subset of XNAT's REST API, the paths that begin `xapi` that give us access to XNAT's internal methods. To make requests that refer to XNAT objects, like scans and resources, we are going to need to use another subset of XNAT's API, the paths that begin with `data`.  This [page](https://wiki.xnat.org/display/XAPI/XNAT+REST+API+Directory) documents the paths that you'll need to access scans, resources, and file objects.

You may have a set of dicoms already on your XNAT instance that you're interested in converting to NIFTI, but if you don't, we'll need to upload some, and now is a useful time to do it to explore the data API structure a bit more.  XNAT provides some [sample  dicoms](https://central.xnat.org/app/action/ProjectDownloadAction/project/Sample_DICOM).  Now that you've created your project, from the Project page you can go to Upload Images, choose Option 2, XNAT Compressed Uploader, select Archive, navigate to the downloaded .zip file, and click Begin Upload.

![Upload Image Files](UploadImageFiles.png)

Now when you naviage to the Container Service Tutorial Project page, you should see your sample data.

![Sample Data](SampleSubject.png)

Now navigate to `<your-xnat-url>/data/experiments` (if you are using the Vagrant VM and haven't changed any settings, you can replace `<your-xnat-url>` with http://10.1.1.17). This is the equivalent to making a GET request to that route in the API. You should see at least that experiment listed.   

![Get Experiments](GetExperiments.png)

You can also see that an experiment has several properties: an ID, a project, a date, an xsiType, a label, and insert_date, and a URI.  If you now find the ID for your experiment (it should be associated with the project CS_TUTORIAL) and add it to the path in your URL bar you can find an XML page with information about that particular experiment.  But let's look at something a little more formatted: enter 

`<your-xnat-url>/data/experiments/<your-experiment-id>/scans`

in the URL bar.  

This experiment has one scan, with an ID of 2.

![Scans](Scans.png)

Now try `<your-xnat-url>/data/experiments/<your-experiment-id>/scans/2/resources`

And finally `<your-xnat-url>/data/experiments/<your-experiment-id>/scans/2/resources/DICOM/files`.

As you can see, the API can give you detailed information about the experiments, scans, resources, and files in your project, and that information will be important in interpreting (and later writing) the external and derived inputs in commands.


# A Closer Look at External and Derived Inputs

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

```

So the external input, the thing the program (in other words, us at the Swagger UI) provides, is going to be the path to an XNAT scan.  In our sample data that we uploaded in the previous section, it would look like like this: `experiments/<your-experiment-id>/scans/2`

Where is the `/data` prefix you ask?  Good question!  When providing API routes to XNAT you leave off the prefix.  There's no clear reason why.

The matcher is a piece of syntax JSON filter.  It forms as a check and a winnower.   

The matcher on scan is 

` "'DICOM' in @.resources[*].label"`

`@` in the filter signifies the thing in this input, in this case, a scan.  `@.resources` generates a list of resources, and `@.resources[*]` means "look at all of them". `@.resources[*].label` then generates the list of their labels from each resource's label property.  This filter gets one thing, the scan we passed it, and ideally it returns one thing.  If the scan we passed didn't have any dicom resources `'DICOM' in @.resources[*].label` would be false, the filter wouldn't pass anything through, and our command wouldn't run.  

```
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

In this command, the derived input is a resource that it derived from the external input scan.  We tell XNAT its type, Resource, and which input its derived from -- the input named "scan".    A scan could have more than one resource associated with it -- in fact, when we execute this command will have a DICOM resource and a NIFTI resource.  So the matcher in this case, `@.label == 'DICOM'`  performs an important winnowing function -- it makes sure we execute a command on the DICOM resource, and not another XNAT object.  We also use this entry to provide files for our input mount.  A Resource is an abstraction, but it acts like a directory that contains files, and if we mount the resource on our container, we can now access that directory from the command line of our container.  

Recall what we told XNAT about our input mount:

```
 "mounts": [
        {
            "name": "dicom-in",
            "writable": "false",
            "path": "/input"
        },
```

and the command line:

```
 "command-line": "dcm2niix [BIDS] [OTHER_OPTIONS] -o /output /input",
 ```

Since our derived input tells us that it provides files for command mount "dicom-in", and mount "dicom-in" has the path `/input` now `/input` from command line refers to a directory with the files form our derived input, the DICOM resource, which contains the DICOM files.

An important note about derived inputs: they only work if the matcher matches exactly one thing.  In this case, we have exactly one resource with the label 'DICOM'.  If we had more than one, it would be a problem.  

