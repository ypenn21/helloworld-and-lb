# helloworld-and-lb

for CTX in ${CTX_1} ${CTX_2}
do
    kubectl create --context=${CTX} namespace sample
    kubectl label --context=${CTX} namespace sample \
        istio-injection- istio.io/rev=REVISION --overwrite
done


Create the HelloWorld service in both clusters:


kubectl create --context=${CTX_1} \
    -f ${SAMPLES_DIR}/samples/helloworld/helloworld.yaml \
    -l service=helloworld -n sample

kubectl create --context=${CTX_2} \
    -f ${SAMPLES_DIR}/samples/helloworld/helloworld.yaml \
    -l service=helloworld -n sample
    
Deploy hello world

kubectl create --context=${CTX_1} \
  -f ${SAMPLES_DIR}/samples/helloworld/helloworld.yaml \
  -l version=v1 -n sample

kubectl create --context=${CTX_2} \
  -f ${SAMPLES_DIR}/samples/helloworld/helloworld.yaml \
  -l version=v2 -n sample
  
for CTX in ${CTX_1} ${CTX_2}
do
    kubectl apply --context=${CTX} \
        -f ${SAMPLES_DIR}/samples/sleep/sleep.yaml -n sample
done

  kubectl get pod --context=${CTX_1} -n sample -l app=sleep
  
  kubectl get pod --context=${CTX_1} -n sample
  
  kubectl exec --context="${CTX_1}" -n sample -c sleep \
    "$(kubectl get pod --context="${CTX_1}" -n sample -l \
    app=sleep -o jsonpath='{.items[0].metadata.name}')" \
    -- /bin/sh -c 'for i in $(seq 1 20); do curl -sS helloworld.sample:5000/hello; done'
