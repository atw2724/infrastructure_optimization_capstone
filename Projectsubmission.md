# DevOps Capstone Infrastructure Optimization Submission


## Section A: Creation of the Cluster
1.	Log onto AWS practice lab to retrieve Access Key, Secret Key, and Security Token
2.	Open AWS Web Console and Create a Directory
3.	Export Access Key, Secret Key, and Security Token for Terraform to use
```
export AWS_ACCESS_KEY_ID="ASIAZZTBKIIAKXATQNGN"
export AWS_SECRET_ACCESS_KEY="GG9CJdpUWUaQ48DJnKgsdXXPwCL1fHYeRv9CIsUk"
export AWS_SESSION_TOKEN="***************"
echo $AWS_ACCESS_KEY_ID
echo $AWS_SECRET_ACCESS_KEY
echo $AWS_SESSION_TOKEN
```

4.	Create aws.tf to define region
```
    provider "aws" {
    region        = "us-east-1"
    }
```

## Section B: Provisioning Automation Using Terraform

1.	Verify Terraform is available
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform version

Terraform v1.1.6
on linux_amd64

Your version of Terraform is out of date! The latest version
is 1.2.3. You can update by downloading from https://www.terraform.io/downloads.html

```
![image](https://user-images.githubusercontent.com/72522796/175634712-6b921233-2f82-43b2-965d-af8d5f44e3ce.png)

2.	Initialize Terraform

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform init

Initializing the backend...

Initializing provider plugins...
- Finding latest version of hashicorp/aws...
- Installing hashicorp/aws v4.20.0...
- Installed hashicorp/aws v4.20.0 (signed by HashiCorp)

Terraform has created a lock file .terraform.lock.hcl to record the provider
selections it made above. Include this file in your version control repository
so that Terraform can guarantee to make the same selections by default when
you run "terraform init" in the future.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```
![image](https://user-images.githubusercontent.com/72522796/175635064-e5c7694e-e8c4-452e-92ee-2954da1a4837.png)

3.	Create main.tf to define aws instance and keypair resources 

```
cat main.tf

resource "aws_instance" "rhel" {
  ami           = "ami-052efd3df9dad4825"
  instance_type = "t2.micro"
  key_name      = "${aws_key_pair.generated_key.key_name}"
  tags = {
    Name        = "terraform_instance"
  }
}

output "myEC2IP" {
  value = "${aws_instance.ubuntu.public_ip}"
}

resource "tls_private_key" "example" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

resource "aws_key_pair" "generated_key" {
  key_name   = "mykey2"
  public_key = tls_private_key.example.public_key_openssh

provisioner "local-exec" { # Create "myKey.pem" to your computer!!
    command = "echo '${tls_private_key.example.private_key_pem}' > ./myKey.pem"
  }
}
```
![image](https://user-images.githubusercontent.com/72522796/175636886-0568ef95-6524-4aa9-b67f-6d2134fe5ca9.png)

4.	Run Terraform Plan

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform plan
```
![image](https://user-images.githubusercontent.com/72522796/175643556-26553f65-e84e-4aa4-afae-cbb1ac5eb016.png)

5.	Run Terraform Apply to deploy EC2 Instance

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform apply
```
![image](https://user-images.githubusercontent.com/72522796/175644072-e3b4da0d-21e6-4064-9c93-270b629f9750.png)

6.	Verify Instance and Keypair Creation

![image](https://user-images.githubusercontent.com/72522796/175644328-4798d9de-fa46-4466-ba4a-fa8e3f500e83.png)

7.	Update main.tf to add security group

![image](https://user-images.githubusercontent.com/72522796/175644480-b5891ef8-8c08-4e56-88f5-fbb2e992eeaf.png)

8.	Run Terraform Apply and change permissions to verify ability to SSH

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$  chmod 400 myKey.pem
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$  ssh -i myKey.pem ec2-user@44.206.230.44
```
![image](https://user-images.githubusercontent.com/72522796/175644684-62d28b21-f746-4f84-9f34-dfba54994604.png)

9.	Scale the instances by modifying main.tf with updated count, updated name, and updated value

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ vi main.tf
resource "aws_instance" "ubuntu" {
  ami           = "ami-052efd3df9dad4825"
  count = 3
  instance_type = "t2.micro"
  key_name      = "${aws_key_pair.generated_key.key_name}"
  vpc_security_group_ids = [aws_security_group.ab_sg.id]
  tags = {
    Name        ="terraform_instance${count.index+1}"
  }
}

output "myEC2IP" {
  value = "${aws_instance.ubuntu.*.public_ip}"
}

```

10.	Run Terraform Apply again to provision the additional instances

```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ terraform apply
```
![image](https://user-images.githubusercontent.com/72522796/175647097-6a1ee61e-e783-45cb-b48c-4828ab1e1ffa.png)

## Section C: Kubernetes Installation on the Master
1.	Create install.sh
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ vi install.sh
```
![image](https://user-images.githubusercontent.com/72522796/175649953-a9fe376d-77f2-419b-99e7-c838a594da2b.png)

2.	Execute docker install script
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sh install.sh
```
![image](https://user-images.githubusercontent.com/72522796/175651227-7e8b101d-8d9b-4045-948f-3ceaffc65f62.png)

3.	Run commands to enable docker, reload daemon, and restart docker
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sudo systemctl enable docker
Synchronizing state of docker.service with SysV service script with /lib/systemd/systemd-sysv-install.
Executing: /lib/systemd/systemd-sysv-install enable docker
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sudo systemctl daemon-reload
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ 
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sudo systemctl restart docker
```
![image](https://user-images.githubusercontent.com/72522796/175650284-9d020417-1c0a-4d35-9f27-62ad190e2176.png)

4.	Initialize Kubernetes, make home directory, copy configs, and edit ownership
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sudo kubeadm init
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ mkdir -p $HOME/.kube
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
![image](https://user-images.githubusercontent.com/72522796/175650542-8d327598-ec0b-4b61-b711-c73b2ce33b13.png)

5.	Verify Master Node was installed
```
austinwoodngc@ip-172-31-19-140:~/Desktop/infrastructure_optimization_capstone$ kubectl get nodes
NAME                 STATUS     ROLES    AGE   VERSION
master.example.com   NotReady   <none>   63m   v1.23.2
```
![image](https://user-images.githubusercontent.com/72522796/175650805-65561b9a-612a-4cd9-9e58-9243b61b96a6.png)

6.	Generate the token join command
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ sudo kubeadm token create --print-join-command
kubeadm join 172.31.19.140:6443 --token hois6m.yxj0xrtzfkz3vy18 --discovery-token-ca-cert-hash sha256:4b2eed87c88da2ca083b060293152987530d93259887dc7ead3c020ccbf6b648 
```
![image](https://user-images.githubusercontent.com/72522796/175680614-6062afed-683a-4acb-9588-71a5418244e9.png)

7.	Change hostnames of both the instances
```
ubuntu@ip-172-31-84-62:~$ sudo hostnamectl set-hostname node1.example.com
ubuntu@ip-172-31-81-125:~$ sudo hostnamectl set-hostname node2.example.com
```
![image](https://user-images.githubusercontent.com/72522796/175657848-3b406a2a-5cfd-4fdf-934b-eaf45c29135b.png)
![image](https://user-images.githubusercontent.com/72522796/175657875-e96249fd-2e75-44f8-9d02-2a64271ab914.png)
![image](https://user-images.githubusercontent.com/72522796/175667579-f001e261-c905-44f2-96f9-325049c18acd.png)

8. Create and run the node.sh on Node1 and Node2

![image](https://user-images.githubusercontent.com/72522796/175664956-1ed4b727-72be-467b-a283-934768e98500.png)

9.	Access the AWS console and allows TCP traffic within the subnet

![image](https://user-images.githubusercontent.com/72522796/175664819-a8cc1f6d-9f01-4503-8b46-0a18f91f896d.png)

10.	On each node, run this command to join both Node1 and Node 2 to the Master
```
sudo kubeadm join 172.31.19.140:6443 --token hois6m.yxj0xrtzfkz3vy18 --discovery-token-ca-cert-hash sha256:4b2eed87c88da2ca083b060293152987530d93259887dc7ead3c020ccbf6b648
```
11. Verify Kubernetes Installation on the master
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl get nodes
NAME                 STATUS     ROLES                  AGE     VERSION
master.example.com   NotReady   control-plane,master   3h27m   v1.23.2
node1.example.com    NotReady   <none>                 4m15s   v1.23.6
node2.example.com    NotReady   <none>                 85s     v1.23.6
``` 
11. Create an overlay network for cluster communications
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')"
serviceaccount/weave-net created
clusterrole.rbac.authorization.k8s.io/weave-net created
clusterrolebinding.rbac.authorization.k8s.io/weave-net created
role.rbac.authorization.k8s.io/weave-net created
rolebinding.rbac.authorization.k8s.io/weave-net created
daemonset.apps/weave-net created
```
![image](https://user-images.githubusercontent.com/72522796/175697154-47cdaf44-c56a-4d06-aea1-f6f22a430aeb.png)

11. Create a multi-tier application
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ vi mydb.yml
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl create -f mydb.yml
deployment.apps/mydb created
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl expose deployment mydb --port=3306
service/mydb exposed
```
![image](https://user-images.githubusercontent.com/72522796/175697427-960932da-8831-4bac-b306-be8d9210218b.png)

11. Create deployment apps and run commands to expose deployment
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ vi wp.yml
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl create -f wp.yml
deployment.apps/wp created
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl expose deployment wp --port=80 --type=NodePort
service/wp exposed
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl get pods
NAME                    READY   STATUS   RESTARTS   AGE
mydb-659c7949cd-zl6gr   1/1     Running   0          6m16s
wp-946c66d98-hrkxx      1/1     Running   0          52s
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl get svc
NAME         TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        3h50m
mydb         ClusterIP   10.97.231.121   <none>        3306/TCP       6m9s
wp           NodePort    10.99.52.76     <none>        80:32448/TCP   18s
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl get nodes -o wide
NAME                 STATUS   ROLES    AGE     VERSION   INTERNAL-IP     EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION    CONTAINER-RUNTIME
master.example.com   Ready    <none>   3h51m   v1.23.2   172.31.19.140   <none>        Ubuntu 20.04.3 LTS   5.11.0-1027-aws   docker://20.10.17
```

## Section D: Implementation of Network Policies
1.	Create network policy
```austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ vi np.yml
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl create -f np.yml
networkpolicy.networking.k8s.io/api-allow created
```
![image](https://user-images.githubusercontent.com/72522796/175698355-843d869e-ead5-4696-ac29-38719f4826a2.png)

## Section E: Creation of New User Permissions
1.	Create Users and ACL RBAC
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl create serviceaccount newroleadded
serviceaccount/newroleadded created
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl create clusterrole newroleadded --verb=get --verb=list --verb=create --verb=update --resource=pods
clusterrole.rbac.authorization.k8s.io/newroleadded created
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl create clusterrolebinding newroleadded --serviceaccount=default:newroleadded --clusterrole=newroleadded
clusterrolebinding.rbac.authorization.k8s.io/newroleadded created
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ TOKEN=$(kubectl describe secrets "$(kubectl describe serviceaccount newroleadded | grep -i Tokens | awk '{print $2}')" | grep token: | awk '{print $2}')
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl config set-credentials myuser1 --token=$TOKEN
User "myuser1" set.
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl config set-context newcontextadded --cluster=kubernetes --user=myuser1
Context "newcontextadded" created.
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl config use-context newcontextadded
Switched to context "newcontextadded".
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl get all
```
![image](https://user-images.githubusercontent.com/72522796/175698794-ce09c481-2c19-4eeb-9ebe-83bb70acff4c.png)

## Section F: Application Configuration on the Pod
1.	Configure applications on the Pod
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl auth can-i get pods --all-namespaces
yes
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ 
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl auth can-i get pods --all-namespaces
yes
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl auth can-i get deployment --all-namespaces
no
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl config use-context kubernetes-admin@kubernetes
Switched to context "kubernetes-admin@kubernetes".
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl auth can-i get deployment --all-namespaces
yes
```
![image](https://user-images.githubusercontent.com/72522796/175699009-aefc2c3c-9400-45c1-9764-4f15cb20ca4b.png)

## Section G: ETCD Database Snapshot
1.	Take a snapshot/backup of the etcd database
```
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ sudo apt update -y
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ sudo apt install etcd-client
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ hostname -i
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ kubectl get nodes -o wide
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ export advertise_url=172.31.19.140:2379
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ echo $advertise_url
https://172.31.19.140:2379
austinwoodngc@master:~/Desktop/infrastructure_optimization_capstone$ sudo ETCDCTL_API=3 etcdctl --endpoints $advertise_url --cacert /etc/kubernetes/pki/etcd/ca.crt --key /etc/kubernetes/pki/etcd/server.key --cert /etc/kubernetes/pki/etcd/server.crt snapshot save test1.db
Snapshot saved at test1.db
```
![image](https://user-images.githubusercontent.com/72522796/175713494-4791a8a9-28d2-4bf4-8ecb-a4c5c7f31409.png)

## Section H: Configuration of CPU Memory Environment Scaling
1.	TBD

## Section I:  Conclusion
1.	TBD
