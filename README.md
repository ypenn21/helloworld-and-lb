# helloworld-and-lb

#setup abm https://github.com/GoogleCloudPlatform/anthos-samples/tree/main/anthos-bm-edge-deployment#5-configure-the-reverse-proxy-to-route-external-traffic-to-abms-bundled-metal-load-balancer

#setup GKE standard


https://cloud.google.com/service-mesh/docs/unified-install/multi-cloud-hybrid-mesh
curl https://storage.googleapis.com/csm-artifacts/asm/asmcli_1.16 > asmcli
chmod +x asmcli
sudo mv asmcli /bin

gcloud container fleet memberships register --enable-workload-identity


gcloud container fleet memberships generate-gateway-rbac      --membership=c2     --role=clusterrole/cluster-admin     --users=admin@yannipeng.altostrat.com     --project=edge-retail-374401     --kubeconfig=~/.kube/config     --context=gke_edge-retail-374401_us-central1-c_c2     --apply

gcloud container fleet memberships register c2 --enable-workload-identity --gke-cluster us-central1-c/c2

#https://cloud.google.com/service-mesh/docs/managed/troubleshoot-managed-anthos-service-mesh
#gke
gcloud container node-pools list --cluster=c2 --region=us-central1-c
gcloud container clusters resize c2 --node-pool default-pool --num-nodes 3 --region us-central1-c
./asmcli install   --project_id edge-retail-374401   --cluster_name c2  --cluster_location us-central1-c   --fleet_id edge-retail-374401   --output_dir /home/admin_/git-projects/asm/c2   --enable_all   --ca mesh_ca

./asmcli install   --project_id edge-retail-374401   --cluster_name autopilot-cluster-1   --cluster_location us-central1   --fleet_id edge-retail-374401   --output_dir /home/admin_/git-projects/asm/asm-autopliot-c1   --enable_cluster_labels   --ca mesh_ca

#auto pilot doesn't allow to install control plane istiod-asm-1162-2-6454d88c8-bgwbl

#hybrid
 asmcli install   --fleet_id edge-retail-374401   --kubeconfig /var/abm-install/kubeconfig/kubeconfig --output_dir ~/asm   --platform multicloud   --enable_all   --ca mesh_ca

asmcli install   --fleet_id edge-retail-374401   --kubeconfig /var/abm-install/kubeconfig/kubeconfig --output_dir ~/asm   --platform multicloud   --enable_cluster_labels   --ca mesh_c


$maybe need to run 
asmcli install   --fleet_id edge-retail-374401   --kubeconfig /var/abm-install/kubeconfig/kubeconfig --output_dir ~/asm   --platform multicloud   --enable_gcp_components   --ca mesh_c

asm/istio/expansion/gen-eastwest-gateway.sh     --mesh proj-11833724782 --network default --revision asm-1162-2 | istioctl --kubeconfig=~/bmctl-workspace/cluster1/cluster1-kubeconfig install -y -f -


asm/istio/expansion/gen-eastwest-gateway.sh \
    --mesh proj-11833724782 \
    --network default \
    --revision asm-1162-2 | \
    istioctl --kubeconfig=~/bmctl-workspace/cluster1/cluster1-kubeconfig install -y -f -



 asm/istio/expansion/gen-eastwest-gateway.sh \
    --mesh proj-11833724782 \
    --network default  \
    --revision asm-1162-2 | \
    istioctl --kubeconfig=./asm_kubeconfig install -y -f -



 ./asmcli create-mesh \
      edge-retail-374401 \
      /home/admin_/git-projects/asm/c2/asm_kubeconfig \
      /home/admin_/git-projects/asm/c2/bm_kubeconfig

 ./asmcli validate \
  --project_id edge-retail-374401 \
  --cluster_name cnuc-1 \
  --cluster_location CLUSTER_LOCATION \
  --fleet_id FLEET_PROJECT_ID \
  --output_dir DIR_PATH

  asmcli validate   --kubeconfig /var/abm-install/kubeconfig/kubeconfig   --fleet_id edge-retail-374401   --output_dir ~/asm/validate   --platform multicloud


    ./asmcli create-mesh \
      edge-retail-374401 \
      /home/admin_/git-projects/asm/c2/asm_kubeconfig \
      ~/.kube/config

kubectl exec -n sample -c sleep \
    "$(kubectl get pod -n sample -l \
    app=sleep -o jsonpath='{.items[0].metadata.name}')" \
    -- /bin/sh -c 'for i in $(seq 1 20); do curl -sS helloworld.sample:5000/hello; done'

kubectl exec -n sample -c sleep \
    "$(kubectl get pod -n sample -l \
    app=sleep -o jsonpath='{.items[0].metadata.name}')" \
    -- /bin/sh -c 'for i in $(seq 1 20); do curl -sS 34.120.74.207/hello; done'


---
apiVersion: v1
kind: Namespace
metadata:
  annotations:
    sidecar.istio.io/inject: "true"
  creationTimestamp: "2023-03-16T04:33:56Z"
  labels:
    istio.io/rev: asm-1162-2
    kubernetes.io/metadata.name: sample
  name: sample
  resourceVersion: "2553652"
  uid: 864a854b-50e2-4207-b34b-497ea1bdf876
spec:
  finalizers:
  - kubernetes
status:
  phase: Active
---
apiVersion: v1
kind: Namespace
metadata:
  creationTimestamp: "2023-03-16T03:28:32Z"
  labels:
    istio.io/rev: asm-1162-2
    kubernetes.io/metadata.name: istio-system
    topology.istio.io/network: edge-retail-374401-default
  name: istio-system
  resourceVersion: "115153"
  uid: 8963a2a7-8fc6-4070-a1a4-f97136c1c948
spec:
  finalizers:
  - kubernetes
status:
  phase: Active



issue:

I got it working but only from anthos bm cluster Hello version: v1 deployed on anthos bm.

If you are running the command curl loop from gke only gets results from gke CTX_2 is gke cluster. Hello v2 deployed on gke


Debugging & checking service discovery:

  ./istioctl validate
  ./istioctl bug-report
  ./istioctl bug-report
  ./istioctl proxy-status
  ./istioctl proxy-config endpoint helloworld-v1-78b9f5c87f-nndtk -n sample | grep hello
  ./istioctl proxy-config endpoint helloworld-v2-79bf565586-pv96g  -n sample | grep hello
  

abm ingress ip for gclb: 34.120.74.207


  566  telnet 10.128.0.37 30810
  567  telnet 10.128.0.37 30811
  568  telnet 10.128.0.37 15443
  569  curl 10.128.0.37:31661/healthz/ready
  570  curl -I 10.128.0.37:31661/healthz/ready
  571  kubetl get svc -n istio-system
  572  kubectl get svc -n istio-system
  573  kubectl edit svc istio-eastwestgateway -n istio-system
  # externalIPs mapped to the gclb load balancer
  externalIPs:
  - 34.149.127.162
  574  kubectl get svc -n istio-system
  575  curl 34.149.127.162:15443



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
