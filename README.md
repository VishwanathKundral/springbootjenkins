# spring-boot-jenkins
- stop whatever process running on the deployed port. 
- delete the files of the previous deploy 
- copy the files to deploy location 
- start application with nohup command, java - jar
- start listening to the server log until it reaches a specific instruction.


```
-- spring-boot

---- dev
------ resources
-------- application.yml
------ initServer.log
------ my-app-jar

---- sit
------ resources
-------- application.yml
------ initServer.log
------ my-app-jar

---- uat
------ resources
-------- application.yml
------ initServer.log
------ my-app-jar
```
* dev - develop, sit - system integration testing, uat - user acceptance testing, application.yml - external app configuration file


```
-- my-project
---- resources
------ application.yml
---- api
------ src
------ (other project file)
------ build.gradle
```

example

```bash

#!/bin/bash

# COMMAND LINE VARIABLES
#enviroment FIRST ARGUMENT 
# Ex: dev | sit | uat
env=$1
# deploy port SECOND ARGUMENT
# Ex: 8090 | 8091 | 8092 
serverPort=$2
# THIRD ARGUMENT project name, deploy folder name and jar name
projectName=$3 #spring-boot
# FOURTH ARGUMENT external config file name
# Ex: application-localhost.yml
configFile=$4


#### CONFIGURABLE VARIABLES ######
#destination absolute path. It must be pre created or you can
# improve this script to create if not exists
destAbsPath=/home/rcoli/Desktop/$projectName/$env
configFolder=resources
##############################################################

#####
##### DONT CHANGE HERE ##############
#jar file
# $WORKSPACE is a jenkins var
sourFile=$WORKSPACE/api/build/libs/$projectName*.jar
destFile=$destAbsPath/$projectName.jar

#config files folder
sourConfigFolder=$WORKSPACE/$configFolder*
destConfigFolder=$destAbsPath/$configFolder

properties=--spring.config.location=$destAbsPath/$configFolder/$configFile

#CONSTANTS
logFile=initServer.log
dstLogFile=$destAbsPath/$logFile
#whatToFind="Started Application in"
whatToFind="Started "
msgLogFileCreated="$logFile created"
msgBuffer="Buffering: "
msgAppStarted="Application Started... exiting buffer!"

### FUNCTIONS
##############
function stopServer(){
    echo " "
    echo "Stoping process on port: $serverPort"
    fuser -n tcp -k $serverPort > redirection &
    echo " "
}

function deleteFiles(){
    echo "Deleting $destFile"
    rm -rf $destFile

    echo "Deleting $destConfigFolder"
    rm -rf $destConfigFolder

    echo "Deleting $dstLogFile"
    rm -rf $dstLogFile
    
    echo " "
}

function copyFiles(){
    echo "Copying files from $sourFile"
    cp $sourFile $destFile

    echo "Copying files from $sourConfigFolder"
    cp -r $sourConfigFolder $destConfigFolder

    echo " "
}

function run(){

   #echo "java -jar $destFile --server.port=$serverPort $properties" | at now + 1 minutes

   nohup nice java -jar $destFile --server.port=$serverPort $properties $> $dstLogFile 2>&1 &

   echo "COMMAND: nohup nice java -jar $destFile --server.port=$serverPort $properties $> $dstLogFile 2>&1 &"

    echo " "
}
function changeFilePermission(){

    echo "Changing File Permission: chmod 777 $destFile"

    chmod 777 $destFile

    echo " "
}   

function watch(){
 
    tail -f $dstLogFile |

        while IFS= read line
            do
                echo "$msgBuffer" "$line"

                if [[ "$line" == *"$whatToFind"* ]]; then
                    echo $msgAppStarted
                    pkill  tail
                fi
        done
}
```

<!-- just updating the readme for testing the Pull Req -->
