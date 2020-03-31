# Uploading Data from St. Jude `hpc` to DNAnexus

This guide describes how to upload data from St. Jude's research computing
cluster to DNAnexus. It covers logging in to `hpc`, creating an interactive
session, installing DNAnexus Upload Agent, and uploading files to DNAnexus.

The research cluster is restricted to St. Jude employees, as it is only
accessible on St. Jude's intranet.

## Logging in to `hpc`

The SSH (Secure Shell) protocol is used to log in to `hpc`, the hostname of
the entry point into St. Jude's research cluster. SSH provides a secure
method to log in to a remote computer.

Regardless of platform, logging in requires a St. Jude username and will
prompt you for your password. They are the same username and password used to
log in to all St. Jude services.

Upon a successful log in, you will see a prompt similar to
`[username@splprhpc06 ~]$`.

### Windows

Windows does not have an SSH client preinstalled. As of Windows 10 1809,
OpenSSH is included as [a feature] that can be installed.

  * Open the Settings app (search "Settings" in the Start menu).
  * Under Apps > Apps & features > Apps & features, click on _Optional
    features_.
  * Under the OpenSSH Client entry, click _Install_.

Open PowerShell and run

```
> ssh <username>@hpc
```

Alternatively (or on older versions of Windows), install and use the terminal
emulator [PuTTY].

[a feature]: https://docs.microsoft.com/en-us/windows-server/administration/openssh/openssh_install_firstuse
[PuTTY]: https://www.chiark.greenend.org.uk/~sgtatham/putty/

### macOS

macOS includes OpenSSH by default. Open Terminal and run

```
$ ssh <username>@hpc
```

### Linux

Most Linux distributions have OpenSSH installed by default. Open a terminal
and run

```
$ ssh <username>@hpc
```

## Creating an interactive session

When logging in to `hpc`, you are placed on a _head node_, a controlled gateway
configured to allow external access to the cluster. This node is not meant for
computation, which is done on a cluster node instead.

To move to a cluster node, start an interactive session. The cluster's
workload is managed by [IBM Spectrum LSF], and even though LSF commands can
be used to create an interactive session, the High-Performance Computer
Facility (HPCF) provides a convenience script named `hpcf_interactive` for a
simpler invocation.

```
$ hpcf_interactive
```

This moves you from the head node to a node in the research cluster, and
tasks that require CPU time, memory, high bandwidth network access, etc., can
be executed.

When an interactive session starts, the prompt will look similar to the
following:

```
Job <99871669> is submitted to queue <interactive>.
<<Waiting for dispatch ...>>
<<Starting on nodecn013>>
[username@nodecn013 ~]$
```

[IBM Spectrum LSF]: https://www.ibm.com/us-en/marketplace/hpc-workload-management

## DNAnexus Upload Agent

DNAnexus provides multiple methods to upload files to their platform. In this
section, Upload Agent (UA) is used. It is simple to install and supports
resuming interrupted transfers.

### Install

DNAnexus Upload Agent is available from DNAnexus Documentation's
[Downloads][download] article. Download and install Upload Agent:

```
$ cd /tmp
$ wget https://dnanexus-sdk.s3.amazonaws.com/dnanexus-upload-agent-1.5.31-linux.tar.gz
$ mkdir -p $HOME/.local/bin
$ tar xf dnanexus-upload-agent-*-linux.tar.gz --strip-components 1 --directory $HOME/.local/bin
```

The research cluster runs Red Hat Enterprise Linux 7.3, which is why the
Linux package is used.

The rest of this chapter assumes `$HOME/.local/bin` is in your `PATH`.
Otherwise, when calling `ua`, use `$HOME/.local/bin/ua`.

[download]: https://documentation.dnanexus.com/downloads#Upload-Agent

### Setup

Set the environment variable `DX_SECURITY_CONTEXT` with [a DNAnexus
authentication token]. Rather than exposing your username and password, an
authentication token is an alternative way to log in to DNAnexus using a
secret phrase. Replace `<auth-token>` with your own token.

```
$ export DX_SECURITY_CONTEXT='{"auth_token_type": "bearer", "auth_token": "<auth-token>"}'
```

This must be set once for ever new interactive session started.

[a DNAnexus authentication token]: https://documentation.dnanexus.com/user/login-and-logout#authentication-tokens

## Uploading files

With DNAnexus Upload Agent installed and configured, files can be uploaded by
running `ua`.

