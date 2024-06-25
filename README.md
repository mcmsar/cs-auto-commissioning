# COSPAS/SARSAT automated commissioning scripts
A repository of commissioning scripts that can be run to test COSPAS/SARSAT Mission Control Centers

## Purpose
The aim of this project is to provide a centralized version of MCC commissioning tests that can be run quickly and with a minimized human-in-the-loop element, which should drastically reduce commissioning time, effort, and pain due to the complexity in preparing and running tests by hand, the time it takes to resolve issues when there are test failures due to manual error, configuration error, test definition error, test result expectation error, etc. All MCCs will also have access to the scripts at any point in time so they can understand the requirements they must meet.

## Venus
The Venus application, developed by Techno-Sciences, inc., is an end-to-end testing framework with the ability to put files into an MCC pickup directory and to collect and collate received files from an MCC output directory. It can run a individual tests or a suite of tests at one time, where each step is either a message to be delivered to the MCC or a PowerShell script to be run locally. Portions of the message can be replaced with proper values via special template parameters, such as the SIT-format time or satellites in or out of view of a position, and arbitrary parameters defined in the suite config.xml file.

## Structure
In order to provide the tests in a manner that is easier to keep in harmony, the following structure is suggested.

A parent directory should be maintained for each category of tests, e.g., MCC LGM Commissioning, MCC FGB ELT(DT) Delta Commissioning, MCC SGB Delta Commissioning. Within each general category, a subdirectory should be maintained for tests for nodal MCCs, non-nodal MCCs, and non-nodal CDDR MCCs. Example:

+ MCC LGM Commissioning
  + Nodal MCCs
    + Test Table
    + Venus Tests
  + Non-Nodal MCCs
    + Test Table
    + Venus Tests
  + Non-Nodal CDDR MCCs
    + Test Table
    + Venus Tests
+ MCC FGB ELT(DT) Delta Commissioning
  + Nodal MCCs
    + Test Table
    + Venus Tests
  + Non-Nodal MCCs
    + Test Table
    + Venus Tests
  + Non-Nodal CDDR MCCs
    + Test Table
    + Venus Tests
+ MCC SGB Delta Commissioning
  + Nodal MCCs
    + Test Table
    + Venus Tests
  + Non-Nodal MCCs
    + Test Table
    + Venus Tests
  + Non-Nodal CDDR MCCs
    + Test Table
    + Venus Tests

A Venus test suite consists of a general config.xml file, a series of XML files in the format <test number>.xml that define the individual tests, and an "in", "out", and "expected" directory with each containing individual subdirectories for each test. Example:

+ Venus Tests
  + config.xml
  + 1.xml
  + 2.xml
  + ...
  + in
    + 1
    + 2
    + ...
  + out
    + 1
    + 2
    + ...
  + expected
    + 1
    + 2
    + ...
   
## Writing Venus test scripts

### Parent test XML files
When creating a new Venus test, first select a new test number (they do not need to be consecutive), then create a corresponding XML file, in, out, and expected directory. **Because GitHub does not support empty directories, store a blank ".gitkeep" file in the out and expected directories.** For example, for test 1, there will be 1.xml, in/1, out/1, out/1/.gitkeep, expected/1, and expected/1/.gitkeep. It is recommended to use a previous or blank XML test for a template.

A Venus test XML file is constructed like this:
```
<test
  name="Test 1: Site confirmation with I7 DOA + GNSS"
  description="Tests that a site with a single I7 incident results in confirmation."
  config="LGM Commissioning nodal MCC">
  <inputs>
    <sit>in\1\1.1.147.txt</sit>                   <!-- I7 MEO in local DDR, local MID -->
    <script delay="601">in\1.2.noop.ps1</script>  <!-- Wait 10 minutes and 1 second -->
    <sit>in\1\1.3.147.txt</sit>                   <!-- I7 MEO same positions -->
    <script delay="300">in\1.4.noop.ps1</script>  <!-- Wait 5 minutes -->
    <sit>in\1\1.5.147.txt</sit>                   <!-- I7 MEO same positions -->
  </inputs>
  <outputs>
    <!-- Step 1: Sw0 + I7 -> Aw7 to "RIP" -->
    <!-- Local RCC(s): SIT 185 INITIAL LOCATED ALERT, contains MCC REFERENCE -->
    <!-- Local RCC(s): SIT 185 NOCR, contains MCC REFERENCE -->

    <!-- Step 2: No messages -->

    <!-- Step 3: Sw7 + I7 -> Ct0 due to DBE = 1 (under 15 minutes) -->

    <!-- Step 4: No messages -->

    <!-- Step 5: Sw7 + I7 -> Ct7 due to DBE = 0 (more than 15 minutes) -->
    <!-- Local RCC(s): SIT 185 POSITION UPDATE ALERT, contains MCC REFERENCE -->
  </outputs>
</test>
```

The name, description, and config attributes in the <test> tag are for reference only. All items betweem <!-- and --> tags are comments, and should be used liberally to document the exact purposes and expected outcomes of the test. The <outputs> section is not currently used by Venus and is for reference only, but it may be used in the future so Venus can analyze actual vs expected outputs.


### Input test messages


### Expected output messages


## Running Venus tests
Venus can be opened by running the Venus.exe file or a shortcut to it. To run a particular test suite, or an individual test within it, you must first update the system configuration (Configuration / System Options menu) and set the Venus Test Working Directory to that of your test suite. The Test Environment should be set to Local. The input message directory is the place where Venus will put messages destined for the DMCC (FTP will be added as an alternative), and the output message directory is the place where Venus will look for messages sent to it by the DMCC. The delay between test steps should be the maximum amount of time that is expected to be needed under normal circumstances for simple cases; individual steps can override this value if longer delays are going to be needed.

All configuration values except the working directory may be overridden by the config.xml in the test suite.

![image](https://github.com/mcmsar/cs-auto-commissioning/assets/19142607/e37d7503-fea7-42fd-b75e-c478ada9e251)
### File / Run Test Directory
Select this step to run the entire suite. Every <number>.xml file will be run in numerical order. They do not need to be consecutive. The working directory will be automatically selected in the list.
### File / Run Test
Select this step to run a single test in the suite.
### File / Run Directory from Selected Test
Deprecated.

## Troubleshooting
As Venus was originally written for internal testing only, it may not provide as many guardrails to alert the user for malformed tests or issues while running.

Many errors will be shown directly in the UI, in the lower half of the screen.

If there appears to be an error that was not shown in the UI, the first place to look is venus.log, located in the working test directory. Many common issues will result in a useful log message there.

However, there still could be cases where the UI seems to not respond and there is no log message.

### Common errors
+ The XML format in config.xml or any individual XML file is broken (e.g.: missing tags, misspelled tags, missing comment closures, etc.).
+ The name/path to a input file is incorrect.
+ The corresponding out or expected file for a test is missing (e.g.: for test 1, out/1 and expected/1 must exist). This could occur if the directories were pushed to github without any files within (solution: put a blank .gitkeep file in any such directories).
