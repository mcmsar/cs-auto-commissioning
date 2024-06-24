# COSPAS/SARSAT automated commissioning scripts
A repository of commissioning scripts that can be run to test COSPAS/SARSAT Mission Control Centers

## Purpose
The aim of this project is to provide a centralized version of MCC commissioning tests that can be run quickly and with a minimized human-in-the-loop element, which should drastically reduce commissioning time, effort, and pain due to the complexity in preparing and running tests by hand, the time it takes to resolve issues when there are test failures due to manual error, configuration error, test definition error, test result expectation error, etc. All MCCs will also have access to the scripts at any point in time so they can understand the requirements they must meet.

## Venus
The Venus application, developed by Techno-Sciences, inc., is an end-to-end testing framework with the ability to put files into an MCC pickup directory and to collect and collate received files from an MCC output directory. It can run a individual tests or a suite of tests at one time, where each step is either a message to be delivered to the MCC or a PowerShell script to be run locally. Portions of the message can be replaced with proper values via special template parameters, such as the SIT-format time or satellites in or out of view of a position, and arbitrary parameters defined in the suite config.xml file.

## Structure
In order to provide the tests in a manner that is easier to keep in harmony, the following structure is suggested.

A parent directory should be maintained for each category of tests, e.g., MCC LGM Commissioning, MCC FGB ELT(DT) Delta Commissioning, MCC SGB Delta Commissioning. Within each general category, a subdirectory should be maintained for tests for nodal MCCs, non-nodal MCCs, and non-nodal CDDR MCCs.

Example:

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
