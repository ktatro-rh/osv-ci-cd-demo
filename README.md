# osv-ci-cd-demo

# team-eco-sa-virtualization-cicd

# Ansible CI/CD Demo

These are the steps for setting up the AAP CI/CD Demo on a fresh OpenShift 4.x install.

1. Install AAP Operator
2. Install Openshift Pipelines Operator
3. Create AutomationController instance named automationcontroller
4. Follow route (aap namespace -> routes) to newly created automation controller instance
5. Login AAP as admin using credentials in secret automationcontroller-admin-password in namespace aap
6. Subscribe AAP Instance
7. Create ansible-sa service account in aap namespace (oc apply -f ./openshift-resources/openshift-ansible-sa.yaml) (note this gives the cluster admin privileges, if doing this in a non-testing environment, you'll want to limit the scope of this users access)
8. In AAP, create credential with type OpenShift or Kubernetes API Endpoint using the ansible service account token you just created and the cluster API URL (ensure there are no trailing spaces in the API URL)
9. Create Project using this Github Repo (or your fork) https://github.com/ktatro-rh/ansible-ci-cd-demo
10. Create inventory openshift with host: {'ansible_host': '127.0.0.1', 'ansible_connection': 'local'}
11. Create Job Templates with Job Type: Run, Inventory: openshift, Project ktatro-demo, Default Execution Environment, Kubernetes Bearer Token
    
    | Template Name             | Playbook                                  | Survey                     | Extra Variables        |  
    |---------------------------|-------------------------------------------|----------------------------|------------------------|
    | create-namespaces         | playbooks/create-namespaces.yaml          | -                          | -                      |
    | create-pipeline           | playbooks/create-openshift-pipeline       | aap_password, aap_base_url, github_url, workflow_id | -                      | 
    | job-deploy-hello-app-prod | playbooks/hello-app-deploy.yaml           | -                          | namespace: demo-prod   | 
    | job-deploy-hello-app-test | playbooks/hello-app-deploy.yaml           | -                          | namespace: demo-test   | 
    | promote-image-to-prod     | playbooks/hello-app-promote-image.yaml    | -                          | CHECK PROMPT ON LAUNCH |

12. Create Workflow Template with Job Type: Run, Inventory: openshift, Extra variables: CHECK PROMPT ON LAUNCH

    [job-deploy-hello-app-test] -> [Approval to promote to production (Approval Step)] -> [promote-image-to-prod] -> [job-deploy-hello-app-prod] 

13. Run Playbooks create-namespaces, create-pipeline. Note github_url is for the node application you will be building, not the Ansible Repo. workflow_id is the workflow id of workflow you created in step 12 (will be in the URL when you view the workflow).
14. Add Webhook to Github source repo
15. Check in code to trigger pipeline (or trigger manually)
16. Navigate to route in test
17. Approve Production Release in Ansible
18. Navigate to route in production

## Playbook/Job Descriptions

### One Shots run from Ansible
create-namespaces  - Run once to create demo-test and demo-prod namespaces  
create-pipeline  - Run once to create pipeline, custom tasks, aap credentials secret, etc.
### Called from workflow (initiated by Openshift Pipeline)  
job-deploy-hello-app-test - Creates / Updates deployment, service, and route in demo-test  
job-deploy-hello-app-prod - Creates / Updates deployment, service, and route in demo-prod  
promote-image-to-prod  - Moves a specific image tag from demo-test to demo-prod  