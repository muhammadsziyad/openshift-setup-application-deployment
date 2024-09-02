# openshift-setup-application-deployment

To create an OpenShift project with Ansible, we will automate the deployment of an application onto an OpenShift cluster. OpenShift, built on Kubernetes, provides additional tools and features, including a web console, CI/CD pipelines, and enhanced security capabilities. For this project, we will use Ansible to:

1. Create a new project in OpenShift.
2. Deploy an application from a Docker image.
3. Expose the application via a Route.

### Prerequisites

1. **OpenShift Cluster**: You need access to an OpenShift cluster. This can be a local OpenShift instance (like Minishift or CRC) or a remote OpenShift cluster.
    
2. **OC CLI (OpenShift Client)**: Ensure you have the OpenShift CLI (`oc`) installed on your machine where Ansible will be run. You can download it from the OpenShift official site.
    
3. **Ansible Installed**: Make sure Ansible is installed on your control node. You can install Ansible using:

```bash
sudo yum install epel-release -y
sudo yum install ansible -y
```

### Step-by-Step Guide to Create an OpenShift Project with Ansible

#### Step 1: Set Up Ansible Environment

1. **Verify Ansible Installation**:

```bash
ansible --version
```

2. **Install OpenShift Ansible Collection**:

- The OpenShift collection allows Ansible to interact with OpenShift clusters.

```bash
ansible-galaxy collection install community.kubernetes
```

3. **Ensure OC CLI is Installed**:

- Verify that `oc` is installed and accessible:

```bash
oc version
```

#### Step 2: Create an Ansible Playbook

1. **Create the Playbook File**:
    
    - Create a file named `openshift_project.yml` with the following content:

```yml
---
- name: OpenShift Project Setup
  hosts: localhost
  gather_facts: no
  tasks:
    - name: Log in to OpenShift
      command: >
        oc login https://openshift-cluster-url:6443
        --username={{ openshift_user }}
        --password={{ openshift_password }}
      register: login_output
      ignore_errors: yes
    
    - name: Create a new OpenShift project
      command: >
        oc new-project {{ project_name }}
      register: create_project_output
      ignore_errors: yes

    - name: Deploy an application from Docker image
      command: >
        oc new-app --name={{ app_name }}
        --docker-image={{ docker_image }}
      register: deploy_app_output
      ignore_errors: yes

    - name: Expose the application via a Route
      command: >
        oc expose svc/{{ app_name }}
      register: expose_route_output
      ignore_errors: yes
```

**Explanation of Playbook**:

- **Log in to OpenShift**: Uses the `oc login` command to log in to the OpenShift cluster.
- **Create a New OpenShift Project**: Creates a new project using the `oc new-project` command.
- **Deploy an Application**: Deploys an application using a Docker image with the `oc new-app` command.
- **Expose the Application**: Exposes the deployed application using a Route, making it accessible externally.

1. **Variables**:
    
    - The playbook uses the following variables:
        - `openshift_user`: OpenShift username.
        - `openshift_password`: OpenShift password.
        - `project_name`: Name of the OpenShift project.
        - `app_name`: Name of the application to deploy.
        - `docker_image`: Docker image to deploy.

#### Step 3: Run Your Ansible Playbook

1. **Create a Variable File**:
    
    - Create a file named `vars.yml` to store the variables:

```yaml
openshift_user: "your_username"
openshift_password: "your_password"
project_name: "example-project"
app_name: "example-app"
docker_image: "docker.io/library/nginx:latest"
```


- Replace the placeholder values with your actual OpenShift credentials and desired application details.
    
2. **Execute the Playbook**:
    
    - Run the playbook using the following command:

```bash
ansible-playbook -i localhost, -e @vars.yml openshift_project.yml
```

- This command tells Ansible to use the `localhost` inventory (as the tasks run on the control node) and pass the variables from `vars.yml`.
    
- **Verify the Deployment**:
    
    - After the playbook completes, you can verify the project and application in the OpenShift web console or by using `oc` commands:

```bash
oc get projects
oc get all -n example-project
```


### Step 4: Clean Up (Optional)

1. **Delete the OpenShift Project**:
	If you want to clean up and delete the project, run:

```bash
oc delete project example-project
```


## Conclusion

This project demonstrates how to use Ansible to automate the setup of an OpenShift project and deployment of an application. You can extend the playbook to include more complex configurations and deployments as needed.
