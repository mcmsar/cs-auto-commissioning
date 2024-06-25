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
The name, description, and config attributes in the `<test>` tag are for reference only. All items betweem <!-- and --> tags are comments, and should be used liberally to document the exact purposes and expected outcomes of the test. The `<outputs>` section is not currently used by Venus and is for reference only, but it may be used in the future so Venus can analyze actual vs expected outputs.

Within the <inputs> section, the following tags are available:
`<sit>`: Takes a text file, replaces any test parameters, and writes the resulting file to the configured input message directory. An option to use FTP to write these messages to remote DMCCs will be added.
`<script>`: Runs a specified PowerShell script.
For both `<sit>` and `<script>`, the "delay" attribute will cause Venus to wait the given number of seconds before completing that step. Any inputs received during the delay period will be attributed to that step.

### Input test messages
Input messages are free-form, except for test parameters that Venus will replace, allowing a generic test to be run with SIT time values updated to be based on the current system time, with satellites chosen based on when positions will be in view, and with beacons and positions and other values selected based on the specific DMCC under test.

#### #CurrentSitDateTimeMinuteAccuracy#
The `#CurrentSitDateTimeMinuteAccuracy#` test parameter is predefined and will be replaced with the current system time in UTC in SIT format `YY DDD HHMM`. Offsets to this time can be specified by adding parentheses at the end of the parameter name with one or more signed integer offset sizes followed by a 'y', 'd', 'h', or 'm'.

Examples:

`#CurrentSitDateTimeMinuteAccuracy(1y2d3h4m)#` will offset the current time by +1 year, +2 days, +3 hours, and +4 minutes.

`#CurrentSitDateTimeMinuteAccuracy(+1d)#` will offset the current time by +1 day.

`#CurrentSitDateTimeMinuteAccuracy(-1h+15m)#` will offset the current time by -1 hour and +15 minutes.

#### #LeoSatelliteInView#, #MeoSatelliteInView#, #LeoSatelliteNotInView#, #MeoSatelliteNotInView#
The `#[LEO/MEO]Satellite[Not]InView#` test parameters are predefined and will be replaced with the three-digit satellite number of the defined type that has the nth-highest elevation, or lowest for NotInView, relative to the location being observed. Each of these parameters take the following arguments (any optional can be omitted, but order does matter):
+ Location being observed (required); format is `([+-]DD.DDD/[+-]DDD.DDD)`; sign can be omitted, precision does not have to be exact
  + Example: `(+30.000/-070.000)`
  + Example: `(1.20/0.0000005)`
+ Time of observation (optional); default is current time; format is SIT format with or without seconds
  + Example: `(24 100 2227)`
  + Example: `(24 100 2227 12.34)`
+ Satellite ranking (optional); default is 1; format is the integer value indicating the nth-best or nth-worst satellite
  + Example: `(1)`
  + Example: `(15)`

The parameters for satellite in/out-of view will be replaced last, so `#CurrentSitDateTimeMinuteAccuracy#` or any user-defined test parameter can be embedded into the satellite parameter and will be included.
+ Example: `#LeoSatelliteInView(20.0/-19.5)#`
+ Example: `#LeoSatelliteNotInView(#LocalLocationTest1#)#`
+ Example: `#MeoSatelliteInView(#LocalLocationTest1#)(#DetectTimeTest1Step1#)#`
+ Example: `#MeoSatelliteNotInView(#LocalLocationTest1#)(#CurrentSitDateTimeMinuteAccuracy(-1h-9m)#)#`
+ Example: `#MeoSatelliteInView(#LocalLocationTest1#)(#CurrentSitDateTimeMinuteAccuracy(-1h-9m)#)#`

#### User-defined test parameters

In the config.xml file, there is a `<TestParameters>` section that defines the generic versions of all the items that might need to be replaced with specific data for the DMCC under test. Template parameters are defined with individual tags, where any instance of two \# symbols surrounding text that matches a name in the `<TestParameters>` section will be replaced with the value currently in config.xml. For example, `<LocalMccCode>4310</LocalMccCode>` will cause all instances of `#LocalMccCode#` in input `<sit>` steps to be replaced with `4310`.

