# Hello World - Dart sample

A simple web app written in the [Dart](https://www.dartlang.org) programming language
that you can use for testing. It reads in the env variable `TARGET` and prints
`"Hello $TARGET"`. If `TARGET` is not specified, it will use `"World"` as
`TARGET`.

## Prerequisites

- A working [gcloud](https://cloud.google.com/sdk/gcloud) console

- A [dart-sdk](https://www.dartlang.org/tools/sdk#install) installed and
  configured if you want to run the program locally.

## Recreating the sample code

While you can clone all of the code from this directory, it is useful to know
how to build a hello world Dart application step-by-step. This application can
be created using the following instructions.

1. Create a new directory and write `pubspec.yaml` as follows:

   ```yaml
   name: hello_world_dart
   private: True # let's not accidentally publish this to pub.dartlang.org
   description: >-
     Hello world server example in dart.
   dependencies:
     shelf: ^0.7.3
   environment:
     sdk: ">=2.0.0 <3.0.0"
   ```

2. If you want to run locally, install dependencies. If you only want to run in
   cloud run you can skip this step.

   ```shell
   pub get
   ```

3. Create a new file `bin/main.dart` and write the following code:

   ```dart
   import 'dart:io';

   import 'package:shelf/shelf.dart';
   import 'package:shelf/shelf_io.dart';

   void main() {
     // Find port to listen on from environment variable.
     var port = int.tryParse(Platform.environment['PORT']);

     // Read $TARGET from environment variable.
     var target = Platform.environment['TARGET'] ?? 'World';

     // Create handler.
     var handler = Pipeline().addMiddleware(logRequests()).addHandler((request) {
       return Response.ok('Hello $target');
     });

     // Serve handler on given port.
     serve(handler, InternetAddress.anyIPv4, port).then((server) {
       print('Serving at http://${server.address.host}:${server.port}');
     });
   }
   ```

4. Create a new file named `Dockerfile`, this file defines instructions for
   dockerizing your applications, for dart apps this can be done as follows:

    ```Dockerfile
    # Use Google's official Dart image.
    # https://hub.docker.com/r/google/dart-runtime/
    FROM google/dart-runtime

    # Service must listen to $PORT environment variable.
    # This default value facilitates local development.
    ENV PORT 8080

    # The target variable
    ENV TARGET "World + Dog"
    ```


## Building and deploying the sample

Once you have recreated the sample code files (or used the files in the sample
folder) you're ready to build and deploy the sample app.

1. Build your container image using Cloud Build, by running the following command from 
   directory containing the Dockerfile:
   
   ``` builds submit --tag gcr.io/[PROJECT-ID]/helloworld ```
   
   where [PROJECT-ID] is your GCP project ID. You can get it by running gcloud config get-value project.
   
   Upon success, you will see a SUCCESS message containing the image name (gcr.io/[PROJECT-ID]/helloworld). The image is stored in Container Registry and can be re-used if desired.

2. Deploy the build above using the following command:

   ```gcloud beta run deploy --image gcr.io/[PROJECT-ID]/helloworld```
   
    where [PROJECT-ID] is again your GCP project ID. You can get it by running gcloud 
    config get-value project.

    When prompted, select a region (for example us-central1), confirm the service name, 
    and respond y to allow unauthenticated invocations.

    Then wait a few moments until the deployment is complete. On success, the 
    command line displays the service URL. Visit your deployed container by opening the 
    service URL in a web browser.
    
    Congratulations! You have just deployed an application packaged in a container 
    image to Cloud Run. Cloud Run automatically and horizontally scales your 
    container image to handle the received requests, then scales down when demand 
    decreases. You only pay for the CPU, memory, and networking consumed during 
    request handling. If you change your project you can just re-build and redeploy 
    as you wish.


## Removing the sample app deployment

To remove the built image you can [delete](https://cloud.google.com/container-registry/docs/managing#deleting_images) the image.
You can also delete your GCP project to avoid incurring charges. Deleting your GCP project stops billing for all the resources used within that project.