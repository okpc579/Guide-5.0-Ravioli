## <div id='19-1'/>3.6.	PaaS-TA Container service 설치 및 Prometheus Agent 정보확인

PaaS-TA CaaS 서비스는 Kubernetes Cluster, Workload, Pod 및 Alarm 정보를 수집을 위하여 paasta-container-service에 Prometheus Agent가 설치되어야 한다. 
```
$ cd ~/workspace/paasta-5.0/deployment/service-deployment/paasta-container-service-2.0
```

### <div id='19-2'/>3.6.1	manifests/paasta-container-service-vars-{Cloud Provider}.yml
Deployment YAML에서 사용하는 변수들을 서버 환경에 맞게 수정한다. 

```
# INCEPTION OS USER NAME
inception_os_user_name: "ubuntu"

# RELEASE
caas_projects_release_name: "paasta-container-service-projects-release"
caas_projects_release_version: "1.0"

# IAAS
auth_url: 'http://<IAAS-IP>:5000/v3'
openstack_domain: '<OPENSTACK_DOMAIN>'
openstack_username: '<OPENSTACK_USERNAME>'
openstack_password: '<OPENSTACK_PASSWORD>'
openstack_project_id: '<OPENSTACK_PROJECT_ID>'
region: '<OPENSTACK_REGION>'
ignore-volume-az: true

# STEMCELL
stemcell_os: "ubuntu-trusty"
stemcell_version: "315.36"
stemcell_alias: "trusty"

# VM_TYPE
vm_type_small: "small"
vm_type_small_highmem_16GB: "small-highmem-16GB"
vm_type_caas_small: "small"
vm_type_caas_small_api: "minimal"

# NETWORK
service_private_networks_name: "default"
service_public_networks_name: "vip"

# IPS
caas_master_public_url: "115.68.151.178"   # CAAS-MASTER-PUBLIC-URL
haproxy_public_ips: "115.68.151.177"       # HAPROXY-PUBLIC-URL

# CREDHUB
credhub_server_url: "10.20.0.7:8844"       # Bosh credhub server URL
credhub_admin_client_secret: "<CREDHUB_ADMIN_CLIENT_SECRET>"

# CF
cf_uaa_oauth_uri: "https://uaa.<DOMAIN>"
cf_api_url: "https://api.<DOMAIN>"
cf_uaa_oauth_client_id: "<CF_UAA_OAUTH_CLIENT_ID>"
cf_uaa_oauth_client_secret: "<CF_UAA_OAUTH_CLIENT_SECRET>"

# HAPROXY
haproxy_http_port: 8080
haproxy_azs: [z1]

# MARIADB
mariadb_port: "<MARIADB_PORT>"
mariadb_azs: [z2]
mariadb_persistent_disk_type: "10GB"
mariadb_admin_user_id: "<MARIADB_ADMIN_USER_ID>"
mariadb_admin_user_password: "<MARIADB_ADMIN_USER_PASSWORD>"
mariadb_role_set_administrator_code_name: "Administrator"
mariadb_role_set_administrator_code: "RS0001"
mariadb_role_set_regular_user_code_name: "Regular User"
mariadb_role_set_regular_user_code: "RS0002"
mariadb_role_set_init_user_code_name: "Init User"
mariadb_role_set_init_user_code: "RS0003"

# DASHBOARD
caas_dashboard_instances: 1
caas_dashboard_port: 8091
caas_dashboard_azs: [z3]
caas_dashboard_management_security_enabled: false
caas_dashboard_logging_level: "INFO"

# API
caas_api_instances: 1
caas_api_port: 3333
caas_api_azs: [z1]
caas_api_management_security_enabled: false
caas_api_logging_level: "INFO"

# COMMON API
caas_common_api_instances: 1
caas_common_api_port: 3334
caas_common_api_azs: [z2]
caas_common_api_logging_level: "INFO"

# SERVICE BROKER
caas_service_broker_instances: 1
caas_service_broker_port: 8888
caas_service_broker_azs: [z3]

# (OPTIONAL) PRIVATE IMAGE REPOSITORY
private_image_repository_release_name: "private-image-repository-release"
private_image_repository_release_version: "1.0"
private_image_repository_azs: [z1]
private_image_repository_port: 5000
private_image_repository_root_directory: "/var/lib/docker-registry"
private_image_repository_user_name: "<PRIVATE_IMAGE_REPOSITORY_USER_NAME>"
private_image_repository_user_password: "#################" # cloudfoundry (encoding by Bcrypt)
private_image_repository_public_url: "115.68.151.99"     # PRIVATE-IMAGE-REPOSITORY-PUBLIC-URL
private_image_repository_persistent_disk_type: "10GB"

# ADDON
caas_apply_addons_azs: [z2]

# MASTER
caas_master_backend_port: 8443
caas_master_port: 8443
caas_master_azs: [z3]
caas_master_persistent_disk_type: 5120

# WORKER
caas_worker_instances: 3
caas_worker_azs: [z1,z2,z3]
```

### <div id='19-3'/>3.6.2.	PaaS-TA Container service 설치
Cloud Provider 환경에 맟는 deploy shell 스크립트를 실행한다. 

```
$  ./deploy-{Cloud Provider}.sh
```

### <div id='19-4'/>3.6.3.	PaaS-TA Container service의 설치 완료를 확인한다. 
![PaaSTa_paasta_container_service_vms]

### <div id='19-5'/>3.6.4.	Kubernetes Prometheus Pods 정보를 확인한다.  
```
$  bosh -e {director_name} ssh -d paasta-container-service master
$  /var/vcap/packages/kubernetes/bin/kubectl get pods --all-namespaces -o wide
```
![PaaSTa_paasta_container_service_pods]

### <div id='19-6'/>3.6.5.	prometheus-prometheus-prometheus-oper-prometheus-0 POD의 Node IP를 확인한다.
```
$  /var/vcap/packages/kubernetes/bin/kubectl get nodes -o wide
```
![PaaSTa_paasta_container_service_nodes]

### <div id='19-7'/>3.6.6.	Kubernetes API URL(serverAddress)를 확인한다.
```
$  curl localhost:8080/api
```
![PaaSTa_paasta_container_service_kubernetes_api]

### <div id='19-8'/>3.6.7.	Kubernetes API Request 호출시 Header(Authorization) 인증을 위한 Token값을 확인한다. 
```
$  /var/vcap/packages/kubernetes/bin/kubectl -n kube-system describe secret $(/var/vcap/packages/kubernetes/bin/kubectl -n kube-system get secret | grep monitoring-admin | awk '{print $1}')
```
![PaaSTa_paasta_container_service_kubernetes_token]




[PaaSTa_paasta_container_service_vms]:./images/paasta-container-service-vms.png
[PaaSTa_paasta_container_service_pods]:./images/paasta-container-service-pods.png
[PaaSTa_paasta_container_service_nodes]:./images/paasta-container-service-nodes.png
[PaaSTa_paasta_container_service_kubernetes_api]:./images/paasta-container-service-kubernetes-api.png
[PaaSTa_paasta_container_service_kubernetes_token]:./images/paasta-container-service-kubernetes-token.png
