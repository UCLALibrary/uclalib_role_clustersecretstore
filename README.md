uclalib_role_clustersecretstore
=========

Ansible role to prepare UCLA Library's Kubernetes clusters to use the [External Secrets](https://external-secrets.io) Operator

What is this role doing?
------------------------

This role creates the following Kuberentes resources for each team specified in the `external_secrets_aws_access` dictonary variable:
  * `Secret` - a Secret for each specified team to hold their respective AWS access credentials
  * `ClusterSecretStore` - a ClusterSecretStore for each specified team that references a cooresponding Secret containing AWS access credentials

Why are we doing this?
----------------------

The purpose of this role is to minimize the amount of manual or out-of-band set-up required to allow a new Kubernetes application to use External Secrets.

For any new application to use External Secrets, it needs the following items:
  1. Entries in the AWS Parameter store defining all applicable "secret" values needed by the application
  2. Secret K8S resource holding the AWS credentails to access the Parameter Store entries
  3. SecretStore K8S resource that references the Secret with the AWS credentials
  4. ExternalSecret K8S resource that defines which entries in the Parameter Store should be pulled for this application using credentails from the SecretStore

The objective with this role is to provison items `2` and `3` one time for each team so they don't have to be a part of the process to deploy a new application. 

The team can focus on writing a Helm chart for a standard Kubernetes application using an ExternalSecret resource. 

This role should only be run when:
  * a new team is created to establish their AWS credentials
  * an existing team's AWS credentials need to be changed

Requirements
------------

  * The External Secrets Operator is installed in the Kuberentes cluster
  * A Namespace named `external-secerts` exists in the Kubernetes cluster
    * each of the teams' Secret resources are provisioned into this Namespace

Role Variables
--------------

  * `k8s_user` - Defines the user to run the kubectl commmands to provision the Kubernetes resources (default: `helm`)
  * `external_secrets_aws_access` - Defines the dictionary variable specifying each teams AWS access credentials
    * Example definition of this variable:
    ```
    external_secrets_aws_access:
      team1:
        aws_access_key_b64: ""
        aws_secret_access_key_b64: ""
      team2:
        aws_access_key_b64: ""
        aws_secret_access_key_b64: ""
      team3:
        aws_access_key_b64: ""
        aws_secret_access_key_b64: ""
    ```
    * `team1/team2/team3/etc` - Is any arbitrary name for the team for which the subsequent AWS credentials are assigned
    * `aws_access_key_b64` - Defines the AWS Access Key assigned to this team (base 64 encoded)
    * `aws_secret_access_key_b64` - Defines the AWs Secret Access Key assigned to this team (base 64 encoded)

Tags
----

Use the following tags to run specific role functions:

  * `create-secret` - create the Secret resources for each specified team
  * `create-css` - create the ClusterSecretStore for each specified team

Dependencies
------------

None

Example Playbook
----------------

```
---

- name: uclalib_clustersecretstore.yml
  become: yes
  become_method: sudo
  gather_facts: no
  hosts:
    - k8s_ssm_external_secrets

  roles:
    - { role: uclalib_role_clustersecretstore }
```
