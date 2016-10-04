Example of Image with WLS Domain in OpenShift
=============================================
This Dockerfile is an OpenShift adaptation of the sample Oracle WebLogic
domain distributed by Oracle. It extends rhel7-weblogic-domain.

The upstream reference can be found here:
https://github.com/oracle/docker-images/tree/master/OracleWebLogic/samples/1221-appdeploy

Once deployed, the sample application will be available on port 8001 of
the pod.

Deploying this Application within OpenShift
===========================================

This application can be deployed using `oc new-app` pointing to the git
repository. Alternately you can build it using the following YAML:

    kind: List
    apiVersion: v1
    items:

    - kind: ImageStream
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-sampleapp
        name: rhel7-weblogic-sampleapp
      spec: {}

    - kind: BuildConfig
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-sampleapp
        name: rhel7-weblogic-sampleapp
      spec:
        output:
          to:
            kind: ImageStreamTag
            name: rhel7-weblogic-sampleapp:latest
        source:
          git:
            uri: https://github.com/jkupferer/openshift-weblogic.git
          contextDir: rhel7-weblogic-sampleapp
          type: Git
        strategy:
          dockerStrategy:
            from:
              kind: ImageStreamTag
              name: rhel7-weblogic-domain:latest
          type: Docker
        triggers:
        - type: ConfigChange
        - imageChange: {}
          type: ImageChange

    - kind: DeploymentConfig
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-sampleapp
        name: rhel7-weblogic-sampleapp
      spec:
        replicas: 1
        selector:
          app: rhel7-weblogic-sampleapp
          deploymentconfig: rhel7-weblogic-sampleapp
        strategy:
          resources: {}
        template:
          metadata:
            annotations:
              openshift.io/container.rhel7-weblogic-sampleapp.image.entrypoint: '["startWebLogic.sh"]'
            labels:
              app: rhel7-weblogic-sampleapp
              deploymentconfig: rhel7-weblogic-sampleapp
          spec:
            containers:
            - image: rhel7-weblogic-sampleapp:latest
              name: rhel7-weblogic-sampleapp
              ports:
              - containerPort: 8001
                protocol: TCP
        triggers:
        - type: ConfigChange
        - imageChangeParams:
            automatic: true
            containerNames:
            - rhel7-weblogic-sampleapp
            from:
              kind: ImageStreamTag
              name: rhel7-weblogic-sampleapp:latest
          type: ImageChange

    - kind: Service
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-sampleapp
        name: rhel7-weblogic-sampleapp
      spec:
        ports:
        - name: 8001-tcp
          port: 8001
          protocol: TCP
          targetPort: 8001
        selector:
          app: rhel7-weblogic-sampleapp
          deploymentconfig: rhel7-weblogic-sampleapp
