===================
Testing style guide 
===================

Purpose of testing
=================
The purpose of testing is to minimize the amount of time spent debugging code. 
As new code is introduced into a complex repository regularly, it is important 
to have information on the effects that new features and adjustments have on the outcome of the software. 
We aim to minimize the amount of time spent debugging code by catching bugs early, and isolating bugs as explicitly as possible.

Early detection
------------------------------
In general, the time it takes to detect and fix a bug `grows exponentially <https://deepsource.io/blog/exponential-cost-of-fixing-bugs/>`_ with the amount of time it has existed in the development
workflow. The primary factors of detecting bugs early are code coverage and test runtime. Code coverage should be 80%+ before every 
major release. Besides the metric of code coverage, it it is good practice to create tests that probe functionality at multiple levels of abstraction. 
At the least abstract end of the spectrum we have unittests, which examine a very specific feature or use case in the code. 

`Unittests <https://pylonsproject.org/community-unit-testing-guidelines.html>`_ should have the following characteristics:

1. Tests should be as simple as possible while also exercising the software completely
2. Tests should run as quickly as possible to encourage running them frequently during development
3. Tests should be completely independent and should not couple with other tests or parts of the software they are not responsible for testing

In other words, the main purpose of unittests is to isolate bugs explicitly so that the user can fix the bug quickly. 
They are especially important to run in the beginning of the development process because they confirm that basic features of 
the software meet expectations without being used in tandem with other features.

At the most abstract end of this spectrum, we have integration tests which validate that the modules within the code operate together as expected. 
They specifically target system-level issues. These tests reduce the amount of time it takes to debug the code by providing additional code coverage. 
Although unittests might satisfy the metric of code coverage, integration tests expand coverage by probing common module combinations (such as running a sim and plotting).

`Integration tests <https://www.testingxperts.com/blog/what-is-integration-testing#What%20is%20Integration%20Testing?>`_ should have the following characteristics:

1. Tests should combine multiple features in a way that is common in the user’s workflow
2. Tests should prioritize feature combinations with `high churn and complexity <https://repository.lib.ncsu.edu/bitstream/handle/1840.4/4092/TR-2009-10.pdf?sequence=1#:~:text=Complexity%20metrics%20measure%20the%20structural,occurred%20during%20development%20of%20code>`_
3. Tests should be reasonably fast so that they can be run whenever code is contributed to the repository

If an issue has escaped unittests but is caught by the integration tests then it is likely a systems level issue. 
Rather than standardized and specific output which helps isolate the error, it may be necessary to use developer tools like an IDE’s debugger to further isolate the issue. 
Once the issue is detected, the unittests should be patched to detect this issue again if appropriate.

To maintain a low runtime, in both types of test, we aim to make all tests parallelizable and computationally efficient. This should be kept in mind during code reviews. 
It also the reason why every test suite of either type should be runnable with ``pytest``, which has a wealth of time profiling functionality.

Isolating Bugs
------------------------------
Developers often spend a lot of time probing the software to determine which part of the code produces a bug. 
Tests should reduce this amount of time by providing relevant and interpretable information about an error. 
Unittests should be the developer’s first tool in isolating test failure because they are designed to be as granular as possible. 
In general:

1. Error messages should be sufficient to create a standard bug report with summary of issue, expected result, and actual result

    If someone is reviewing a PR and sees a test failure, they should be able to write a bug report and assign it to the PR owner without having to inspect further

2. An optional verbose mode should provide all information necessary in isolating the bug
    
    The granularity of unittests should make it easy to understand what information is relevant to finding the cause of the issue.

Predicting what information is necessary to isolate a bug is not always straightforward. 
Because of this problem, we recommend the following debug / update workflow:

1.	When an individual writes an entirely new feature, they must either add a unittest or request a unittest to cover the new feature before merging.
2.	In the case of errant code, the tests must produce error messages in GHA that should specify the nature of the test failure, what value is expected, what the actual value is, and ideally some context for the bug (such as a variable’s values leading up to the error). The error message should contain all the information required to write a bug report.
3.	If the user can't determine the source of the bug from the error message, they can run the test locally using command line arguments to specify verbose mode

    ``pytest test_states.py::TestStates::test_gestation -v``

4.	Verbose mode should provide all information that might be relevant for debugging the code. It does this by saving files (or "artifacts") that contain all relevant information in a well-formatted and interpretable file.

    Part of the design structure of the test suite should be to clear out old output files during teardown. Then, if the user runs the suite without verbose on it will delete all the outdated files.

5.	As a last resort, the user should be able to use a debugger with the test, specifically the built-in python debugger package.
6.	If a debugger is ever required to find a bug, the user must specify what information was not made available by the test's verbose mode and make an issue for a test patch. If the failing test is an integration test, the user may suggest an improvement to the error messaging that will help isolate the bug next time
7.	If the information was included in the artifacts left behind by verbose mode but is not easily interpretable, an issue must be made to patch the tests with a better formatted artifact.
8.	The tester (or whoever is in charge of test) must incorporate this new information (or format) into the test's verbose mode in a timely manner.


Concrete requirements
=====================

Running tests
--------------
- Should be runnable with pytest test-runner so that time-profiling and parallel processing are available
- Ability to run tests from command-line with the python command through the __main__ block
- Be able to run with spyder (F5 for that script) and individual tests from a script (F9 on that line in the main block in pytest scripts)

Coverage
--------------
- Should have the option to check coverage for unittests and all tests separately as well as together
- There should be a one-line script to check coverage in parallel like `check_coverage <https://github.com/amath-idm/fp_analyses/blob/master/tests/check_coverage>`_
- Coverage should be 80%+ for all tests at bare minimum, and ideally 90%+

Automated runs
--------------
- GHA should run all tests, finishing in less than 3 minutes
- Should be a one-line script to run all tests in parallel like `run_tests.py <https://github.com/amath-idm/fp_analyses/blob/master/tests/run_tests>`_ finishing in less than 30 seconds

Test design
-----------
- Tests should run efficiently with minimal runtime
- Tests should be as interpretable as possible, generally this means less code but not always
- Anything tests write to disk should be easily removable
- Tests should not output files by default

Unittests

- Each individual test should contain docstring that details what is being tested, how it is tested (what it's being checked against), and the expected value
- Setup should be encapsulated as a function to group together shared configuration resources
- Cleaning the environment (necessary for test independence) and optionally logging relevant output should be encapsulated in a single function
- Must have a test class name the same as the filename (for example test_states.py and class TestStates) if applicable
- Must display error message information that is sufficient to create a bug report (summary, expected value, and actual value)
- Must be able to log all data that is relevant to detecting a bug in the domain of the test case, ideally through an optional verbose mode

Integration tests
- Ease of use with a debugger is top priority
- Must have time profiling for each test script

Compatibility
-----------
- Tests must be easy to run and debug in PyCharm
- Test must be easy to run and debug in VsCode
- All tests and scripts should work in both Windows and Linux
- Should be easy to run with Spyder (F5 for that script)
- Should be easy to run individual tests from a script (F9 on that line in the main block in pytest scripts when using Spyder)

New tests
---------
- New tests should prioritize code with `churn and complexity <https://repository.lib.ncsu.edu/bitstream/handle/1840.4/4092/TR-2009-10.pdf?sequence=1#:~:text=Complexity%20metrics%20measure%20the%20structural,occurred%20during%20development%20of%20code>`_
- Every new feature should have a corresponding unittest in the same PR, ideally by the developer
- Every feature that introduces a new workflow should have a corresponding integration test






