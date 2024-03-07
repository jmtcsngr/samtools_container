# NPG Samtools Docker images

## The images ##

This is a Docker image of samtools

## Build instructions ##

A makefile is supplied that will by build the image and add
metadata based on `git describe`.

    cd ./docker
    make

## Release instructions

When releasing/tagging a new version please use annotated tags in the
command line interface.

```bash
git checkout master
git tag -a 'x.y.z' -m 'release x.y.z'
git push origin x.y.z
```

Using the web interface to create a release is known to produce
undesirable effects when picked up by automation.

# NPG Singularity wrappers

Each container that provides command line programs is self-documenting
and is able to install its own proxy wrappers outside of the container,
to allow these programs to be run transparently.

The images include the singularity-wrapper tool which allows programs to
be listed and their wrappers installed. The install target should be set
to a volume mounted into the container and the -p (install prefix) option
of the tool set accordingly. The -h option will show online help.

e.g. Show online help:

    $ docker run wsinpg/ub-18.04-irods-clients-4.2.11:latest \
        singularity-wrapper -h

e.g. List the programs provided by a container:

    $ docker run wsinpg/ub-18.04-irods-clients-4.2.11:latest \
        singularity-wrapper list
    {
        "package": "irods-clients",
        "executable": [
            "baton-chmod",
            ...
            "samtools"
        ]
    }

e.g. Install wrappers to $PREFIX/bin:

    $ docker run -v $PREFIX:/mnt/tmp \
        wsinpg/ub-18.04-irods-clients-4.2.11:latest \
          singularity-wrapper -p /mnt/tmp install

    $ ls $PREFIX/bin
    -rwxr-xr-x 1 kdj staff 406 Apr 12 15:47 baton-chmod
    ...
    -rwxr-xr-x 1 kdj staff 409 Apr 12 15:47 samtools


## Author

Keith James kdj@sanger.ac.uk
Jennifer Liddle js10@sanger.ac.uk
