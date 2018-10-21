# nlnz-m11n-tools-repository-splitter-updater National Library of New Zealand Modernisation Tools repository splitter and updater

## Synopsis

This repository contains a gradle build that splits the a source repository into path-based repositories and
create patches for updating those split repositories when the source repository changes.

## Motivation

Many legacy repositories have grown large with the combination of many components. This often makes them difficult
to decouple. Splitting large repositories into smaller repositories based on specific functionality makes the work of
of decoupling monoliths much easier. There are other reasons why splitting a large repository makes sense as well.

## Requirements

This tool requires the following plugins:
 - `nz.govt.natlib.m11n.tools:automation-plugin`, which contains the necessary classes to do the splitting work
 - `nz.govt.natlib.m11n.tools:gradle-plugin`, which contains useful classes for gradle builds

## Conceptual model

### Source repository to destination repositories

Splitting takes place by taking a source repository and splitting it into one or more split-target repositories based on
specific file paths in the source repository. The parameters used to split the source repository are specified in a
JSON file. The structure of that file is described in `src/main/resources/sampleProjectParameters.json`.

### Patching once the repositories have been split

Once the source repository has been split, the split-target repositories can be updated by changes to the source
repository by creating patch files that are applied to the split-target repositories.

## Usage

### Common parameters

#### showCommandOutput

Show the output of commands on the console. This is useful for understanding what the tool is doing.

Example:
```
showCommandOutput=true
```

#### tempWorkFolder

`tempWorkFolder` is the folder where temporary files used in processing are created. Currently only the repositoryWorkflow
operations use this. This value must be set.

Example:
```
tempWorkFolder=/path/to/tmp/folder
```

#### reportsFolder

`reportsFolder` is the folder where reports from the repository conversion stored. Currently only the repositoryWorkflow
operations use this. This value must be set if reports are generated.

Example:
```
reportsFolder=/path/to/reports/folder
```

#### autocreateReportsFolder

`autocreateReportsFolder` indicates whether `reportsFolder` needs to be created. If this
is false and the `reportsFolder` does not exist and reports are run then an error will occur. The default is false.

Example:
```
autocreateReportsFolder=true
```

#### sourceRepositoryWorkingFolder

`sourceRepositoryWorkingFolder` is the root folder for copies of the source repository. The source repository goes through
several iterations to extract the necessary git history for the split-target repositories.

Example:
```
sourceRepositoryWorkingFolder="/go/split-work/source-repository-work"
```

#### autocreateSourceRepositoryWorkingFolder

`autocreateSourceRepositoryWorkingFolder` indicates whether sourceRepositoryWorkingFolder needs to be created. If this
is false and the `sourceRepositoryWorkingFolder` does not exist then an error will occur. The default is false.

Example:
```
autocreateSourceRepositoryWorkingFolder=true
```

#### sourceRepositoryNameBase

`sourceRepositoryNameBase` is the base name for the source code repository. The base name is used in combination
with the `repositoryWorkflowStartingSequenceIndex` to produce the starting repository, as in
"${sourceRepositoryNameBase}-${repositoryWorkflowStartingSequenceIndex}". This value must be set. This repository
is found in `sourceRepositoryWorkingFolder`.

Example:
```
sourceRepositoryNameBase="my-repository-name"
```

#### repositoryWorkflowStartingSequenceIndex

`repositoryWorkflowStartingSequenceIndex` is the starting sequence index used by RepositoryWorkflow when processing a
repository. The RepositoryWorkflow has the expectation that there is a repository with the name
"${sourceRepositoryNameBase}-${repositoryWorkflowStartingSequenceIndex}" and this is the starting repository for all
its configured operations. The default value is 0.

Example:
```
repositoryWorkflowStartingSequenceIndex=4
```

#### sourceCodeRepositoryServerUrl

The git repository URL for the originating code repository. This value must be set if the task `cloneSourceRepository`
is called.

Example:
```
sourceCodeRepositoryServerUrl="git@github.com:MyOrganization/my-repository-name.git'
```

#### sourceRepositoryGitBranch

The source repository will be checked out in the `sourceRepositoryGitBranch`. This value must be set.

Example:
```
sourceRepositoryGitBranch="my-split-branch-name"
```

#### preserveBranchNames

preserveBranchNames is the comma-separated list of branch names to preserve when extracting specific paths and files
from the source repository. These branch names are also checked out of the cloned repositories to ensure
they are visible if the remote origin is removed. The default is an empty list.

Example:
```
preserveBranchNames="branch-name-one,branch-name-two"
```

