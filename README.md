# Tool Service Template
This is an example hello world tool service that can be used as a template for developing your own tool service for the MDENet Education Platform.

## Running the example

### Start an instance of the education platform
Have an running instance of the education platform by starting the docker demo from [here](https://github.com/mdenet/educationplatform-docker).

### Checkout the repository 
Use either command to clone the repository.

**Via https -**
```
git clone https://github.com/mdenet/educationplatform-tooltemplate.git
```

**Via ssh -**
```
git clone git@github.com:mdenet/educationplatform-tooltemplate.git
```
> Note that for ssh access you must [configure](https://docs.github.com/en/authentication/connecting-to-github-with-ssh) your account with a key.

### Build and run the docker image
This builds the docker images and starts the hello world tool service.

```
cd ./educationplatform-tooltemplate
docker compose up --build
```
> Note it may take approximately 10 minutes the first time the tool service is built.

## Access the example

Once the docker tool service has started the hello world tool service will be available at http://127.0.0.1:8500/ with its function endpoint being http://127.0.0.1:8500/services/RunHelloWorldFunction.

A demo activity will also be available at http://localhost:8083/helloworld/helloworld_activity.json. With a platform instance running on the default port 8080 the hello world activity can be accessed at the following url http://localhost:8080/?activities=http://localhost:8083/helloworld/helloworld_activity.json.

## Stopping the example

To safely stop the example use `ctrl-c`  in the terminal running the platform.

## Building the tool service for development
When developing a tool service, building it locally takes less time than recompiling the docker container for each change and allows an IDE to be used.

Prerequisites:
- [Maven](https://maven.apache.org/)

### Build core dependencies
The MDENet core tool service dependencies *must* be available in the local Maven m2 repository. The following steps build and install these dependencies to the local m2 repository.

Use either command to clone the [platformtools](https://github.com/mdenet/platformtools) repository.

**Via https -**
```
git clone https://github.com/mdenet/platformtools.git
```

**Via ssh -**
```
git clone git@github.com:mdenet/platformtools.git
```

To build the tool service dependencies from the root directory of the platformtool repository run the following commands.

```
cd ./services
mvn clean install -Pall
```

> Note for windows the platformtools tests may fail so add the maven option `-DskipTests` to the build command.




### Build the example
To build the tool service, at the root directory of the educationplatform-tooltemplate repository  run the following commands.

```
cd ./services
mvn clean install -Phelloworld
```


## Modifying the Example

Before modifying the example, [use this template](https://github.com/mdenet/educationplatform-tooltemplate/generate) to create a repository for the new tool service that is populated with the helloworld example files. Ensure that the example builds successfully.

There are two components to an education platform tool, its services and its static configuration files and resources. Additionally, for a tool to be displayed in the platform there needs to be an activity that uses it. This section describes how the helloworld example is structured and how it can be modified to create a new tool. 

> Note - When using Eclipse for development ensure it is set up as detailed in [this section](#eclipse-ide-setup).

The helloworld has the following directory structure. 

```
├── config/
├──helloworld-example/
├──services/
├──static.helloworld/
├──docker-compose.yml
├──package-lock.json
└──package.json
```

**config/** contains web server configuration files used for serving the helloworld activity.

**helloworld-example/** contains the helloworld activity files.

**services/** contains the Java based tool services.

**static.helloworld/** contains the static configuration files and resources.

The `docker-compose.yml` defines two containers one is the helloworld platform tool service and the other is a web server making the activity contents of the `helloworld-example/` available. This file also specifies the ports that tool service and activity are available on and can be changed if the default 8500 and 8083 are unavailable.

The `package.json` lists the location of the npm modules for building the static files and `package-lock.json` is updated by the npm package manager.

The first step in modifying the example is to customise the tool service. 

### Tool Services

The helloworld example is java based. However, any language can be used provided the tool service conforms to the [json http interface](https://github.com/mdenet/educationplatform/wiki/Adding-a-Tool#tool-service).

There are two packages `com.mdenetnetwork.ep.toolfunctions.helloworld` and `com.mdenetnetwork.ep.toolfunctions.helloworldfunction`, the suggested [structure for eclipse based tools](https://github.com/mdenet/educationplatform/wiki/Adding-a-Tool#java-based-implementation). The `helloworld` package's HelloWorldTool class invokes the tool and the `helloworldfunction` package's RunHelloWorldFunction class handles the http json request using the class provided by `helloworld`.


```
services/
├── com.mde-network.ep.toolfunctions.helloworld/
|   ├── META-INF/MANIFEST.MF
|   ├── src/main/java/com/mdenetnetwork/ep/toolfunctions/helloworld
|   |   └── HelloWorld.java
|   ├── target/
|   ├── .classpath
|   ├── .project
|   ├── build.properties
|   ├── mdenettool-helloworld.target
|   └── pom.xml
|
├── com.mde-network.ep.toolfunctions.helloworldfunction/
|   ├── src/
|   |   ├── main/java/com/mdenetnetwork/ep/toolfunctions/helloworldfunction/
|   |   |   ├── RunConversionXToY.java
|   |   |   └── RunHelloWorldFunction.java
|   |   └── test/src/main/java/com/mdenetnetwork/ep/toolfunctions/helloworld/
|   |       └──HelloWorldToolTests.java
|   ├── target/
|   ├── .classpath
|   ├── .project
|   └──pom.xml
|
└── pom.xml

```

The **`helloworld`** package is a maven [tycho](https://tycho.eclipseprojects.io/doc/latest/index.html) eclipse plugin project and has the standard directory structure which includes a MANIFEST file.

> Note - When using the Eclipse IDE for devlopment ensure that the target file is active and being used by Eclipse as shown [here](#using-a-target-file).

The **`helloworldfunction`** is a standard maven java project.

> Note - If the new tool does not have any dependency on eclipse features then the helloworld package can be replaced with a standard maven java project. 

#### Rename the tool
Rename all instances of helloworld to use the new tool name for all file contents, filenames, and directories.


#### Modify the Tool Class 
Modify the tool package helloworld:
- Add tool dependency sources to the `mdenettool-helloworld.target` file. Dependencies in the pom file are ignored for eclipse tycho maven projects so they must be added to the target file. Details on how to manage dependencies of target files using eclipse can be found [here](https://www.vogella.com/tutorials/EclipseTargetPlatform/article.html#exercise-setting-up-a-target-platform-for-eclipse-based-development).
- add required tool dependencies to the `MANIFEST.MF`.
- For HelloWorld.class
    - initialise the new tool as required by its api.
    - parse and load the input parameters that are strings of file contents. 
    - invoke the tool to process the input parameters to obtain an output(s).

Modify the tool package helloworldfunction:
- For `RunHelloWorldFunction.java`
    - Modify input parameters that are used to invoke `HelloWorld.run()`
- For `HelloWorldToolTests`
  - Implement tests that check the output of tool function is as expected.

The second step in modifying the example is to customise the static configuration files and resources.


### Static Configuration Files

The static.helloworld directory contains the tool service's static configuration files, resources. Documentation on the format and expected structure of these files can be found [here](https://github.com/mdenet/educationplatform/wiki/Adding-a-Tool#configuration-file-and-resources). Additionally, there is a dockerfile for creating a tool service container.

The static resources include highlighting rules, icons, and a configuration file. The javascript npm package manager is used to manage building of the highlight rules. 

**`dist/`** following building the project using `npm run build` contains all of the built static configuration files and resources for a tool service.

**`public/`** contains the static files and resources that do not need to be bundled.

**`src/`** contains javascript highlighting rules that are bundled.

The `Dockerfile` configures a web server to make the tool service static files from from the `dist/` directory available. It also runs the tool services using `start.sh`. The web server is configured to proxy requests to the tool services  configured by locations in `nginx.conf.template`. The npm project is configured by `package.json` and `package-lock.json` is updated by the npm package manager. Webpack is used to bundle the javascript files and is configure by`webpack.config.js`. 


```
static.helloworld/
├── dist/
├──public/
|   ├── icons/globe.png
|   ├── helloworld_tool.json
|   └── icons.css
├──src/
|   ├── helloworld.js
|   └── highlighting.js
├──Dockerfile
├──nginx.conf.template
├──package-lock.json
├──start.sh
└──webpack.config.js
```


#### Rename the Static Files

Rename all instances of helloworld to use the new tool name for all file contents, filenames, and directories. In particular, don't forget to update the names in `start.sh` and `Dockerfile` otherwise the tool service will fail to start.


#### Modify the Static files
- Replace icons
    - Replace the `globe.png` in `public/icons/` with icon images of the new tool.
    - Update `public/icons.css` to have entries for each icon added to the icons directory.
- For each language that doesn't already have an existing [ace mode](https://ace.c9.io/#nav=higlighter), copy `src/helloworld.js` and modify the [highlighting rules](https://github.com/mdenet/educationplatform/wiki/Adding-a-Tool#highlighting-rules) of each.
    - Replace the keywords.
    - Add highlighting rules
    - update `highlighting.js` to import the file.
- Modify the `helloworld_tool.json` [configuration file](https://github.com/mdenet/educationplatform/wiki/Adding-a-Tool#configuration-file) for the new tool service.
    - Define the available tool functions.
    - Define the available panels and their layout.


#### Update Container Configuration

If using the included Docker container:
- Define tool service urls for each tool services in the  `nginx.conf.template` file.
- Add entries for each tool service in the `start.sh` file so that the tool services are started by the container.




The final step in modifying the example is to customise the activity.

### Activity 

The activity contains any project files plus an activity configuration file. The  project files an activity has is dependant on the tool used in the activity and what a teacher would like to demonstrate. The helloworld example displays the `input.txt` in a panel that is used as the input to the helloworld tool service. 


```
helloworld-example/helloworld/
├── helloworld_activity.json
└── input.txt
```

The activity file configuration format documentation can be found [here](https://github.com/mdenet/educationplatform/wiki/Creating-an-Activity#activity-configuration)

#### Modify activity configuration
- Update the `tools` entry to include the new tool.
- Add panels to the `panels` object for: 
   - each new tool panel
   - displaying any other required input or outputs
- Update the `layout` object with the panel references so the panels are displayed in the desired positions.
- Update the `actions` object to map panels to use as inputs to tool services on an action button press.

> Note if a new tool has more than one new panel, multiple activities can be defined in a configuration file.


## Eclipse IDE Setup

This section contains notes for setting up the Eclipse IDE for developing tool services.

### Prerequisite Plug-ins

The following plugins are required for developing a tool service using Eclipse. 

 | Name | Version Tested | ID |
 |  ---    |   ---             | ---   |
  Eclipse Modeling Tools  |	4.26.0 | epp.package.modeling	|
  Eclipse Plug-in Development | 3.14 | org.eclipse.pde.feature.group | 
  EMF - Eclipse Modeling Framework SDK |	2.32.0 |	org.eclipse.emf.sdk.feature.group |
  M2E - Maven Integration for Eclipse |	2.1.2 |	org.eclipse.m2e.feature.feature.group |
  M2E - PDE Integration |	2.1.2 |	org.eclipse.m2e.pde.feature.feature.group | 	
 
 Depending on the languages that your tool makes use of additional plugins may be requried to support them, for example, Emfatic. The folloing table shows plugins supporting other languages that have been used previously.

 | Name | Version Tested | ID |
 |  ---    |   ---             | ---   |
  Emfatic |	1.1.0|	org.eclipse.emf.emfatic.feature.group |  
  Epsilon EMF Integration |	2.4.0 |	org.eclipse.epsilon.emf.feature.feature.group |
  Flexmi |	2.4.0 |	org.eclipse.epsilon.flexmi.feature.feature.group |
  OCL Classic SDK: Ecore/UML Parsers,Evaluator,Edit |	5.18.0 |	org.eclipse.ocl.all.sdk.feature.group	Eclipse OCL



### Using a Target File
If you are developing a tool that is Eclipse based and makes used of the `com.mde-network.ep.toolfunctions.helloworld` package that includes a target file, it is important to configure eclipse to use the target file of the of tool being developed. This ensures that the dependencies specified by the target file are used. 

1. From the Eclipse main window with the project open, select from the menu  Window > Preferences. 

2. In preferences panel dialog box that appears under Plug-in Development > Target Platform in the left hand panel, make sure the mdenettool-helloworld target file is active (shown in the screenshot).

> Note - Or select the relevant target for the project you are developing if you have renamed it.

![Preferences dialog showing helloworld target configuration.](/.images/preferences_target.png)


3. Click Apply and close.

> Note - You may have to restart eclipse for the changes to be applied.


