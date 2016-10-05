HTTPD 2.4 with WebLogic module
==============================

This Dockerfile provides an example of building a httpd image to proxy
requests to weblogic using the WebLogic module.

Required Files
==============

This build requires that you host a local copy of the WebLogic installer in
a fileserver within your environment. At the time of this writing the latest
release was fmw_12.2.1.1.0_wlsplugins_Disk1_1of1.zip.

This software may be downloaded from:
http://www.oracle.com/technetwork/middleware/webtier/downloads/index-jsp-156711.html 

Image Build Configuration
=========================

You _must_ specify environment variables for the OpenShift build to obtain
the WebLogic Fusion Middleware software. These are as follows:

* FMW_BASEURL - Base http or https bath from which to download the weblogic zip archive.

* FMW_VERSION - Set to the desired version as matching zip archive (ex: 12.2.1.1.0).

For example, if you have downloaded fmw_12.2.1.1.0_wlsplugins_Disk1_1of1.zip
and made it available on a private local server under
http://fileserv.example.com/weblogic/ then you would set these in your
buildconfig as:

    dockerStrategy:
      env:
      - name: FMW_BASEURL
        value: http://fileserv.example.com/weblogic/
      - name: FMW_VERSION
        value: 12.2.1.1.0

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
          app: rhel7-weblogic-httpd
        name: httpd-24-rhel7
      spec:
        tags:
        - annotations:
            openshift.io/imported-from: registry.access.redhat.com/rhscl/httpd-24-rhel7
          from:
            kind: DockerImage
            name: registry.access.redhat.com/rhscl/httpd-24-rhel7
          importPolicy: {}
          name: latest
    
    - kind: ImageStream
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-httpd
        name: rhel7-weblogic-httpd
      spec: {}
    
    - kind: BuildConfig
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-httpd
        name: rhel7-weblogic-httpd
      spec:
        output:
          to:
            kind: ImageStreamTag
            name: rhel7-weblogic-httpd:latest
        source:
          type: Git
          git:
            uri: https://github.com/jkupferer/openshift-weblogic.git
          contextDir: rhel7-weblogic-httpd
        strategy:
          type: Docker
          dockerStrategy:
            from:
              kind: ImageStreamTag
              name: httpd-24-rhel7:latest
            env:
            - name: FMW_BASEURL
              value: http://fileserv.libvirt:8008/
            - name: FMW_VERSION
              value: 12.2.1.1.0
        triggers:
        - type: ConfigChange
        - type: ImageChange
          imageChange: {}
    
    - kind: DeploymentConfig
      apiVersion: v1
      metadata:
        labels:
          app: rhel7-weblogic-httpd
        name: rhel7-weblogic-httpd
      spec:
        replicas: 1
        selector:
          app: rhel7-weblogic-httpd
          deploymentconfig: rhel7-weblogic-httpd
        strategy:
          resources: {}
        template:
          metadata:
            annotations:
              openshift.io/container.rhel7-weblogic-httpd.image.entrypoint: '["/usr/local/bin/run-httpd24.sh","httpd","-DFOREGROUND"]'
            labels:
              app: rhel7-weblogic-httpd
              deploymentconfig: rhel7-weblogic-httpd
          spec:
            containers:
            - image: rhel7-weblogic-httpd:latest
              name: rhel7-weblogic-httpd
              ports:
              - containerPort: 8080
                protocol: TCP
              volumeMounts:
              - mountPath: /var/log/httpd24
                name: rhel7-weblogic-httpd-log-volume
              - mountPath: /opt/rh/httpd24/root/var/www
                name: rhel7-weblogic-httpd-www-volume
            volumes:
            - emptyDir: {}
              name: rhel7-weblogic-httpd-log-volume
            - emptyDir: {}
              name: rhel7-weblogic-httpd-www-volume
        test: false
        triggers:
        - type: ConfigChange
        - type: ImageChange
          imageChangeParams:
            automatic: true
            containerNames:
            - rhel7-weblogic-httpd
            from:
              kind: ImageStreamTag
              name: rhel7-weblogic-httpd:latest
      status: {}
    
    - apiVersion: v1
      kind: Service
      metadata:
        annotations:
          openshift.io/generated-by: OpenShiftNewApp
        creationTimestamp: null
        labels:
          app: rhel7-weblogic-httpd
        name: rhel7-weblogic-httpd
      spec:
        ports:
        - name: 8080-tcp
          port: 8080
          protocol: TCP
          targetPort: 8080
        selector:
          app: rhel7-weblogic-httpd
          deploymentconfig: rhel7-weblogic-httpd
      status:
        loadBalancer: {}