The `<TestParameters>` section may look like this:
```
  <TestParameters>
    <LocalMccCode>4310</LocalMccCode>
    <LocalMccGeolut1Code>4315</LocalMccGeolut1Code>
    <LocalMccMeolut1Code>4317</LocalMccMeolut1Code>

    <NodalMccDdr1Code>3660</NodalMccDdr1Code>
    <NodalMccGeolut1Code>3661</NodalMccGeolut1Code>
    <NodalMccMeolut1Code>3385</NodalMccMeolut1Code>

    <GeoSatelliteOverLocalMcc1>222</GeoSatelliteOverLocalMcc1>

    <!-- Test 1 -->
    <!-- EPIRB Maritime User Protocol with MID of local MCC, MMSI 400589, default location -->
    <Test1Epirb1>5AF451A68260668822FBC000000000</Test1Epirb1>

    <!-- Test 2 -->
    <!-- EPIRB Standard Location Protocol with foreign MID, MMSI 104540, refined location within local MCC SRR -->
    <Test2Epirb1>9DD21985C023D1574FA3B7000005BF</Test2Epirb1>
  </TestParameters>
```
Test parameters should be written in such a way as to maximize their clarity and reusability from the perspective of an MCC that needs to configure the tests for an arbitrary DMCC. It is strongly recommended to describe exactly what the test is doing in comments around the test, and what exactly the user is expected to provide for given configurable parameters. A suggested best-practice is to use the same test definition text above the parameters specific to that test as is used for its entry in the accompanying test table produced by C/S to explain the test plan to DMCCs.

#### Example input test message
```
/00001 00000/#NodalMccDdr1Code#/#CurrentSitDateTimeMinuteAccuracy#
/142/#LocalMccCode#/01
/#NodalMccMeolut1Code#/+99999.9 999.9 +99.99/#CurrentSitDateTimeMinuteAccuracy(-6m)# 12.30
/#CurrentSitDateTimeMinuteAccuracy(-1m)# 12.31/07/FFFE2F#Test1Epirb1#
/12.99/00/01/002
/#MeoSatelliteInView(#LocalLocation#)# #MeoSatelliteInView(#LocalLocation#)(2)# #MeoSatelliteInView(#LocalLocation#)(3)# 000 000 000 000 000 000 000 000 000 000 000 000 000 000
/LASSIT
/ENDMSG
```

### Expected output messages
Expected output messages are useful in comparing the exact message file names and message contents that the HMCC expected to receive from the DMCC. When Venus completes all steps in a test, it will compare all the output message filenames that were received and placed in their corresponding out/n folder with those in the corresponding expected/n folder. Out files are named by the test number, step number, and the name of the facility that sent the message to us, as taken from the original filename. E.g., if a message produced during test 1, step 1 was originally found with the name `DMCC_HMCC_00001.txt`, it would be moved to the out/1 directory with the name `1.1.HMCC.txt`. If it had no corresponding file with the same exact name in the expected/1 directory, it will be shown on the UI as `(+1.1.HMCC)`. If, on the other hand, that file exists in expected/1 and is not received during that test step, it will be shown on the UI as `(-1.1.HMCC)`. If it is received late, during another test step, the numbers may be off, and something like `(-1.1.HMCC) (+1.2.HMCC)` might be displayed, which is a good reason to be permissive with the test step delays. If all output messages were received as expected, Venus will instead just show `Finished`.

The actual contents of the files cannot (currently) be compared with Venus, however there are many applications out there that can do a diff of the directories and provide that information. At TSi, we tend to use [WinMerge](https://winmerge.org/downloads/?lang=en). Compare the out and expected directories in that application, and you can see which files are not identical and what differences there are inside them. This is not quite as accurate when re-using the tests for different DMCCs, and may need more development before it can be very useful here.

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
