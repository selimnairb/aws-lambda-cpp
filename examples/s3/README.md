# Example using the AWS C++ SDK with Lambda

We'll build a lambda that downloads an image file from S3 and sends it back in the response as Base64 encoded that can be displayed in a web page for example.
To also show case how this can be done on a Linux distro other than Amazon Linux, you can use the Dockerfile in this directory to create an Alpine Linux environment in which you can run the following instructions.

That being said, the instructions below should work on any Linux distribution.

## Create Lambda runtime Docker image in which to build our lambda
Create image:
```bash
docker build -t lambda-env .
```

Run container with current directory on host bound to /build on container:
```bash
docker run --mount type=bind,source="$(pwd)",target=/build -it lambda-env 
```


## Build the AWS C++ SDK
Start by building the SDK from source.
```bash
$ git clone --recurse-submodules https://github.com/aws/aws-sdk-cpp.git
$ cd aws-sdk-cpp
$ mkdir build
$ cd build
$ cmake3 .. -DBUILD_ONLY="s3" \
  -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCUSTOM_MEMORY_MANAGEMENT=OFF \
  -DCMAKE_INSTALL_PREFIX=/usr \
  -DENABLE_UNITY_BUILD=ON

$ make
$ make install
```

## Build the Runtime
Now let's build the C++ Lambda runtime, so in a separate directory clone this repository and follow these steps:

```bash
$ git clone https://github.com/awslabs/aws-lambda-cpp-runtime.git
$ cd aws-lambda-cpp-runtime
$ mkdir build
$ cd build
$ cmake3 .. -DCMAKE_BUILD_TYPE=Release \
  -DBUILD_SHARED_LIBS=OFF \
  -DCMAKE_INSTALL_PREFIX=/usr
$ make
$ make install
```

## Build the application
The last step is to build the Lambda function in `main.cpp` and run the packaging command as follows:

```bash
$ mkdir /tmp/install
$ mkdir build
$ cd build
$ cmake3 .. -DCMAKE_BUILD_TYPE=Release -DCMAKE_PREFIX_PATH=/usr
$ make
$ make aws-lambda-package-encoder
```

You should now have a zip file called `encoder.zip`.

## Upload lambda
Create new:
```bash
aws lambda create-function --function-name demo \
--role arn:aws:iam::?????:role/lambda-demo \
--runtime provided --timeout 15 --memory-size 128 \
--handler demo --zip-file fileb://encoder.zip
```

Upload a new version:
```bash
aws lambda update-function-code --function-name demo \
--zip-file fileb://encoder.zip
```


Run with:
```bash
aws lambda invoke \
--cli-binary-format raw-in-base64-out \
--function-name demo \
--payload '{"s3bucket":"ccom-test-lambda-in","s3key": "helloworld.txt"}' \
output.txt
```
