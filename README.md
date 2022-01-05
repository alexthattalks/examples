# UPDATE: Setup Workflow for Recurring Monthly Patching

This update allows you to setup a workflow that runs the Patching Job Template two times (or more), for each version of RHEL
to create filters to lock content in place to a certain date, publish, and promote the content and composite content views.


1. Complete installation steps in "Setup Instructions", creating your publish and promotoe Job Template

2. Once the Job Template is created, go to Templates > + (Workflow Template)

3. NAME: Publish & Promote Monthly
   -ORGANIZATION: {YOUR ORG}
   -INVENTORY: use the same as below, ensure you are only including the Satellite host
   -Click SAVE
   
4. Go into Workflow Visualizer

5. Create a node downstream from START
   -DROPDOWN: Template
   -Select Job Template created in below steps
   -CONVERGENCE: All
   -Click PROMPT
   
6. Fill out Survey for RHEL 7
   Leave ERRATUM END DATE BLANK (will be filled by Workflow survey
   PRIMARY CONTENT VIEW: {Main RHEL 7 CV}
   ALL CONTENT VIEWS: {All RHEL 7 CVs including Main)
   COMPOSITE CONTENT VIEW: {RHEL 7 CCV}
   Click NEXT, Then CONFIRM, Then SELECT
 
7. Create a node downstream from RHEL 7
   DROPDOWN: Template
   Select Job Template created in below steps
   CONVERGENCE: All
   Click PROMPT
   
8. Fill out Survey for RHEL 8
   Leave ERRATUM END DATE BLANK (will be filled by Workflow survey
   PRIMARY CONTENT VIEW: {Main RHEL 8 CV}
   ALL CONTENT VIEWS: {All RHEL 8 CVs including Main)
   COMPOSITE CONTENT VIEW: {RHEL 8 CCV}
   Click NEXT, Then CONFIRM, Then SELECT, Then SAVE Workflow
   
9. Click EDIT SURVEY for Workflow
   Only FIELD (END DATE)
    a. PROMPT: END DATE
    b. DESCRIPTION: YYYY-MM-DD
    c. ANSWER VARIABLE NAME: end_date
    d. ANSWER TYPE: text
    e. MIN & MAX: 10
    f. REQUIRED: true
   Click SAVE
   
 10. SAVE Workflow
 
 Now you can run this Workflow each month by launching it, entering the required date, and running the workflow.
 

# MAIN SETUP INSTRUCTIONS:

****DOWNLOAD ALL 4 FILES - publish.yml, filters.yml, cvs.yml, and composite_cvs.yml, and place all in same directory on Git

***IF USING AUTO-DOWNLOADED COLLECTIONS

1. Create "collections" directory in the top level of your Git repo for the project containing the content view playbooks

2. Create a "requirements.yml" file inside the "collections" directory, containing the following:

---
collections:
  - name: redhat.satellite
    source: https://console.redhat.com/api/automation-hub/

This enables Ansible Tower to automatically fetch needed collections/modules/roles from Automation hub

3. Ensure Automation Hub is setup as your Galaxy source in Ansible Tower


***IF USING COLLECTIONS PART OF GIT/SCM

####
#### WRITE IN HERE
#### WRITE IN HERE
####

4. Create secrets file with Satellite username and password
    a. Create a sat_login_secret.yml file alongside your playbooks
    b. Inside the file, place the following:

    secret:
      sat_user: "YOUR ADMIN USERNAME"
      sat_pass: "YOUR ADMIN PASSWORD"

    c. Save and close the file
    d. Encrypt the file using Ansible Vault with the following command:

    [root@ansible ~]# ansible-vault encrypt sat_login_secret.yml
    New Vault password:
    Confirm New Vault password:
    Encryption successful

5. Using the password you just set, create a Vault credential in Ansible Tower
    a. Go to Credentials > Green "+"
    b. Enter the name, password you just set, a description, and the organization
    c. Click Save

6. When setting up the template, ensure BOTH the machine credential AND the Vault credential you just created are both selected in the Credentials field

7. When selecting an inventory, ensure you are only including the Satellite host

8. You must create the following survey to work properly:
    1st FIELD (SATELLITE URL)
    a. PROMPT: Satellite URL
    b. ANSWER VARIABLE NAME: sat_url
    c. ANSWER TYPE: text
    d. DEFAULT ANSWER: http://yoursatellite.example.com
    e. REQUIRED: true

    2nd FIELD (ORGANIZATION)
    a. PROMPT: Organization
    b. ANSWER VARIABLE NAME: org_name
    c. ANSWER TYPE: text
    d. DEFAULT ANSWER: YOUR ORGANIZATION
    e. REQUIRED: true

    3rd FIELD (DESCRIPTION)
    a. PROMPT: Description
    b. ANSWER VARIABLE NAME: description
    c. ANSWER TYPE: text
    d. DEFAULT ANSWER: ENTER MONTH Patch Set
    e. REQUIRED: true
    
    4th FIELD (ERRATUM END DATE)
    a. PROMPT: Erratum End Date
    b. DESCRIPTION: YYYY-MM-DD
    c. ANSWER VARIABLE NAME: end_date
    d. ANSWER TYPE: text
    e. MIN & MAX LENGTH: 10
    f. REQUIRED: false
    
    5th FIELD (ERRATA ID)
    a. PROMPT: Erratum End Date
    b. DESCRIPTION: RHSA:2021-XXXXX
    c. ANSWER VARIABLE NAME: errata_id
    d. ANSWER TYPE: text
    e. REQUIRED: false
    
    6th FIELD (PRIMARY CONTENT VIEW)
    a. PROMPT: Primary Content View
    b. ANSWER VARIABLE NAME: primary_content_view
    c. ANSWER TYPE: text
    d. REQUIRED: false

    7th FIELD (ALL CONTENT VIEWS)
    a. PROMPT: All Content Views
    b. DESCRIPTION: (Including Primary Content View)
    c. ANSWER VARIABLE NAME: content_views
    d. ANSWER TYPE: textarea
    e. REQUIRED: false

    8th FIELD (COMPOSITE CONTENT VIEW)
    a. PROMPT: Composite Content View
    b. ANSWER VARIABLE NAME: composite_content_views
    c. ANSWER TYPE: text
    d. REQUIRED: false
