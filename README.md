# Coding Standards Enforcement

## Objective
Maintain code quality between multiple developers without additional work.

## Tools Used
* Composer
    * https://getcomposer.org/
* PHP_CodeSniffer
    * https://github.com/squizlabs/PHP_CodeSniffer
* Git Repository
    *  Pre-Commit Hook
        * https://git-scm.com/docs/githooks
* Bash Emulator (if using Windows)
    * http://gitforwindows.org/

## In Action
If the code does not meet the defined standard, a message will prompt the developer to correct any errors in the offensive file.

```
Checking PHP Lint...
Running Code Sniffer...
E 1 / 1 (100%)
FILE: C:\locallamp\web_root\your-project\src\directory\FileName.php
----------------------------------------------------------------------
FOUND 2 ERRORS AFFECTING 1 LINE
----------------------------------------------------------------------
 133 | ERROR | [x] Expected 1 space after IF keyword; 0 found
 133 | ERROR | [x] Expected 1 space after closing parenthesis; found
     |       |     0
----------------------------------------------------------------------
PHPCBF CAN FIX THE 2 MARKED SNIFF VIOLATIONS AUTOMATICALLY
----------------------------------------------------------------------
```

## Setup
1. Create Git Pre Commit Hook Scripts
    * Create a contribution directory named ```contrib```
    * Create a file named ```setup-hook.sh```
    * Copy/Paste the below code into ```setup-hook.sh```
        ```
        #!/bin/sh
         
        cp contrib/pre-commit .git/hooks/pre-commit
        chmod +x .git/hooks/pre-commit
        ```
    * Within the same directory, create a file named ```pre-commit```
    * Copy/Paste the below code into ```pre-commit```
        ```
        #!/bin/sh
         
        PROJECT=`php -r "echo dirname(dirname(dirname(realpath('$0'))));"`
        STAGED_FILES_CMD=`git diff --cached --name-only --diff-filter=ACMR HEAD | grep \\\\.php`
         
        # Determine if a file list is passed
        if [ "$#" -eq 1 ]
        then
           oIFS=$IFS
           IFS='
           '
           SFILES="$1"
           IFS=$oIFS
        fi
        SFILES=${SFILES:-$STAGED_FILES_CMD}
         
        echo "Checking PHP Lint..."
        for FILE in $SFILES
        do
           php -l -d display_errors=0 $PROJECT/$FILE
           if [ $? != 0 ]
           then
              echo "Fix the error before commit."
              exit 1
           fi
           FILES="$FILES $PROJECT/$FILE"
        done
         
        if [ "$FILES" != "" ]
        then
           echo "Running Code Sniffer..."
           ./vendor/bin/phpcs --standard=vendor/loganconnor44/coding-standard-enforcement/standards/ruleset.xml --encoding=utf-8 -n -p $FILES
           if [ $? != 0 ]
           then
              echo "Fix the error before commit."
              exit 1
           fi
        fi
         
        exit $?
        ```
2. Tell Composer To Run Scripts Upon Installation
    * Copy/Paste the below code into the project's existing ```composer.json```
        ```
        "repositories": [
           {
           "type": "git",
           "url": "git@github.com:LoganConnor44/coding-standard-enforcement.git"
           }
        ],
        "require-dev": {
           "squizlabs/php_codesniffer": "3.*",
           "loganconnor44/coding-standard-enforcement": "dev-master",
        },
        "scripts": {
           "post-install-cmd": [
              "bash contrib/setup-hook.sh"
           ]
        }
        ```
3. Install Composer
    * Install composer by typing ```composer install``` 
        * Windows users - remember to use a Bash emulator

## How It Works
1. Developer Installs Composer
2. PHP_CodeSniffer is brought in as a dependency
3. Composer runs the post script which makes a copy of the ```pre-commit``` file into the local .git files
4. Developer commits and the staged files are checked for standards violations
