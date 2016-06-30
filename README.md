# VstsBuildIssueRepro
Contains files and instructions for reproducing a build issue we encountered in VSTS

(Unfortunately this apparently requires control of the github account, so FORK this repository and use that)

1. Get a local copy of the GitHub repository
** You can clone the repository, or downloading the source Zip.

1. Create a new project to host the build
** Main VSTS Portal - Choose 'New project'
** Give project a name, such as BuildIssueRepro
** Choose version control 'Git'
** Create project (wait while project is created)

2. Upload the sample files
** Open the "Code" page
** Open or clone the project into Visual Studio however you like
** Copy the GitHub project into the folder
** Commit all files

3. Set up the build
** TODO: One step to verify the path
** TODO: The testcloud step with failing wildcard example
