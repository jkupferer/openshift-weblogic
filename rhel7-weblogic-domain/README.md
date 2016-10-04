Example of Image with WLS Domain in OpenShift
=============================================

This Dockerfile is an OpenShift adaptation of the sample Oracle WebLogic
empty domain distributed by Oracle. It extends rhel7-weblogic.

The upstream reference can be found here:
https://github.com/oracle/docker-images/tree/master/OracleWebLogic/samples/1221-domain


Image Build Configuration
=========================

This build may be configured using the following environment variables in the
OpenShift BuildConfig:

* ADMIN_PASSWORD  - Default random

* ADMIN_PORT      - Default 8001

* CLUSTER_NAME    - Default DockerCluster

* DEBUG_FLAG      - Default false

* PRODUCTION_MODE - Default prod


Deploying this Application within OpenShift
===========================================

This application can be deployed using the following YAML:

    kind: List
    apiVersion: v1
    items:

    - kind: ImageStream
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-domain
        name: rhel7-weblogic-domain
      spec: {}

    - kind: BuildConfig
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-domain
        name: rhel7-weblogic-domain
      spec:
        output:
          to:
            kind: ImageStreamTag
            name: rhel7-weblogic-domain:latest
        source:
          type: Git
          git:
            uri: https://github.com/jkupferer/openshift-weblogic.git
          contextDir: rhel7-weblogic-domain
        strategy:
          dockerStrategy:
            from:
              kind: ImageStreamTag
              name: rhel7-weblogic:latest
          type: Docker
        triggers:
        - type: ConfigChange
        - type: ImageChange
          imageChange: {}
