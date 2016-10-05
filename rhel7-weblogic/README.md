WebLogic base Image
===================

This Dockerfile will build a base image with WebLogic. It extends the
rhel7-java-180-oracle image, which is a basic RHEL 7 image with Red Hat
distributed oracle java packages installed.

This is an adaptation of the Dockerfile provided by Oracle here:
https://github.com/oracle/docker-images/tree/master/OracleWebLogic/dockerfiles/


Required Files
==============

This build requires that you host a local copy of the WebLogic installer in
a fileserver within your environment. You can use either the quick or generic
installers.

For example, the generic installer: fmw_12.2.1.1.0_wls_Disk1_1of1.zip 

Or for quick installer: fmw_12.2.1.1.0_wls_quick_Disk1_1of1.zip 

These may be downloaded from:
http://www.oracle.com/technetwork/middleware/weblogic/downloads/wls-for-dev-1703574.html 

Image Build Configuration
=========================

You _must_ specify a few environment variables for the OpenShift build to obtain
the WebLogic Fusion Middleware software. These are as follows:

* FMW_BASEURL - Base http or https bath from which to download the weblogic zip archive.

* FMW_VERSION - Set to the desired version as matching zip archive (ex: 12.2.1.1.0).

* FMW_QUICK   - If set then the quick/developer version of weblogic will be used.

For example, if you have downloaded the quick installer, fmw_12.2.1.1.0_wls_quick_Disk1_1of1.zip
and made it available on a private local server under http://fileserv.example.com/weblogic/ then
you would set these in your buildconfig as:

    dockerStrategy:
      env:
      - name: FMW_BASEURL
        value: http://fileserv.example.com/weblogic/
      - name: FMW_VERSION
        value: 12.2.1.1.0
      - name: FMW_QUICK
        value: true

Deploying this Application within OpenShift
===========================================

This image builder can be deployed using YAML as shown below. Be certain to
customize the the environment variables as described above:

    kind: List
    apiVersion: v1
    items:

    - kind: ImageStream
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic
        name: rhel7-weblogic
      spec: {}

    - kind: BuildConfig
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic
        name: rhel7-weblogic
      spec:
        output:
          to:
            kind: ImageStreamTag
            name: rhel7-weblogic:latest
        source:
          type: Git
          git:
            uri: https://github.com/jkupferer/openshift-weblogic.git
          contextDir: rhel7-weblogic
        strategy:
          dockerStrategy:
            env:
            - name: FMW_BASEURL
              value: http://fileserv.example.com/weblogic/
            - name: FMW_VERSION
              value: 12.2.1.1.0
            - name: FMW_QUICK
              value: "True"
            from:
              kind: ImageStreamTag
              name: rhel7-java-180-oracle:latest
          type: Docker
        triggers:
        - type: ConfigChange
        - type: ImageChange
          imageChange: {}
