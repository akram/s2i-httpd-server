# Apache HTTPD Web Server

This repository implements a means of hosting a static web site using the Apache HTTPD web server. It is implemented as a [Source-to-Image](https://github.com/openshift/source-to-image) (S2I) builder and can be used to generate a standalone Docker-formatted image which you can run with any container runtime supporting Docker-formatted images. Alternatively the S2I builder can be used in conjunction with OpenShift to handle both the build and deployment of a static web site.

## Integration With OpenShift

To use this with OpenShift, it is a simple matter of creating a new application within OpenShift, pointing the S2I builder at the Git repository containing your static web site.

As an example, to build and host a simple site with you only need run:

```
oc new-app getwarped/s2i-httpd-server~https://github.com/getwarped/httpd-site-maintenance --name site-maintenance

oc expose svc/site-maintenance
```

To have any changes to your document source automatically redeployed when changes are pushed back up to your Git repository, you can use the [web hooks integration](https://docs.openshift.com/container-platform/latest/dev_guide/builds.html#webhook-triggers) of OpenShift to create a link from your Git repository hosting service back to OpenShift.

The repository used with the builder can host any static files assets, including HTML files, images, style sheets, JavaScript etc. It is possible to include ``.htaccess`` files in directories to control the Apache HTTPD server.

## Creation Using Template

To make it easier to deploy a static web site a template for OpenShift is also included. This can be loaded into your project using:

```
oc create -f https://raw.githubusercontent.com/getwarped/s2i-httpd-server/master/template.json
```

Once loaded, select the ``httpd-server`` template from the web console when wanting to add a new site to the project.

The template details are:

```
Name:		httpd-server
Description:	Apache HTTPD Server
Annotations:	tags=instant-app,httpd

Parameters:
    Name:		APPLICATION_NAME
    Description:	The name of the application.
    Required:		true
    Value:		httpd-server

    Name:		SOURCE_REPOSITORY
    Description:	Git repository for source.
    Required:		true
    Value:		<none>
    
    Name:		SOURCE_DIRECTORY
    Description:	Sub-directory of repository for source files.
    Required:		false
    Value:		<none>

Objects:
    ImageStream		${APPLICATION_NAME}
    ImageStream		${APPLICATION_NAME}-s2i
    BuildConfig		${APPLICATION_NAME}
    DeploymentConfig	${APPLICATION_NAME}
    Service		${APPLICATION_NAME}
    Route		${APPLICATION_NAME}

```

The ``APPLICATION_NAME`` and ``SOURCE_REPOSITORY`` must be specified.

## Standalone Docker Images

To create a standalone Docker-formatted image, you need to [install](https://github.com/openshift/source-to-image/releases) the ``s2i`` program from the Source-to-Image (S2I) project locally. Once you have this installed, you would run within your Git repository:

```
s2i build . getwarped/s2i-httpd-server myhttpdsite
```

In this case this will create a Docker-formatted image called ``myhttpdsite``. You can then run the image using:

```
docker run --rm -p 8080:8080 myhttpdsite
```

You can also change default port by setting the LISTEN_PORT environment variable
```
LISTEN_PORT=9000 docker run --rm -p $LISTEN_PORT:$LISTEN_PORT myhttpdsite
```