#### projectParametersJsonFile

The JSON file that contains the project parameters used for the various pipeline operations. An example project
parameters file is found in `src/main/resources/sampleProjectsParameters.json`. This file is required.

Example:
```
projectParametersJsonFile="/my/work/my-repository-splitting-project-parameters.json"
```

#### splitTargetProjectsRootFolder

`splitTargetProjectsRootFolder` is the root folder for the split-target repository projects.

Example:
```
splitTargetProjectsRootFolder="${sourceRepositoryWorkingFolder}/targetRepositories"
```

#### autocreateSplitTargetProjectsRootFolder

`autocreateSplitTargetProjectsRootFolder` indicates whether the `splitTargetProjectsRootFolder` needs to be created.
If this is false and the `autocreateSplitTargetProjectsRootFolder` does not exist then an error will occur.
The default is false.

Example:
```
autocreateSplitTargetProjectsRootFolder=true
```

#### splitTargetRepositoryServerUrlBase

The git repository base URL for the split code repositories. This value must be set. The project name and '.git' is
appended to this for a specific code repository.

Example:
```
splitTargetRepositoryServerUrlBase="git@github.com:MyOrganization/my-split-target-repository-base-name-'
```

#### cloneSplitTargetRepositoriesRemoveRemoteOrigin

When cloning source repositories with the task `cloneSplitTargetRepositories`, this determines whether the remote origin is
removed (as in 'git remote rm origin') after the branches have been created and/or preserved. This is a safety measure
to ensure that changes to the cloned split-target repository cannot be pushed to the origin (if by chance the pipeline
operations cause some corruption or unwanted to state). The default is true.

Example:
```
cloneSplitTargetRepositoriesRemoveRemoteOrigin=true
```

#### patchTargetRepositoryFolder

patchTargetRepositoryFolder is the folder where the projects used to create patches can be found. This value must be set.

Example:
```
patchTargetRepositoryFolder="${splitTargetProjectsRootFolder}"
```

#### patchTargetRepositoryBranch

The patchTargetRepositoryBranch is the branch used as the end of the revision range for creating patches (as in the 'to'
argument in from..to).

Example:
```
patchTargetRepositoryBranch="my-changes-since-the-split-branch-name"
```

#### patchTargetRepositoryBaseBranch

The patchTargetRepositoryBaseBranch is the branch used as the base branch when creating the patchTargetRepositoryBranch,
as in 'git checkout --no-track -b ${patchTargetRepositoryBranch} origin/${patchTargetRepositoryBaseBranch}'.

Example:
```
patchTargetRepositoryBaseBranch="my-split-branch-name"
```

#### createPatchTargetRepositoryBranch

createPatchTargetRepositoryBranch indicates whether or not to create the patchTargetRepositoryBranch in the clone of
the source repositories. The default is false.

Example:
```
createPatchTargetRepositoryBranch=true
```

#### patchesFolderPath

patchesFolderPath is an override of the generated patches folder path used by the RepositoryWorkflow. Set this value
when applying specific patches from a specific folder, usually in conjunction with the task applyPatchesOnly and when
the includeProjectNames value is set to a single project. This value only makes sense when processing multiple projects.
By default this is not set (the value becomes null) and the patches folder is generated.

Example:
```
patchesFolderPath="/tmp/work/patches/patches-for-changes-from-my-split-branch-name-to-my-changes-since-the-split"
```

#### includeProjectNames

includeProjectNames is a comma-separated list of project names to include in the processing. includeProjectNames takes
precedence over excludeProjectNames (so if includeProjectNames is non-empty, then a project must be in the
includeProjectNames and not in the excludeProjectNames to be included in the list). May be empty.

Example:
```
includeProjectNames="split-project-one"
```

#### excludeProjectNames

excludeProjectNames is a comma-separated list of project names to exclude from the processing. includeProjectNames takes
precedence over excludeProjectNames (so if includeProjectNames is non-empty, then a project must be in the
includeProjectNames and not in the excludeProjectNames to be included in the list). May be empty.

Example:
```
excludeProjectNames="split-project-two"
```

### Common operations

Note that these common operations assume that the common parameters as listed above have already been set. The examples
do provide examples of setting the parameters. Operations can be combined and should work in proper order (for example
cloneSourceRepository will happen before cloneRepositories which happens before extractAndPatchProjects).

#### cloneSourceRepository

Clones the source repository into sourceRepositoryWorkingFolder. The repository name is
"${sourceRepositoryNameBase}-${repositoryWorkflowStartingSequenceIndex}". By default this would be
"${sourceRepositoryNameBase}-0".

