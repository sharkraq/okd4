OKD4 install 

1-Create  cofig maninfest after creating nova/install-config.yaml see example below
openshift-install create manifests --dir nova/
2-Edit the scheduler to prevent pods on control plane
sed -i 's/mastersSchedulable: true/mastersSchedulable: False/' nova/manifests/cluster-scheduler-02-config.yml
3-Create ignition config and give 755 permission
openshift-install create ignition-configs --dir nova/ && chmod -R  755 nova/
4-Boot bootstrap nnode and master
coreos-installer install --ignition-url=http://ns1.nova.io:8080/nova/bootstrap.ign /dev/sda --insecure-ignitioin
coreos-installer install --ignition-url=http://ns1.nova.io:8080/nova/master.ign /dev/sda --insecure-ignitioin
coreos-installer install --ignition-url=http://ns1.nova.io:8080/nova/worker.ign /dev/sda --insecure-ignitioin
5-Monitor  install
 openshift-install --dir=nova/ wait-for bootstrap-complete --log-level=info
6-Set oc enviroment
export KUBECONFIG=/var/www/html/nova/auth/kubeconfig
oc whoami
oc get nodes
oc get csr
6-Approve certs may need to do multiple times
oc adm certificate approve `oc get csr|awk -F' ' '{print $1}'|grep -v NAME`
7-Login to console
kubeadmin
cat nova/auth/kubeadmin-password
8-List the nodes and view operator status
oc get nodes
watch -n5 oc get clusteroperators
11-Shutdown  cluster
for node in $(oc get nodes -o jsonpath='{.items[*].metadata.name}'); do oc debug node/${node} -- chroot /host shutdown -h 1; done 





#################

OKD4 install install-config.yaml

apiVersion: v1
baseDomain: nova.io
metadata:
  name: dev

compute:
- hyperthreading: Enabled
  name: worker
  replicas: 2

controlPlane:
  hyperthreading: Enabled
  name: master
  replicas: 3

networking:
  clusterNetwork:
  - cidr: 10.128.0.0/14 
    hostPrefix: 23 
  networkType: OpenShiftSDN
  serviceNetwork: 
  - 172.30.0.0/16

platform:
  none: {}

fips: false

pullSecret: '{"auths":{"fake":{"auth": "bar"}}}' 
sshKey: 'ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQCZObAvCJbpdbh0LjubGzMzktmsGpowEB/PuQOFNvLT0b7ZB7WZtd/zmu+RRac1pTteYUO85cIgZPKpBROkjWT/ThxLEpqDb73z3gByhxPKwzXnJ5lZqlBXZ3yFtpC2KIeqq28kGFecILIeEHRr44b5Xv1DYdeNEA0JiZzieIjlf1M25R6aXbxJbbsZV9anfbDz6l33om7+WZEjrWqNW9B9tu49tlz6VSBeUTWK0wV6A0wf55o3mhWVGHY8raZcVO2SZLEhUMNHfg9qHUAf/hRxlsXvPWuLPXse1YFXnWdUDQgp01GLXnrsox9jE2T6lnJ3qVtfLmKYYnkeOGIGel+YN8kT+XPC/V2s4KAOI0WWm8+irVnZKjGXl/o6SOYEtG/m+Ht3FQTmyA2Imy5vNUbtTJVZ5L68LpUfkq1k0TFgZoKCgezWfZmt7L2Ru05+1s9bwhce7kqs8Nm+KAW84EEhNuuQVJN1gw+5SKIdAE+MoHwo5KUVUVZ1fFIPFr4WJys= root@ns1'






#######Configure OATH Identifier############
1-Create secret object
oc create secret generic github-secret --from-literal=clientSecret=XXXXXXXXXXXXXXXX -n openshift-config


2 Create github-cr.yaml

apiVersion: config.openshift.io/v1
kind: OAuth
metadata:
  name: cluster
spec:
  identityProviders:
  - name: github
    chalenge: false
    login: true
    mappingMethod: claim
    type: GitHub
    github:
      clientID: "b67e673b8f0020df36a3"
      clientSecret:
        name: github-secret
      organizations: 
      - amara-donastia


    oc apply -f github-cr.yaml
