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
The MDENet core tool service dependencies need to available in the local Maven m2 repository. 

Use either command to clone the platformtools repository.

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


### Build the example
To build the tool service, at the root directory of the educationplatform-tooltemplate repository  run the following commands.

```
cd ./services
mvn clean install -Phelloworld
```



