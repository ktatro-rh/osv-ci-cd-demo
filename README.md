# osv-ci-cd-demo

# team-eco-sa-virtualization-cicd

# OpenShift Virtualization Ansible + OpenShift Pipelines CI/CD Demo

These are the steps for setting up the AAP CI/CD Demo on from the Day 2 Operations RHDP Deployment.

1. Login to AAP and create a project using this GitHub Repo or a fork: https://github.com/ktatro-rh/osv-ci-cd-demo  
2. Create Job Templates for the following:  
    
    | Template Name             | Playbook                                  | Survey                     | Extra Variables        |  
    |---------------------------|-------------------------------------------|----------------------------|------------------------|
    | create-namespaces         | playbooks/create-namespaces.yml           | -                          | -                      |
    | install-pipelines         | playbooks/install-openshift-pipelines-operator.yml || -                | -                      |
    | create-pipline            | create-openshift-pipline.yml              | Aap_password, aap_base_url, github_url, workflow_id, base_url, app_base_url, artifact_base_url | -| 
    | create-vm                 | playbooks/create-vm.yml                   | ssh_public_key             | -                      | 
    | validate-deployment       | playbooks/validate-deployment.yml         | -                          | -                      |
    | cutover-vm-and-clean-up   | playbooks/update-vm.yml                   | -                          | -                      |
    | start artifact repo       | playbooks/artifact-repo-state.yml         | -                          | -                      |
    | stop artifact repo        | playbooks/artifact-repo-state.yml         | -                          | -                      |
4. Generate a public / private key (optional) to allow ssh.   
5. Create Workflow Template with Job Type: Run, Inventory: openshift, Extra variables: CHECK PROMPT ON LAUNCH  

    [start artifact repo] -> [create-vm] -> [validate-deployment] -> [cutover-vm-and-clean-up] -> [validate-deployment] -> [stop artifact repo]   

6. Run Playbooks create-namespaces, install-pipelines, create-pipeline. Note github_url is for the node application you will be building, not the Ansible Repo. workflow_id is the workflow id of workflow you created in step 3 (will be in the URL when you view the workflow).  
7. Create the trigger on the pipeline (manual)  
8. Add Webhook to Github source repo and Trigger to the pipeline that was created in the demo-test namespace  
9. Check in code to trigger pipeline (or trigger manually)  


## Playbook/Job Descriptions

### One Shots run from Ansible
create-namespaces  - Run once to create demo-test and demo-prod namespaces  
create-openshift-pipeline  - Run once to create pipeline, custom tasks, aap credentials secret, etc.  
install-openshift-pipelines-operator - Run to install openshift pipelines  
### Called from workflow (initiated by Openshift Pipeline)  
artifact-repo-state - starts or stops the artifact repo  
create-vm - Creates vm in demo-test  
update-vm - does the cutover by updating the service. cleans up temporary resources and old vm.  
validate-deployment  - curls the url until it comes up.  