Example:
```
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="my-source-repository"
autocreateSourceRepositoryWorkingFolder=true
sourceRepositoryGitBranch="my-source-git-branch"
preserveBranchNames="my-preserve-branch-name"
sourceCodeRepositoryServerUrl="git@github.com:MyOrganization/my-repository-name.git"

gradle cloneSourceRepository \
 -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
 -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
 -PautocreateSourceRepositoryWorkingFolder=true \
 -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
 -PpreserveBranchNames="${preserveBranchNames}" \
 -PsourceCodeRepositoryServerUrl="${sourceCodeRepositoryServerUrl}" \
 --stacktrace --info
```

#### cloneSplitTargetRepositories

Clones the split-target repositories into the given folder. The repository names are determined by the project names
found in the `projectParametersJsonFile` file in combination with the includeProjectNames and excludeProjectNames. Note
that the final branch name in preserveBranchNames is the one that is checked out after the repository is cloned.

Example:
```
projectParametersJsonFile="/my/project-parameters.json"
splitTargetProjectsRootFolder="/my/split-repositories-work"
autocreateSplitTargetProjectsRootFolder="true"
preserveBranchNames="master,preserve-branch-one,preserve-branch-two"
cloneSplitTargetRepositoriesRemoveRemoteOrigin=false
includeProjectNames="my-include-project-names-one,my-include-project-names-two"
excludeProjectNames=

gradle cloneSplitTargetRepositories \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder="${autocreateSplitTargetProjectsRootFolder}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PcloneSplitTargetRepositoriesRemoveRemoteOrigin=${cloneSplitTargetRepositoriesRemoveRemoteOrigin} \
  --stacktrace --info
```


#### createSplitTargetRepositories

Creates the split-target repositories from the source repository based on the given parameteres.
This does all RepositoryWorkflow operations and produces a series of repositories.

Example:
```
projectParametersJsonFile="/my/project-parameters.json"
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="my-source-repository"
repositoryWorkflowStartingSequenceIndex="0"
sourceRepositoryGitBranch="my-split-origin"
splitTargetProjectsRootFolder="${sourceRepositoryWorkingFolder}/split-target-repos"
autocreateSplitTargetProjectsRootFolder="true"
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="patch-branch-name"
patchTargetRepositoryBaseBranch="base-branch-name"
createPatchTargetRepositoryBranch=true
preserveBranchNames="base-branch-name"
includeProjectNames=
excludeProjectNames=
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

gradle createSplitTargetRepositories \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  --stacktrace --info
```

#### extractAndPatchProjects

Extracts and patches a given set of projects. This does all RepositoryWorkflow operations.

Example:
```
projectParametersJsonFile="/my/project-parameters.json"
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="my-source-repository"
repositoryWorkflowStartingSequenceIndex="0"
sourceRepositoryGitBranch="my-split-origin"
splitTargetProjectsRootFolder="${sourceRepositoryWorkingFolder}/split-target-repos"
autocreateSplitTargetProjectsRootFolder="true"
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="patch-branch-name"
patchTargetRepositoryBaseBranch="base-branch-name"
createPatchTargetRepositoryBranch=true
preserveBranchNames="base-branch-name"
includeProjectNames=
excludeProjectNames=
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

gradle extractAndPatchProjects \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  --stacktrace --info
```

#### createAndApplyPatchesOnly

Only creates and applies the patches. Does not do the source repository filter-branch and other operations.
Note that the `repositoryWorkflowStartingSequenceIndex` is 3 because that version has been filtered by the
previous steps.

```
projectParametersJsonFile="/my/project-parameters.json"
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="my-source-repository-name"
repositoryWorkflowStartingSequenceIndex="3"
sourceRepositoryGitBranch="my-split-origin"
splitTargetProjectsRootFolder="${sourceRepositoryWorkingFolder}/split-target-repos"
autocreateSplitTargetProjectsRootFolder="true"
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="patch-branch-name"
patchTargetRepositoryBaseBranch="base-branch-name"
createPatchTargetRepositoryBranch=true
preserveBranchNames="base-branch-name"
includeProjectNames=
excludeProjectNames=
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

gradle createAndApplyPatchesOnly \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  --stacktrace --info
```

#### applyPatchesOnly

Only applies the patches. Does not do the source repository filter-branch and other operations. This operation would be
used when there has been some manual editing of the patch files and an attempt is made to apply the patches to
the project repository clone. These manually edited patched would be found in `patchesFolderPath`.
Note that the `repositoryWorkflowStartingSequenceIndex` is 3 because that version has been filtered by the
previous steps.

