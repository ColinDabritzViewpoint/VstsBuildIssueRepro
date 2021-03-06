# VstsBuildIssueRepro
Contains files and instructions for reproducing a release issue we encountered in VSTS.
Note this does NOT appear to affect the script in a build step context, ONLY in a release scenario.

Steps to reproduce:

1. Get a local copy of the GitHub repository
  * You can clone the repository, or download the source Zip.

2. Create a new project to host the build
  * Main VSTS Portal - Choose 'New project'
  * Give project a name, such as BuildIssueRepro
  * Choose version control 'Git'
  * Create project (wait while project is created)

3. Upload the sample files
  * Open the "Code" page
  * Open or clone the project into Visual Studio however you like
  * Copy the GitHub project into the folder
  * Commit all files
  * Verify that the files are present on vsts in the code tab, especially the files:
  * `/SomeProject.iOS/bin/iPhone/UITest/SomeProjectiOS 2016-06-30 09-39-16/SomeProject.ipa`
  * `packages/Xamarin.UITest.1.3.8/tools/test-cloud.exe`
  
4. Set up the build
  * On the 'Build' tab for the project, add a new build
  * Choose 'Empty' for the build definition template (> Next)
  * Choose the TeamProject repo, defaults should be fine, leave agent queue as "Hosted" (> Next)
    * Optional: Set system.debug variable to 'true' - This add extra debug information to logs
  * General build defaults should be fine, under Build, add a step
    * Choose "Copy Publish Aritfact: drop"
	* Optional: Add a task > Test > Xamarin Test Cloud step for comparison

5. Configure the steps
  * Configure the artifact drop step to drop ALL files as artifacts
     * On the 'Copy Publish Artifact: drop' step, add these settings:
	   * Contents: `**/*`
	   * Artifact name: drop
	   * Artifact Type: Server
  * Configure the optional TestCloud step
    * This is optional and is only used to confirm that the step CAN resolve wildcards in a build scenario
	* Configure the following settings:
      * App File: `$(Build.ArtifactStagingDirectory)/drop/SomeProject.iOS/bin/iPhone/**/*.ipa`
        * Team API Key: `Fake-Not-Used-In-Test`
	    * user email: `fakenotusedintest@example.com`
        * Devices: `fakedevices`
        * Test Assembly Directory: `SomeProject.iOS/bin/`

6. Run a build
  * If running a normal build
    * Ensure if you have an optional TestCloud verification step, it is NOT enabled
    * Confirm that the ipa file and test-cloud.exe files are in the artifacts directory
    * Ensure there is a successful build, and note the build number for reference in the release process
  * If running the optional TestCloud verification
    * Ensure the step is set to run
    * Run the build and examine the failiure
    * Note this step will always fail, but the error occurs AFTER the process has resolved the test-cloud.exe and ipa paths.
	* The task log should show the test-cloud.exe command invocation with full valid paths. It should NOT error while resoling these paths.
	* An example successful verification will look like:
```
******************************************************************************
Starting task: Test $(Build.ArtifactStagingDirectory)/drop/SomeProject.iOS/bin/iPhone/**/*.ipa with Xamarin.UITest in Xamarin Test Cloud
******************************************************************************
Set workingFolder to default: C:\LR\MMS\Services\Mms\TaskAgentProvisioner\Tools\agents\1.101.2\tasks\XamarinTestCloud\1.0.23
C:/a/1/s/packages/Xamarin.UITest.1.3.8/tools/test-cloud.exe submit C:/a/1/a/drop/SomeProject.iOS/bin/iPhone/UITest/SomeProjectiOS 2016-06-30 09-39-16/SomeProject.ipa Fake-Not-Used-In-Test --user fakenotusedintest@example.com --devices fakedevices --series master --locale en_US --assembly-dir C:\a\1\s\SomeProject.iOS\bin\ --nunit-xml C:\a\1\s\SomeProject.iOS\bin\xamarintest_13.0.xml
Unable to find the nunit.framework.dll in the assembly directory. In Xamarin Studio you may have to right-click on the nunit.framework reference and choose Local Copy for it to be included in the output directory.
Unable to process logging event:##vso[results.publish type=NUnit;resultFiles=;]
Return code: 1
```

