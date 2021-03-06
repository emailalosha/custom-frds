# This requires all the details which is required to build customized frds image

You need to make the base image and setup in a way so that kubernetes can access it locally or the private repo 

## check imagePullPolicy from containers list

       apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: raining-lab-depl-1
            spec:
              replicas: 2
              selector:
                matchLabels:
                  raining-lab-pod: raining-lab-pod-label-1
              template:
                metadata:
                  name: raining-lab-pod
                  labels:
                    raining-lab-pod: raining-lab-pod-label-1
                spec:
                  containers:
                    - name: raining-lab-pod-container-1
                      image: frds:1.0
                      imagePullPolicy: Never
                      ports:
                        - containerPort: 1389
                      env:
                        - name: USE_DEMO_KEYSTORE_AND_PASSWORDS
                          value: "true"
                      command: ["sh","-c"]
                      args: ["start-ds"]

 
### There are couple of things we need to set it up to make the pod running
          
              Environment variables
              ---------------------

              DS containers can be parameterized at run-time using the following environment
              variables:

                  DS_GROUP_ID                         The group ID, which usually identifies
                                                      the data center or region in which the
                                                      server is located. Servers prefer
                                                      connecting to other servers having the
                                                      same group ID. Default: "default".

                  DS_SERVER_ID                        A unique identifier for the server
                                                      which will identify the server within
                                                      a deployment. Default: HOSTNAME.

                  DS_ADVERTISED_LISTEN_ADDRESS        The advertised address(es) which 
                                                      clients should use for connecting 
                                                      to this server. Default: FQDN.

                  DS_BOOTSTRAP_REPLICATION_SERVERS    The addresses of one or more servers 
                                                      within the deployment which this 
                                                      server should connect to in order to 
                                                      discover the rest of the deployment. 
                                                      Default is aligned with Kubernetes
                                                      stateful set naming conventions:
                                                      <statefulset-name>-0.<domain-name>,
                                                      <statefulset-name>-1.<domain-name>

                  DS_SET_UID_ADMIN_AND_MONITOR_PASSWORDS Set to true in order to set the
                                                      admin and monitor passwords before
                                                      intializing or starting the server.

                  DS_USE_PRE_ENCODED_PASSWORDS        By default the admin and monitor
                                                      passwords are provided as plain-text.
                                                      Set this to true if the passwords have
                                                      been pre-encoded externally.

                  DS_UID_ADMIN_PASSWORD               This variable should contain the
                                                      plain-text password or pre-encoded
                                                      password for the uid=admin account.
                                                      If it is not set then fetch the
                                                      password from a file instead.

                  DS_UID_ADMIN_PASSWORD_FILE          This variable should contain the
                                                      name of a file, relative to
                                                      /opt/opendj, containing the plain-text
                                                      password or pre-encoded password for
                                                      the uid=admin account. Defaults to
                                                      secrets/admin.pwd.

                  DS_UID_MONITOR_PASSWORD             This variable should contain the
                                                      plain-text password or pre-encoded
                                                      password for the uid=monitor account.
                                                      If it is not set then fetch the
                                                      password from a file instead.

                  DS_UID_MONITOR_PASSWORD_FILE        This variable should contain the
                                                      name of a file, relative to
                                                      /opt/opendj, containing the plain-text
                                                      password or pre-encoded password for
                                                      the uid=monitor account. Defaults to
                                                      secrets/monitor.pwd.

                  DS_CLUSTER_TOPOLOGY                 An optional comma separated list of
                                                      cluster region names where DJ is
                                                      deployed.
                                                      Used by multi region deployments,
                                                      it is assumed service names in each
                                                      region have suffixes matching the
                                                      region, and are a subdomain of the pod
                                                      FQDN.
                                                      For all pods in a cluster, DS_GROUP_ID
                                                      is changed to the cluster name;
                                                      DS_SERVER_ID is changed to include pod
                                                      and cluster name, for uniqueness
                                                      purposes.
                                                      DS_BOOTSTRAP_REPLICATION_SERVERS is
                                                      also changed to include one pod per
                                                      region cluster.

              Volumes and secrets
              -------------------

              DS containers use the following directories which may be mounted at run-time:

                  /opt/opendj/config          Contains the configuration, including
                                              schema in the "schema" sub-directory.

                                              This directory should be copied out to
                                              development environment when building
                                              custom images (see above).

                  /opt/opendj/data            Contains run-time persisted data, such as
                                              database files and replication changelog
                                              files. This directory will not usually be
                                              mounted in dev environments, where data
                                              is discarded when the container stops.
                                              In production environments this directory
                                              must be mounted from an external persistent
                                              volume.

                  /opt/opendj/secrets         Contains the PKCS12 "keystore" file and its
                                              PIN in "keystore.pin". The keystore file must
                                              contain three secrets, the CA public key with
                                              the alias "ca-cert", the SSL key pair with the
                                              alias "ssl-key-pair" and signed by the CA,
                                              and the RSA master key pair, with the alias 
                                              "master-key", which should be used for 
                                              protecting and distributing any generated 
                                              symmetric keys.

                                              The CA and master key-pair must be the same
                                              on all servers in the deployment.

                                              Note that derived images may have different
                                              requirements in terms of secrets. For example,
                                              the keystore PIN may be obtained from an
                                              environment variable, or the master key may
                                              be obtained from a service like KMS.

              Network ports
              -------------

              DS containers expose the following network ports:

                  4444    The LDAPS administration port.
                  1389    The LDAP port, supporting StartTLS.
                  1636    The LDAPS port.
                  8080    The HTTP port.
                  8443    The HTTPS port.
                  8989    The TLS protected replication port.

              All TLS enabled ports support TLSv1.3 and are protected with the SSL key-pair
              found in the PKCS12 keystore (see the previous section).
         
