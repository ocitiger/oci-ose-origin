# oci-ose-origin
openshift origin install on oracle oci

# 一、设置主机名和基本网络
sudo hostnamectl set-hostname  okd.acme.com
sudo yum install NetworkManager.x86_64
#这一步在Oracle Cloud版本的Centos需要执行
#sudo systemctl unmask NetworkManager
#sudo vi /etc/oci-hostname.conf ->2 
sudo systemctl start NetworkManager
sudo systemctl status NetworkManager

# 二、设置互信
ssh-keygen
ssh-copy-id localhost

# 三、安装基础软件
sudo yum update -y; sudo yum install wget git net-tools bind-utils yum-utils iptables-services bridge-utils bash-completion kexec-tools sos psacct -y; sudo reboot
sudo yum install -y epel-release; sudo sed -i 's/enabled=1/enabled=0/' /etc/yum.repos.d/epel.repo
sudo yum install -y --enablerepo=epel ansible pyOpenSSL
sudo yum install -y docker-1.13.1

# 四、获取Openshft3.11的安装包
cd
git clone https://github.com/ocitiger/openshift-ansible
cd openshift-ansible
git checkout release-3.11 


openssl passwd -apr1 Abcd1234
得到 $apr1$oLQGNVUd$lA2kmDJM7ZxgnTBwCxy.R0
第五步会用

# 五、 设置openshif安装需要用到的inventory文件
vi  ~/openshift-ansible/inventory.ini ，以下内容

[OSEv3:children]
masters
nodes
etcd

[OSEv3:vars]
#ansible_ssh_user=root

ansible_ssh_user=opc
ansible_become=true

openshift_deployment_type=origin
openshift_master_identity_providers=[{'name': 'htpasswd_auth', 'login': 'true', 'challenge': 'true', 'kind': 'HTPasswdPasswordIdentityProvider'}]
openshift_master_htpasswd_users={'admin': '$apr1$oLQGNVUd$lA2kmDJM7ZxgnTBwCxy.R0'}

openshift_docker_insecure_registries=172.30.0.0/16

openshift_public_hostname=okd.acme.com
openshift_master_default_subdomain=apps.okd.acme.com
openshift_disable_check=disk_availability,docker_storage,memory_availability,docker_image_availability
openshift_enable_service_catalog=false
[masters]
okd.acme.com openshift_schedulable=true 

[etcd]
okd.acme.com 

[nodes]
okd.acme.com openshift_schedulable=true openshift_node_group_name="node-config-all-in-one"



# 六、执行安装
cd ~/openshift-ansible
ansible-playbook -i ~/inventory.ini playbooks/prerequisites.yml
ansible-playbook -i ~/inventory.ini playbooks/deploy_cluster.yml


# 七、验证安装结果
1）
[opc@okd ~]$ oc get node 
NAME           STATUS    ROLES                  AGE       VERSION
okd.acme.com   Ready     compute,infra,master   11m       v1.11.0+d4cacc0

2）
[opc@okd ~]$ oc get pod -owide
NAME                       READY     STATUS    RESTARTS   AGE       IP           NODE           NOMINATED NODE
docker-registry-1-km89h    1/1       Running   0          8m        10.128.0.4   okd.acme.com   <none>
registry-console-1-2gcbn   1/1       Running   0          8m        10.128.0.6   okd.acme.com   <none>
router-1-dfbqh             1/1       Running   0          8m        10.0.0.9     okd.acme.com   <none>

3）
[opc@okd ~]$ oc login 
Authentication required for https://okd.acme.com:8443 (openshift)
Username: admin
Password: 
Login successful.

You don't have any projects. You can try to create a new project, by running

    oc new-project <projectname>


4）本地浏览器可以访问
在本地hosts文件中加入okd.acme.com的映射，比如
202.xxx.13.1xx okd.acme.com okd

https://okd.acme.com:8443/
用户名admin
密码Abcd1234



# 八、Note
Health Check
/usr/share/ansible/openshift-ansible/playbooks/openshift-checks/pre-install.yml

Node Bootstrap
/usr/share/ansible/openshift-ansible/playbooks/openshift-node/bootstrap.yml

etcd Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-etcd/config.yml

NFS Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-nfs/config.yml

Load Balancer Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-loadbalancer/config.yml

Master Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-master/config.yml

Master Additional Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-master/additional_config.yml

Node Join
/usr/share/ansible/openshift-ansible/playbooks/openshift-node/join.yml

GlusterFS Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-glusterfs/config.yml

Hosted Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-hosted/config.yml

Monitoring Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-monitoring/config.yml

Web Console Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-web-console/config.yml

Metrics Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-metrics/config.yml

Logging Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-logging/config.yml

Prometheus Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-prometheus/config.yml

Availability Monitoring Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-monitor-availability/config.yml

Service Catalog Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-service-catalog/config.yml

Management Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-management/config.yml

Descheduler Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-descheduler/config.yml

Node Problem Detector Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-node-problem-detector/config.yml

Autoheal Install
/usr/share/ansible/openshift-ansible/playbooks/openshift-autoheal/config.yml
