# Setup CI/CD pipeline using Jenkins

## prerequisites

* jenkins

* gradle

* node

* nginx

* git

* jdk


## steps
1. In Jenkins, create a new job by clicking the "New Item"
1. Enter the name and choose "Freestyle project", then click "ok"
1. In "Source Code Management" tab, choose the "Git" and in the input "Repository URL", specify your url as your github repository.
1. In "Build Triggers", choose the "Poll SCM" and input in the "Schedule" as below:
    ```
    H/3 * * * *
    ```
1. Create a directory on your disk to store your artifacts and the launch script, for example: 
    ```
    D:\deploy
    ```
3. In "Build" tab, click "Add build step", int the "Command" input box, add the following command:
    ```
    gradle test
    ```
1. Add a new build step, with the following command:
    ```
    gradle build
    ```
1. Add a new build step, with the following command:
    ```
    copy build\libs\*.jar D:\deploy\
    cd D:\deploy
    run.bat
    ```
1. In directory `D:\deploy`, create a launch scirpt `run.bat`, in the file, add the following scripts:
    ```
    for /f "token=5" %%a in ('netstat -ano ^| findstr "8090"') do set pid=%%s
    if %pid% neq "" if %pid% neq "0" (taskkill /f /pid %pid%)
    start java -jar spring-boot-employee-0.0.1-SNAPSHOT.jar
    ```
1. Click "Save" and then "Build now", you can see the pipeline started to run.

1. For frontend project, the build steps should include:
    ```
    npm config set registry https://registry.npm.taobao.org
    npm install
    npm run test
    npm run build
    xcopy build\* D:\deploy\frontend /e /y
    ```

    