1. Set up the release
  * On the release tab, add a new release > Create release definition, and choose an 'Empty' definition template (Next >)
  * The artifacts settings are correct by default (Source  'Build', Project is your main project, Source (Build definition) = Your test build with artifacts, Queue = Hosted)
  * Click "Create", and under the default environment, click 'Add Task' > Test > Xamarin Test Cloud > Add
  * Close the dialog and enter these TestCloud configuration settings:
    * App File: `$(Agent.ReleaseDirectory)/Issue Repro Build/drop/SomeProject.iOS/bin/iPhone/UITest/**/*.ipa`
	* Team API Key: `Fake-Not-Used-In-Test`
	* user email: `fakenotusedintest@example.com`
	* Devices: `fakedevices`
	* Test Assembly Directory: `SomeProject.iOS/bin/`
  * Save the release
  * Optional: Add verification powershell script
    * Add a Powershell Script task
	* Set type to 'Inline Script'
	* Copy the following inline script: `Resolve-Path SomeProject.iOS\bin\iPhone\UITest\**\*.ipa`
    * Under 'Advanced' set the working directory: `$(System.DefaultWorkingDirectory)/Issue Repro Build/drop`
	
8. Execute the test
  * Confirm successful test build (without optional test step run)
    * Open the build definition from before, and find a successful test run
    * If you cannot find one, ensure that the optional test step is NOT enabled (or is not present), and run a new build
	* Note the build name and number
  * Verify the test build artifacts
    * Open the specific instance of the successful test build
    * On the summary page, open the 'artifacts' tab
    * Explore the 'drop' folder, and verify the following paths are present:
     * drop/packages/Xamarin.UITest.1.3.8/tools/test-cloud.exe
	 * drop/SomeProject.ios/bin/iPhone/UITest/SomeProjectiOS 2016-06-30 09-39-16/SomeProject.ipa
  * Run a release from that build
    * On the 'Release' tab of the project, select the configured release definition, and add a release
	* In the 'Create new release' dialog, choose the build name and number you noted before and click create
  * The release should run automatically, read the logs
    * Open the relase you created, and open the log tab
	* Wait for the release to complete if needed
	* Locate the error message, and compare it to the expected results below
	
9. Compare Test Results
  * As a reminder, in ALL CASES we EXPECT an error message! Please compare the errors
    * Optional note: If you included the verification powershell script note that:
      * The script step passed
      * The script was able to resolve the path
      * The located path starts with the same root directory as the search pattern that fails below.
	  
=== EXPECTED ===
* The release should get past resolving all paths, and should get to the point of executing the test-cloud.exe tool before erroring.
* Errors may resemble:
```
******************************************************************************
Starting task: Test $(Build.ArtifactStagingDirectory)/drop/SomeProject.iOS/bin/iPhone/**/*.ipa with Xamarin.UITest in Xamarin Test Cloud
******************************************************************************
Set workingFolder to default: C:\LR\MMS\Services\Mms\TaskAgentProvisioner\Tools\agents\1.101.2\tasks\XamarinTestCloud\1.0.23
C:/a/1/s/packages/Xamarin.UITest.1.3.8/tools/test-cloud.exe submit C:/a/1/a/drop/SomeProject.iOS/bin/iPhone/UITest/SomeProjectiOS 2016-06-30 09-39-16/SomeProject.ipa Fake-Not-Used-In-Test --user fakenotusedintest@example.com --devices fakedevices --series master --locale en_US --assembly-dir C:\a\1\s\SomeProject.iOS\bin\ --nunit-xml C:\a\1\s\SomeProject.iOS\bin\xamarintest_13.0.xml
Unable to find the nunit.framework.dll in the assembly directory. In Xamarin Studio you may have to right-click on the nunit.framework reference and choose Local Copy for it to be included in the output directory.
Unable to process logging event:##vso[results.publish type=NUnit;resultFiles=;]
Return code: 1
```
=== ACTUAL ===
* The incorrect behavior is an error resolving the path to the ipa file (or test-cloud.exe tool location)
* The step never reaches the point of execuing the test-cloud.exe tool
* Errors look like:
```
2016-06-30T21:33:31.0820561Z ##[error]No matching app files were found with search pattern: C:\a\958d17f14\Issue Repro Build\drop\SomeProject.iOS\bin\iPhone\UITest\**\*.ipa
 
2016-06-30T21:33:31.0830569Z ##[debug]task result: Failed
 
2016-06-30T21:33:31.0840573Z ##[error]Return code: 1
```