```
projectParametersJsonFile="/my/project-parameters.json"
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="my-source-repository-name"
repositoryWorkflowStartingSequenceIndex="3"
sourceRepositoryGitBranch="my-split-origin"
splitTargetProjectsRootFolder="${sourceRepositoryWorkingFolder}/split-target-repos"
autocreateSplitTargetProjectsRootFolder="true"
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="patch-branch-name"
patchTargetRepositoryBaseBranch="base-branch-name"
patchesFolderPath="/path/to/my/special/edited/patches/folder"
createPatchTargetRepositoryBranch=true
preserveBranchNames="base-branch-name"
includeProjectNames=
excludeProjectNames=
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

gradle applyPatchesOnly \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PpatchesFolderPath="${patchesFolderPath}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  --stacktrace --info
```

#### applyPatchesOnly in combination with cloneRepositories

When trying to get a sequence of patches to work it's probably best to clone the target project repository and try
to apply the manually edited patches. This operation would be used when there has been some manual editing of the patch
files and an attempt is made to apply the patches to the project repository clone. These manually edited patched would
be found in 'patchesFolderPath'.
Note that the `repositoryWorkflowStartingSequenceIndex` is 3 because that version has been filtered by the
previous steps.

```
projectParametersJsonFile="/my/project-parameters.json"
sourceRepositoryWorkingFolder="/my/repository-split-work"
sourceRepositoryNameBase="my-source-repository-name"
repositoryWorkflowStartingSequenceIndex="3"
sourceRepositoryGitBranch="my-split-origin"
splitTargetProjectsRootFolder="${sourceRepositoryWorkingFolder}/split-target-repos"
autocreateSplitTargetProjectsRootFolder="true"
patchTargetRepositoryFolder="/my/repository-split-work/patch-repositories"
patchTargetRepositoryBranch="patch-branch-name"
patchTargetRepositoryBaseBranch="base-branch-name"
patchesFolderPath="/path/to/my/special/edited/patches/folder"
createPatchTargetRepositoryBranch=true
preserveBranchNames="base-branch-name"
cloneRepositoriesRemoveRemoteOrigin=true
includeProjectNames=
excludeProjectNames=
tempWorkFolder="/my/temp-folder"
reportsFolder="/my/reports-folder"
autocreateReportsFolder="true"

gradle cloneRepositories applyPatchesOnly \
  -PprojectParametersJsonFile="${projectParametersJsonFile}" \
  -PsplitTargetProjectsRootFolder="${splitTargetProjectsRootFolder}" \
  -PautocreateSplitTargetProjectsRootFolder=true \
  -PsourceRepositoryWorkingFolder="${sourceRepositoryWorkingFolder}" \
  -PsourceRepositoryNameBase="${sourceRepositoryNameBase}" \
  -PrepositoryWorkflowStartingSequenceIndex="${repositoryWorkflowStartingSequenceIndex}" \
  -PsourceRepositoryGitBranch="${sourceRepositoryGitBranch}" \
  -PpreserveBranchNames="${preserveBranchNames}" \
  -PpatchTargetRepositoryFolder="${patchTargetRepositoryFolder}" \
  -PpatchTargetRepositoryBranch="${patchTargetRepositoryBranch}" \
  -PpatchTargetRepositoryBaseBranch="${patchTargetRepositoryBaseBranch}" \
  -PpatchesFolderPath="${patchesFolderPath}" \
  -PcreatePatchTargetRepositoryBranch="${createPatchTargetRepositoryBranch}" \
  -PcloneRepositoriesRemoveRemoteOrigin="${cloneRepositoriesRemoveRemoteOrigin}"
  -PincludeProjectNames="${includeProjectNames}" \
  -PexcludeProjectNames="${excludeProjectNames}" \
  -PtempWorkFolder="${tempWorkFolder}" \
  -PreportsFolder="${reportsFolder}" \
  -PautocreateReportsFolder="${autocreateReportsFolder}" \
  --stacktrace --info
```

#### restoreRemoteOriginForProjects

Restores the remote origin for the filtered project list.

#### pushBranchOriginForProjects

Pushes the branch to origin.

#### createMergeBranchForProjects

Creates a merge branch for the filtered project list.

## Versioning

This pipeline does not produce any artifacts.

## Contributors

See git commits to see who contributors are. Issues are tracked through the github issue tracking.

## License

&copy; 2018 National Library of New Zealand. MIT License. All rights reserved.
