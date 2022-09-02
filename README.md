Using the k6 operator to run a distributed load test in your Kubernetes cluster:
--------------------------------------------------------------------------------

Cloning the k6-operator repository on your kubernetes workstation:
------------------------------------------------------------------

git clone https://github.com/grafana/k6-operator && cd k6-operator


Deploying the operator:
-----------------------

make deploy


Writing our test script:
-------------------------

Refer the test.js file

Extract of the script:

We’ve set up two stages that will run for 30 seconds each. The first one will ramp up linearly to 200 VUs over 30 seconds. The second one will ramp down to 0 again over 30 seconds.

In this case the operator will tell each test runner to run only a portion of the total VUs. For instance, if the script calls for 40 VUs, and parallelism is set to 4, the test runners would have 10 VUs each.

Each VU will then loop over the default function as many times as possible during the execution. It will execute an HTTP GET request against the URL we’ve configured, and make sure that the responds with HTTP Status 200. In a real test, we'd probably throw in a sleep here to emulate the think time of the user, but as the purpose of this article is to run a distributed test with as much throughput as possible, I've deliberately skipped it.

Create a separate namespace:
----------------------------

kubectl create ns k6-load


Deploying our test script:
--------------------------

kubectl create -n k6-load configmap crocodile-stress-test --from-file test.js


Creating our custom resource (CR):
----------------------------------

To communicate with the operator, we’ll use a custom resource called K6. 

In this case, the data of the custom resource contains all the information necessary for k6 operator to be able to start a distributed load test

refer k6-crd-testjs.yaml


Deploy the custom resource (CR):
--------------------------------

kubectl apply -f k6-crd-testjs.yaml


Make sure everything went as expected:
--------------------------------------

kubectl get k6 

