# Infrastructure Optimization Capstone
Devops Capstone Project for Caltech

**Project Description**
Create a DevOps infrastructure for an e-commerce application to run on high-availability mode. A popular payment application, EasyPay where users add money to their wallet accounts, faces an issue in its payment success rate. The timeout that occurs with the connectivity of the database has been the reason for the issue. While troubleshooting, it is found that the database server has several downtime instances at irregular intervals. This situation compels the company to create their own infrastructure that runs in high-availability mode. Given that online shopping experiences continue to evolve as per customer expectations, the developers are driven to make their app more reliable, fast, and secure for improving the performance of the current system.

**Tools required**: 
EC2, Kubernetes, Docker, Ansible or Chef or Puppet

**Implementation Requirements**: 
1.	Create the cluster (EC2 instances with load balancer and elastic IP in case of AWS)
2.	Automate the provisioning of an EC2 instance using Ansible or Chef Puppet
3.	Install Docker and Kubernetes on the cluster
4.	Implement the network policies at the database pod to allow ingress traffic from the front-end application pod
5.	Create a new user with permissions to create, list, get, update, and delete pods
6.	Configure application on the pod
7.	Take snapshot of ETCD database
8.	Set criteria such that if the memory of CPU goes beyond 50%, environments automatically get scaled up and configured

**Expected Deliverables**: 
1.	Document the steps and write the algorithms in them.
2.	Submit/share GitHub repository link.
3.	Document the step-by-step process starting from creating test cases, executing them, and recording the results.
4.	Submit the final specification document, which includes:
a.	Project and tester details
b.	Concepts used in the project
c.	Links to the GitHub repository to verify the project completion
d.	Conclusion on enhancing the application and defining the USPs (Unique Selling Points)