### Basic usage

The simplest usage of `ua` is providing the DNAnexus project to upload to and
a source file.

```
$ ua --do-not-compress --project <project-name-or-id> <src>
```

For example, with a DNAnexus project named `flagstat` and a file named
`sample.1.bam`, the command would be

```
$ ua --do-not-compress --project flagstat sample.1.bam
```

!!! question "Why is `--do-not-compress` always set?"
    Upload Agent uses an unfortunate default where uncompressed files are
    automatically gzipped. For example, uploading the text file
    `samplesheet.txt` results in the file `samplesheet.txt.gz` on DNAnexus.
    This is confusing and unexpected, and file management is more
    straightforward with the option disabled.

### Batched jobs

When working in an interactive session, if the session is closed (e.g., a
network disconnection or you quit the terminal), all running commands are
killed. To avoid this from happening, a _job_ is submitted in its place,
which continues to run even after the session is closed.

To submit a job, use the LSF command [bsub], where `-P` is an arbitrary
project name for the job submission.

```
$ bsub \
    -P <project> \
    -R "rusage[mem=2882]" \
    ua \
    --do-not-compress \
    --project <project-name-or-id> \
    <src>
```

When uploading a large batch of files, HPCF requests the
`/stjudecloud/uploads` job group be used to rate limit upload jobs. This can
be done using the `-g` option when submitting a job.

```
$ bsub \
    -P dx-upload \
    -g /stjudecloud/uploads \
    -R "rusage[mem=2882]" \
    ua \
    --do-not-compress \
    --project <project-name-or-id> \
    <src>
```

!!! question "Where does the 2882 MiB resource requirement come from?"
    There is a [note in the source code of Upload Agent][ua-main-mem] that
    gives an estimate of how much memory is used for a transfer:

    `<read-threads> + 2 * (<compress-threads> + <upload-threads>) * <chunk-size>`

    Thus, using the default values (see `ua --help`) and adding 20% as a buffer:

    `(2 + 2 * (8 + 8) * 75 MiB) * 1.2 = ~2882 MiB`

[bsub]: https://www.ibm.com/support/knowledgecenter/en/SSWRJV_10.1.0/lsf_command_ref/bsub.man_top.1.html
[ua-main-mem]: https://github.com/dnanexus/dx-toolkit/blob/9e6398e1ce1e8b210df3f3f50abc82932084b2b3/src/ua/main.cpp#L151-L162

## Examples

The following examples are common usages of Upload Agent.

### Upload multiple files

`ua` takes multiple source arguments.

```
$ ua --do-not-compress --project <project-name-or-id> <src...>
```

For example, for two files `sample.1.bam` and `sample.1.bam.bai`, the command
is

```
$ ua --do-not-compress --project flagstat sample.1.bam sample.1.bam.bai
```

### Upload a file to a directory on DNAnexus

By default, all files uploaded via `ua` get placed at the root of the
project, i.e., `/`. To upload to a particular directory, use the `--folder`
option.

```
$ ua --do-not-compress --project <project-name-or-id> --folder /data sample.1.bam
```

The resulting uploaded file will be located at `/data/sample.1.bam`.

The directory does not have a previously exist in the DNAnexus project and
will be created automatically, along with its parents (similar to `mkdir -p`).

### Upload a directory

When uploading a directory, `ua` will only use the files at the top level
(similar to `find data -type f -maxdepth 1`). For example, take the following
directory structure and `ua` command.

```
$ tree data
data
├── extra
│   ├── sample.2.bam
│   └── sample.2.bam.bai
├── sample.1.bam
└── sample.1.bam.bai

1 directory, 4 files

$ ua --do-not-compress --project <project-name-or-id> data
```

The resulting files will be `/sample.1.bam` and `/sample.1.bam.bai`. Use the
`--folder` option and name it the same as the local source directory to
include the directory on DNAnexus.

```
$ ua --do-not-compress --project <project-name-or-id> --folder /data data
```

The resulting files will be `/data/sample.1.bam` and `/data/sample1.bam.bai`.

To include the subdirectories, use the `--recursive` option.

```
$ ua --do-not-compress --project <project-name-or-id> --folder /data --recursive data
```

Thus, using both `--folder` and `--recursive` uploads an entire directory:
`/data/sample.1.bam`, `/data/sample.1.bam.bai`, `/data/extras/sample.2.bam`,
and `/data/extras/sample.2.bam.bai`.