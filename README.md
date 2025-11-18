# osv-ci-cd-demo

# team-eco-sa-virtualization-cicd

# OpenShift Virtualization Ansible + OpenShift Pipelines CI/CD Demo

These are the steps for setting up the AAP CI/CD Demo on from the Day 2 Operations RHDP Deployment.

1. Login to AAP and create a project using this GitHub Repo or a fork: https://github.com/ktatro-rh/osv-ci-cd-demo
2. Create Job Templates for the following:
    
    | Template Name             | Playbook                                  | Survey                     | Extra Variables        |  
    |---------------------------|-------------------------------------------|----------------------------|------------------------|
    | create-namespaces         | playbooks/create-namespaces.yaml          | -                          | -                      |
    | install-pipelines         | playbooks/install-openshift-pipelines-operator.yml || -                | -                      |
    | create -pipline           | create-openshift-pipline.yml              | Aap_password, aap_base_url, github_url, workflow_id | -| 
    | create-vm                 | playbooks/                                | -                          | namespace: demo-test   | 
    | validate-deployment       | playbooks/                                | -                          | -                      |
    | cutover-vm-and-clean-up   | playbooks/                                | -                          | -                      |
    | start artifact repo       | playbooks/                                | -                          | -                      |
    | stop artifact repo        | playbooks/                                | -                          | -                      |

3. Create Workflow Template with Job Type: Run, Inventory: openshift, Extra variables: CHECK PROMPT ON LAUNCH

    [start artifact repo] -> [create-vm] -> [validate-deployment] -> [cutover-vm-and-clean-up] -> [validate-deployment] -> [stop artifact repo] 

4. Run Playbooks create-namespaces, install-pipelines, create-pipeline. Note github_url is for the node application you will be building, not the Ansible Repo. workflow_id is the workflow id of workflow you created in step 3 (will be in the URL when you view the workflow).
5. Add Webhook to Github source repo and Trigger to the pipeline that was created in the demo-test namespace
6. Check in code to trigger pipeline (or trigger manually)


## Playbook/Job Descriptions

### One Shots run from Ansible
create-namespaces  - Run once to create demo-test and demo-prod namespaces  
create-pipeline  - Run once to create pipeline, custom tasks, aap credentials secret, etc.
### Called from workflow (initiated by Openshift Pipeline)  
job-deploy-hello-app-test - Creates / Updates deployment, service, and route in demo-test  
job-deploy-hello-app-prod - Creates / Updates deployment, service, and route in demo-prod  
promote-image-to-prod  - Moves a specific image tag from demo-test to demo-prod  