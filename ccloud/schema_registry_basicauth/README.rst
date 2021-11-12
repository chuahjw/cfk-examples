Deploy Schema Registry (with Basic Authentication) using managed Confluent Cloud Kafka
================================================================

Adapted from official repo here: https://github.com/confluentinc/confluent-kubernetes-examples/tree/master/hybrid/ccloud-integration

===============================
Deploy Confluent for Kubernetes
===============================

#. Set up the Helm Chart:

   ::

     helm repo add confluentinc https://packages.confluent.io/helm


#. Install Confluent For Kubernetes using Helm:

   ::

     helm upgrade --install operator confluentinc/confluent-for-kubernetes
  
#. Check that the Confluent For Kubernetes pod comes up and is running:

   ::
     
     kubectl get pods


============================
Deploy configuration secrets
============================

Provide secrets according to the encryption and/or authentication mechanism selected.
Refer to: https://docs.confluent.io/operator/current/co-security-overview.html

Provide secrets to connect to Confluent Cloud
::

  kubectl create secret generic cloud-plain \
  --from-file=plain.txt=./creds-client-kafka-sasl-user.txt

The file creds-client-kafka-sasl-user.txt will comprise the following:

::

  username=<api-key>
  password=<api-secret>

Provide Schema Registry users
::

  kubectl create secret generic sr-creds \
  --from-file=basic.txt=./creds-client-sr.txt


=========================
Deploy Confluent Platform
=========================

Edit the ``confluent-platform.yaml`` deployment YAML, and add your respective
Confluent Cloud URLs in the following places:

- ``<cloudKafka_url>``

#. Deploy Confluent Platform with the above configuration:

   ::

     kubectl apply -f confluent-platform.yaml

#. Check that all Confluent Platform resources are deployed:

   ::
   
     kubectl get pods

========
Validate
========

#. Curl Schema Registry load balancer (this should return an Unauthorized error):

   ::

     curl -X GET http://rb-sr.my.domain:80/subjects/

#. Curl Schema Registry load balancer with credentials:

   ::

     curl -u admin-user:admin-password -X GET http://rb-sr.my.domain:80/subjects/

=========
Tear down
=========

::

  kubectl delete -f confluent-platform.yaml

::

  kubectl delete secrets cloud-plain 

::

  helm delete operator
